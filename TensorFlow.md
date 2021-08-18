# TensorFlow简介

## 什么是神经网络

**神经网络**是一种数学模型，是存在于计算机的神经系统，由大量的神经元相连接并进行计算，在外界信息的基础上，改变内部的结构，常用来对输入和输出间复杂的关系进行建模。

神经网络由大量的节点和之间的联系构成，负责传递信息和加工信息，神经元也可以通过训练而被强化。

这个图就是一个神经网络系统，它由很多层构成。输入层就是负责接收信息，比如说一只猫的图片。输出层就是计算机对这个输入信息的认知，它是不是猫。隐藏层就是对输入信息的加工处理。



**神经网络是如何被训练的**，首先它需要很多数据。比如他要判断一张图片是不是猫。就要输入上千万张的带有标签的猫猫狗狗的图片，然后再训练上千万次。

神经网络训练的结果有对的也有错的，如果是错误的结果，将被当做非常宝贵的经验，那么是**如何从经验中学习的呢？**就是对比正确答案和错误答案之间的区别，然后把这个区别反向的传递回去，对每个相应的神经元进行一点点的改变。那么下一次在训练的时候就可以用已经改进一点点的神经元去得到稍微准确一点的结果。

**神经网络是如何训练的呢？**每个神经元都有属于它的激活函数，用这些函数给计算机一个刺激行为。



## 神经网络 梯度下降

梯度的本意是一个向量（矢量），表示某一函数在该点处的方向导数沿着该方向取得最大值，即函数在该点处沿着该方向（此梯度的方向）变化最快，变化率最大（为该梯度的模）。

### Optimization 

[![神经网络 梯度下降](https://morvanzhou.github.io/static/results/ML-intro/gd2.png)](https://morvanzhou.github.io/static/results/ML-intro/gd2.png)

学习机器学习的同学们常会遇到这样的图像, 我了个天, 看上去好复杂, 哈哈, 不过还挺好看的. 这些和我们说的梯度下降又有什么关系呢? 原来这些图片展示出来了一个家族的历史, 这个家族的名字就是-”optimization” (优化问题). 优化能力是人类历史上的重大突破, 他解决了很多实际生活中的问题. 从而渐渐演化成了一个庞大的家族.

比如说牛顿法 (Newton’s method), 最小二乘法(Least Squares method), 梯度下降法 (Gradient Descent) 等等. 而我们的神经网络就是属于梯度下降法这个分支中的一个. 提到梯度下降, 我们不得不说说大学里面学习过的求导求微分. 因为这就是传说中”梯度下降”里面的”梯度” (gradient)啦. 听到求导微分可别后怕, 因为这个短视频只是让你有一个直观上的理解, 并不会涉及太过复杂的东西.

### 梯度下降 

[![神经网络 梯度下降](https://morvanzhou.github.io/static/results/ML-intro/gd3.png)](https://morvanzhou.github.io/static/results/ML-intro/gd3.png)

初学神经网络的时候, 我们通常会遇到这样一个方程, 叫做误差方程 (Cost Function). 用来计算预测出来的和我们实际中的值有多大差别. 在预测数值的问题中, 我们常用平方差 (Mean Squared Error) 来代替. 我们简化一下这个方程, W是我们神经网络中的参数, x, y 都是我们的数据, 因为 xy 都是实实在在的数据点, 在这个假设情况中, 是多少都无所谓, 然后我们再厚颜无耻地像这样继续简化一下, (注意, 这个过程在在数学中并不正确, 不过我们只是为了看效果), 所以现在误差值曲线就成了这样. 假设我们初始化的 W 在这个位置. 而这个位置的斜率是这条线, 这也就是梯度下降中的梯度啦. 我们从图中可以看出, Cost 误差最小的时候正是这条 cost 曲线最低的地方, 不过在蓝点的 W 却不知道这件事情, 他目前所知道的就是梯度线为自己在这个位置指出的一个下降方向, 我们就要朝着这个蓝色梯度的方向下降一点点. 在做一条切线, 发现我还能下降, 那我就朝着梯度的方向继续下降, 这时, 再展示出现在的梯度, 因为梯度线已经躺平了, 我们已经指不出哪边是下降的方向了, 所以这时我们就找到了 W 参数的最理想值. 简而言之, 就是找到梯度线躺平的点. 可是神经网络的梯度下降可没这么简单.

[![神经网络 梯度下降](https://morvanzhou.github.io/static/results/ML-intro/gd4.png)](https://morvanzhou.github.io/static/results/ML-intro/gd4.png)

神经网络中的 W 可不止一个, 如果只有一个 W, 我们就能画出之前那样的误差曲线, 如果有两个 W 也简单, 我们可以用一个3D 图来展示, 可是超过3个 W, 我们可就没办法很好的可视化出来啦. 这可不是最要命的. 在通常的神经网络中, 误差曲线可没这么优雅.

### 全局 and 局部最优 

[![神经网络 梯度下降](https://morvanzhou.github.io/static/results/ML-intro/gd5.png)](https://morvanzhou.github.io/static/results/ML-intro/gd5.png)

在简化版的误差曲线中, 我们只要找到梯度线躺平的地方, 就能能迅速找到误差最小时的 W. 可是很多情况是这样的, 误差曲线并不只有一个沟, 而且梯度躺平的点也不止一个. 不同的 W 初始化的位置, 将会带来不同的下降区域. 不同的下降区域, 又会带来不同的 W 解. 在这个图像当中, W 的全局最优解(Global minima)在这个位置, 而其它的 解都是局部最优(Local minima). 全局最优固然是最好, 但是很多时候, 你手中的都是一个局部最优解, 这也是无可避免的. 不过你可以不必担心, 因为虽然不是全局最优, 但是神经网络也能让你的局部最优足够优秀, 以至于即使拿着一个局部最优也能出色的完成手中的任务.

## 神经网络在干嘛？

### 拟合曲线 

机器学习 其实就是让电脑不断的尝试模拟已知的数据. 他能知道自己拟合的数据离真实的数据差距有多远, 然后不断地改进自己拟合的参数,提高拟合的相似度.

本例中蓝色离散点是我们的数据点, 红线是通过神经网络算法拟合出来的曲线,

[![神经网络在干嘛](https://morvanzhou.github.io/static/results/tensorflow/1_3_1.png)](https://morvanzhou.github.io/static/results/tensorflow/1_3_1.png)

它是对我们数据点的一个近似表达. 可以看出, 在开始阶段, 红线的表达能力不强, 误差很大. 不过通过不断的学习, 预测误差将会被降低. 所以学习到后来. 红线也能近似表达出数据的样子.

[![神经网络在干嘛](https://morvanzhou.github.io/static/results/tensorflow/1_3_2.png)](https://morvanzhou.github.io/static/results/tensorflow/1_3_2.png)

### 拟合参数 

如果红色曲线的表达式为：`y = a*x + b` 其中`x`代表`inputs`, `y`代表`outputs`, `a`和`b`是神经网络训练的参数. 模型训练好了以后,`a`和`b`的值将会被确定, 比如 `a=0.5`, `b=2`,当我们再输入`x=3`时, 我们的模型就会输出 `0.5*3 + 2` 的结果. 模型通过学习数据, 得到能表达数据的参数, 然后对我们另外给的数据所作出预测.















# Tensorflow 基础框架

## 处理结构

计算图纸 

Tensorflow 首先要定义神经网络的结构, 然后再把数据放入结构当中去运算和 training.

[![处理结构](https://www.tensorflow.org/images/tensors_flowing.gif)](https://www.tensorflow.org/images/tensors_flowing.gif)

因为TensorFlow是采用数据流图（data　flow　graphs）来计算, 所以首先我们得创建一个数据流流图, 然后再将我们的数据（数据以张量(tensor)的形式存在）放在数据流图中计算. 节点（Nodes）在图中表示数学操作,图中的线（edges）则表示在节点间相互联系的多维数据数组, 即张量（tensor). 训练模型时tensor会不断的从数据流图中的一个节点flow到另一节点, 这就是TensorFlow名字的由来.

## 例子1

tensorflow version 1代码如下

```python
import tensorflow as tf
import numpy as np

# create data
x_data = np.random.rand(100).astype(np.float32)
y_data = x_data*0.1 + 0.3

#搭建模型
Weights = tf.Variable(tf.random_uniform([1], -1.0, 1.0))
biases = tf.Variable(tf.zeros([1]))
y = Weights*x_data + biases

#计算误差
loss = tf.reduce_mean(tf.square(y-y_data))

#传播误差
#反向传递误差的工作就教给optimizer了, 我们使用的误差传递方法是梯度下降法: Gradient Descent 让后我们使用 optimizer 来进行参数的更新.
optimizer = tf.train.GradientDescentOptimizer(0.5)
train = optimizer.minimize(loss)


#训练
#到目前为止, 我们只是建立了神经网络的结构, 还没有使用这个结构. 在使用这个结构之前, 我们必须先初始化所有之前定义的Variable, 所以这一步是很重要的!
# init = tf.initialize_all_variables() # tf 马上就要废弃这种写法
init = tf.global_variables_initializer()  # 替换成这样就好

#接着,我们再创建会话 Session. 我们用 Session 来执行 init 初始化步骤. 并且, 用 Session 来 run 每一次 training 的数据. 逐步提升神经网络的预测准确性.
sess = tf.Session()
sess.run(init)          # Very important

for step in range(201):
    sess.run(train)
    if step % 20 == 0:
        print(step, sess.run(Weights), sess.run(biases))
```

tensorflow version 2代码如下

```python
import tensorflow as tf
import numpy as np

#creat data
x_data = np.random.rand(100).astype(np.float32)
y_data = x_data*01.+0.3

### creat tensorflow structure start ###
Weights = tf.Variable(tf.random.uniform((1,),-1.0,1.0))
biases = tf.Variable(tf.zeros([1]))

y = Weights*x_data+biases

loss= lambda: tf.keras.losses.MSE(y_data,Weights * x_data +biases)
optimizer = tf.keras.optimizers.SGD(learning_rate=0.5)
### creat tensorflow structure end ###


for step in range(201):
    optimizer.minimize(loss, var_list=[Weights, biases])

    if step %20==0:
        #print(step,sess.run(Weights),sess.run(biases))
        print("{} step, weights = {}, biases = {}".format(step, Weights.read_value(), biases.read_value()))  # read_value函数可用numpy替换
```



## Session 会话控制

这一次我们会讲到 Tensorflow 中的 `Session`, `Session` 是 Tensorflow 为了控制,和输出文件的执行的语句. 运行 `session.run()` 可以获得你要得知的运算结果, 或者是你所要运算的部分.

首先，我们这次需要加载 Tensorflow ，然后建立两个 `matrix` ,输出两个 `matrix` 矩阵相乘的结果。

```python
import tensorflow as tf

# create two matrixes

matrix1 = tf.constant([[3,3]])
matrix2 = tf.constant([[2],
                       [2]])
product = tf.matmul(matrix1,matrix2)
```

因为 `product` 不是直接计算的步骤, 所以我们会要使用 `Session` 来激活 `product` 并得到计算结果. 有两种形式使用会话控制 `Session` 。

```python
# method 1
sess = tf.Session()
result = sess.run(product)
print(result)
sess.close()
# [[12]]

# method 2
with tf.Session() as sess:
    result2 = sess.run(product)
    print(result2)
# [[12]]
```

tensorflow version 2已经不需要用session控制了

```python
import tensorflow as tf

matrix1=tf.constant([[3,3]])
matrix2=tf.constant([[2],[2]])
product=tf.matmul(matrix1,matrix2)
tf.print(product)
```



## Variable 变量

在 Tensorflow 中，定义了某字符串是变量，它才是变量，这一点是与 Python 所不同的。

定义语法： `state = tf.Variable()`

```python
import tensorflow as tf

state = tf.Variable(0, name='counter')

# 定义常量 one
one = tf.constant(1)

# 定义加法步骤 (注: 此步并没有直接计算)
new_value = tf.add(state, one)

# 将 State 更新成 new_value
update = tf.assign(state, new_value)
```

如果你在 Tensorflow 中设定了变量，那么初始化变量是最重要的！！所以定义了变量以后, 一定要定义 `init = tf.initialize_all_variables()` .

到这里变量还是没有被激活，需要再在 `sess` 里, `sess.run(init)` , 激活 `init` 这一步.

```python
# 如果定义 Variable, 就一定要 initialize
init = tf.global_variables_initializer()  # 替换成这样就好
 
# 使用 Session
with tf.Session() as sess:
    sess.run(init)
    for _ in range(3):
        sess.run(update)
        print(sess.run(state))
```

注意：直接 `print(state)` 不起作用！！

一定要把 `sess` 的指针指向 `state` 再进行 `print` 才能得到想要的结果！

附上v2代码

```python
import tensorflow as tf

state=tf.Variable(0,name='counter')
one=tf.constant(1)
new_value=tf.add(state,one)

for _ in range(3):
	#The variable value can be changed using one of the assign methods.
	state.assign_add(new_value)
	tf.print(state,new_value)
```



## Placeholder 传入值

placeholder 是 Tensorflow 中的占位符，暂时储存变量.

Tensorflow 如果想要从外部传入data, 那就需要用到 `tf.placeholder()`, 然后以这种形式传输数据 `sess.run(***, feed_dict={input: **})`.

示例：

```python
import tensorflow as tf

#在 Tensorflow 中需要定义 placeholder 的 type ，一般为 float32 形式
input1 = tf.placeholder(tf.float32)
input2 = tf.placeholder(tf.float32)

# mul = multiply 是将input1和input2 做乘法运算，并输出为 output 
ouput = tf.multiply(input1, input2)
```



接下来, 传值的工作交给了 `sess.run()` , 需要传入的值放在了`feed_dict={}` 并一一对应每一个 `input`. `placeholder` 与 `feed_dict={}` 是绑定在一起出现的。



```python
with tf.Session() as sess:
    print(sess.run(ouput, feed_dict={input1: [7.], input2: [2.]}))
# [ 14.]
```



v2貌似已经换掉了placeholder ，看到我再添加。



###  Activation Function激励函数

这里的 AF 就是指的激励函数. 激励函数拿出自己最擅长的”掰弯利器”, 套在了原函数上 用力一扭, 原来的 Wx 结果就被扭弯了.

其实这个 AF, 掰弯利器, 也不是什么触不可及的东西. 它其实就是另外一个非线性函数. 比如说relu, sigmoid, tanh. 将这些掰弯利器嵌套在原有的结果之上, 强行把原有的线性结果给扭曲了. 使得输出结果 y 也有了非线性的特征. 举个例子, 比如我使用了 relu 这个掰弯利器, 如果此时 Wx 的结果是1, y 还将是1, 不过 Wx 为-1的时候, y 不再是-1, 而会是0.

你甚至可以创造自己的激励函数来处理自己的问题, 不过要确保的是这些激励函数必须是可以微分的, 因为在 backpropagation 误差反向传递的时候, 只有这些可微分的激励函数才能把误差传递回去.

想要恰当使用这些激励函数, 还是有窍门的. 比如当你的神经网络层只有两三层, 不是很多的时候, 对于隐藏层, 使用任意的激励函数, 随便掰弯是可以的, 不会有特别大的影响. 不过, 当你使用特别多层的神经网络, 在掰弯的时候, 玩玩不得随意选择利器. 因为这会涉及到梯度爆炸, 梯度消失的问题. 因为时间的关系, 我们可能会在以后来具体谈谈这个问题.

最后我们说说, 在具体的例子中, 我们默认首选的激励函数是哪些. 在少量层结构中, 我们可以尝试很多种不同的激励函数. 在卷积神经网络 Convolutional neural networks 的卷积层中, 推荐的激励函数是 relu. 在循环神经网络中 recurrent neural networks, 推荐的是 tanh 或者是 relu (这个具体怎么选, 我会在以后 循环神经网络的介绍中在详细讲解).



## 创建第一个神经网络

### 例子3 添加层 def add_layer()



# 计算机视觉介绍

## Anaconda使用

```python
%config IPCompleter.greedy=True  #按tag可以自动补全代码
```

## 加载Fashion MNIST数据集

70000张图片
10个类别
28*28像素
训练神经元网络
![image-20201209220451516](img/TensorFlow/image-20201209220451516.png)

```python
import tensorflow as tf
from tensorflow import keras
import matplotlib.pyplot as plt

#导入数据集
fasion_mnist = keras.datasets.fashion_mnist
#划分数据集
(train_images,train_labels),(test_images,test_labels) = fasion_mnist.load_data()

#看数据集大小
print(train_images.shape)

#展示图片
plt.imshow(train_images[1])

```

![image-20201209220813905](img/TensorFlow/image-20201209220813905.png)

## 构建神经元网络模型

![image-20201209220900053](img/TensorFlow/image-20201209220900053.png)

![image-20201209220929783](img/TensorFlow/image-20201209220929783.png)

全连接的网络结构

```python
model = keras.Sequential()
model.add(keras.layers.Flatten(input_shape=(28,28)))
model.add(keras.layers.Dense(128,activation=tf.nn.relu))
model.add(keras.layers.Dense(10,activation=tf.nn.softmax))

model.summary()

#以下为console
Model: "sequential_1"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
flatten_1 (Flatten)          (None, 784)  个像素             0         
_________________________________________________________________
dense (Dense)                (None, 128)               100480    （784个像素+1权重）*128个神经元
_________________________________________________________________
dense_1 (Dense)              (None, 10)                1290      （128个神经元+1权重）*10个类别
=================================================================
Total params: 101,770
Trainable params: 101,770
Non-trainable params: 0
_________________________________________________________________
```

## 训练和评估模型



```python
#scaling or normalization 把灰度变成只有1和0
train_imges_scaled=train_images/255

#metrics=['accuracy'] 是为了看到指标accuracy的变化，
model.compile(optimizer=tf.optimizers.Adam(),loss=tf.losses.sparse_categorical_crossentropy,metrics=['accuracy'])
#训练，epochs重复次数5次
model.fit(train_imges_scaled,train_labels,epochs=5)

#模型评估
test_images_scaled = test_images/255
model.evaluate(test_images_scaled,test_labels)


import numpy as np

print(np.argmax( model.predict([[test_images[0]/255]])))
print(test_labels[0])
plt.imshow(test_images[0])
```



## 自动终止训练

![image-20201210194942305](img/TensorFlow/image-20201210194942305.png)

```python
#重载一个callback方法
class myCallback(tf.keras.callbacks.Callback):
    def on_epoch_end(self,epoch,logs={}):
        if(logs.get('loss')<0.4):
            print("\n Loss is low so cancelling training!")
            self.model.stop_training = True
            
callbacks= myCallback()
mnist =tf.keras.datasets.fashion_mnist
(training_images,training_labels),(test_images,test_labels) = mnist.load_data()
training_images_scaled = training_images/255
test_images_scaled = test_images/255

model =tf.keras.models.Sequential([
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(512,activation = tf.nn.relu),
    tf.keras.layers.Dense(10,activation = tf.nn.softmax)
])
model.compile(optimizer='adam',loss='sparse_categorical_crossentropy',metrics=['accuracy'])
model.fit(training_images_scaled,training_labels,epochs=5,callbacks=[callbacks])

---console:---
Epoch 1/5
1875/1875 [==============================] - 5s 3ms/step - loss: 0.4734 - accuracy: 0.8292
Epoch 2/5
1867/1875 [============================>.] - ETA: 0s - loss: 0.3585 - accuracy: 0.8697
 Loss is low so cancelling training!
1875/1875 [==============================] - 5s 3ms/step - loss: 0.3583 - accuracy: 0.8697
```

# 卷积介绍

## 卷积神经网络

卷积神经网络(Convolutional Neural Network) CNN
大部分的图片并不是正正方方摆好的，会有些偏移，这时候全连接的计算机视觉就不太好使了。
他是识别物品的特征，来判断物品的；

![image-20201210200313433](img/TensorFlow/image-20201210200313433.png)



从图像当中取到的像素，通过过滤器，与过滤器（Filter）相乘，最后相加，得到一个新的像素集；

不同的filter会有不同的效果

![image-20201210200607255](img/TensorFlow/image-20201210200607255.png)

![image-20201210200657614](img/TensorFlow/image-20201210200657614.png)

每次卷积完了之后还要再做一个Max Pooling，他的作用是增强图像的特征，如下是取最大值，最后就剩下一个2x2的矩阵

![image-20201210200734124](img/TensorFlow/image-20201210200734124.png)

![image-20201210200907168](img/TensorFlow/image-20201210200907168.png)

Max Pooling以后数据减少了，但是特征增强了。卷积的材料可以看https://bit.ly/2UGa7uH

## 卷积神经程序

```python
import tensorflow as tf
from tensorflow import keras

fasion_mnist = keras.datasets.fashion_mnist
(train_images,train_labels),(test_images,test_labels) = fasion_mnist.load_data()

model = keras.Sequential()
#Conv2D二维的卷积层，64个过滤器，每个过滤器3X3的像素，
model.add(keras.layers.Conv2D(64,(3,3),activation='relu',input_shape=(28,28,1)))
#Maxpooling 2x2，四个像素，
model.add(keras.layers.MaxPooling2D(2,2))
model.add(keras.layers.Conv2D(64,(3,3),activation='relu'))
model.add(keras.layers.MaxPooling2D(2,2))

model.add(keras.layers.Flatten())
model.add(keras.layers.Dense(128,activation=tf.nn.relu))
model.add(keras.layers.Dense(10,activation=tf.nn.softmax))

train_imges_scaled=train_images/255
model.compile(optimizer=tf.optimizers.Adam(),loss=tf.losses.sparse_categorical_crossentropy,metrics=['accuracy'])
#reshape ，-1是不管无视行数
model.fit(train_imges_scaled.reshape(-1,28,28,1),train_labels,epochs=5)
```

## 卷积网络结构

```python
model.summary()

---console---
Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
conv2d (Conv2D)              (None, 26, 26, 64)        640  
//输入是28x28，但是filter去掉了两个像素，64个filter
//640 = (3*3+1)*64  
_________________________________________________________________
max_pooling2d (MaxPooling2D) (None, 13, 13, 64)        0    //尺寸缩小一半。没有调整参数所以Param=0
_________________________________________________________________
conv2d_1 (Conv2D)            (None, 11, 11, 64)        36928  //再卷积，又去掉两个像素
//36928=(3*3*64+1)*64
_________________________________________________________________
max_pooling2d_1 (MaxPooling2 (None, 5, 5, 64)          0       //尺寸继续锁校
_________________________________________________________________
flatten (Flatten)            (None, 1600)              0       //展平数据，
_________________________________________________________________
dense (Dense)                (None, 128)               204928  //
_________________________________________________________________
dense_1 (Dense)              (None, 10)                1290      
=================================================================
Total params: 243,786
Trainable params: 243,786
Non-trainable params: 0
_________________________________________________________________
```



```python
#读取MODEL的每个层
layer_outputs = [layer.output for layer in model.layers]
#将input和output放在一起，构成一组对象
activation_model = tf.keras.models.Model(inputs=model.input,outputs=layer_outputs )
#用在一张图片上，预测结果
pred = activation_model.predict(test_images[0].reshape(1,28,28,1))
len(pred)
#输出7，有7个层的输出
pred[0].shape
#(1,26,26,64) 

pred[0][0,:,:,1]
#第0层，所有的行，所有的列，filter
```

第0层，conv2d，filter=1
![](img/TensorFlow/image-20201210214820333.png)
第1层，max_pooling2d，filter=1
![image-20201210215457034](img/TensorFlow/image-20201210215457034.png)
第2层，conv2d_1，filter =1
![image-20201210215623822](img/TensorFlow/image-20201210215623822.png)
第3层，max_pooling2d_1，filter=1
![image-20201210215648245](img/TensorFlow/image-20201210215648245.png)

第4层，flatten
![image-20201210215701092](img/TensorFlow/image-20201210215701092.png)
第5层，dense
![image-20201210215709794](img/TensorFlow/image-20201210215709794.png)
第6层，dense_1，一共6层，输出预测结果
![image-20201210215719873](img/TensorFlow/image-20201210215719873.png)

第0层，filter=50，不同filter的样子
![image-20201210215805919](img/TensorFlow/image-20201210215805919.png)

# 更复杂的图像应用

项目过程：准备训练数据，构建模型，训练模型，优化参数

真实数据的特点：
图片尺寸大小不一，需要裁剪成一样大小；
数据量比较大，不能一下转入内存；
需要经常修改参数，例如输出的尺寸，增补图像拉伸等；

```python

!wget --no-check-certificate \
    https://storage.googleapis.com/laurencemoroney-blog.appspot.com/horse-or-human.zip \
    -O /tmp/horse-or-human.zip
!wget --no-check-certificate \
    https://storage.googleapis.com/laurencemoroney-blog.appspot.com/validation-horse-or-human.zip \
    -O /tmp/validation-horse-or-human.zip
#用OS库来调用文件系统，用zipfile库来解压数据。
#Zip文件的内容会被解压到目录/tmp/horse-or-human，而该目录下又分别包含horses和humans子目录。
import os
import zipfile

local_zip = '/tmp/horse-or-human.zip'
zip_ref = zipfile.ZipFile(local_zip, 'r')
zip_ref.extractall('/tmp/horse-or-human')
zip_ref.close()

local_zip = '/tmp/validation-horse-or-human.zip'
zip_ref = zipfile.ZipFile(local_zip, 'r')
zip_ref.extractall('/tmp/validation-horse-or-human')
zip_ref.close()

#看一下训练目录中的文件名
train_horse_names = os.listdir(train_horse_dir)
print(train_horse_names[:10])
#['horse22-1.png', 'horse13-0.png', 'horse22-4.png', 'horse19-2.png', 'horse23-5.png', 'horse18-5.png', 'horse10-1.png', 'horse27-8.png', 'horse28-7.png', 'horse35-2.png']

train_human_names = os.listdir(train_human_dir)
print(train_human_names[:10])
#['human04-12.png', 'human09-19.png', 'human10-16.png', 'human13-04.png', 'human15-21.png', 'human09-29.png', 'human13-11.png', 'human17-12.png', 'human03-12.png', 'human16-15.png']

#查看总数
print('total training horse images:', len(os.listdir(train_horse_dir)))
print('total training human images:', len(os.listdir(train_human_dir)))
#total training horse images: 500
#total training human images: 527


#查看几张照片
#配置matplot
%matplotlib inline

import matplotlib.pyplot as plt
import matplotlib.image as mpimg

# 设置参数Parameters，长宽
nrows = 4
ncols = 4

# Index for iterating over images
pic_index = 0

# Set up matplotlib fig, and size it to fit 4x4 pics
fig = plt.gcf()
fig.set_size_inches(ncols * 4, nrows * 4)

pic_index += 8
next_horse_pix = [os.path.join(train_horse_dir, fname) 
                for fname in train_horse_names[pic_index-8:pic_index]]
next_human_pix = [os.path.join(train_human_dir, fname) 
                for fname in train_human_names[pic_index-8:pic_index]]

for i, img_path in enumerate(next_horse_pix+next_human_pix):
  # Set up subplot; subplot indices start at 1
  sp = plt.subplot(nrows, ncols, i + 1)
  sp.axis('Off') # Don't show axes (or gridlines)

  img = mpimg.imread(img_path)
  plt.imshow(img)

plt.show()
```

![image-20201213213800322](img/TensorFlow/image-20201213213800322.png)

## ImageDataGenerator

```python
import os
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from tensorflow.keras.optimizers import RMSprop
#定义 ImageDataGenerator
train_datagen = ImageDataGenerator(rescale=1/255)
validation_datagen = ImageDataGenerator(rescale=1/255)
train_generator = train_datagen.flow_from_directory('/tmp/horse-or-human/',  
        target_size=(150, 150),batch_size=32,class_mode='binary')
validation_generator = validation_datagen.flow_from_directory('/tmp/validation-horse-or-human/',  
        target_size=(150, 150), batch_size=32,class_mode='binary')


```



## 调参

因为我们面对的是一个二类分类的问题，所以用sigmoid激活函数作为模型的最后一层，这样我们网络的输出将是一个介于0和1的有理数，即当前图像是1类的概率。

Kerastuner就是一个可以自动搜索模型训练参数的库。它的基本思路是在需要调整参数的地方插入一个特殊的对象（可指定参数范围），然后调用类似训练那样的search方法即可。



准备训练数据

```python
import os
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from tensorflow.keras.optimizers import RMSprop
train_datagen = ImageDataGenerator(rescale=1/255)
validation_datagen = ImageDataGenerator(rescale=1/255)
train_generator = train_datagen.flow_from_directory('/tmp/horse-or-human/',  
        target_size=(150, 150),batch_size=32,class_mode='binary')
validation_generator = validation_datagen.flow_from_directory('/tmp/validation-horse-or-human/',  
        target_size=(150, 150), batch_size=32,class_mode='binary')


```

加载库，`!pip3 install -U keras-tuner`记得安装keras-tuner

```python
from kerastuner.tuners import Hyperband
from kerastuner.engine.hyperparameters import HyperParameters
import tensorflow as tf
```

接着创建HyperParameters对象，然后在模型中插入Choice、Int等调参用的对象。

```python
hp=HyperParameters()
def build_model(hp):
    model = tf.keras.models.Sequential()        
    model.add(tf.keras.layers.Conv2D(hp.Choice('num_filters_top_layer',values=[16,64],default=16), (3,3), 
                                     activation='relu', input_shape=(150, 150, 3)))
    model.add(tf.keras.layers.MaxPooling2D(2, 2))
    for i in range(hp.Int("num_conv_layers",1,3)):
        model.add(tf.keras.layers.Conv2D(hp.Choice(f'num_filters_layer{i}',values=[16,64],default=16), (3,3), activation='relu'))
        model.add(tf.keras.layers.MaxPooling2D(2,2))
    model.add(tf.keras.layers.Flatten())
    model.add(tf.keras.layers.Dense(hp.Int("hidden_units",128,512,step=32), activation='relu'))
    model.add(tf.keras.layers.Dense(1, activation='sigmoid'))
    model.compile(loss='binary_crossentropy',optimizer=RMSprop(lr=0.001),metrics=['acc'])
    return model

```

然后创建Hyperband对象，这是Kerastuner支持的四种方法的其中一种，其优点是能较四童话第查找参数。具体资料可以到Kerastuner的网站获取。

最后调用search方法。

```python
tuner=Hyperband(
    build_model,
    objective='val_acc',
    max_epochs=10,
    directory='horse_human_params',
    hyperparameters=hp,
    project_name='my_horse_human_project'
)
tuner.search(train_generator,epochs=10,validation_data=validation_generator)
```

搜索到最优参数后，可以通过下面的程序，用tuner对象提取最优参数构建神经元网络模型。并调用summary方法观察优化后的网络结构。

```python
best_hps=tuner.get_best_hyperparameters(1)[0]
print(best_hps.values)
model=tuner.hypermodel.build(best_hps)
model.summary()

```

