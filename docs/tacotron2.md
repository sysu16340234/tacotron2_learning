# tacotron2
## 1.模型结构
![tacotron2](https://github.com/sysu16340234/tacotron2_learning/blob/master/tacotron2.png?raw=true)

论文中的模型有两个主要的组件,第一部分是一个具有attention的seq2seq循环预测网络,通过输入的字符序列预测Mel-Frequency帧;第二部分是一个WaveNet的修改版本,用来从预测的Mel-Frequency帧中生成时域波形样本;
### 1.1 Mel-Frequency Spectrum特征表示
模型的两个组件由[Mel-Frequency Spectrum](https://zh.wikipedia.org/wiki/%E6%A2%85%E7%88%BE%E5%80%92%E9%A0%BB%E8%AD%9C)连接,它可以由时域信号简单地计算得到,便于将两部分模型分开训练,它比波形表示更加平滑,更容易使用平方误差损失训练.Mel-Frequency频谱图与线性频谱图即[STFT](https://en.wikipedia.org/wiki/Short-time_Fourier_transform)幅度相关,它是通过对STFT的频轴进行非线性变换得到的.可以用更少的维度来代表频率内容;

### 1.2频谱图预测网络
使用125 Hz到7.6 kHz的80通道梅尔滤波器组将STFT幅度转换为梅尔尺度，然后进行对数动态范围压缩.在对数压缩之前，滤波器组输出幅度被限幅到最小值0.01，以便限制对数域中的动态范围.

网络由编码器和解码器组成,采用attention机制,编码器将字符序列转换为隐藏的特征表示，解码器消耗该特征表示以预测频谱图.

编码器:

![encoder](https://github.com/sysu16340234/tacotron2_learning/blob/master/encoder.png?raw=true)

编码器输入字符使用经过学习的512维character embedding来表示，它们通过3个卷积层的堆栈，每个卷积层包含512个5×1的过滤器，即每个过滤器跨度为5个字符，然后是批量标准化和ReLU激活。最终卷积层的输出被传递到包含512个单元(每个方向256个)的单个双向[LSTM](https://sysu16340234.github.io/tacotron2_learning/LSTM)层,以生成编码特征.

编码器输出由attention网络消费，attention网络将完整编码序列概括为每个解码器输出步骤的固定长度上下文矢量.本文使用的attention模型是location-sensitive attention,其扩展了附加注意机制,使用来自先前解码器时间步长的累积attention权重作为附加特征.其中一些子序列被解码器重复或忽略.在将输入和位置特征投影到128维隐藏表示之后计算attention概率.使用32个长度为31的1-D卷积滤波器计算位置特征.

解码器:

![decoder](https://github.com/sysu16340234/tacotron2_learning/blob/master/decoder.png?raw=true)
