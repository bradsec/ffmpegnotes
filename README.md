## FFMPEG Notes

- Change file name extensions as required, examples using .mkv

### Batch extract audio from video files

```terminal
for i in *.mkv; do
  ffmpeg -i "$i" -q:a 0 -map a "${i%.mkv}.wav"
done
```

### Batch extract center channel audio from 5.1 video files

```terminal
for i in *.mkv; do
  ffmpeg -i "$i" -filter_complex "[0:a:0]channelsplit=channel_layout=5.1:channels=FC[FC]" -map "[FC]" "${i%.mkv}_center.wav"
done
```
