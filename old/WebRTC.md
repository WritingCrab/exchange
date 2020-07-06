主要看modules/audio_processing中的内容。

编译方法

```shell
gn gen out/Default
#gn clean out/Default
ninja -C out/Default audio_processing
ninja -C out/Default audioproc_f
```

audioproc_f的使用方法

```shell
./audioproc_f -i cap.wav -ri play.wav -o out.wav -aec 1
```

注：2019.3.6Per Ahgern的提交中，将AEC3作为目前的默认aec了，之前的aec需使用选项-use_legacy_aec

今天开始看代码

### `modules/audio_processing/include/audio_processing.h`

其中包含了较多注释

- `ExtendedFilter`，用以在AEC中enable扩展滤波器模式
- `RefinedAdapteiveFilter`，用以在回声消除中使能refined滤波器，该配置仅可用于非移动设备回声消除，可在constructor中使用，也可以使用`AudioProcessing::SetExtraOptions()`
- `DelayAgonstic`，enable不可知延迟回声消除，该特性依赖内部延迟估计，不依赖上报延迟，该配置仅可用于非移动设备回声消除，可在constructor中使用，也可以使用`AudioProcessing::SetExtraOptions()`
- `ExperimentalAgc`，用以使能试验性AGC，启动时将试验性AGC将麦克风音量放大到`startup_min_volume`，需要通过`AudioProcessingBuilder().Create(config)`
- `ExperimentalNs`，用以使能试验性ns，可在constructor中使用，也可以使用`AudioProcessing::SetExtraOptions()`

Audio Processing Module(APM)为实时通信软件提供了声音处理的工具集

APM逐帧处理两个音频流，应用所有操作的主流的frame，传到`ProcessStream()`，反向流的frame传到`ProcessReverseStream()`，在client侧，这些分别是near-end (capture) and far-end (render) 流。APM应该尽可能位于信号链上最接近硬件抽象层的部分

在server侧，处理传入流时，通常不会使用反向流

**这里提到的server和client是什么**

组件接口遵循类似的模式，并且可以通过APM中响应的获取器进行访问，所有的组件在创建时都是使用大多数情况的默认设置并且disabled，新的设置生效无需enable组件，enable组件时触发内存分配和初始化来允许组件开始处理流

在以下设想情况下，提供线程安全性以降低lock开销：
- 流getter和setter与ProcessFrame()在同一个线程进行调用，更精确地说，流函数不会与ProcessFrame()同时调用
- 参数getter与其对应的setter不会同时调用

**流setter是什么？上述内容是指它代码中实现的东西是吗？**

APM仅接受一大块10ms的线性音频数据，int16接口使用交错数据，而float接口使用非交错数据

**int16接口和float接口是什么？交错数据与费交错数据是什么？**

使用方法举例，忽略错误检查

```cpp
AudioProcessing* apm = AudioProcessingBuilder().Create();

AudioProcessing::Config config;
config.echo_canceller.enabled = true;
config.echo_canceller.mobile_mode = false;
config.high_pass_filter.enabled = true;
config.gain_controller2.enabled = true;
apm->ApplyConfig(config)

apm->noise_reduction()->set_level(kHighSuppression);
apm->noise_reduction()->Enable(true);

apm->gain_control()->set_analog_level_limits(0, 255);
apm->gain_control()->set_mode(kAdaptiveAnalog);
apm->gain_control()->Enable(true);

apm->voice_detection()->Enable(true);

// Start a voice call...

// ... Render frame arrives bound for the audio HAL ...
apm->ProcessReverseStream(render_frame);

// ... Capture frame arrives from the audio HAL ...
// Call required set_stream_ functions.
apm->set_stream_delay_ms(delay_ms);
apm->gain_control()->set_stream_analog_level(analog_level);

apm->ProcessStream(capture_frame);

// Call required stream_ functions.
analog_level = apm->gain_control()->stream_analog_level();
has_voice = apm->stream_has_voice();

// Repeate render and capture processing for the duration of the call...
// Start a new call...
apm->Initialize();

// Close the application...
delete apm;
```

接下来是一个类`AudioProcessing`

**Config**

- *`struct Config`*，结构体由音频处理的新参数方案组成，逐渐引入同时在完全引入前都有可能变化。通过修改`AudioProcessing::Config`中的默认值来控制APM的参数和行为，将结构体传入`ApplyConfig()`来应用config。此配置应该在setup期间使用，来enable/disable顶层处理效果，在处理期间使用可能导致不期望的子模块重置，影响音频质量，使用`RuntimeSetting`来进行运行时配置
- `Config::struct Pipeline`，设置音频处理流程的属性
- `Config::Pipeline::maximum_internal_processing_rate`，内部使用的处理rate，只能设置为32000和48000，其他值将被视为48000，目前默认值基于CPU架构进行选择，但将来有可能变化 **!!!**
- `Config::Pipeline::experimental_multi_channel`，强制对playout和capture音频进行多通道处理，试验性特性，很有可能不经警告进行修改
- `Config::struct PreAmplifier`，enable预放大器，在其他处理完成前放大采集信号 **完成前？**
- `Config::struct HighPassFilter`，*高通滤波器*，**干吗用的**
- `Config::struct EchoCanceller`，*默认应该就是aec3，其中的`enabled`是总开关吧？*
- `Config::struct NoiseSuppression`，enable背景噪声抑制
- `Config::struct VoiceDetection`，enable `webrtc::AudioProcessingStats`中的`has_audio` **audio_processing之外的东西对我们可能就用处不大了**
- `Config::struct GainController1`，enable AGC功能，AGC将信号带到合适的范围，应用数字增益实现，模拟模式下，规定模拟增益应用到音频HAL，建议在client侧启动 **重点是启动还是client侧？**
- `Config::struct GainController2`，enable下一代AGC功能
- 其他AGC的内容粗略看了一下，需要的时候再看
- `Config::struct ResidualEchoDetector`，enable剩余回声检测器 **不太清楚干嘛用的**
- `Config::struct LevelEstimation`，enable `webrtc::AudioProcessingStats`中的`output_rms_dbfs`
- `Config::Config& operator=`，重载`=`运算符以进行显式copy
- `Config::std::string ToString() const`，其实就是返回该实例的一个字符串，里面是什么都无所谓，开发者定义好了的

**ChannelLayout**

- `enum ChannelLayout`，就是单通道，立体声这些，将来应该是会被移除

**RuntimeSetting**

- `class RuntimeSetting`，当AudioProcessing运行时传入属性，这个类应该不是具体操作，应该是调用类的静态函数，根据传入的参数，通过`RTC_DCHECK_XX`检查参数合法性，然后返回一个`class RuntimeSetting`，将类作为参数再通过某个接口传入`AudioProcessing`

**方法**
定义了一些成员虚函数
- `int Initialize()`，初始化内部状态同时保留用户设置，应当在开始处理一个新的音频流之前调用，在开始第一个流的处理之前就没必要调这个了
- `int Initialize(const ProcessingConfig& processing_config)`，仅使用`NativeRate`，input，output，reverse的rate要匹配，`processing_config.output_stream()`和`processing_config.input_stream()`要匹配。float接口接受任意rate并支持input和output具有不同的layout，output必须有要么1个channel，要么其channel数与input一致
- `int Initialize(int capture_input_sample_rate_hz, int capture_output_sample_rate_hz, int render_sample_rate_hz, ChannelLayout capture_input_layout, ChannelLayout capture_output_layout, ChannelLayout render_input_layout)`，跟前面一样，但是这个后面应该会被**删除**
- `void ApplyConfig(const Config& config)`，用于控制APM参数的临时解决方案，可能会改动，参数就是前面的`struct Config`
- `void SetExtraOptions(const webrtc::Config& config)`，传进额外的没有显式setter的选项，保证选项可以立刻被应用
- `int proc_sample_rate_hz()`，emmm似乎是内部使用的，所以不知道对我们有没有用
- `int proc_split_sample_rate_hz()`
- `size_t num_input_channels()`
- `size_t num_proc_channels()`
- `size_t num_output_channels()`
- `size_t num_reverse_channels()`
- `void set_output_will_be_muted(bool muted)`，当AudioProcessing的output被静音或其他方式不使用output时设置为`true`，理想情况，采集音频依旧会被处理，但一些组件可能因为这个而改变行为，默认为`false`
- `void SetRuntimeSetting(RuntimeSetting setting)`，将运行时设置Enqueue，参数就是前面的`class RuntimeSetting`
- `int ProcessStream(AudioFrame* frame)`，处理一段主音频流的10ms的`frame`，在client侧，就是near-end或captured音频。当任何已启用功能需要时，任何`set_stream_`开头的函数应该先于处理当前frame之前调用，任何需要的`stream_`开头的getter函数应在处理后调用。`frame`的成员`sample_rate_hz_`，`num_channels_`，`samples_per_channel_`必须是有效的。If changed from the previous call to this method, it will trigger an initialization. **不知道该怎么理解，是说从前一次调用到这次调用参数有变化的时候会触发初始化吗？重点是主语是什么，是指前面提到的参数吗？还有没找到`AudioFrame`的定义**
- `int ProcessStream(const float* const* src, size_t samples_per_channel, int input_sample_rate_hz, ChannelLayout input_layout, int output_sample_rate_hz, ChannelLayout output_layout, float* const* dest)`，接收去交错的float音频，范围[-1, 1]，`src`中的每个元素指向一个channel buffer，按`input_layout`进行排列。对于output，channels按`output_layout`在`output_sample_rate_hz`排列在`dest`。output要么有一个通道，要么与input一致，`src`和`dest`会占用相同的内存，这个后面可能会被删除
- `int ProcessStream(const float* const* src, const StreamConfig& input_config, const StreamConfig& output_config, float* const* dest)`，跟前面一样，只不过参数使用StreamConfig而不是散着写
- `int ProcessReverseStream(AudioFrame* frame)`，处理一段反向音频流的10ms的`frame`，frame可能被修改，在client侧，就是far-end或rendered音频。当回声处理enable时，必须提供反向流，反向流组成了回声参考信号。当AGC enable时，建议提供反向流，但不必须。在server侧一般不会用到，如果你不确定该往里传啥，那估计你根本就用不到。`frame`的成员`sample_rate_hz_`，`num_channels_`，`samples_per_channel_`必须是有效的。
- `int AnalyzeReverseStream(const float* const* data, size_t samples_per_channel, int sample_rate_hz, ChannelLayout layout)`，输入参数看就知道是啥意思，但是接口的具体作用不清楚
- `ProcessReverseStream(const float* const* src, const StreamConfig& input_config, const StreamConfig& output_config, float* const* dest)`，跟前面的一样，我在想前面的一个是不是写错了或者还没统一起来
- `void set_stream_analog_level(int level)`，当且仅当自适应模拟增益控制enabled时，必须在`ProcessStream()`调用之前调用这个，将当前来自audio HAL来的模拟电平传进去，必须设置`Config::GainController1`提供的范围内
- `int recommended_stream_analog_level()`，当模拟模式enabled，应该在`ProcessStream()`调用之后调用这个来获取推荐的新的提供给audio HAL的新的电平，应用电平应该用户去做
- `int set_stream_delay_ms(int delay)`，当且仅当回声处理enabled时必须调用。在`ProcessReverseStream()`接收到一个far-end frame和`ProcessSreeam()`接收到一个包含对应的回声的near-end frame之间，以ms为单位设置`delay`，在client侧，可以被表示为如下，其中`t_analyze`是frame传到`ProcessReverseStream()`的时间，`t_render`是同一frame的第一个sample被声音硬件渲染的时间，`t_capture`是采集设备采集到一个frame的第一个sample的时间，`t_process`是同一frame传到`ProcessStream()`的时间。考虑`t_render`和`t_capture`时间差应该是很小的（因为mic和play应该离得很近），所以实际上就是`t_process-t_analyse`
```
delay = (t_render - t_analyze) + (t_process - t_capture)
```
- `int stream_delay_ms()`，应该是获取当前的`delay`吧
- `bool was_stream_delay_set()`，判断现在是否已经设置了`delay`？
- `void set_stream_key_pressed(bool key_pressed)`，通知按键按下？key press指的是什么？
- `void set_delay_offset_ms(int offset)`，设置一个`offset`，加到通过`stream_delay_ms()`设置的`delay`上，可正可负。注意，这可能导致设置到`stream_delay_ms()`的其他有效值返回错误 **也就是设置新的`delay`之前要考虑`offset`**
- `int delay_offset_ms()`，相应的，应该是获取当前的`offset`吧
- `void AttachAecDump(std::unique_ptr<AecDump> aec_dump)`，attach提供的`webrtc::AecDump`来记录调试信息，log文件和最大文件大小通过`AecDump`的实例实现，当有另一个AecDump已经attach时调用该方法将替换为新的一个。将导致更早些的AecDump调用其析构，可能阻塞直到所有等待中的log任务完成 **d-tor应该就是析构函数destructor**
- `void DetachAecDump()`，没有`AecDump` attach时将没有影响，如果有attach的，将调用其析构函数，析构会阻塞到所有等待中的log任务完成
- `void AttachPlayoutAudioGenerator(std::unique_ptr<AudioGenerator> audio_generator)`，attach提供的`webrtc::AudioGenerator`来修改playout音频，当另一个AudioGenerator attach时，替换为新的一个
- `void DetachPlayoutAudioGenerator()`，没有attach的就没有影响，有的话就调析构
- `void UpdateHistogramsOnCallEnd()`，调用结束时发送UMA直方图，将被移除 **UMA**
- `AudioProcessingStats GetStatistics(bool has_remote_tracks)`，获取音频处理统计 **不知道remote track是什么**
- `GainControl* gain_control()`，这几个都已经const了，应该是马上就完全移除了，几个接口的功能都由`AudioProcessing::ApplyConfig`和`AudioProcessing::SetruntimeSetting`代替了
- `LevelEstimator* level_estimator()`
- `NoiseSuppression* noise_suppression()`
- `VoiceDetection* voice_detection()`
- `AudioProcessing::Config GetConfig()`，返回最后的配置
- `enum Error`，错误返回值表
- `enum NativeRate`，支持的rate，8k，16k，32k，48k **另需要注意前面说的，内部使用的就只有32k和48k，不知是否有关联？**

**class AudioProcessing到这就结束了**

- `class RTC_EXPORT AudioProcessingBuilder`，好像创建AudioProcessing的时候会用到，中间的宏`RTC_EXPORT`不知道用来做什么，里面的接口，不是虚的，允许设置一些外部定义的东西，比如SetEchoControlFactory，SetCapturePostProcessing等等
- `AudioProcessingBuilder::AudioProcessing* Create()`和`AudioProcessingBuilder::AudioProcessing* Create(const webrtc::Config& config)`，使用前面设置的组件创建一个APM实例，也就是上面的那个类的，Create操作会重置`AudioProcessingBuilder`到初始状态

- `class StreamConfig`，就是流的一些信息，不知道keyboard channel是什么

- `class ProcessingConfig`，这个类在前面AudioProcessing的构造函数中用到了，但是没看到这个类自己的构造函数，它只是提供了一些查询用的接口，可以猜测是传进去给AudioProcessing进行初始化的时候它调的，而非给用户调的

**下面这几个还用得到吗？开关不是都在上面那个大类中吗，没标deprecate，还是看一下**

- `class LevelEstimator`，用于检索级别指标的估算组件
- `LevelEstimator::RMS()`，返回根均方等级，会计算直到最后一次调用`RMS()`的所有主流frame，返回值为正，但应该解释为负，值被限制在[0, 127]
- `class NoiseSuppression`，NS组件在保留话音的同时去除噪声，建议在client侧启用
- `NoiseSuppression::Level`，决定抑制的激进程度，增大等级将降低噪声等级但会带来较高的话音失真，应该是调用接口`int set_level()`和`Level level()`进行set和get
- `NoiseSuppression::float speech probability()`，返回在输出通道上平均的内部计算的当前帧的在先语音概率，不知道是什么，感觉用不到
- `NoiseSuppression::std::vector<float> NoiseEstimate()`，返回所有channel上每个频点的噪声估计

**这个可能会有用**

- `class CustomAudioAnalyzer`，试验性的自定义分析模块的接口，包含了几个虚函数，`void Initialize(int sample_rate_hz, int num_channels)`，`void Analyze(const AudioBuffer* audio)`，`std::string ToString()`
- `class CustomProcessing`，类似，这个是用于处理的，几个虚函数，`void Initialize(int sample_rate_hz, int num_channels)`，`void Process(AudioBuffer* audio)`，`std::string ToString()`，`void SetRuntimeSetting(AudioProcessing::RuntimeSetting setting)`
- `class EchoDetector`，回声监测子模块接口，虚函数`Initialize`，`void AnalyzeRenderAudio(rtc::ArrayView<const float> render_audio)`，`void AnalyzeCaptureAudio(rtc::ArrayView<const float> capture_audio)`，`void PackRenderAudioBuffer(AudioBuffer* audio, std::vector<float>* packed_buffer)`，应该是几个通道一起分析
- `class VoiceDetection`，分析流来确定是否有声音，提供了外部VAD结果的传入功能，另外对于`stream_has_voice()`，VAD结论通过`AudioFrame`传到`ProcessFrame()`来提供VAD结论，`vad_activity_`成员会被修改来反应当前结论
- `VideoDetection::bool stream_has_voice()`，检测到声音返回true，在`ProcessFrame()`之后调用
- `VideoDetection::int set_stream_has_voice()`，APM的一些功能需要VAD结论，在外部就有当前frame的VAD结论时，可以通过这个接口传入，在`ProcessFrame()`被调用之前。使用这个接口不需要VAD启用，如果在VAD启用时使用此接口，对于提供了外部VAD结论的任何frame，都会跳过检测（也就是优先使用外部给的）
- `VideoDetection::enum Likelihood`
- `VideoDetection::int set_likelihood(Likelihood likelihood)`和`VideoDetection::Likelihood likelihood()`，设置可可能性，较高的值避免语音被忽略但也容易将噪声视为语音
- `int set_frame_size_ms(int size)`和`int frame_size_ms()`，设置VAD操作的frame `size`，大一些的frame可以有更高的检测精度，但会降低更新频率，不会对传到`ProcessStream()`的frame大小产生影响

**至此文件audio_processing.h基本上就看完了，我是这么认为的，这个文件是提供给我们用户使用的，一大堆类及抽象接口，而开发者在另一边继承这些类后实现其中的虚方法，对于我们来说，实际使用的就是各个类的方法，在更底层应该是调用的其子类中的具体实现，另实现应该在audio_processing/audio_processing_impl.cc中，接下来将看一下这部分内容**

### `modules/audio_processing/audio_processing_impl`

**由于这部分有可能会是具体实现，而我们的主要目的还是使用，所以粗略一点看这部分**

通过一个AudioProcessingImpl : AudioProcessing，在其中完成了具体实现，没有特别实际的内容

---

### demo工作流程

暂时还是不深入看实现，我想了解一下工作流程，可以从demo下手，在modules/audio_processing/test/audioproc_float_main.cc中发现了主函数，main调用`webrtc::test::AudioprocFloat`，创建`AudioProcessingBuilder`，将argc和argv一起传进去（不是apbuilder，是传到函数中），`AudioprocFloat()`位于api/test/audioproc_float.cc，里面就这么一个接口（还有另一个重载），调用`webrtc::test::AudioprocFloatImpl()`，将ap_builder、argc和argv传进去，这个函数在audio_processing_impl.cc中实现，首先解析参数并检查参数个数，然后调`CreateSettings()`搞默认设置，由于audioproc_f的命令行形式调用的时候没有`aec_dump`，所以这里接下来也不使用这种形式，创建`std::unique_ptr<AudioProcessingSimulator> processor`时，有两种，一种是`AecDumpBasedSimulator`，另一种是`WavBasedSimulator`，两个类都派生自同一个基类`AudioProcessingSimulator`，同样，这里没有aec_dump，所以通过`WavBasedSimultor`进行创建，类中除了构造函数，就只有一个公有方法`Process()`，用于处理输入的wav，私有方法`Initialize()`，`HandleProcessStreamCall()`，处理输入流，`HandleProcessReverseStreamCall()`，处理反向流，`PrepareProcessStreamCall()`，准备处理输入流，`PrepareReverseProcessStreamCall()`，准备处理反向流？这个不确定，`GetDefaultEventChain()`和`GetCustomEventChain(const std::string& filename)`不清楚用于做什么

#### `WavBasedSimluator::Process()`

根据`settings`调用，Custom的先不管，`WavBasedSimulator::GetDefaultEventChain()`默认事件链好像是指定`call_chain[0]`为输入流，`call_chain[1]`为反向流，然后调用test/audio_processing_simulator.cc中的`AudioProcessingSimulator::CreateAudioProcessor()`，根据参数去实例化一些模块，**发现了`EchoCanceller3Factory`，这个可能我们要用**，前面是new了一个，后面通过`AudioProcessingBuilder::SetEchoControlFactory()`将这个factory设置进去了

代码看累了，于是尝试实践了一下，用AU去混音作为测试样本进行测试
- 双讲
  首先将两个阅读样本带有一个时间差混音到一块作为cap.wav，然后将被调慢的一个作为play.wav，进行消除测试，play.wav的声音几乎完全被消除了，但同时cap.wav的音质也受到了一定影响，这应该就是双讲上的难题
- 单讲
  单讲更简单，直接将一个样本时间后移即可作为cap.wav，未后移的版本作为play.wav，分别测试了legacy_aec，aec和aecm，aecm效果最差，残留很多，legacy_aec是有一定效果的，但去除不干净，aec在短暂的时间之后（可能是收敛时间，可以考虑用正弦波去测试），后面几乎完全被消除。

如果操作没有错误的地方，那么可以看出aec的效果还是很不错的。

**`AudioProcessingSimulator::CreateAudioProcessor()`**，这个是`WavBasedProcessor`的基类成员函数，首先根据`Settings`的内容给`config`设置新的内容（`new`了一些对象出来），根据`Settings`的参数，确定要是用什么，比如使用什么aec这种，同时检测一些参数是否正确，还有一些比如设置自适应滤波器这种，还有我们关注的`EchoCanceller3Factory`，再回头看一下，这里有两个`config`，一个是`webrtc::Config config`，另一个是`AudioProcessing::Config apm_config`，`config`主要用于保存一些对象，比如`new`出来的`ExperimentNs`这种，好像是一个比较通用的配置，而`apm_config`是提供给我们的在`AudioProcessing`这个类里面的配置，好像声明了一些特性的启动或关闭

前面就是根据`settings`（是命令行解析出来的参数）来初始化`config`和`apm_config`，然后使用`AudioProcessingBuilder::Create()`，根据`config`来创建`AudioProcessing`对象，首先实例化的是`AudioProcessingImpl`，就是实现了`AudioProcessing`虚函数的派生类，再使用`AudioProcessing::ApplyConfig()`来应用`apm_config`的配置，最后好像是根据启用的功能进行了一些检查，到这，`CreateAudioProcesser()`结束

**`WavBasedSimulator::Initialize()`**，创建`WavReader`和`ChannelBufferReader`，估计他俩就是用来读wav文件及其中各声道的数据的，输入文件的采样率及通道数是从wav文件中直接读出来的，而输出：
```cpp
  int output_sample_rate_hz = settings_.output_sample_rate_hz
                                  ? *settings_.output_sample_rate_hz
                                  : input_sample_rate_hz;
  int output_num_channels = settings_.output_num_channels
                                ? *settings_.output_num_channels
                                : input_num_channels;
```
这里跟我前面想的一样，其实是先判断参数是否存在，即是否从命令行传入了，然后再决定是用其中的值（\*）还是用输入的值，反向流的参数先初始化，然后再创建WavReader读反向流文件，一样要从wav中读取其采样率通道数等参数，再决定反向流输出的参数（但是还不知道这个-ro是干嘛用的，这个反向流不就是个参考吗？），然后调用`AudioProcessingSimulator::SetupBuffersConfigsOutputs()`，给每个输入输出（-i/-o/-ri/-ro）创建`StreamConfig`，给`AudioProcessingSimultor`中的各个buffer检查参数并创建`ChannelBuffer`，然后打印调试信息

最后还要调用`AudioProcessingSimulator::SetupOutput()`，处理一些关于输出的配置一个是-o的，一个是-ro的

主要就是读取wav信息，处理输入文件信息和输出文件信息，创建`ChannelBuffer`，猜测是用于存储读取文件进来的数据，到这，`Initialize()`结束，回到`WavBasedSimulator::Process()`中来

*回到`WavBasedSimulator::Process()`*

是一个循环，根据默认的`call_chain`来调用，默认的`call_chain`为0-Stream，1-ReverseStream，先看一下不同`call_chain`时两个处理函数做了啥

**WavBasedSimulator::HandleProcessStreamCall()**
读文件，会返回是否还有剩下没处理的，然后`WavBasedSimulator::PrepareProcessStreamCall()`，然后调`WavBasedSimulator::ProcessStream()`

对于`WavBasedSimulator::PrepareProcessStreamCall()`，首先`webrtc::test::CopyToAudioFrame()`，将`ChannelBuffer`中的样copy到`AudioFrame`中，如果配置了`stream_delay`，则调用`AudioProcessing`的`set_stream_delay_ms()`

对于`AudioProcessingSimulator::ProcessStream(bool fixed_interface)`，先是利用fake device来仿真模拟增益，这里用的时候可能需要将设置模拟增益也就是3104的接口给弄进来，然后就是处理流了，调用的是`AudioProcessing`的`ProceStream()`，疑问是为什么可以单独处理输入流？不是应该跟反向输入流一块处理输入流吗？

处理完之后回到循环，退出循环，然后`call_chain_index`递增余2（默认的`call_chain`就2个），也就是`call_chain_index`就是0和1循环，也就是处理一个正向`frame`，再处理反向frame，反向流的处理应该是差不多的

**`WavBasedSimulator::HandleProcessReverseStreamCall()`**

与上面一样，只不过处理的是反向流

对于`WavBasedSimulator::PrepareReverseProcessStreamCall()`，不需要进行stream_delay的设置，其他都一样，

`AudioProcessingSimulator::ProcessReverseStream(bool fixed_interface)`，与正向流差不多

处理完之后调用`void AudioProcessingSimulator::DestroyAudioProcessor()`，将`aec_dump detach`掉，`WavBasedSimulator::Process()`就结束了，

回到`AudioprocFloatImpl`，打印状态信息等，最后`return 0`整个就结束了

对比前面的例子，基本上都是对得上的，不一样的地方在于前面有个apm->Initialize()，这个是在进行新的处理的时候调用的，并且刚new出来的apm是不需要的，因此就变成了结束的时候调用（我们应该都用不到这个接口）

*这里的多态，AudioProcessingSimulator这个基类有两个子类，AecDumpBasedSimulator和WavBasedSimulator，运行时根据我们给的参数决定我们的processor最终指向的是哪个类，然后再调用processor->Prcesse()时，实际上就是基类对象执行派生类的方法*

---

### 接下来结合audio_processing.h中的接口和demo的顺序再捋一捋

首先解析命令行输入，根据解析结果进行`CreateSettings()`，得到`SimulationSettings settings`，弄好的`settings`用来创建`AudioProcessingSimulator`的派生类`AecDumpBasedSimulator`和`WavBasedSimulator`的对象，作为`processor`，同时通过基类构造函数实例化了`ap_builder_`

在`processor->Process()`中，调`AudioProcessingSimulator::CreateAudioProcessor()`构建实际的或者说具体的`AudioProcessor`，根据`settings`中的内容，构造`Config config`，`AudioProcessing::Config apm_config`和`std::unique_ptr<EchoControlFactory> echo_control_factory`，设置`ap_builder_->SetEchoControlFactory(echo_control_factory)`，通过`(*ap_builder).Create(config)`创建`AudioProcessingImpl`类对象并给到`AudioProcessing`类指针，调`ap_->ApplyConfig(apm_config)`应用`apm_config`，再根据`settings`中的一些参数，调一些`AudioProcessing`中的小接口

继续在`processor->Process()`中，调`WavBasedSimulator::Initialize()`，利用`WavReader`和`ChannelBufferWavReader`读取wav信息，然后调`AudioProcessingSimulator::SetupBuffersConfigsOutputs()`设置每个输入和输出的`webrtc::StreamConfig`，调`AudioProcessingSimulator::SetupOutput()`处理一些输出文件信息

还是在`processor->Process()`中，循环处理-i和-ri，最后结束

---

分析一下供用户侧使用的参数，用户的命令行输入都会体现在SimulationSettings settings，只找了一部分，剩下的有的没找到，有的是仅供simulator用的，慢慢深入吧
```cpp
struct SimulationSettings {
  SimulationSettings();  // 构造函数
  SimulationSettings(const SimulationSettings&);  // 构造函数
  ~SimulationSettings();  // 析构函数
  absl::optional<int> stream_delay; // AudioProcessing::set_stream_delay_ms(int delay)
  absl::optional<bool> use_stream_delay;
  absl::optional<int> stream_drift_samples;
  absl::optional<int> output_sample_rate_hz;  // 应该输出ne的采样率
  absl::optional<int> output_num_channels;  // 应该是输出ne的通道数（注意限制）
  absl::optional<int> reverse_output_sample_rate_hz;  // 输出fe的采样率（输出ne到底是干嘛用的）
  absl::optional<int> reverse_output_num_channels;  // 输出ne的通道数
  absl::optional<std::string> output_filename;  // 输出ne文件名，就是处理后的那个
  absl::optional<std::string> reverse_output_filename;  // 输出fe文件名，不知用来做什么
  absl::optional<std::string> input_filename;  // 输入ne文件名
  absl::optional<std::string> reverse_input_filename;  // 输入fe文件名
  absl::optional<std::string> artificial_nearend_filename;  // ???????????????
  absl::optional<bool> use_aec;  // AudioProcessing::Config::EchoCanceller::enabled aec3
  absl::optional<bool> use_aecm;  // AudioProcessing::Config::EchoCanceller::mobile_mode aecm
  absl::optional<bool> use_ed;  // AudioProcessing::Config::ResidualEchoDetector::enabled Residual Echo Detector.
  absl::optional<std::string> ed_graph_output_filename;  //输出一个xxx.py，运行得到echo likelihood
  absl::optional<bool> use_agc;  // AudioProcessing::Config::GainController1::enabled agc1
  absl::optional<bool> use_agc2;  // AudioProcessing::Config::GainController2::enabled agc2
  absl::optional<bool> use_pre_amplifier;  // AudioProcessing::Config::PreAmplifier::enabled 在其他处理前放大ne信号
  absl::optional<bool> use_hpf;  // AudioProcessing::Config::HighPassFilter::enabled 高通滤波器，不知道用在哪里做什么
  absl::optional<bool> use_ns;  // AudioProcessing::Config::NoiseSuppression::enabled ns
  absl::optional<bool> use_ts;  // ExperimentalNs::enabled, AudioProcessing::set_stream_key_pressed transient suppressor 好像用于决定测试性噪声抑制
  absl::optional<bool> use_ie;
  absl::optional<bool> use_vad;  // VoiceDetection::enabled 声音活动检测器
  absl::optional<bool> use_le;  // LevelEstimator::enabled
  absl::optional<bool> use_all;  // 
  absl::optional<int> aec_suppression_level;  // AudioProcessing::Config::EchoCanceller::legacy_moderate_suppression_levelaec压制等级，legacy，将移除
  absl::optional<bool> use_delay_agnostic;  // DelayAgnostic::enabled 未知延迟，将依赖内部估计延时
  absl::optional<bool> use_extended_filter;  // ExtendedFilter::enabled 不知做什么用的
  absl::optional<bool> use_drift_compensation;  // deprecated setting
  absl::optional<bool> use_legacy_aec;  // AudioProcessing::Config::EchoCanceller::use_legacy_aec 使用过时aec
  absl::optional<bool> use_experimental_agc;  // ExperimentalAgc::enabled 使用测试agc（agc3？
  absl::optional<bool> use_experimental_agc_agc2_level_estimator;  // ExperimentalAgc::enabled_agc2_level_estimator 电平检测？
  absl::optional<bool> experimental_agc_disable_digital_adaptive;  // ExperimentalAgc::digital_adaptive_disabled
  absl::optional<bool> experimental_agc_analyze_before_aec;  // ExperimentalAgc::analyze_before_aec
  absl::optional<int> agc_mode;  // AudioProcessing::Config::GainController1::mode
  absl::optional<int> agc_target_level;  // AudioProcessing::Config::GainController1::target_level_dbfs
  absl::optional<bool> use_agc_limiter;  // AudioProcessing::Config::GainController1::enable_limiter
  absl::optional<int> agc_compression_gain;  // AudioProcessing::Config::GainController1::compression_gain_db
  absl::optional<bool> agc2_use_adaptive_gain;  // AudioProcessing::Config::GainController2::adaptive_digital::enabled
  absl::optional<float> agc2_fixed_gain_db;  // AudioProcessing::Config::GainController2::fixed_digital::gain_db
  AudioProcessing::Config::GainController2::LevelEstimator
      agc2_adaptive_level_estimator;
  absl::optional<float> pre_amplifier_gain_factor;  // AudioProcessing::Config::PreAmplifier::enabled
  absl::optional<int> vad_likelihood;  // VoiceDetection::set_likelihood(VoiceDetection::Likelihood likelihood)
  absl::optional<int> ns_level;  // NoiseSuppression::set_level(NoiseSuppression::Level level)
  absl::optional<int> maximum_internal_processing_rate;  // AudioProcessing::Config::Pipeline::maximum_internal_processing_rate
  absl::optional<bool> use_refined_adaptive_filter;  //RefinedAdaptiveFilter::enabled
  int initial_mic_level;
  bool simulate_mic_gain = false;
  absl::optional<bool> experimental_multi_channel;  // AudioProcessing::Config::Pipeline::experimental_multi_channel
  absl::optional<int> simulated_mic_kind;
  bool report_performance = false;
  absl::optional<std::string> performance_report_output_filename;
  bool report_bitexactness = false;
  bool use_verbose_logging = false;
  bool use_quiet_output = false;  // demo用
  bool discard_all_settings_in_aecdump = true;
  absl::optional<std::string> aec_dump_input_filename;  //aec_dump输入，知道如何得到aec_dump，不知是做什么的
  absl::optional<std::string> aec_dump_output_filename;  //aec_dump输出，不知是做什么的
  bool fixed_interface = false;
  bool store_intermediate_output = false;
  bool print_aec_parameter_values = false;
  bool dump_internal_data = false;
  absl::optional<std::string> dump_internal_data_output_dir;
  absl::optional<std::string> call_order_input_filename;
  absl::optional<std::string> call_order_output_filename;
  absl::optional<std::string> aec_settings_filename;
  absl::optional<absl::string_view> aec_dump_input_string;
  std::vector<float>* processed_capture_samples = nullptr;
};
```

---

没办法了，稍微分析一下echo_remover.cc中的`void EchoRemoverImpl::ProcessCapture()`，以确定dump出来的都是哪些数据。`const std::vector<std::vector<std::vector<float>>>& x`是个三维向量的引用，指向render的一个Block，`std::vector<std::vector<std::vector<float>>>* y`是个三维向量的指针，指向capture。各个向量的尺寸
```
(echo_remover.cc:262): x.size() = 1, x[0].size() = 1, x[0][0].size() = 64
(echo_remover.cc:265): y->size() = 1, (*y)[0].size() = 1, (*y)[0][0].size() = 64
```
1s数据，1个通道，这个64不太清楚该怎么理解
申请e_stack，2通道，每个通道存长度64的fft;
申请Y2_stack、E2_stack、R2_stack、S2_linear_stack，均为2通道，长度65的fft；（这里具体存什么还不确定）
申请Y_stack、E_stack、comfort_noise_stack、high_band_comfort_noise_stack，2通道的FftData；
申请subtractor_output_stack，2通道SubtractorOutput，应该是减法器输出数据
利用上述申请的所有东西，申请名为rtc::ArrayView的数据结构，分别命名为e、Y2、E2、R2、S2_linear、Y、E、comfort_noise、high_band_comfort_noise、subtractor_output
接下来判断，如果输入的通道数大于2（好像还支持6通道），则继续申请一些空间，由于我们最多也就是双通道，所以这里不用
x0为render数据的引用，y0为capture数据的引用
检测增益、回声路径变化，先不管了
通过RenderSignalAnalyzer分析render信号，识别窄带和识别强窄带，不知道是干嘛的
对减法器调用ExitInitialState()，切换状态，似乎是完成了减法器的初始化状态
通过减法器进行线性回声消除，参数为render数据，capture各通道数据（(*y)[0]），render信号分析结果，aec状态（这个前面也有，可能是一个整体的处理状态），然后是前面初始化分配过空间的subtractorOutput（应该是用于存放减法器进行线性回声消除的输出结果），减法器处理中，输出文件aec3_subtractor_G_main.raw，是有一定规律的数据，是将G的实部虚部写进数据中的，似乎是增益，main滤波器的增益？？？，（可能就是main滤波器的频谱吧），后面还有输出的aec3_subtractor_G_shadow.raw，跟前面一样，那可能就是shadow滤波器的频谱吧，然后滤波器的输出就是e_main和e_shadow，分别dump到aec3_main_filter_output.wav和aec3_shadow_filter_output.wav中
y数据对应的fft为Y，e对应的fft为E，Y-E（频域实部虚部对应相减）为线性回声强度S2_linear，Y的Spectrum为Y2，E的Spectrum为E2
根据aec_state_.UseLinearFilterOutput()选择是否使用线性滤波器的输出，这里Y是y的fft，是capture，E是e的fft，应该是想，线性滤波器输出的（可能经过了选择），将y0和e（即Y和E的时域信号）dump出来为aec3_output_linear.wav和aec3_output_linear2.wav，这块分析根据~L410选择是否使用线性滤波器输出，是则选择Y_fft = E，不是则选择Y_fft = Y，Y应该是未经改动的输入，看打印是1
继续处理，估计估计残余回声能量，到R2中
估计舒适噪声，到high_band_comfort_noise
抑制器回声估计，根据aec_state_.UsableLinearEstimate()，选择echo_spectrum = S2_linear
抑制器近端估计，使用std::transform，根据aec_state_.UsableLinearEstimate()，取E2与Y2的极小值，到nearend_spectrum_bounded，选择nearend_spectrum = nearend_spectrum_bounded
计算首选增益，最小增益决定最终增益（不知道这部分会不会有什么影响）
然后调ApplyGain()对y进行处理，处理前y就是capture，处理后就是输出的样子，我的想法是，实际上这里将所有的因素都处理为一个Gain（滤波器？），然后Apply到源信号上，完成对信号的处理


|dump_data|含义|来源|
|---|---|---|
|aec3_echo_remover_capture_input.wav|未经回声处理的输入capture数据（但可能有其他预处理）|echo_remover.cc/~L330|
|aec3_echo_remover_render_input.wav|未经回声处理的输入render数据（但可能有其他预处理）|echo_remover.cc/~L330|
|aec3_echo_remover_capture_input.dat|未经回声处理的输入capture数据（可能有其他内容）|echo_remover.cc/~L330|
|aec3_echo_remover_render_input.dat|未经回声处理的输入render数据（可能有其他内容）|echo_remover.cc/~L330|
|aec3_subtractor_G_main.dat|猜测是main滤波器的频谱（re和im分量都有）|subtractor.cc/~L245|
|aec3_subtractor_G_shadow.dat|猜测是shadow滤波器的频谱（re和im分量都有）|subtractor.cc/~L267|
|aec3_main_filter_output.wav|main滤波器的输出|subtractor.cc/~L277|
|aec3_shadow_filter_output.wav|shadow滤波器的输出|subtractor.cc/~L279|
|aec3_output_linear.wav|应该是未经线性滤波的输入的capture信号|echo_remover.cc/~L410|
|aec3_output_linear2.wav|应该是经过线性滤波器并form（可能是main和shadow筛选）过的|echo_remover.cc/~L410|

简要记录各个dat文件的获取和大概形状
css_speech_3：
- 18999KB = 19454720B = 303980ms * 64B = 303980ms * 16 * 1S16，即位深16，16kHz，
- 1782KB = 1823880B = 303980ms * 6B，aec3_all_erle_3-0.dat，aec3_erle_correction_factor_3-0.dat，aec3_ref_erle_3-0.dat，4字节，全是3f080000
- 1188KB = 1215920B = 4B * 303980ms，aec3_er_acum_numerator_3-0.dat，全是0
- 594KB = 607960 = 8B * 75995block，
  - aec3_latency_blocks_3-0.dat，4字节，数值只有0123，看变化规律对应运行打印的实际延时，一般5块一个循环，20ms，应该影响不大啊
  - aec3_min_latency_blocks_3-0.dat，8字节为单位（即block），前面一点点0，后面全1，
  - aec3_processblock_call_order_3-0.dat，基本为01交替，为capture和render处理的顺序，render为1，capture为0（看代码是以int32_t写的，但是看数据应该是16位啊，这个比较奇怪），
  - aec3_render_delay_controller_buffer_delay_3-0.dat，8字节为单位，为每个block处理时使用的delay，
  - aec3_render_delay_controller_delay_3-0.dat，也是8字节的block为单位，处理结果不明白是什么，但波形上可以与处理后的波形有一定对应关系，没处理的部分为0较多
- 297KB = 303980 = 4B * 75995block，
  - aec3_capture_saturation_3-0.dat，全0，
  - aec3_echo_path_delay_estimator_best_index_3-0.dat，4字节的block为单位，也是与输出波形有一定关系，但不知道含义，选择最大的lag作为最好的，
  - aec3_echo_path_delay_estimator_delay_3-0.dat，4字节的block，与前述类似，
  - aec3_erl_time_domain_3-0.dat，4字节，与输出波形关系不明显，这个erl可能是echo return loss，
  - aec3_erle_instantaneous_quality_3-0.dat，4字节，基本上处理了的就是很大的值，没处理的就是0（瞬时质量？），
  - aec3_external_delay_avaliable_3-0.dat，4字节，这个可能是说明什么时候用的外部delay，什么时候用的是内部delay，
  - aec3_filter_delay_3-0.dat，4字节，波形看不出与输出波形有什么关系，
  - aec3_fullband_erle_inst_log2_3-0.dat，4字节，也是，处理了值就很大，没处理就是0，
  - aec3_fullband_erle_log2_3-0.dat，4字节，同上，不知道具体含义，
  - aec3_fullband_erle_max_log2_3-0.dat和aec3_fullband_erle_min_log2_3-0.dat，4字节，与上面有一些不一样，与输出波形有一定关联，估计参考意义不大，
  - aec3_inv_misadjustment_factor_3-0.dat什么失调因数，与输出波形有关联，
  - aec3_late_reverb_end_3-0.dat，aec3_late_reverb_end_3-0.dat，全0，
  - aec3_narrow_render_3-0.dat，4字节，01交替，
  - aec3_num_reverb_decay_blocks_3-0.dat，
  aec3_reverb_alpha_3-0.dat，全0，aec3_reverb_decay_3-0.dat，全程同一个数值3f547ae1，aec3_reverb_tail_energy_3-0.dat，全0，
  - aec3_using_subtractor_output[0]_3-0.dat，前面一小部分0后面全1，是标识是否使用了减法器？
- 238KB = 234184B，aec3_call_order_3-0.dat，似乎是ProcessCapture()和AnalyzeRender()的调用顺序，是完全规则的10交替
- 149KB = 151990B = 2B * 75995block，
  - aec3_consistent_filter_3-0.dat，全0，
  - aec3_converged_filter_3-0.dat，收敛滤波器，处理了的地方01交替，未处理的地方全0，有关联，
![aec3_converged_filter_3-0.png](E:\work_dir\PROJECTS\AECM\work\css_speech_3\pic\aec3_converged_filter_3-0.png)
  - aec3_diverged_filter_3-0.dat，有关系，但关系不大，
  - aec3_dominant_nearend_3-0.dat，关系强烈，可能是用于判定此时是近端信号还是回声信号，双讲模式会怎么样呢？
  - aec3_echo_saturation_3-0.dat，aec3_echo_saturation_3-0.dat，全0，aec3_initial_state_3-0.dat，两组话音后面有1，其他是0，关系较小，aec3_transparent_mode_3-0.dat，全0，aec3_usable_linear_estimate_3-0.dat，类似前面的，两组话音后为1
- 119KB = 121592B = ，stream_delay_1-0.dat，全0，experimental_gain_control_compression_gain_db_0-3.dat，experimental_gain_control_set_stream_analog_level_1-3.dat，experimental_gain_control_stream_analog_level_1-3.dat，4字节，有关系，但关系不明显，从notepad++明明是2字节的，但是就是需要按4字节读，这样的话，前面的是不是也有不对的？

除了10交替的应该都是可以取出波形的，所以看一下那几个中哪个不正确

测试，外部给延时，如果延时给得比较准确，效果是可以的

首先测试了调整过延时的，这里测试了172、175，效果都比较不错，

然后测试了未经过延时调整的，使用延时572ms，效果与对经过延时调整的信号使用172ms的效果基本一样

当use_external_delay_estimator = true，其延时自动调整幅度很小，看打印，其实际应用的延时为143~145block，

这里测试了一个大致范围，在520ms到580ms的范围内，都有相对较好的处理效果，暂时不更加详细去测了

另关于css信号逐渐增大的问题，看了输出的波形，并没有在css范围内发生变化的中间输出，会不会是滤波器的特性之类的？或者相应的中间输出没有拿出来？

测试，哪些在all_default中有较大作用：

1. 使用all_default：
```sh
./audioproc_f_internal -i d2pc_-450.wav -ri pc2d.wav -o o_def_-450.wav -all_default

```

2. 不使用all_default：
```sh
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_def_-400.wav -le 1 -vad 1 -ts 1 -ns 1 -hpf 1 -agc 1 -agc2 0 -pre_amplifier 0 -aec 1 -aecm 0 -ed 0
```
上述两条可以得到一样的效果

3. 只开启aec：
```sh
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_onlyaec_-400.wav -aec 1
```
相比前2个效果差一些

4. 增加所有为0的选项，排除关闭选项带来的影响
```sh
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_aecandall0_-400.wav -agc2 0 -pre_amplifier 0 -aec 1 -aecm 0 -ed 0
```
确定与仅开启aec的效果一样

5. 上述2可以简化为
```shell
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_only1_-400.wav -le 1 -vad 1 -ts 1 -ns 1 -hpf 1 -agc 1 -aec 1
```
验证与1，2一样

6. 依次关闭，看哪个影响最大，命令：
```bash
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_le0_-400.wav -le 0 -vad 1 -ts 1 -ns 1 -hpf 1 -agc 1 -aec 1
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_vad0_-400.wav -le 1 -vad 0 -ts 1 -ns 1 -hpf 1 -agc 1 -aec 1
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_ts0_-400.wav -le 1 -vad 1 -ts 0 -ns 1 -hpf 1 -agc 1 -aec 1
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_ns0_-400.wav -le 1 -vad 1 -ts 1 -ns 0 -hpf 1 -agc 1 -aec 1
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_hpf0_-400.wav -le 1 -vad 1 -ts 1 -ns 1 -hpf 0 -agc 1 -aec 1
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_agc0_-400.wav -le 1 -vad 1 -ts 1 -ns 1 -hpf 1 -agc 0 -aec 1
```
测试结果：hpf，vad和le关闭时对输出波形没有影响（但是之前hpf是有影响的）

7. 验证关闭vad和le是否确实不影响：
```sh
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_le0vad0_-400.wav -ts 1 -ns 1 -agc 1 -aec 1
```
验证与all_default输出相同

8. 验证ns、ts、agc单独开启时的效果
```sh
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_ts_-400.wav -ts 1 -aec 1
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_ns_-400.wav -ns 1 -aec 1
./audioproc_f_internal -i d2pc_-400.wav -ri pc2d.wav -o o_manual_agc_-400.wav -agc 1 -aec 1
```
测试结果：ts影响很小，ns影响最大，单独启用agc看起来似乎起了相反的效果，但是三个一起启用的时候达到的效果确实是最好的，其次是单独启用ns时的效果

# 回声消除小总结

回声消除这块，一般都是自适应滤波，开源的有speex和webrtc两个，各有优缺点。

speex是一个音频格式，也有一些音频处理功能，后来被分开了，分到了speexDsp库中，现在已经都不更新了。

webrtc是一整套东西，很多，Google的，源码就很多，目前依旧在更新。

先说speex，它也有agc和ns，但是我只用了aec，speex的aec最大的优点是双讲效果良好，消除效果不够好，但也基本够用，不具备非线性处理算法，由采集设备等引起的非线性失真是没有办法解决的，没有延时处理，要么你自己保证你的ne和fe两个流延时在正确的范围内，即fe比ne先，同时时间差较小，基本上要低于filter length，我是自己想办法实现的手动控制，但是同样有问题，主要问题是ne端与fe端延时无法保持固定导致的，ne端信号时始终有的，fe端却不一定，所以fe来的时候ne的ring中有多少buffer就不一定了，我尝试过此时清空ne侧的buffer，但仍旧存在一些问题。

这里只提它的audio_processing模块，包含的功能也很多，agc、aec、ns等等webrtc的模块是一整个整体，应用的话基本上也会是一整个应用进去，webrtc具备一个延时估计模块，用于估计ne和fe的延时，效果还算比较不错，确实可以考虑提取出来用，webrtc有三个回声消除，一个是用于低计算能力设备的aecm，即aec的移动版本，消除效果稍差，一个是aec2，已经过时的aec，不建议再使用，一个是aec3，目前仍在优化中，效果比较好，webrtc单讲回声消除的效果非常好，消除很彻底，但问题是双讲吃音很严重，基本就是无法听得清到底是在说什么，同时aec3的运算资源占用也很大，双讲吃音问题源于其算法，自适应滤波器进行回声消除，需要在消除过程中进行滤波器系数的更新，而双讲情况出现时，不应该按照双讲的波形进行更新，如果此时仍旧继续更新，就会出现话音信号被消除的问题出现，这需要一个dtd（double-talk-detector）双讲检测器来处理，但webrtc并没有，所以双讲吃音问题无法解决。webrtc的agc也可以说说，它是通过调整设备的采集音量来实现的增益控制，即自动控制采集增益，避免出现数字增益带来的数字噪声，但是暂时应该不用动它了。