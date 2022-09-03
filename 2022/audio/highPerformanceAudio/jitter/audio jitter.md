



# 1. Audio Jitter Buffer(音频抖动缓冲区)

Jitter Buffer简单讲就是一个共享数据区,用来接收/存放, 并把数据均匀地发送给音频处理模块.数据包不同的到达时间被称为jitter(抖动). 它可能是由于网络拥塞，定时漂移或路由变更而产生. 在VOIP语音通话中, Jitter Buffer通常实现在语音连接的接收端, 通过目的性延迟数据包的接收,从而消除网络传输带来的不良影响, 使得终端用户有更好的音频体验. 

### 1.1 丢包

我们虽然可以通过[**冗余、NACK、FEC**](#Reference)等机制去抗丢包，但是这些做法不是百分百地恢复。这里的丢包我们可以分成两部分：

- **网络丢包**：报文在网络中被丢掉，通过NACK、FEC、opus Inband FEC无法恢复。一旦出现丢包，对语音肯定存在损伤。这种丢包丢包可以通过**丢包隐藏（Packet Loss Concealment）**来减轻丢包带来的影响，但是PLC毕竟不能完全恢复语音信号，对语音损伤肯定是有的，特别是连续丢包时听感更明显。PLC原理是NetEQ信号处理的部分，等到后续部分讲解。
- **报文迟到**：由于网络拥塞可能会导致大量报文迟到，这些报文错过了播放时间，对于音频播放来说就是“丢包”了。合理的Jitter Buffer缓冲长度能够减轻甚至避免过多PLC导致的卡顿。这种丢包可以可以归入抖动的影响中。

### 1.2 抖动

网络中存在抖动会导致接收端收包不均匀、甚至乱序，如果按照收包顺序去播放，肯定会有时快时慢、来不及播放导致卡顿、错音等问题，在比较差的网络场景中语音基本是没法听。因此，一般语音应用都会有buffer的存在，先缓存一定量的数据保证后续均匀播放，可以牺牲一些延迟去抗抖动，抗抖动的能力取决于buffer的大小。

按照抖动的频率，也可以分为：

- **稳定抖动**：这种抖动是网络和算法固有属性导致的，有一定的统计上稳定性，在一段时间内呈现统计上的稳定性；
- **突发抖动**：这些抖动是在稳定抖动基础上叠加上突发的抖动尖峰，突发通常没有太大规律，无法找到统计上的规律性。

一般的jitter算法都是尽力去解决稳定抖动，对于突发抖动解决的并不是太好。

## 1.3  抖动的计算方法



![image-20220814183936855](audio%20jitter.assets/image-20220814183936855.png)

<center>抖动缓冲区工作原理示意图</center>



## 2. Jitter Buffer分类以及NetEQ中Jitter Buffer

Jitter buffer自语音通话以来一直在发展，一般来说Jitter buffer主要有两种，即静态Jitter Buffer和动态Jitter Buffer：

- **静态jitter buffer**，jitter buffer深度Depth固定，可以抗Depth以下的抖动。一些固话应用中因为线路稳定，还会会使用静态jitter buffer。这种静态深度方式实现简单，存在固定延迟，在稳定信道中表现不俗，但是一旦抖动超过范围就导致语音有损伤。
- **动态jitter buffer**，会自动根据网络抖动调整Jitter Buffer深度。一般各家做法中设计都比较简单，会根据当前抖动情况调整jitter buffer深度，检测到更高的抖动后会hold比较长的时间才能降下来，会导致长时间的延迟比较大。



# 2. WebRTC Jitter Buffer

![image-20220903182220007](audio%20jitter.assets/image-20220903182220007.png)

音频引擎主要包括:

iSAC(Internet Speech Audio Codec)/iLBC(internet Low Bitrate Codec)解码器, 回声消除(Acoustic Echo Canceler-AEC)/噪声抑制(Noise Reduction-NR)以及NetEQ三大部分;

![image-20220814183248406](audio%20jitter.assets/image-20220814183248406.png)





## 2.1 抖动的定义

![image-20220903183716396](audio%20jitter.assets/image-20220903183716396.png)

## 2.2 抖动消除

- 抖动缓冲区算法

  - 静态抖动缓冲算法控制
  - 动态抖动缓冲算法控制

- 丢包隐藏原理

  ![image-20220903191038607](audio%20jitter.assets/image-20220903191038607.png)

- 

  

## 2.3 NetEQ

WebRTC NetEQ中的jitter buffer，设计相对更复杂，效果也是比普通的静态jitter buffer、动态jitter buffer更好。NetEQ中的jitter buffer和上面说的jitter buffer一样也会动态调整jitter buffer深度，但是调整更精细，还配合了加速、减速、PLC等策略：

- 和传统jitter buffer类似，可以配置jitter buffer的min buffer、max buffer、start buffer。
- 从统计上估计当前当前网络jitter，动态调整jitter buffer的深度。
- 调整jitter buffer深度时同时使用快放、慢放等策略，保障语音质量。
- 考虑到突发抖动的影响（老代码中存在peak detector检测）。
- 统计上发现网络变好时调整jitter buffer深度，兼顾语音质量和延迟。

![image-20220814182448700](audio%20jitter.assets/image-20220814182448700.png)

NetEQ主要有四个部分, 

- Adaptive packet buffer(自适应缓冲器)
- Speech  decoder(语音解码器)
- Jitter control and error concealment(抖动控制及丢包隐藏)
- playout(播放)

Jitter control and error concealment主要有三大主要操作所组成:

- Expansion: 扩展操作, 即对语音时长的拉伸, 其中包括expand及preemptive_expand 两种模式. 前者为丢包隐藏操作, 其作用是等待延迟包并补偿丢包, 由于补偿丢包是实现数据从无到有, 因此使用expand, 表示语音数据的扩展; 后者以为有限扩展, 及在原有数据的基础上拉伸语音时长, 因此可以实现减速播放功能;
- norma: 正常播放操作, 即在网络状况正常且相对无抖动的操作;
- Accelerate: 加速操作, 即对语音信号处理以实现快速播放;

## 2.4 NetEQ介绍

![image-20220903191516668](audio%20jitter.assets/image-20220903191516668.png)

NetEQ模块主要包含MCU和DSP量大处理单元, 音频解码器模块以及抖动缓冲区(Packet/Jitter buffer)和语音缓冲区(Speech Buffer)

![image-20220903194136266](audio%20jitter.assets/image-20220903194136266.png)

当语音引擎运行时, 会根据网络状况

NetEQ主要有两个线程，一个线程往NetEQ中插入报文（InsertPacket），并估计当前网络jitter情况；另外一个线程10ms调度一次，解码处理得到10ms音频（GetAudio）。

下面深入下这两个线程操作，逐步剖析NetEQ的jitter buffer原理。

### 3.1 估计当前需要buffer的时间：target delay

这个任务在InsertPacket所在线程完成，主要依靠`DelayManager`中的`直方图`和`DelayPeakDetector`估计当前的jitter，并得到一个合理地延迟时间：target delay。直方图可以估计一个稳定的值，而peak是瞬态的，所以他们分别估计的是稳定抖动和突发抖动。

target delay估计入口如下，需要注意几点：

- RFC2198冗余已经在此处去掉，只处理主包main packet
- 如果没有开启`enable_rtx_handling_`，那么则会忽略重传导致的乱序，这个开关对于有重传场景非常重要，如果不开启会导致该场景检测不准

```cpp
    // Update statistics.
    if ((enable_rtx_handling_ || (int32_t)(main_timestamp - timestamp_) >= 0) &&
        !new_codec_) {
      // Only update statistics if incoming packet is not older than last played
      // out packet or RTX handling is enabled, and if new codec flag is not
      // set.
      delay_manager_->Update(main_sequence_number, main_timestamp, fs_hz_);
    }
```

这里的jitter估计是根据相邻包的**包间间隔（IAT，*inter-arrival time*）**来计算的，然后得到直方图，再取95%值作为估计。

直接根据代码来讲解吧！每次update会根据包计算timestamp来更新一个包的长度，这个长度后面用来计算以packet为单位的IAT。

```cpp
  // Try calculating packet length from current and previous timestamps.
  int packet_len_ms;
  if (!IsNewerTimestamp(timestamp, last_timestamp_) ||
      !IsNewerSequenceNumber(sequence_number, last_seq_no_)) {
    // Wrong timestamp or sequence order; use stored value.
    packet_len_ms = packet_len_ms_;
  } else {
    // Calculate timestamps per packet and derive packet length in ms.
    int64_t packet_len_samp =
        static_cast<uint32_t>(timestamp - last_timestamp_) /
        static_cast<uint16_t>(sequence_number - last_seq_no_);
    packet_len_ms =
        rtc::saturated_cast<int>(1000 * packet_len_samp / sample_rate_hz);
  }
```

计算IAT也比较简单，上次seq到当前seq经历了多长时间，换算成packet为单位。所以IAT就是两次收到seq的间隔。

```cpp
    // Cannot update statistics unless |packet_len_ms| is valid.
    // Calculate inter-arrival time (IAT) in integer "packet times"
    // (rounding down). This is the value used as index to the histogram
    // vector |iat_vector_|.
    int iat_packets = packet_iat_stopwatch_->ElapsedMs() / packet_len_ms;
```

大部分情况下，这个计算是准确的，但是万一有乱序和丢包呢？对于丢包，需要扣除丢包的gap；对于乱序需要加上乱序的包数。

```cpp
    if (IsNewerSequenceNumber(sequence_number, last_seq_no_ + 1)) {
      // Compensate for gap in the sequence numbers. Reduce IAT with the
      // expected extra time due to lost packets, but ensure that the IAT is
      // not negative.
      iat_packets -= static_cast<uint16_t>(sequence_number - last_seq_no_ - 1);
      iat_packets = std::max(iat_packets, 0);
    } else if (!IsNewerSequenceNumber(sequence_number, last_seq_no_)) {
      iat_packets += static_cast<uint16_t>(last_seq_no_ + 1 - sequence_number);
      reordered = true;
    }
```

通过对丢包和乱序的修正，再将这个数据输入到直方图中。直方图在这里是64个（可以配置）bucket，分别对应IAT 1个包~64个包的个数占比（实际上在代码实现上，采用了稍微复杂的方式，但是这里只说思想哈，大家看代码）。

```cpp
    // Saturate IAT at maximum value.
    const int max_iat = kMaxIat;
    iat_packets = std::min(iat_packets, max_iat);
    UpdateHistogram(iat_packets);
    // Calculate new |target_level_| based on updated statistics.
    target_level_ = CalculateTargetLevel(iat_packets, reordered);
```

直方图取95%值：

```cpp
  size_t index = 0;           // Start from the beginning of |iat_vector_|.
  int sum = 1 << 30;          // Assign to 1 in Q30.
  sum -= iat_vector_[index];  // Ensure that target level is >= 1.

  do {
    // Subtract the probabilities one by one until the sum is no longer greater
    // than limit_probability.
    ++index;
    sum -= iat_vector_[index];
  } while ((sum > limit_probability) && (index < iat_vector_.size() - 1));
```

再计算peak值，和上面直方图计算的值取大：

```cpp
  // Update detector for delay peaks.
  bool delay_peak_found =
      peak_detector_.Update(iat_packets, reordered, target_level);
  if (delay_peak_found) {
    target_level = std::max(target_level, peak_detector_.MaxPeakHeight());
  }
```

DelayPeakDetector是用来检测一些峰值，这些峰值是偶发的，通过有平滑功能的直方图是无法检测出来的。而峰值出现一次意味着后续还回出现，所以出现几次峰值后就必须调整delay。这里大概介绍下峰值计算方法：

- 峰值认定条件：当前计算出的iat（一般忽略重传导致的乱序）减去上次最终target delay差，超过阈值78ms；或者当前iat超过上次target delay两倍。

- 第一次检测出peak，不计入

- 距离上次peak在10s（可配置）内，加入history，并记录距离上次peak间隔

- 距离上次peak在10s~2*10s内，超时，不计入

- 距离上次peak超过2*10s，两次peak太长，需要reset，清空history，后续还要从第一次算起

- peak找到的条件需要满足一下两个条件

- - history中有2个（可配置）及以上peak
  - 当前检查时间距离上次peak间隔小于两倍最大peak间隔，其实肯定是小于2*10s的！

peak检测算法：

```cpp
if (inter_arrival_time > target_level + peak_detection_threshold_ ||
      inter_arrival_time > 2 * target_level) {
    // A delay peak is observed.
    if (!peak_period_stopwatch_) {
      // This is the first peak. Reset the period counter.
      peak_period_stopwatch_ = tick_timer_->GetNewStopwatch();
    } else if (peak_period_stopwatch_->ElapsedMs() > 0) {
      if (peak_period_stopwatch_->ElapsedMs() <= kMaxPeakPeriodMs) {
        // This is not the first peak, and the period is valid.
        // Store peak data in the vector.
        Peak peak_data;
        peak_data.period_ms = peak_period_stopwatch_->ElapsedMs();
        peak_data.peak_height_packets = inter_arrival_time;
        peak_history_.push_back(peak_data);
        while (peak_history_.size() > kMaxNumPeaks) {
          // Delete the oldest data point.
          peak_history_.pop_front();
        }
        peak_period_stopwatch_ = tick_timer_->GetNewStopwatch();
      } else if (peak_period_stopwatch_->ElapsedMs() <= 2 * kMaxPeakPeriodMs) {
        // Invalid peak due to too long period. Reset period counter and start
        // looking for next peak.
        peak_period_stopwatch_ = tick_timer_->GetNewStopwatch();
      } else {
        // More than 2 times the maximum period has elapsed since the last peak
        // was registered. It seams that the network conditions have changed.
        // Reset the peak statistics.
        Reset();
      }
    }
  }
```

peak是否找到的检查条件：

```cpp
bool DelayPeakDetector::CheckPeakConditions() {
  size_t s = peak_history_.size();
  if (s >= x &&
      peak_period_stopwatch_->ElapsedMs() <= 2 * MaxPeakPeriod()) {
    peak_found_ = true;
  } else {
    peak_found_ = false;
  }
  return peak_found_;
}
```

直方图计算的target delay和peak detector计算出来的target delay取大，就是最终的target delay。注意下，这里的target delay是以packet（target level）为单位计算的，使用时需要注意下。



### 3.2 根据target delay决策加速、减速、PLC

计算完target delay后，neteq的目标是保证buffer的数据维持在这个target delay左右，太高或者太低都不行。buffer的数据高于target delay很多，需要尽快排空导致加速播放；buffer 的数据低于target delay很多，需要做慢放避免很快排空。

上面所说的buffer数据包含两部分，一个是packet buffer中的数据，一个是sync buffer中的数据（sync buffer中的数据是已经解码可以用于播放的数据）。这个buffer数据长度有做过简单滤波，见BufferLevelFilter类，建议大家看代码了解详细，大概来说就是做一个平滑，平滑系数和当前target level有关，level越高平滑越大。

线程每次从NetEQ中去10ms数据用于播放，并做一些决策。在当前取的数据没有丢的情况下，会根据buffer大小和target delay差距做匀速、加速、减速。

- 如果buffer数据远小于target delay，说明数据太少，要做减速拖延时间避免无数据播放；
- 如果buffer数据远大于target delay，说明累积了太多数据，需要加速播放避免累积数据太多延迟太大；
- 否则不做处理直接播放。
- 如果取数据发现没有数据可以播放，则做PLC；
- 如果取数据发现当前的数据和上次数据之间有gap，则做PLC；
- 如果之前已经做过PLC，当前数据没有丢可以播放，但是buffer低于target delay太多，避免很快排空buffer而导致抗抖动能力低，会继续做PLC。

一个关键的流程看这个地方，对应着没有丢包的加速、减速处理，以及有丢包的（FeaturePacketAvailable）处理：

```cpp
  // Check if the required packet is available.
  if (target_timestamp == available_timestamp) {
    return ExpectedPacketAvailable(prev_mode, play_dtmf);
  } else if (!PacketBuffer::IsObsoleteTimestamp(
                 available_timestamp, target_timestamp, five_seconds_samples)) {
    return FuturePacketAvailable(
        sync_buffer, expand, decoder_frame_length, prev_mode, target_timestamp,
        available_timestamp, play_dtmf, generated_noise_samples);
  } else {
    // This implies that available_timestamp < target_timestamp, which can
    // happen when a new stream or codec is received. Signal for a reset.
    return kUndefined;
  }
```





规则大概如上面所写，这里只讲基本原理，去掉了一些无关紧要的细节。大家可以再看代码去仔细了解。



> PLC 语音修补（ Packet loss concealment） ： PLC机制可以在语音数据封包遗失的情况下仍然维持一定水准的通话品质
>
> NetEQ
>
> 

# 其它流媒体协议

## xquic

## rtsp



# 低码率

## Lyra



# Reference

## 1. [白话解读 WebRTC 音频 NetEQ 及优化实践](https://segmentfault.com/a/1190000039425822)

## 2. [NetEQ 算法](https://blog.csdn.net/u014338577/article/details/46010983?spm=a2c6h.12873639.0.0.5e54504c4dHXyK)
## 3. 



