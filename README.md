# FFMPEG Dictionary
Maximizing hardware for video and audio encoding and decoding.

There are two versions I have established: Windows 11 and MacOS. Windows has 12 iterations, and I have picked the most efficient/notable ones. MacOS has 4 iterations, and all are listed. 

In similarities, all offload to the GPU in order to speed up encoding and decoding at a resolution of 2560x1440.

## Codecs, Accelerators Identified

Before starting, here are the codecs used. These can be found with the ```ffmpeg -hwaccels``` and ```ffmpeg -codecs``` commands.

**Win 11 Hardware Accelerators**:
- cuda (tested)
- vaapi
- dxva2
- qsv
- d3d11va
- opencl
- vulkan
- d3d12va

**Win 11 Codecs**:
There are countless codecs listed, and the following below are used. Windows codecs follow the DE(V/A/S/D/T)ILS format.
- h264_nvenc
----------------------------------------
**MacOS Hardware Accelerators**:
- videotoolbox

**MacOS Codecs**: 
There are countless codecs here too, but three have been used here as they support the ```videotoolbox``` accelerator. MacOS codecs follow the DEVILS format.
- h264_videotoolbox
- hevc_videotoolbox
- prores_videotoolbox

## Windows (x86, CUDA) 

AMF, or AMD's equivalent GPU support for FFMPEG is not covered here, but you are welcome to add to this readme. 


1) ```ffmpeg -y -threads 4 -hwaccel cuda -hwaccel_output_format cuda -i "input.mp4" -vf "scale_cuda=2560:1440" -c:v h264_nvenc -cq 20 -crf 23 -c:a copy output.mp4```
- This creates a 1.72 GB file at 15.8x speed.

2) ```ffmpeg -y -threads 4 -hwaccel cuda -hwaccel_output_format cuda -extra_hw_frames 2 -i "input.mp4" -vf "scale_cuda=2560:1440" -c:v h264_nvenc -preset p1 -tune hq -rc vbr -cq:v 15 -b:v 0 -c:a copy output.mp4```
- This creates a 4.23GB file at 27.6x speed.

3) ```ffmpeg -y -threads 2 -hwaccel cuda -hwaccel_output_format cuda -extra_hw_frames 2 -i "input.mp4" -vf "scale_cuda=2560:1440" -c:v h264_nvenc -preset p1 -tune hq -rc vbr -cq:v 17 -b:v 0 -c:a copy output.mp4```
- This creates a 3.14GB file at 28.4x speed. At a ```cq``` of 15, speed starts to level at 27.7x-28x.

4) ```ffmpeg -y -threads 2 -hwaccel cuda -hwaccel_output_format cuda -extra_hw_frames 2 -i "input.mp4" -vf "scale_cuda=2560:1440" -c:v h264_nvenc -preset p1 -tune hq -rc vbr -cq:v 16 -b:v 0 -c:a copy output.mp4```
- This creates a 3.5GB file at 28.2x speed. Bitrate was changed to accomodate high motion, blur on moving characters.

5) ```ffmpeg -y -threads 2 -hwaccel cuda -hwaccel_output_format cuda -extra_hw_frames 2 -i "input.mp4" -vf "scale_cuda=2560:1440" -c:v h264_nvenc -preset p3 -tune hq -rc vbr -cq:v 16 -b:v 0 -c:a copy output.mp4```
- This creates a 3.61GB file at 23x speed. Here, it starts to raise concern that a hardware limitation arises. Will confirm with GPU-Z.

6) ```ffmpeg -y -threads 2 -hwaccel cuda -hwaccel_output_format cuda -extra_hw_frames 2 -i "input.mp4" -vf "scale_cuda=2560:1440" -c:v h264_nvenc -preset p2 -tune hq -rc vbr -cq:v 16 -b:v 0 -c:a copy output.mp4```
- This creates a 3.67GB file at 27.8x speed. The ```p3``` profile is downgraded to 2 as it causes overexposure in the output footage.

7) ```ffmpeg -y -threads 2 -hwaccel cuda -hwaccel_output_format cuda -extra_hw_frames 2 -i "input.mp4" -vf "scale_cuda=2560:1440" -c:v h264_nvenc -preset p2 -rc-lookahead 0 -bf -0 -tune hq -rc vbr -cq:v 16 -b:v 0 -c:a copy output.mp4```
- This creates a 3.67GB file at 27.8x speed. Adding the ```-rc-lookahead 0``` and ```-bf 0``` flags decreases the load of the GPU as it doesn't have to queue frames as it's concurrently rendering frames.

<img width="1557" height="715" alt="Screenshot 2026-04-26 110227" src="https://github.com/user-attachments/assets/02cebf1a-e996-45ce-b59a-b753a464f74d" />
At 100% load, it can be confirmed that a hardware limitation can be seen as speed ranges from 26.4x to 27.8x at a cq of 15-17, p2, and cuda acceleration.


## MacOS (Apple Silicon Version)

1) ```ffmpeg -y -hwaccel videotoolbox -i "input.mp4" -vf "scale=2560:1440" -c:v h264_videotoolbox -tune hq -c:a copy output.mp4```
- With the ```h264_videotoolbox``` flag, a negligible increase in speed with just CPU acceleration at 5x speed. A warning compiles with ```color range not set for nv12```.

2) ```ffmpeg -y -hwaccel videotoolbox -i "input.mp4" -vf "scale=2560:1440" -c:v hevc_videotoolbox -pix_fmt nv12 -tune hq -c:a copy -tag:v hvc1 output.mp4```
- With the ```hevc_videotoolbox``` flag, it helps suppress the color range error prior with help from the ```-pix_fmt``` flag.
- The problem with this codec is that the file can only open with the ```-tag:v hvc1``` flag. This allows it to open in QuickTime Player.
- Speed decreases in this iteration, to 4.65x.

3) ```ffmpeg -y -hwaccel videotoolbox -i "input.mp4" -vf "scale=2560:1440" -c:v hevc_videotoolbox -q:v 70 -c:a copy -tag:v hvc1 output.mp4```
- Sticking with the hevc codec, this one creates a ProRes like quality, and quality seems to increase from ```q:v``` of 65 or higher.
- This one is also the most space efficient at 693MB. Speed is at 4.71x.

4) ```ffmpeg -y -hwaccel videotoolbox -i "input.mp4" -vf "scale=2560:1440" -pix_fmt p210le -c:v prores_videotoolbox -allow_sw 1 -profile:v 3 -c:a copy output.mov```
- The ```prores_videotoolbox``` codec is one of the most demanding codecs both in compute and storage.
- Very picky in terms of pixel format, demands only ```p210le```.
- Adding the ```-allow_sw 1``` flag allows for the command to go through without crashing.
- ```profile:v 3``` is similar to the ```p3``` flag with nvenc in producing the quality desired.
- Requires the output footage to be in the .mov format.
- DO NOT ENCODE IN PRORES DUE TO SPACE! 45 minutes out of 51 minute test footage surpasses 60GB. 
