## FFMPEG Notes and Terminal Commands

- Change file name extensions as required, examples using .mkv


### Compile FFMPEG with CUDA support Debian 11 and 12
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

# Usage - ffmpeg -y -hwaccel cuda (normal ffmpeg arguments)

Example output sucessfully compiled
ffmpeg version N-112061-g654e4b00e2 Copyright (c) 2000-2023 the FFmpeg developers
  built with gcc 12 (Debian 12.2.0-14)
  configuration: --enable-nonfree --enable-gpl --enable-nonfree --enable-libmp3lame --enable-libx264 --enable-libx265 --enable-libfdk-aac --enable-cuvid --enable-nvenc --enable-cuda-nvcc --enable-libnpp --extra-cflags=-I/usr/local/cuda/include --extra-ldflags=-L/usr/local/cuda/lib64
  libavutil      58. 24.100 / 58. 24.100
  libavcodec     60. 26.100 / 60. 26.100
  libavformat    60. 12.100 / 60. 12.100
  libavdevice    60.  2.101 / 60.  2.101
  libavfilter     9. 11.100 /  9. 11.100
  libswscale      7.  3.100 /  7.  3.100
  libswresample   4. 11.100 /  4. 11.100
  libpostproc    57.  2.100 / 57.  2.100
Hyper fast Audio and Video encoder
usage: ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...

Use -h to get full help or, even better, run 'man ffmpeg'
```

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
