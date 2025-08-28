ffmpeg
=====

- Only 1st stream of Video (H265) and 2nd stream of Audio (as AAC), no subtitles, extracting 1min fragment from 1 hour 10 seconds point
```shell
  ffmpeg -i "movie.mkv" -c:v libx265 -sn -map 0:v:0 -map 0:a:1 -ss 01:00:10 -t 00:01:00 sample.mp4
```
- To convert all videos in a folder to their H.265 counterpart in Windows, just create a .BAT file containing:

```shell
  for %%a in ("*.avi") do ffmpeg -i "%%a" -c:v libx265 -vtag hvc1 "%%~na.mp4"
```
- Leverage the graphics card to accelerate conversion into HEVC video:
```shell
  # AMD Radeon
  ffmpeg -i input.mp4 -c:v hevc_amf -rc cqp -qp_i 24 -qp_p 24 output.mp4
  # For Intel ones use hevc_qsv instead
```
- To simply concatenate 3 videos which have the same codec into a single one, first create a `mylist.txt` containing:
```
  file 'video1.mp4'
  file 'video2.mp4'
  file 'video3.mp4'
```
  and then execute:
```shell
  ffmpeg -f concat -safe 0 -i mylist.txt -c copy output.mp4
```
- **DVD** VIDEO_TS/*.VOB into a single MP4 file:

```shell
  ffmpeg -i "concat:VTS_01_1.VOB|VTS_01_2.VOB|VTS_01_3.VOB|VTS_01_4.VOB|VTS_01_5.VOB" outfile.mp4
```
- Extract fragment of audio only converting from stereo to mono

```shell
  ffmpeg -i "BobEsponja.avi" -ss 00:09:06 -t 00:00:05.500 -q:a 0 -map a -ac 1 sample.mp3
```
- Extract audio stream only converting it to MP3 with default options:

```shell
  ffmpeg -i "Sir Mix-A-Lot - Baby Got Back.mkv" -vn -ss 00:00:13 output.mp3
```
- Remove a section of the video that's between certain floating seconds:

```shell
  ffmpeg -i input.mp4 -vf "select='not(between(t,6.5,15))', setpts=N/FRAME_RATE/TB" -af "aselect='not(between(t,6.5,15))', asetpts=N/SR/TB" output.mp4
```
- Resize/Scale keeping aspect ratio

```shell
  ffmpeg -i input.mp4 -vf scale=1920:-1 output.mp4
```