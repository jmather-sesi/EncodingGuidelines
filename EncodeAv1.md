---
layout: default
nav_order: 4.5
title: AV1 Encoding
parent: Codec Comparisons
---

# AV1

AV1 is an open-source and royalty free codec developed by the [Alliance for Open Media](https://en.wikipedia.org/wiki/Alliance_for_Open_Media) (AOMedia), a non-profit industry consortium. It can be 50% higher efficiency than h264. 

The royalty free and open-source features of the codec make it extremely interesting to the VFX community, however most of the codecs are significantly slower than h264, which may be a major hurdle for adoption. Having said that, they are actively being developed, so this could change at some point.

General ffmpeg info on AV1 is [here](https://trac.ffmpeg.org/wiki/Encode/AV1).

AV1 has browser support in:
   * Chrome
   * Edge
   * Firefox
   * Opera
the main missing one is Safari. Apple has now joined the AOM group, and we are starting to see AV1 hardware support in their latest hardware, but currently safari does still not support it.

Outside of the web browser, AV1 support is pretty much limited to ffmpeg and VLC.

AV1 is supported by mp4 and webm containers, no support exists for mov.

There are three encoders available via ffmpeg:
* [libsvtav1](#libsvtav1)
* [libaom-av1](#libaom-av1)
* [librav1e](#librav1e)


## Libsvtav1
[SVT-AV1](https://gitlab.com/AOMediaCodec/SVT-AV1) was originally developed by Intel with Netflix. In 2022 it was adopted by AOMedia as the basis of future development for AV1.
It has a more limited support for pix-formats currently only supporting: yuv420p yuv420p10le

Supported pixel formats: yuv420p yuv420p10le

Color modes - 

| main | 8 or 10 bit | 4:2:0 |
| high | 8 or 10 bit | 4:2:0 or 4:4:4 |
| professional | 8 or 10 or 12 bit | 4:2:0 4:2:2 or 4:4:4 |

Example encoding:

<!---
name: test_libsvtav1
sources: 
- sourceimages/chip-chart-1080-16bit-noicc.png.yml
comparisontest:
   - testtype: idiff
     compare_image: ../sourceimages/chip-chart-1080-16bit-noicc-yuv420p10le.png
   - testtype: assertresults
     tests:
     - assert: less
       value: max_error
       less: 0.00195
-->
```
ffmpeg -r 24 -start_number 1 -i inputfile.%04d.png -frames:v 200 -c:v libsvtav1 \
        -pix_fmt yuv420p10le -crf 18 -preset 9  -svtav1-params tune=0 -sws_flags lanczos \
        -vf "scale=in_range=full:in_color_matrix=bt709:out_range=tv:out_color_matrix=bt709" \
        -color_range tv -colorspace bt709 -color_primaries bt709 -color_trc iec61966-2-1\
        -y outputfile.mp4
```

| --- | --- |
| **-crf 18** | This is the constant rate factor, controlling the default quality in the range 0-63. By default this is set to 50, which is a little on the low side, using values closer to 18 is recommended, but this does come at the expense of file-size. For more on this see the [CRF comparison](CRF Comparison) below. |
| **-preset 9** | Help with a trade-off between encoding speed and compression efficiency. Supported preset range in the 0-13. See below for comparisons |

See also: 
   * [SVT-AV1 ffmpeg](https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/Ffmpeg.md)
   * [SVT-AV1 arguments](https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/Parameters.md)
   * [SVT-AV1 Common Args](https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/CommonQuestions.md)

### CRF Comparison for libsvtav1

To help pick appropriate values with the CRF flag, we have run the [Test Framework](enctests/README.html) through some of the [reference media](enctests/sources/enc_sources/README.html).

| ![](enctests/reference-results/av1-crf-test-encode_time.png)  This is showing CRF values against encoding time. |
| ![](enctests/reference-results/av1-crf-test-filesize.png) This is showing CRF values against file size. |
| ![](enctests/reference-results/av1-crf-test-vmaf_harmonic_mean.png) This is showing CRF values against VMAF harmonic mean |


### Preset values for libsvtav1

See: [SVT-AV1 Common Questions](https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/CommonQuestions.md)

## libaom-av1

This is the reference encoder

Supported pixel formats:
yuv420p yuv422p yuv444p gbrp yuv420p10le yuv422p10le yuv444p10le yuv420p12le yuv422p12le yuv444p12le gbrp10le gbrp12le gray gray10le gray12le


{: .warning }
All our initial testing is showing libaom being more than 10x slower at encoding than svt-av1. It needs further exploration to determine if there are ways of getting better encoding times. Unfortunately for many pixel formats, libaom is the only option for av1 encoding (e.g. 422, or 444 encoding).

Example encoding:

```
ffmpeg -r 24 -start_number 1 -i inputfile.%04d.png -frames:v 200 -c:v libaom-av1  \
        -pix_fmt yuv420p10le -cpu-used 2 -crf 20 -row-mt 1 -sws_flags lanczos \
        -vf "scale=in_range=full:in_color_matrix=bt709:out_range=tv:out_color_matrix=bt709" \
        -color_range tv -colorspace bt709 -color_primaries bt709 -color_trc iec61966-2-1\
        -y outputfile.mp4
```


| --- | --- |
| -cpu-used 2 | This sets how efficient the compression will be. The default is 0, changing this will increase encoding speed at the expense of having some impact on quality and rate control accuracy.  |
| -row-mt 1 | This enables row based multi-threading (see [here](https://trac.ffmpeg.org/wiki/Encode/VP9#rowmt)) which is not enabled by default. |



## librav1e
librav1e is the Xiph encoder for AV1, 

Supported pixel formats:
yuv420p yuvj420p yuv420p10le yuv420p12le yuv422p yuvj422p yuv422p10le yuv422p12le yuv444p yuvj444p yuv444p10le yuv444p12le

There is no CRF flag, so we are ignoring this for now.

TODO: Test.

See Also: 
   * https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/svt-av1_encoder_user_guide.md
   * https://gitlab.com/AOMediaCodec/SVT-AV1/-/blob/master/Docs/Ffmpeg.md