# Summary

通过语言编码中的码率缩减趋势，Lyra与Opus中的区别比较，Lyra的作用，XDN平台上的高效语音编码技术几个方面探讨新的Google Lyra音频编解码器对实时视频流的意义。



# Introduction

Lyra is a high-quality, low-bitrate speech codec that makes voice communication available even on the slowest networks. To do this it applies traditional codec techniques while leveraging advances in machine learning (ML) with models trained on thousands of hours of data to create a novel method for compressing and transmitting voice signals.

[Google lyra](https://github.com/google/lyra)

## key point
- lattency
- test method/benchmark
-  an audio codec capable of delivering a reasonable facsimile of human speech at 3 kbps



## 语音编码中的码率缩减趋势



就目前尖端语音压缩这个晦涩难懂的世界而言，3 kbps并不是那么稀奇。通过将算法处理限制在300hz到18khz之间的全部或部分声波频率，新旧语音编解码器都比支持人类可听到的全范围声音的音频编解码器具有更高的带宽效率。例如，视频流中使用最广泛的音频编解码器——高级音频编码(AAC)，通常覆盖0至96 kHz的频率范围，通过使用低频增强(LFE)、用于环绕声和其他高级声学中使用的低音箱馈源，可将频率范围扩展至120khz。



AAC被纳入H.264/AVC标准，在使用48 kHz编码采样率的典型立体声设置时消耗带宽为96 kbps，尽管纯音乐应用程序通常以更高的采样率使用AAC，码率一直延伸到512 kbps。相比之下，在WebRTC流媒体通信（包括Duo的）中使用最广泛的下一代语音编解码器Opus，仅以32 kbps的速度就能近乎完美地复制语音，并以低至6 kbps的码率提供可行的语音通信。



对Opus以及G.722和G.711的支持是由WebRTC规范要求的，这意味着它们被主流浏览器支持。像Lyra这样的编解码器可以与WebRTC一起使用，只要它们有应用程序插件支持，例如Duo。



包括 Lyra 和 Opus 在内的许多语音编解码器在带宽受到严重限制下，可以通过将声音复制限制在300hz到8khz甚至500hz到3khz的低频范围内。即使是听起来很糟糕的语音，也足以传达可理解的内容。这些频率范围可以将可理解语音使用的最小码率降低到3 kbps以下水平。



能够做到这一点的编码器包括国防部的增强型混合激励线性预测（eMELP）、3GPP的自适应多速率（AMR）以及Opus的开源前身Speex，这两种编码器都是由[http://Xiph.Org](https://link.zhihu.com/?target=http%3A//Xiph.Org)开发的。此外，MPEG-4第3部分为语音编码指定的编码激发线性预测(CELP)和谐波矢量激发编码(HVXC)算法，旨在支持分别以低至3.65 kbps和2 kbps的码率传输可行的语音。



## 比较Lyra与Opus



在最近的一篇博客文章中，Lyra背后的团队开始对Lyra的特别之处进行评估，他们声称，在3kbps的情况下，该编解码器的性能优于其他所有在该码率下运行的编解码器，其质量也优于Opus在6kbps下运行的编解码器。Google的软件工程师Alejandro Luebs和Chrome产品经理Jamieson Brettle表示:“其他编解码器的码率与Lyra不相上下(Speex、MELP、AMR)，但每一种编解码器都会产生更多的干扰，并产生机器人般的声音。”



但博客中提供的测试样本只包括一个简短的语音片段，由Lyra编码为3 kbps，Opus编码为6 kbps，Speex编码为3 kbps。这些是在这里提到的编解码器中的免版税选项，这可能解释了为什么这些测试样本是唯一包含的。



这些测试报告的质量水平差异似乎很有意义。中立的观众以1-5分的标准产生的平均意见分(MOS)的平均值显示，Lyra为3.5分，Opus为2.5分，Speex为1.7分。不过，如果如作者所坚持的那样，额外的测试表明，8 kbps的Opus相当于3 kbps的Lyra，那么人们就会怀疑，这种码率的节省是否足以让Lyra发挥作用。



## Lyra的作用



显然，Duo的人认为Lyra值得他们花时间。他们指出，Lyra 3 kbps与Opus 8 kbps的等效值相当于减少了60%的消耗带宽，他们断言："新兴市场的数十亿用户可以使用一种高效的低码率编解码器，从而获得比以往更高质量的音频。"



有道理。更好的音频质量是一件好事; 如果一个新的编解码器能够以低得多的码率提供另一个编解码器的质量，那么所有的用户，而不仅仅是那些在带宽有限的市场的用户，都会受益。



不过就目前而言，Lyra的真正影响很可能是对那些没有带宽支持视频通信，但能够拥有像样的音频聊天连接的人。事实上据报道，Google正在加速Lyra的实施，以满足人们仍在使用2G连接或有线拨号连接的地区的需求。



对于使用3G连接的用户来说，用Duo取代Opus不可能带来更多的消费者，因为3G对240p视频的支持完全在该标准的吞吐量范围内，无论是使用H.264时的350 kbps，还是使用Duo使用的开源视频编解码器VP9时的200 kbps。通过使用Lyra以3 kbps的最低音频质量与Opus以8 kbps的质量提供同样的音频质量来节省5 kbps，这对于3G用户是否可以参与视频聊天并不具有决定性意义。



Google团队提出，Lyra与AV1结合使用，与VP9相比，编码效率提高了约40%，可以让 "让用户通过56kbps的拨号调制解调器连接到互联网 "实现视频聊天。但AV1/Lyra组合对于使用2G手机的人来说是行不通的，因为这类手机无法支持AV1所需的处理。



事实上，Google去年表示实施的AV1的使用将仅限于电脑和有足够处理能力处理AV1的5G智能手机。在那些高带宽环境下，Lyra是否会起作用还有待观察。



# 构建







