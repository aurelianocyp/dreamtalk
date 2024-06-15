## Installation

```
conda create -n dreamtalk python=3.7.0
conda activate dreamtalk
pip install -r requirements.txt
conda install pytorch==1.8.0 torchvision==0.9.0 torchaudio==0.8.0 cudatoolkit=11.1 -c pytorch -c conda-forge
conda update ffmpeg

pip install urllib3==1.26.6
pip install transformers==4.28.1
pip install dlib
```

## Download Checkpoints
In light of the social impact, we have ceased public download access to checkpoints. If you want to obtain the checkpoints, please request it by emailing mayf18@mails.tsinghua.edu.cn . It is important to note that sending this email implies your consent to use the provided method **solely for academic research purposes**.

Put the downloaded checkpoints into `checkpoints` folder.


## Inference
Run the script:

```
python inference_for_demo_video.py \
--wav_path data/audio/acknowledgement_english.m4a \
--style_clip_path data/style_clip/3DMM/M030_front_neutral_level1_001.mat \
--pose_path data/pose/RichardShelby_front_neutral_level1_001.mat \
--image_path data/src_img/uncropped/male_face.png \
--cfg_scale 1.0 \
--max_gen_len 30 \
--output_name acknowledgement_english@M030_front_neutral_level1_001@male_face
```

`wav_path` specifies the input audio. The input audio file extensions such as wav, mp3, m4a, and mp4 (video with sound) should all be compatible.

`style_clip_path` specifies the reference speaking style and `pose_path` specifies head pose. They are 3DMM parameter sequences extracted from reference videos. You can follow [PIRenderer](https://github.com/RenYurui/PIRender) to extract 3DMM parameters from your own videos. Note that the video frame rate should be 25 FPS. Besides, videos used for head pose reference should be first cropped to $256\times256$ using scripts in [FOMM video preprocessing](https://github.com/AliaksandrSiarohin/video-preprocessing).

`image_path` specifies the input portrait. Its resolution should be larger than $256\times256$. Frontal portraits, with the face directly facing forward and not tilted to one side, usually achieve satisfactory results. The input portrait will be cropped to $256\times256$. If your portrait is already cropped to $256\times256$ and you want to disable cropping, use option `--disable_img_crop` like this:

```
python inference_for_demo_video.py \
--wav_path data/audio/acknowledgement_chinese.m4a \
--style_clip_path data/style_clip/3DMM/M030_front_surprised_level3_001.mat \
--pose_path data/pose/RichardShelby_front_neutral_level1_001.mat \
--image_path data/src_img/cropped/zp1.png \
--disable_img_crop \
--cfg_scale 1.0 \
--max_gen_len 30 \
--output_name acknowledgement_chinese@M030_front_surprised_level3_001@zp1
```

`cfg_scale` controls the scale of classifer-free guidance. It can adjust the intensity of speaking styles.

`max_gen_len` is the maximum video generation duration, measured in seconds. If the input audio exceeds this length, it will be truncated.

The generated video will be named `$(output_name).mp4` and put in the output_video folder. Intermediate results, including the cropped portrait, will be in the `tmp/$(output_name)` folder.

Sample inputs are presented in `data` folder. Due to copyright issues, we are unable to include the songs we have used in this folder.

If you want to run this program on CPU, please add `--device=cpu` to the command line arguments. (Thank [lukevs](https://github.com/lukevs) for adding CPU support.)

## Ad-hoc solutions to improve resolution
The main goal of this method is to achieve accurate lip-sync and produce vivid expressions across diverse speaking styles. The resolution was not considered in the initial design process. There are two ad-hoc solutions to improve resolution. The first option is to utilize [CodeFormer](https://github.com/sczhou/CodeFormer), which can achieve a resolution of $1024\times1024$; however, it is relatively slow, processing only one frame per second on an A100 GPU, and suffers from issues with temporal inconsistency. The second option is to employ the Temporal Super-Resolution Model from [MetaPortrait](https://github.com/Meta-Portrait/MetaPortrait), which attains a resolution of $512\times512$, offers a faster performance of 10 frames per second, and maintains temporal coherence. However, these super-resolution modules may reduce the intensity of facial emotions.

The sample results after super-resolution processing are in the `output_video` folder.

