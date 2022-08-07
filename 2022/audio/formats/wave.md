

 RIFF文件由一个简单的表头（header）跟随着多个"chunks"所组。其格式完全跟IFF一样，除整数的存储方式不一样以外。

- 表头（Header）
  - 4位组（bytes）：固定为"RIFF".
  - 4位组：little-endian 32-bit正整数，整个文件的大小，扣掉识别字符和长度，共8个字节。
  - 4位组：这个文件的类型字符，例如："AVI "或"WAVE".
- 接下来是区块（Chunks），每个区块包含：
  - 4位组：此区块的[ASCII](https://zh.wikipedia.org/wiki/ASCII)识别字，例如："fmt "或"data".
  - 4位组：little-endian 32-bit正整数，表示本区块的长度（这个正整数本身和区块识别字的长度不算在内）。
  - 不固定长度字段：此区块的资料，大小等同前一栏之正整数。
  - 假如区块的长度不为偶数，则填入一个byte。

关于此格式的更多信息，请见互换文件格式（[Interchange File Format](https://zh.wikipedia.org/w/index.php?title=Interchange_File_Format&action=edit&redlink=1)）条目。

![img](wave.assets/wav-sound-format.gif) 

```
Offset  Size  Name             Description

The canonical WAVE format starts with the RIFF header:

0         4   ChunkID          Contains the letters "RIFF" in ASCII form
                               (0x52494646 big-endian form).
4         4   ChunkSize        36 + SubChunk2Size, or more precisely:
                               4 + (8 + SubChunk1Size) + (8 + SubChunk2Size)
                               This is the size of the rest of the chunk 
                               following this number.  This is the size of the 
                               entire file in bytes minus 8 bytes for the
                               two fields not included in this count:
                               ChunkID and ChunkSize.
8         4   Format           Contains the letters "WAVE"
                               (0x57415645 big-endian form).

The "WAVE" format consists of two subchunks: "fmt " and "data":
The "fmt " subchunk describes the sound data's format:

12        4   Subchunk1ID      Contains the letters "fmt "
                               (0x666d7420 big-endian form).
16        4   Subchunk1Size    16 for PCM.  This is the size of the
                               rest of the Subchunk which follows this number.
20        2   AudioFormat      PCM = 1 (i.e. Linear quantization)
                               Values other than 1 indicate some 
                               form of compression.
22        2   NumChannels      Mono = 1, Stereo = 2, etc.
24        4   SampleRate       8000, 44100, etc.
28        4   ByteRate         == SampleRate * NumChannels * BitsPerSample/8
32        2   BlockAlign       == NumChannels * BitsPerSample/8
                               The number of bytes for one sample including
                               all channels. I wonder what happens when
                               this number isn't an integer?
34        2   BitsPerSample    8 bits = 8, 16 bits = 16, etc.
          2   ExtraParamSize   if PCM, then doesn't exist
          X   ExtraParams      space for extra parameters

The "data" subchunk contains the size of the data and the actual sound:

36        4   Subchunk2ID      Contains the letters "data"
                               (0x64617461 big-endian form).
40        4   Subchunk2Size    == NumSamples * NumChannels * BitsPerSample/8
                               This is the number of bytes in the data.
                               You can also think of this as the size
                               of the read of the subchunk following this 
                               number.
44        *   Data             The actual sound data.
```



As an example, here are the opening 72 bytes of a WAVE file with bytes shown as hexadecimal numbers:

```
52 49 46 46 24 08 00 00 57 41 56 45 66 6d 74 20 10 00 00 00 01 00 02 00 
22 56 00 00 88 58 01 00 04 00 10 00 64 61 74 61 00 08 00 00 00 00 00 00 
24 17 1e f3 3c 13 3c 14 16 f9 18 f9 34 e7 23 a6 3c f2 24 f2 11 ce 1a 0d 
```

 ![img](wave.assets/wave-bytes.gif) 



# Wave资源

 wav音频样本可以从维基百科上(https://en.wikipedia.org/wiki/WAV)下载。

注:少数wav格式不支持

| Format                                                       | Bitrate ([kbit/s](https://en.wikipedia.org/wiki/Data_rate_units#Kilobit_per_second)) | 1 minute ([KiB](https://en.wikipedia.org/wiki/Kibibyte)) | Sample                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| 11,025 Hz 16 bit PCM                                         | 176.4                                                        | 1292                                                     | [11k16bitpcm.wav](http://www.nch.com.au/acm/11k16bitpcm.wav) |
| 8,000 Hz 16 bit PCM                                          | 128                                                          | 938                                                      | [8k16bitpcm.wav](http://www.nch.com.au/acm/8k16bitpcm.wav)   |
| 11,025 Hz 8 bit PCM                                          | 88.2                                                         | 646                                                      | [11k8bitpcm.wav](http://www.nch.com.au/acm/11k8bitpcm.wav)   |
| 11,025 Hz [µ-Law](https://en.wikipedia.org/wiki/Μ-Law)       | 88.2                                                         | 646                                                      | [11kulaw.wav](http://www.nch.com.au/acm/11kulaw.wav)         |
| 8,000 Hz 8 bit PCM                                           | 64                                                           | 469                                                      | [8k8bitpcm.wav](http://www.nch.com.au/acm/8k8bitpcm.wav)     |
| 8,000 Hz µ-Law                                               | 64                                                           | 469                                                      | [8kulaw.wav](http://www.nch.com.au/acm/8kulaw.wav)           |
| 11,025 Hz 4 bit [ADPCM](https://en.wikipedia.org/wiki/ADPCM) | 44.1                                                         | 323                                                      | [11kadpcm.wav](http://www.nch.com.au/acm/11kadpcm.wav)       |
| 8,000 Hz 4 bit ADPCM                                         | 32                                                           | 234                                                      | [8kadpcm.wav](http://www.nch.com.au/acm/8kadpcm.wav)         |
| 11,025 Hz GSM 06.10                                          | 18                                                           | 132                                                      | [11kgsm.wav](http://www.nch.com.au/acm/11kgsm.wav)           |
| 8,000 Hz MP3 16 kbit/s                                       | 16                                                           | 117                                                      | [8kmp316.wav](http://www.nch.com.au/acm/8kmp316.wav)         |
| 8,000 Hz GSM 06.10                                           | 13                                                           | 103                                                      | [8kgsm.wav](http://www.nch.com.au/acm/8kgsm.wav)             |
| 8,000 Hz [Lernout & Hauspie](https://en.wikipedia.org/wiki/Lernout_%26_Hauspie) SBC 12 kbit/s | 12                                                           | 88                                                       | [8ksbc12.wav](http://www.nch.com.au/acm/8ksbc12.wav)         |
| 8,000 Hz [DSP Group](https://en.wikipedia.org/wiki/DSP_Group) [Truespeech](https://en.wikipedia.org/wiki/Truespeech) | 9                                                            | 66                                                       | [8ktruespeech.wav](http://www.nch.com.au/acm/8ktruespeech.wav) |
| 8,000 Hz MP3 8 kbit/s                                        | 8                                                            | 60                                                       | [8kmp38.wav](http://www.nch.com.au/acm/8kmp38.wav)           |
| 8,000 Hz Lernout & Hauspie [CELP](https://en.wikipedia.org/wiki/CELP) | 4.8                                                          | 35                                                       | [8kcelp.wav](http://www.nch.com.au/acm/8kcelp.wav)           |



DR lib for wav

https://github.com/mackron/dr_libs/blob/master/dr_wav.h