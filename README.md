## FFMPEG Notes and Terminal Commands

- Change file name extensions as required, examples using .mkv


## Compile FFMPEG with CUDA support Debian 11 and 12
```
# Note CUDA drivers / toolkit needs to already be installed.

# Remove existing version if installed.
sudo apt -y remove ffmpeg

# Install CUDA toolkit as per nvidia site instructions

# Install required libraries and utilities
sudo apt-get update
sudo apt-get install -y build-essential pkg-config yasm git cmake libtool libc6 libc6-dev unzip wget libnuma1 libnuma-dev checkinstall libva-dev libx11-dev libxext-dev libxfixes-dev zlib1g-dev libass-dev libfreetype6-dev libsdl2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo wget zlib1g-dev libomxil-bellagio-dev libunistring-dev libx264-dev libx265-dev libfdk-aac-dev libmp3lame-dev

# Compile and install nvidia codecs
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd ~
cd nv-codec-headers
sudo make install
cd..

# Compile, configure and install ffmpeg
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg/
cd ffmpeg
./configure --enable-nonfree --enable-gpl --enable-nonfree --enable-libmp3lame --enable-libx264 --enable-libx265 --enable-libfdk-aac --enable-cuvid --enable-nvenc --enable-cuda-nvcc --enable-libnpp --extra-cflags=-I/usr/local/cuda/include --extra-ldflags=-L/usr/local/cuda/lib64
make -j $(nproc)
sudo make install
cd..
```

- Run `ffmpeg` command to confirm build configuration with CUDA support
- Example output of a sucessfully compiled version.

```terminal
ffmpeg version N-112061-g654e4b00e2 Copyright (c) 2000-2023 the FFmpeg developers
  built with gcc 12 (Debian 12.2.0-14)
  configuration: --enable-nonfree --enable-gpl --enable-nonfree --enable-libmp3lame --enable-libx264 --enable-libx265 --enable-libfdk-aac --enable-cuvid --enable-nvenc --enable-cuda-nvcc --enable-libnpp --extra-cflags=-I/usr/local/cuda/include --extra-ldflags=-L/usr/local/cuda/lib64
```

#### Using `ffmpeg` with CUDA hardware acceleration
```terminal
ffmpeg -y -hwaccel cuda (normal ffmpeg arguments)
```

## FFMPEG command line helpers

### Batch extract audio from video files
```terminal
for i in *.mkv; do
  ffmpeg -i "$i" -q:a 0 -map a "${i%.mkv}.wav"
done
```

### Batch extract only center channel audio from 5.1 video files
```terminal
for i in *.mkv; do
  ffmpeg -i "$i" -filter_complex "[0:a:0]channelsplit=channel_layout=5.1:channels=FC[FC]" -map "[FC]" "${i%.mkv}_center.wav"
done
```

### Batch number rename files 001.wav, 002.wav etc.
```terminal
i=1
ext="wav"
for f in *.$ext; do
  new=$(printf "%03d.$ext" "$i")
  mv -- "$f" "$new"
  i=$((i+1))
done
```

### Join Concat Video or Audio Files
```terminal
# In directory containing files (change filetype if required .mkv .wav etc)
# Make a list of files in files.txt
for f in *.mkv; do echo "file '$f'" >> files.txt; done

# Concat files (change output filename and filetype if required .mkv .wav etc)
ffmpeg -f concat -safe 0 -i files.txt -c copy output.mkv
```

### Get duration of audio or video file
```terminal
# Using ffprobe, result returned in seconds (set input file name example input.wav).
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 input.wav
```

