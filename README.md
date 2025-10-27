# SAV
Student Action Video (SAV) Dataset in Classroom Scenes

The paper "Towards Student Actions in Classroom Scenes: New Dataset and Baseline" has been accepted by IEEE TRANSACTIONS ON MULTIMEDIA 2025.

## Dataset download process:
## Step 1. Download videos through the web links txt [here](https://drive.google.com/file/d/1S4EONu-faoUxy8y_dIYIqM5LOfyTlGNP/view?usp=sharing).
It should be noted that since we are not the authors of the original videos, we are simply providing the video links here for you to download.

## Step 2. Cut each video and keep the specified slices

You need to download the "clips_to_keep.txt" [here](https://drive.google.com/file/d/1Z3x-farXpzLt7Q3nGSpq5sH_DzVCq-TD/view?usp=drive_link).

Before using the shell, make sure your ffmpeg is compiled with libx264 encoder support. This can be checked by running the following command:
```
ffmpeg -codecs | grep libx264
```
If there is no output, or if the output does not show libx264, it means that your ffmpeg version is not compiled to support the H.264 encoder, and you need to install the libx264 development library.

cut_videos.sh:
```
#!/bin/bash

DATA_DIR="../../data/sav/videos"
OUTPUT_DIR="../../data/sav/video_clips"
SEGMENTS_FILE="clips_to_keep.txt"

if [[ ! -d "${OUTPUT_DIR}" ]]; then
  echo "${OUTPUT_DIR} doesn't exist. Creating it.";
  mkdir -p ${OUTPUT_DIR}
fi

segments=()
while IFS= read -r line; do
    segments+=("$line")
done < "${SEGMENTS_FILE}"

for video_file in "${DATA_DIR}"/*; do
    video_filename=$(basename "${video_file}" .mp4)
    cleaned_name=$(echo "${video_filename}" | sed -e 's/课堂教学视频//g' -e 's/课堂优课实录//g' -e 's/课堂视频实录//g')

    for segment in "${segments[@]}"; do
        original_name=$(echo "${segment}" | cut -d'_' -f1,2)
        required_index=$(echo "${segment}" | cut -d'_' -f3)

        if [[ "${cleaned_name}" == "${original_name}" ]]; then

            start_time=$(echo "scale=2; (${required_index} - 1) * 2 + 1.5" | bc)
            if (( $(echo "${start_time} < 0" | bc -l) )); then
                start_time=0
            fi

            output_segment_path="${OUTPUT_DIR}/${segment}.mp4"
            ffmpeg -i "${video_file}" -ss ${start_time} -t 3 -c:v libx264 -c:a aac -strict experimental "${output_segment_path}"
            
            echo "Processed segment ${segment} from video ${cleaned_name}"
        fi
    done
done
```

```
bash cut_videos.sh
```

## Step 3. Extract frames

extract_frames.sh:
```
#!/bin/bash
DATA_DIR="../../data/sav/video_clips"
OUTPUT_DIR="../../data/sav/frames"

if [[ ! -d "${OUTPUT_DIR}" ]]; then
  echo "${OUTPUT_DIR} doesn't exist. Creating it.";
  mkdir -p ${OUTPUT_DIR}
fi

for VIDEO_FILE in ${DATA_DIR}/*; do
  VIDEO_FILENAME=$(basename ${VIDEO_FILE})
  VIDEO_NAME="${VIDEO_FILENAME%.*}"
  FRAME_DIR="${OUTPUT_DIR}/${VIDEO_NAME}"

  mkdir -p ${FRAME_DIR}

  ffmpeg -i ${VIDEO_FILE} -r 30 -q:v 1 ${FRAME_DIR}/img_%06d.jpg

  FRAME_COUNT=$(ls -1 ${FRAME_DIR} | wc -l)

  echo "Processed video ${VIDEO_FILENAME} with ${FRAME_COUNT} frames."
done
```

```
bash extract_frames.sh
```
If the following problems occur: $'\r': command not found, illegal character: ^M,
use 
```
sed -i 's/\r$//' ../../cut_videos.sh
sed -i 's/\r$//' ../../extract_frames.sh
```

## Step 4. Download "frame list" [train](https://drive.google.com/file/d/1UHlhz6p7-UMy82sy5DBdcrCCBW483fp8/view?usp=drive_link), [val](https://drive.google.com/file/d/1fx7adqC6MiKdQdB3tlGVZkLEpYokUqOS/view?usp=drive_link) and put them in the frame_list folder.

## Step 5. Download the annotations files and put them in the annotations folder.
1. person boxes files: [class_detection_train_boxes_and_labels](https://drive.google.com/file/d/1QI169QjUKZN0PcMzSnTLTQqL7VOZwNOX/view?usp=drive_link), [class_detection_val_boxes_and_labels](https://drive.google.com/file/d/1tf2Fyu1Kl_k55kxsHeLWOlYiz3DLK0MQ/view?usp=drive_link) (CSV format) or [train_person_det](https://drive.google.com/file/d/1EE0q6baN8QiMvKVbSdlHTf_KZcqGHhD7/view?usp=drive_link), [val_person_det](https://drive.google.com/file/d/1T-y1z1bZz8421gdFR6x81WtvxQy9bHRg/view?usp=drive_link) (pkl format).
2. annotation files for train and val: [train](https://drive.google.com/file/d/1rHToVI9pmAqzFdrNJ-o2P_9pzLzYIcLh/view?usp=drive_link), [val](https://drive.google.com/file/d/1lfWBtmJ98cd0xWNqyq0Ax888ZmdZ0p4L/view?usp=drive_link).
3. excluded timestamps files: [train_excluded_timestamps](https://drive.google.com/file/d/1f4OGziejZfpckjNzrzrAeFxJbt0zMEa-/view?usp=drive_link), [val_excluded_timestamps](https://drive.google.com/file/d/1JRLGNTgEKT75ItPO4SBaoLkl7VT6I0Iq/view?usp=drive_link).
4. label name file: [education_first_label](https://drive.google.com/file/d/1bRS5ia_9UUlBTRNuGvc8jBFv3lHLSseg/view?usp=drive_link).

Download the SAV dataset with the following structure:

```
SAV
|_ frames
|  |_ [video name 0]
|  |  |_ img_000001.jpg
|  |  |_ img_000002.jpg
|  |  |_ ...
|  |_ [video name 1]
|     |_ img_000001.jpg
|     |_ img_000002.jpg
|     |_ ...
|_ frame_list
|  |_ train.csv
|  |_ val.csv
|_ annotations
   |_ class_detection_train_boxes_and_labels.csv
   |_ class_detection_val_boxes_and_labels.csv
   |_ train.csv
   |_ val.csv
   |_ train_excluded_timestamps.csv
   |_ val_excluded_timestamps.csv
   |_ education_first_label.pbtxt
   |_ train_person_det.pkl
   |_ val_person_det.pkl


```

## VideoMAE Det.
We run the VideoMAE Det. from mmaction2 [here](https://github.com/open-mmlab/mmaction2) and change the mmaction/datasets/ava_dataset.py into our ava_dataset.py, mmaction/models/roi_heads/bbox_heads/bbox_head.py into our bbox_head.py, mmaction/evalution/metrics/ava_metric.py into our ava_metric.py,  mmaction/evalution/functional/ava_utils.py into our ava_utils.py, test/dataset/test_ava_dataset.py into our test_ava_dataset.py for our SAV Dataset.

## Configs.
[vit-base-p16_videomae-k400-pre_8xb8-16x4x1-20e-adamw_sav-rgb.py](https://github.com/Ritatanz/SAV/blob/main/vit-base-p16_videomae-k400-pre_8xb8-16x4x1-20e-adamw_sav-rgb.py) is the CONFIG_FILE of SAV dataset.
[vit-base-p16_videomae-k400-pre_8xb8-16x4x1-20e-adamw_ava-rgb.py](https://github.com/Ritatanz/SAV/blob/main/vit-base-p16_videomae-k400-pre_8xb8-16x4x1-20e-adamw_ava-rgb.py) is the CONFIG_FILE of AVA dataset.

## Train.
Following the command in [mmaction2](https://github.com/open-mmlab/mmaction2/tree/main/configs/detection/videomae)
```shell
python tools/train.py ${CONFIG_FILE} ${CHECKPOINT_FILE} [optional arguments]
```

## Test.
Following the command in [mmaction2](https://github.com/open-mmlab/mmaction2/tree/main/configs/detection/videomae)
```shell
python tools/test.py ${CONFIG_FILE} ${CHECKPOINT_FILE} [optional arguments]
```

## Citation

```BibTeX
@article{tan2025towards,
  title={Towards Student Actions in Classroom Scenes: New Dataset and Baseline},
  author={Tan, Zhuolin and Gao, Chenqiang and Qin, Anyong and Chen, Ruixin and Song, Tiecheng and Yang, Feng and Meng, Deyu},
  journal={IEEE Transactions on Multimedia},
  volume={27},
  pages={6831--6844},
  year={2025},
  publisher={IEEE}
}
```
