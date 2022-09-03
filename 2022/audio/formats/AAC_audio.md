# AAC格式介绍

首先需要了解的是AAC文件格式有ADIF和ADTS两种，其中ADIF（Audio Data Interchange Format 音频数据交换格式）的特征是解码必须在明确定义的开始处进行，不能从数据流中间开始；而ADTS（Audio Data Transport Stream 音频数据传输流）则相反，这种格式的特征是有同步字，解码可以在这个流中任何位置开始，正如它的名字一样，这是一种和TS流类似的格式。

ADTS格式中每一帧都有头信息，具备流特征，适合于网络传输与处理，而ADIF只有一个统一的头，并且这两种格式的header格式也是不同的。目前主流使用的都是ADTS格式，本文也将以ADTS为重点进行介绍。

ADTS AAC文件格式如下

|             |        |             |        |      |             |        |
| :---------- | :----- | :---------- | :----- | :--- | :---------- | :----- |
| ADTS_header | AAC ES | ADTS_header | AAC ES | …    | ADTS_header | AAC ES |

可以看到每一帧都有头信息，即ADTS_header，其中包含**采样率、声道数、帧长度等信息**。一般ADTS头信息都是7字节，如果有CRC则为9字节。

ADTS帧首部结构如下

| 序号 | 域                              | 长度（bits） | 说明                                                         |
| :--- | :------------------------------ | :----------- | :----------------------------------------------------------- |
| 1    | Syncword                        | 12           | all bits must be 1                                           |
| 2    | MPEG version                    | 1            | 0 for MPEG-4, 1 for MPEG-2                                   |
| 3    | Layer                           | 2            | always 0                                                     |
| 4    | Protection Absent               | 1            | set to 1 if there is no CRC and 0 if there is CRC            |
| 5    | Profile                         | 2            | the MPEG-4 Audio Object Type minus 1                         |
| 6    | MPEG-4 Sampling Frequency Index | 4            | MPEG-4 Sampling Frequency Index (15 is forbidden)            |
| 7    | Private Stream                  | 1            | set to 0 when encoding, ignore when decoding                 |
| 8    | MPEG-4 Channel Configuration    | 3            | MPEG-4 Channel Configuration (in the case of 0, the channel configuration is sent via an inband PCE) |
| 9    | Originality                     | 1            | set to 0 when encoding, ignore when decoding                 |
| 10   | Home                            | 1            | set to 0 when encoding, ignore when decoding                 |
| 11   | Copyrighted Stream              | 1            | set to 0 when encoding, ignore when decoding                 |
| 12   | Copyrighted Start               | 1            | set to 0 when encoding, ignore when decoding                 |
| 13   | Frame Length                    | 13           | this value must include 7 or 9 bytes of header length: FrameLength = (ProtectionAbsent == 1 ? 7 : 9) + size(AACFrame) |
| 14   | Buffer Fullness                 | 11           | buffer fullness                                              |
| 15   | Number of AAC Frames            | 2            | number of AAC frames (RDBs) in ADTS frame minus 1, for maximum compatibility always use 1 AAC frame per ADTS frame |
| 16   | CRC                             | 16           | CRC if protection absent is 0                                |

下面来说一下几个关键字段的含义

## profile



所谓LC即Low Complexity，HE即High Efficiency。

那么profile的取值含义又如何呢？其实在ffmpeg源码中已经有了详细的对应关系，在libavcodec/avcodec.h中，如下

```javascript
#define FF_PROFILE_AAC_MAIN 0
#define FF_PROFILE_AAC_LOW  1
#define FF_PROFILE_AAC_SSR  2
#define FF_PROFILE_AAC_LTP  3
#define FF_PROFILE_AAC_HE   4
#define FF_PROFILE_AAC_HE_V2 28
#define FF_PROFILE_AAC_LD   22
#define FF_PROFILE_AAC_ELD  38
#define FF_PROFILE_MPEG2_AAC_LOW 128
#define FF_PROFILE_MPEG2_AAC_HE  131
```

所以当我们读到profile为28时，就知道这是一个he-aac v2的aac文件了，其每帧sample数目为2048.

## Sampling Frequency Index

表示使用的采样率下标，通过这个下标在 Sampling Frequencies[ ]数组中查找得知采样率的值，对应关系如下：

|      |                                |
| :--- | :----------------------------- |
| 0    | 96000 Hz                       |
| 1    | 88200 Hz                       |
| 2    | 64000 Hz                       |
| 3    | 48000 Hz                       |
| 4    | 44100 Hz                       |
| 5    | 32000 Hz                       |
| 6    | 24000 Hz                       |
| 7    | 22050 Hz                       |
| 8    | 16000 Hz                       |
| 9    | 12000 Hz                       |
| 10   | 11025 Hz                       |
| 11   | 8000 Hz                        |
| 12   | 7350 Hz                        |
| 13   | Reserved                       |
| 14   | Reserved                       |
| 15   | frequency is written explictly |

## Channel Configuration

这个就简单了，就是声道数，对应关系如下 0: Defined in AOT Specifc Config 1: 1 channel: front-center 2: 2 channels: front-left, front-right 3: 3 channels: front-center, front-left, front-right 4: 4 channels: front-center, front-left, front-right, back-center 5: 5 channels: front-center, front-left, front-right, back-left, back-right 6: 6 channels: front-center, front-left, front-right, back-left, back-right, LFE-channel 7: 8 channels: front-center, front-left, front-right, side-left, side-right, back-left, back-right, LFE-channel 8-15: Reserved



## 音频的PTS、DTS

很多文章都说一个aac帧包含1024个sample，其实这是错误的。正确的说法是不同profile决定了每个aac帧含有多少个sample，具体来说，对应关系如下

| PROFILE        | SAMPLES |
| :------------- | :------ |
| HE-AAC v1/v2   | 2048    |
| AAC-LC         | 1024    |
| AAC-LD/AAC-ELD | 480/512 |

音频PTS = DTS，比较简单。

```
F0 F1 F2 F3 ... //编码顺序
 0  1  2  3 ... //DTS
 0  1  2  3 ... //PTS
```

假如音频的采样率为 44100 Hz ,即1秒对声音采样44100次
AAC一帧数据对于LC固定包含1024个采样，那就是一帧数据占用 1024/44100 秒，即 1024/sampling_frequency 秒
那么每编码出一帧的时候，DTS就是 n * 1024/sampling_frequency 秒



# 解决ffmpeg解析aac文件时长不准确的问题

当我们想要利用ffmpeg去获取一个aac文件时长的时候，会发现ffmpeg输出了这么一行warning信息：

```javascript
[aac @ 000000529d929ec0] Estimating duration from bitrate, this may be inaccurate
```



在前面的小节中我们看到，aac文件格式中并没有明确的duration信息，因此ffmpeg选择通过filesize / bitrate来估算时长。

我们闭着眼睛想也知道这种估算方法是不准确的，事实也是如此，而且特别有意思的是，ffmpeg 3.0之前和之后的版本估算出来的结果还不一样，所以开发人员也很贴心的给我们输出了这么一行warning信息。

估算时长这一部分逻辑的代码位于libavformat/utils.c#estimate_timings中

```javascript
if ((!strcmp(ic->iformat->name, "mpeg") ||
         !strcmp(ic->iformat->name, "mpegts")) &&
        file_size && (ic->pb->seekable & AVIO_SEEKABLE_NORMAL)) {
        /* get accurate estimate from the PTSes */
        estimate_timings_from_pts(ic, old_offset);
        ic->duration_estimation_method = AVFMT_DURATION_FROM_PTS;
    } else if (has_duration(ic)) {
        /* at least one component has timings - we use them for all
         * the components */
        fill_all_stream_timings(ic);
        /* nut demuxer estimate the duration from PTS */
        if(!strcmp(ic->iformat->name, "nut"))
            ic->duration_estimation_method = AVFMT_DURATION_FROM_PTS;
        else
            ic->duration_estimation_method = AVFMT_DURATION_FROM_STREAM;
    } else {
    	//aac文件解析时长的时候就会走到这里
        /* less precise: use bitrate info */
        estimate_timings_from_bit_rate(ic);
        ic->duration_estimation_method = AVFMT_DURATION_FROM_BITRATE;
    }
```



那么如何才能获取准确的时长呢？通过上一节的介绍，相信大家都能想到，应该是通过adts frame header取总帧数*每帧时长的值作为duration。

具体来说，获取总帧数就是把整个文件“过”一遍，伪代码如下

```javascript
while (offset < total_size) {
        frame_length = get_adts_frame_length(offset)
        offset += frame_length;
        num_frames++;
    }
```



而获取每帧时长就更简单了：ffmpeg能正确读到每帧的nb_samples和总体的sample_rate，那么两者相除就是每帧的时长了。



## 其它

想要自己详细学习AAC编码格式细节的朋友们当然更推荐直接去看标准文档《MPEG-4 Audio: ISO/IEC 14496-3:2009》





# Reference

> [雷霄骅-视音频编解码技术零基础学习方法](https://blog.csdn.net/leixiaohua1020/article/details/18893769)
>
> 