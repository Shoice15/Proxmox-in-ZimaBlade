# FFmpeg cheat sheet


## Encoding

AVC/H.264/AAC Encoding
```sh
FFmpeg \
-i input.mp4 \
-c:v libx264 \
-preset slow \
-crf 20 \
-c:a aac \
-b:a 128k \
-ac 2 \
-ar 44100 \
-movflags +faststart \
output.mp4
```

> X264 8-bit CRF: `0-51`, 0=lossless 23=default, 51=worst, 18 almost lossless  
> X264 10-bit CRF: `0-63`

X264 Presets:
* `ultrafast`
* `superfast`
* `veryfast`
* `faster`
* `fast`
* `medium`
* `slow`
* `slower`
* `veryslow`
* `placebo`

HEVC/H.265/HE-AAC version 2 Encoding
```sh
ffmpeg \
-i input.mp4 \
-c:v libx265 \
-preset superfast \
-crf 21 \
-c:a aac \
-b:a 128k \
-ac 2 \
-ar 44100 \
-tag:v hvc1 \
-movflags +faststart \
output.mp4
```

> X265 CRF: 28=default

* `ultrafast`
* `superfast`
* `veryfast`
* `faster`
* `fast`
* `medium`
* `slow`
* `slower`
* `veryslow`
* `placebo`

Modify rotation metadata without re-encoding.
```sh
ffmpeg \
-y \
-hide_banner \
-i input.m4v \
-metadata:s:v rotate="90" \
-codec copy \
output.m4v
```

To JPG sequence:
```sh
ffmpeg \
-y \
-hide_banner \
-i input.mp4 \
-an \
-f image2 \
image%04d.jpg
```

To JPG sequence alternative:
```sh
ffmpeg \
-i input.mp4 \
-qscale:v 2 \
image%04d.jpg
```

From images
```sh
ffmpeg \
-y \
-hide_banner \
-f image2 \
-i image%04d.jpg \
-r 12 \
-an \
output.mp4
```

GoPro image sequence to DNxHD at 90Mbps
```sh
ffmpeg \
-y \
-hide_banner \
-start_number 22 \
-i G003%04d.JPG \
-vcodec dnxhd \
-b:v 90M \
-an \
output.mov
```

GoPro image sequence to ProRes
```sh
ffmpeg \
-y \
-hide_banner \
-start_number 22 \
-i G003%04d.JPG \
-r 30000/1001 \
-vcodec prores \
-profile:v 0 \
-an \
output.mov
```

GoPro image sequence to ProRes_ks
```sh
ffmpeg \
-y \
-hide_banner \
-start_number 22 \
-i G003%04d.JPG \
-r 30000/1001 \
-vcodec prores_ks \
-profile:v 0 \
-an \
output.mov
```

Generate 10 thumbnails from a video
```sh
THUMBNAIL_NO=10
FILE=MyVideoFile.mp4
DURATION=$(mediainfo --Inform="General;%Duration%" "${FILE}")
ffmpeg \
-i "${FILE}" \
-vf "fps=${THUMBNAIL_NO}000/${DURATION}" \
out%04d.png
```

ProRes with YUV 4444 support
```sh
ffmpeg -i input.mp4 -c:v prores_ks -profile:v 3 -c:a pcm_s16le output.mov
```

`prores_ks` profiles:
| #  | Profile        | Mbps      | CS   | Chroma  |
|---:|:---------------|----------:|:-----|--------:|
| -1 | auto (default) |           | YUV  |   4:2:2 |
|  0 | Proxy          |  ≈ 45Mbps | YUV  |   4:2:2 |
|  1 | LT             | ≈ 102Mbps | YUV  |   4:2:2 |
|  2 | Standard       | ≈ 147Mbps | YUV  |   4:2:2 |
|  3 | HQ             | ≈ 220Mbps | YUV  |   4:2:2 |
|  4 | 4444           | ≈ 330Mbps | YUVA | 4:4:4:4 |
|  5 | 4444-HQ        | ≈ 500Mbps | YUVA | 4:4:4:4 |



Source: https://trac.ffmpeg.org/wiki/Encode/VFX#Prores

Premiere DNX to H.264/AAC for YouTube with `hqdn3d` denoise and GOP 30
```sh
ffmpeg \
-i input.mxf \
-pix_fmt yuv420p \
-vf "hqdn3d=1.5:1.5:6:6" \
-c:v libx264 \
-preset fast \
-x264opts "keyint=30:scenecut=-1" \
-crf 22 \
-c:a aac \
-b:a 128k \
-ac 2 \
-ar 44100 \
-movflags +faststart \
output.mp4
```

Premiere DNX to H.264/AAC
```sh
ffmpeg \
-i input.mxf \
-pix_fmt yuv420p \
-c:v libx264 \
-preset fast \
-crf 22 \
-c:a aac \
-b:a 128k \
-ac 2 \
-ar 44100 \
-movflags +faststart \
output.mp4
```

Premiere DNX to H.265/AAC
```sh
ffmpeg \
-i input.mxf \
-pix_fmt yuv420p \
-c:v libx265 \
-preset superfast \
-crf 21 \
-c:a aac \
-b:a 128k \
-ac 2 \
-ar 44100 \
-tag:v hvc1 \
-movflags +faststart \
output.mp4
```

## Chroma subsampling and 10bits

X264 10bit 4:2:2 Chroma at CRF 20 using X264 10Bit
```sh
ffmpeg \
-y \
-hide_banner \
-i UHD_10bit_444_INPUT.mov \
-c:v libx264 \
-profile:v high422 \
-crf 20 \
-pix_fmt yuv422p \
output.mp4
```

X264 10bit 4:2:2 Chroma at 135Mbps using X264 10Bit
```sh
ffmpeg \
-y \
-hide_banner \
-i UHD_10bit_444_INPUT.mov \
-c:v libx264 \
-profile:v high422 \
-b:v 120000k \
-pix_fmt yuv422p \
output.mp4
```

X264 10bit 4:4:4 Chroma at 135Mbps using X264 10Bit
```sh
ffmpeg \
-y \
-hide_banner \
-i UHD_10bit_444_INPUT.mov \
-c:v libx264 \
-profile:v high444 \
-crf 20 \
-an \
output.mp4
```

YouTube "Recomended Settings"
```sh
ffmpeg \                # Calling the binary
-i input.mp4 \          # Input video file
-r 30000/1001 \         # Set the frame rate - optional
-vf scale="1920:1080" \ # Resize video - optional
-codec:v libx264 \      # X264 Video Codec
-crf 21 \               # Video Quality
-bf 2 \                 # Maximum 2 B-frames as per guideline
-flags +cgop \          # Closed GOP as per guideline
-pix_fmt yuv420p \      # Chroma subsampling 4:2:0 as per guideline
-c:a aac \              # Fraunhofer FDK AAC codec library
-b:a 128k \             # Audio Bitrate
-ac 2 \                 # Audio channels
-r:a 44100 \            # Audio samplerate
-map 0:v:0 \            # First file : video : first track
-map 0:a:0 \            # First file : audio : first track 
-movflags faststart \   # Put MOOV atom at the front of the file
output.mp4
```

## Alpha Channel

Add alpha channel mask encoding to `qtrle`
```sh
ffmpeg \
-loop 1 \
-i alpha.png \
-i input.mp4 \
-filter_complex \
"[0:v]alphaextract[alf]; \
 [1:v][alf]alphamerge" \
-c:v qtrle \
-an \
-t 1 \
output.mov
```

Add alpha channel mask encoding to `png`
```sh
ffmpeg \
-loop 1 \
-i alpha.png \
-i input.mp4 \
-filter_complex \
"[0:v]alphaextract[alf]; \
 [1:v][alf]alphamerge" \
-c:v png \
-pix_fmt rgb32 \
-an \
-t 1 \
output.mov
```

## Filters

Add blured bars for vertical video
```sh
ffmpeg \
-i "input.mp4" \
-filter_complex "[0:v]scale=ih*16/9:-1,boxblur=luma_radius=min(h\,w)/20:luma_power=1:chroma_radius=min(cw\,ch)/20:chroma_power=1[bg];[bg][0:v]overlay=(W-w)/2:(H-h)/2,crop=h=iw*9/16" \
-pix_fmt yuv420p \
-c:v libx264 \
-preset fast \
-crf 22 \
-c:a aac \
-b:a 128k \
-ar 44100 \
-ac 2 \
-movflags +faststart \
output.mp4
```

Test a filter with `ffplay`
```sh
ffplay \
-filter:v "crop=1920:1080:0:140" \
input.mp4
```

Create blue video with subs
```sh
ffmpeg \
-f lavfi \
-i "color=c=blue:s=1920x1080" \
-vf "subtitles=subs.srt:force_style='Fontsize=26,PrimaryColour=&H00ffff&'" \
-pix_fmt yuv420p \
-c:v libx264 \
-preset fast \
-crf 21 \
-an \
-t "00:14:56.244" \
-movflags +faststart \
subs.mp4
```

YouTube's cubemap to equirectangular
```sh
ffmpeg \
-y \
-hide_banner \
-i input.mkv \
-vf "v360=c3x2:e:cubic:in_forder='lfrdbu':in_frot='000313',scale=3840:1920,setsar=1:1" \
-pix_fmt yuv420p \
-c:v libx264 \
-preset faster \
-crf 21 \
-c:a copy \
-movflags +faststart \
output.mp4
```

ProRes blue screen with subtitles from SRT
```sh
ffmpeg \
-f lavfi \
-i "color=c=blue:s=1920x1080" \
-pix_fmt yuv422p10le \
-vf "subtitles=sub.srt:force_style='Fontsize=26,PrimaryColour=&H00ffff&'" \
-c:v prores_aw \
-an \
-t "00:01:04.993" \
-movflags +faststart \
output.mov
```

Crop
```sh
-vf "crop=1920:816:0:132" \
```

Denoise
```sh
-vf "hqdn3d=4:4:9:9" \
```

Scale
```sh
-vf "scale=640:-16" \
```

Denoise and scale
```sh
# -vf "hqdn3d=4:4:9:9,scale=640:-16" \
```

4K to UHD Pad
```sh
-vf "scale=min(iw*2160/ih\,3840):min(2160\,ih*3840/iw),pad=3840:2160:(3840-iw)/2:(2160-ih)/2" \
```

Add hard yellow subtitles
```sh
-vf "subtitles=subs.srt:force_style='Fontsize=26,PrimaryColour=&H00ffff&'"
```

Deinterlace
```sh
-vf yadif 
```

Change volume
```sh
-filter:a "volume=24dB"
```

## Timelapse

Make a time-lapse fom a video, if you have a GoPro video recorded at normal speed (29.97fps) and you want to convert that video to a time-lapse video using the same GoPro time-lapse speed selection style, like a picture each 0.5, 1, 2, 5, 10, 30, 60 seconds, here is the command, the filter `setpts` changet the presentation timestamp (PTS), the `1/0.5/(30000/1001)*PTS` formula goes as follows, one second divided the pictures per second, the result is devided by the input frame rate, having 29.97 being described as `30000/1001`, and the result, multiplied by PTS. More info about video speed [here](https://trac.ffmpeg.org/wiki/How%20to%20speed%20up%20/%20slow%20down%20a%20video)  

You can also specify the `-filter:v "setpts=0.5*PTS"` to a 2x speed using `0.5`, 4x speed using `0.25`, 8x speed using `0.125`, 16x speed using `0.0625` and so on.

```sh
ffmpeg \
-y \
-hide_banner \
-i GOPR0001.MP4 \
-filter:v "setpts=1/0.5/(30000/1001)*PTS" \
-c:v libx264 \
-preset medium \
-crf 20 \
-an \
-strict experimental \
time-lapse.mp4
```

Time-lapse from GoPro JPGs with crop and resize (`4000×3000` to `1280x720`)
```sh
ffmpeg \
-y \
-hide_banner \
-r 30000/1001 \
-f image2 \
-start_number 5523 \
-i G001%04d.JPG \
-pix_fmt yuv420p \
-vf "crop=4000:2250:0:375,scale=1280:720" \
-c:v libx264 \
-preset faster \
-crf 22 \
-an \
-movflags +faststart \
time-lapse.mp4
```

# Join/Concat video

GoPro join, first, the `mylist.txt` file:
```sh
file 'GOPR5522.MP4'
file 'GP015522.MP4'
file 'GP025522.MP4'
```

Then, the encoding:
```sh
ffmpeg -f concat -i mylist.txt -c copy output.mp4
```

Using the concat protocol:
```sh
ffmpeg \
-i 'concat:GOPR5522.MP4|GP015522.MP4|GP025522.MP4' \
-codec copy output.mp4
```

Split stereo into two mono tracks:
```sh
ffmpeg \
-i stereo.wav \
-map_channel 0.0.0 left.wav \
-map_channel 0.0.1 right.wav
```

## Downmixing audio

Stereo + Stereo → Stereo  
![stereo + stereo → stereo](https://i.imgur.com/AiAGIly.png "stereo + stereo → stereo")

Using the amix filter
```sh
ffmpeg \
-i input0.mp3 \
-i input1.mp3 \
-filter_complex amix=inputs=2:duration=longest \
output.mp3
```

Using the amerge filter
```sh
ffmpeg \
-i input0.mp3 \
-i input1.mp3 \
-filter_complex amerge=inputs=2 \
-ac 2 \
output.mp3
```

Downmix each input into specific output channel  
![Downmix each input into specific output channel](https://i.imgur.com/MPsU7mG.png "Downmix each input into specific output channel")

Using the amerge and pan filters
```sh
ffmpeg \
-i input0.mp3 \
-i input1.mp3 \
-filter_complex "amerge=inputs=2,pan=stereo|c0<c0+c1|c1<c2+c3" \
output.mp3
```

Mono + Mono → Stereo  
![mono + mono → stereo](https://i.imgur.com/uhQZYpZ.png "mono + mono → stereo")

Using the join filter
```sh
ffmpeg \
-i input0.mp3 \
-i input1.mp3 \
-filter_complex "join=inputs=2:channel_layout=stereo" \
output.mp3
```

Using the amerge filter
```sh
ffmpeg \
-i input0.mp3 \
-i input1.mp3 \
-filter_complex "amerge=inputs=2" \
output.mp3
```

Mono + Mono → Mono  
![mono + mono → mono](https://i.imgur.com/y45Hc6j.png "")

Using the amix filter
```sh
ffmpeg \
-i input0.mp3 \
-i input1.mp3 \
-filter_complex "amix=inputs=2:duration=longest" \
output.mp3
```

[FFmpeg Wiki: Audio Channels](https://trac.ffmpeg.org/wiki/AudioChannelManipulation)
[Source](https://stackoverflow.com/a/14528482/218418)

## Video crop factors

### Common Digital Video Formats

Format | Dimensions | Aspect Ratio
|----- |:----------:| -----------:|
|HD    |   1280x720 |         16:9|
|FHD   |  1920x1080 |         16:9|
|4K-UHD|  3840x2160 |         16:9|
|4K-DCI|  4096x2160 |      256:135|
|5K.   |  5120x2880 |         16:9|
|6K    |  6144x3456 |         16:9|
|8K    |  7680x4320 |         16:9|

### Cinemascope (2.35:1)

Format  | 2.35:1 (47:20) | Letterbox |   FFmpeg Crop
| ----- |---------------:|----------:|---------------:|
|HD     |       1280x544 |        88 |   1280:544:0:88|
|FHD    |       1920x816 |       132 |  1920:816:0:132|
|4K-UHD |      3840x1632 |       264 | 3840:1632:0:264|
|4K-DCI |      4096x1744 |       208 | 4096:1744:0:208|
|5K     |      5120x2176 |       352 | 5120:2176:0:352|
|6K     |      6144x2608 |       424 | 6144:2608:0:424|
|8K     |      7680x3264 |       528 | 7680:3264:0:528|

### Cinemascope (2.40:1)

Format  | 2.40:1 (12:5) | Letterbox |   FFmpeg Crop
| ----- |--------------:|----------:|---------------:|
|HD     |      1280x528 |        96 |   1280:528:0:96|
|FHD    |      1920x800 |       140 |  1920:800:0:140|
|4K-UHD |     3840x1600 |       280 | 3840:1600:0:280|
|4K-DCI |     4096x1712 |       224 | 4096:1712:0:224|
|5K     |     5120x2128 |       376 | 5120:2128:0:376|
|6K     |     6144x2560 |       448 | 6144:2560:0:448|
|8K     |     7680x3200 |       560 | 7680:3200:0:560|

### Ultra Panavision 70 (2.76:1)

Format  | 2.76:1/69:25 | Letterbox |  FFmpeg Crop
| ----- |-------------:|----------:|---------------:|
|HD     |     1280x464 |       128 |  1280:464:0:128|
|FHD    |     1920x688 |       196 |  1920:688:0:196|
|4K-UHD |    3840x1382 |       384 | 3840:1382:0:384|
|4K-DCI |    4096x1488 |       336 | 4096:1488:0:336|
|5K     |    5120x1856 |       512 | 5120:1856:0:512|
|6K     |    6144x2224 |       616 | 6144:2224:0:616|
|8K     |    7680x2784 |       768 | 7680:2784:0:768|

### 4K-DCI

Format  | 4K-DCI (256:135) | 2.35:1 Letterbox |   FFmpeg Crop
| ----- | ----------------:|-----------------:|---------------:|
|HD     |         1280x672 |               24 |   1280:672:0:24|
|FHD    |        1920x1008 |               36 |  1920:1008:0:36|
|4K-UHD |        3840x2032 |               64 |  3840:2032:0:64|
|4K-DCI |        4096x2160 |                0 |   4096:2160:0:0|
|5K     |        5120x2704 |               88 |  5120:2704:0:88|
|6K     |        6144x3248 |              104 | 6144:3248:0:104|
|8K     |        7680x4048 |              136 | 7680:4048:0:136|

### Slow motion percentage
From    |  to  | Perc.
|------:|-----:|------:|
|  59.94|    24|   40% |
| 119.88|    24|   20% |
| 239.76|    24|   10% |
|  59.94| 29.97|   50% |
| 119.88| 29.97|   25% |
| 239.76| 29.97| 12.5% |
|  59.94|    60|  100% |
| 119.88|    60|   50% |
| 239.76|    60|   25% |

## HDR Video Characteristics

Based on the [QuickTime File Format and ProRes Video Parameter Editing](https://github.com/bbc/qtff-parameter-editor).


The colour primaries can be selected from the list:

| No.  | Colour Primaries  |
| -----| --------------    |
|0     | Reserved          |
|1     | ITU-R BT.709      |
|2     | Unspecified       |
|3     | Reserved          |
|4     | ITU-R BT.470M     |
|5     | ITU-R BT.470BG    |
|6     | SMPTE 170M        |
|7     | SMPTE 240M        |
|8     | FILM              |
|9     | ITU-R BT.2020     |
|10    | SMPTE ST 428-1    |
|11    | DCI P3            |
|12    | P3 D65            |

The transfer function can be selected from the list:

| No.  | Transfer Function                   |
| -----| ---------------------------------   |
|0     | Reserved                            |
|1     | ITU-R BT.709                        |
|2     | Unspecified                         |
|3     | Reserved                            |
|4     | Gamma 2.2 curve                     |
|5     | Gamma 2.8 curve                     |
|6     | SMPTE 170M                          |
|7     | SMPTE 240M                          |
|8     | Linear                              |
|9     | Log                                 |
|10    | Log Sqrt                            |
|11    | IEC 61966-2-4                       |
|12    | ITU-R BT.1361 Extended Colour Gamut |
|13    | IEC 61966-2-1                       |
|14    | ITU-R BT.2020 10 bit                |
|15    | ITU-R BT.2020 12 bit                |
|16    | SMPTE ST 2084 (PQ)                  |
|17    | SMPTE ST 428-1                      |
|18    | ARIB STD-B67 (HLG)                  |

The colour matrix can be selected from the list:

| No.  | Colour Matrix                  |
| -----| ---------------------------    |
|0     |GBR                             |
|1     |BT709                           |
|2     |Unspecified                     |
|3     |Reserved                        |
|4     |FCC                             |
|5     |BT470BG                         |
|6     |SMPTE 170M                      |
|7     |SMPTE 240M                      |
|8     |YCOCG                           |
|9     |BT2020 Non-constant Luminance   |
|10    |BT2020 Constant Luminance       |


## Misc

Install in maOS
```sh
brew install ffmpeg
```

List streams
```sh
ffmpeg -i input.mp4 2>&1 | grep "Stream #"
```

v360 links
* https://ffmpeg.org/ffmpeg-filters.html#v360
* https://www.cxyzjd.com/article/qiutiantxwd/107283224
* https://www.programmersought.com/article/32807747315/


Links (Deprecated)
* [ffmpeg downloads](http://www.videohelp.com/software/ffmpeg)
* [X264 10-BIT BUILDS](https://ffmpeg.zeranoe.com/blog/?p=435)
* [Video Samples](https://www.arri.com/camera/alexa/learn/alexa_sample_footage/)
* [Sources](https://www.ffmpeg.org/faq.html#How-do-I-encode-single-pictures-into-movies_003f)
* [How to encode 10bit H.264 files](http://video.stackexchange.com/questions/13164/encoding-422-in-10-bit-with-libx264)
* [ProRes Info](https://transcoding.wordpress.com/2012/01/29/prores-ffmpeg/)
* [Homebrew](http://brew.sh/)
* [Most complete FFmpeg compilation for Windows](http://oss.netfarm.it/mplayer/)