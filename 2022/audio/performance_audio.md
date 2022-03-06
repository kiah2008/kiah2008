
# Performance Audio

## query property from audiomanager
```java
private AudioTrack createSynth(int[] exclusiveCores){
    // Obtain the optimal output sample rate and buffer size
    AudioManager am = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
    String frameRateString = am.getProperty(AudioManager.PROPERTY_OUTPUT_SAMPLE_RATE);
    String framesPerBufferString =
            am.getProperty(AudioManager.PROPERTY_OUTPUT_FRAMES_PER_BUFFER);
    int mFrameRate = Integer.parseInt(frameRateString);
    int mFramesPerBuffer = Integer.parseInt(framesPerBufferString);
    native_createEngine(Build.VERSION.SDK_INT);
    return native_createAudioPlayer(
            mFrameRate, mFramesPerBuffer, NUM_BUFFERS, exclusiveCores);
}
```
## Native 实现
```c
JNIEXPORT jobject JNICALL Java_com_example_simplesynth_MainActivity_native_1createAudioPlayer(
    JNIEnv *env,
    jclass clazz,
    jint j_frame_rate,
    jint j_frames_per_buffer,
    jint j_num_buffers,
    jintArray j_cpu_ids) {

  AudioStreamFormat format;
  format.frame_rate = (uint32_t) j_frame_rate;
  format.frames_per_buffer = (uint32_t) j_frames_per_buffer;
  format.num_audio_channels = NUM_AUDIO_CHANNELS;
  format.num_buffers = (uint16_t) j_num_buffers;

  synth = new Synthesizer(format.num_audio_channels, format.frame_rate);

  int64_t callback_period_ns = ((int64_t)format.frames_per_buffer * NANOS_IN_SECOND) / format.frame_rate;
  load_stabilizer = new LoadStabilizer(synth, callback_period_ns);

  player = new AudioPlayer(sl_engine_engine_itf,
                           sl_output_mix_object_itf,
                           load_stabilizer,
                           format,
                           api_level);

  jsize length = env->GetArrayLength(j_cpu_ids);

  if (length > 0){
    // Convert the Java jintArray of CPU IDs into a C++ std::vector
    std::vector<int> cpu_ids;
    jboolean isCopy;
    jint *elements = env->GetIntArrayElements(j_cpu_ids, &isCopy);
    for (int i = 0; i < length; i++){
      cpu_ids.push_back(elements[i]);
    }
    player->setCallbackThreadCPUIds(cpu_ids);
  }

  Trace::initialize();

  player->play();
  return player->getAudioTrack();
}
```


