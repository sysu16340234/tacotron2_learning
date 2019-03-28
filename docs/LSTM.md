# LSTM
## 1.简介
LSTM是Long short-term memory(长短时间记忆网络)的缩写,是一种特殊的RNN,LSTM和传统RNN的区别如下:
![LSTM](https://pic4.zhimg.com/80/v2-e4f9851cad426dfe4ab1c76209546827_hd.jpg)

RNN只有一个隐状态h<sup>t</sup>,LSTM在RNN的基础上新增了一个隐状态c<sup>t</sup>
## 2.LSTM基本结构
### 2.1 遗忘门控
![forget](https://upload-images.jianshu.io/upload_images/42741-96b387f711d1d12c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

得到本层的输入和上一层的隐状态后,将它们拼接后乘以权重矩阵加上偏置经过sigmoid输出到0~1的区间内产生第一个门控信号f<sub>t</sub>,这个门控信号用来决定细胞需要"遗忘"信息的程度,越接近0,保留的信息越少,越接近1,保留的信息越多;

### 2.2 输入门控
![input](https://upload-images.jianshu.io/upload_images/42741-7fa07e640593f930.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

与遗忘门控类似,我们得到一个新的门控值i<sub>t</sub>,称为输入门控,同时使用tanh创建一个新的候选值向量;

### 2.3 确定cell state
![cell](https://upload-images.jianshu.io/upload_images/42741-d88caa3c4faf5353.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这一步确定要向下一个cell传递的c<sub>t</sub>值,将上一层的cell state乘上(哈达玛积)遗忘门控和本层的候选cell state与输入门控的积相加得到下一层的细胞状态;

### 2.4 更新隐含值(输出门控)
![output](https://upload-images.jianshu.io/upload_images/42741-4c9186bf786063d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们用相同的方法产生一个输出门控o<sub>t</sub>来决定将哪些信息输出,得到门控矩阵后与经过tanh层的C<sub>t</sub>值相乘,得到下一层的隐状态h<sub>t</sub>
