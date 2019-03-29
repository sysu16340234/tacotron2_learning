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

解码器是一个自回归递归神经网络,一次一帧地从编码输入序列预测mel频谱图.先前时间的预测值通过两个包含256个隐藏ReLU单元的全连接层的pre-net,pre-net的输出和上下文向量连接后进入具有1024个单元的2个单向LSTM层的堆栈,LSTM的输出和上下文向量连接后进行线性变换来预测频谱图帧.与这一部分预测同时进行的是,将LSTM的输出和上下文向量连接后投影到标量并经过sigmoid函数激活来预测已经完成的概率,在这个预测值大于0.5时,我们认为预测已经完成.最后,经过预测的mel频谱图经过一个五层的post-net卷积网络,这个网络用来预测残差并把它加到预测值上来改善整体重建效果.每一层由512个5x1的过滤器组成,经过批量标准化并且对除最后一层外的所有层采用tanh函数激活.

网络中的卷积层采用0.5的dropout进行正则化,LSTM层采用0.1的zoneout进行正则化,并以0.5的dropout应用于解码器的pre-net.

### 1.3 WaveNet
采用WaveNet的修改版本将mel频谱图特征表示反转为时域波形样本,与原始架构一样,采用30个扩展卷积层,分组为3个扩张周期.与原始架构不同的是,为了使用12.5ms的帧跳,将训练堆栈中的三层下采样层降为两层;不使用softmax层预测离散桶,而采用PixelCNN ++和Parallel WaveNet,并使用10分量混合逻辑分布（MoL）来生成24 kHz的16位样本;WaveNet堆栈输出通过ReLU激活，然后是线性投影，以预测每种组分的参数,损失计算使用实际样本的负对数似然.
