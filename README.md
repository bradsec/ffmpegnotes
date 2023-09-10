# FFMPEG Notes and Commandline Usage

### Batch extract center channel audio from 5.1 mkv video

```terminal
for i in *.mkv; do
  ffmpeg -i "$i" -filter_complex "[0:a:0]channelsplit=channel_layout=5.1:channels=FC[FC]" -map "[FC]" "${i%.mkv}_center.wav"
done
```
