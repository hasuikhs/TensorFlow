## Python_9_4(TensorFlow)

- #### CNN

```python
# convolution example
# cnn 은 이미지 파일을 여러개로 분할하여 부분부분을 뽑아 정해진 칸수 만큼 움직이면서 패턴을 학습
import tensorflow as tf
import numpy as np

# image 형태
# 1장의 이미지는 3차원형태의 데이터
# (이미지의 개수, width, height, color)
# (1, 3, 3, 1)
image = np.array([[[[1], [2], [3]],
                  [[4], [5], [6]],
                  [[7], [8], [9]]]], dtype = np.float32)
print(image.shape)
# 필터를 준비해야 해요
# (width, height, color, 필터의 개수)
# (2, 2, 1, 1)
weight = np.array([[[[1, -5, 10]], [[1, -5, 10]]],
                  [[[1, -5, 10]], [[1, -5, 10]]]])

print(weight.shape)
# strider 지정(사실 2차원이면 충분하지만 행렬 연산때문에)
# (1, stride width, stride height, 1) 4차원으로 표현
# stride = [1, 1, 1, 1]
conv2d = tf.nn.conv2d(image, weight, strides = [1, 1, 1, 1], padding = "VALID")
print(conv2d.shape)
```

```python
(1, 3, 3, 1)
(2, 2, 1, 3)
(1, 2, 2, 3)
```

------

```python
## MNIST 예제를 이용해서 하나의 이미지에 대한
## convolutional image 5개를 생성

from tensorflow.examples.tutorials.mnist import input_data
import tensorflow as tf
import matplotlib.pyplot as plt

# Data Loading
mnist = input_data.read_data_sets("./data/mnist", one_hot=True)

# training 이미지 중 2번째 이미지의 정보를 얻어옴
img = mnist.train.images[1]   # 1차원 데이터를
img = img.reshape(28, 28)     # 2차원 데이터로 변환
plt.imshow(img, cmap = "Greys", interpolation="nearest")
plt.show()
# 해당 이미지를 convolution 이미지로 변형
# 2차원 형태의 img를 4차원 형태의 img로 변환
img = img.reshape(-1, 28, 28, 1)

# 이미지가 준비되었으니 필터를 준비
# 5개의 필터를 이용, 2x2짜리 필터를 이용
# (2, 2, 1, 5)
W = tf.Variable(tf.random_normal([2, 2, 1, 5]), name = "filter")
conv2d = tf.nn.conv2d(img, W, strides = [1, 2, 2, 1], padding = "SAME")
print(conv2d.shape)

# (1, 14, 14, 5) => 14x14짜리 이미지가 5개 생성
# 새로 생성된 이미지를 plt를 이용해서 확인
sess = tf.Session()
sess.run(tf.global_variables_initializer())
conv2d = sess.run(conv2d)

# 배열의 축을 임의로 변경
# (1, 14, 14, 5) => (5, 14, 14, 1)
conv2d = np.swapaxes(conv2d, 0, 3)
print(conv2d.shape)
fig, axes = plt.subplots(1, 5 ) # 1행 5열짜리 subplot을 생성
                                # axes는 subplot의 배열

for idx, item in enumerate(conv2d):
    axes[idx].imshow(item.reshape(14, 14), cmap = "Greys")
plt.show()
```

```python
(1, 14, 14, 5)
(5, 14, 14, 1)

# 하나의 이미지를 
# 14x14의 이미지로 5개로 분할하여 학습하기위해 준비
```

------

```python
#### MNIST with CNN
from tensorflow.examples.tutorials.mnist import input_data
import tensorflow as tf

# 0. 그래프 초기화
tf.reset_default_graph()

# 1. Data Loading & Data 정제
mnist = input_data.read_data_sets("./data/mnist", one_hot = True)

# 2. placeholder 설정
X = tf.placeholder(shape = [None, 784], dtype = tf.float32)
Y = tf.placeholder(shape = [None, 10], dtype = tf.float32)
drop_rate = tf.placeholder(dtype = tf.float32)

# 3. Convolution
# 3.1 Convolution layer 1
x_img = tf.reshape(X, [-1, 28, 28, 1])
W1 = tf.Variable(tf.random_normal([2, 2, 1, 32]), name = "filter1")
L1 = tf.nn.conv2d(x_img, W1, strides = [1, 2, 2, 1], padding = "SAME") # padding 이미지 유지 여부
print(L1.shape)

L1 = tf.nn.relu(L1)
L1 = tf.nn.max_pool(L1, ksize = [1, 2, 2, 1], strides = [1, 2, 2, 1], padding = "SAME")
print(L1.shape) 

# 3.2 Convolution layer 2
W2 = tf.Variable(tf.random_normal([3, 3, 32, 64]), name = "filter2") # 3번째 차원의 숫자 조심 W1의 32
L2 = tf.nn.conv2d(L1, W2, strides = [1, 1, 1, 1], padding = "SAME") # padding 이미지 유지 여부
print(L2.shape)

L2 = tf.nn.relu(L2)
L2 = tf.nn.max_pool(L2, ksize = [1, 1, 1, 1], strides = [1, 1, 1, 1], padding = "SAME")
print(L2.shape)

L2 = tf.reshape(L2, [-1, 7*7*64])

## 4. Neural Network
## 4.1 Weight & bias
# W3 =  tf.get_variable("weight3", shape=["컬럼수", "결과수"])
W3 =  tf.get_variable("weight3", shape=[7*7*64, "256"], initializer=tf.contrib.layers.xavier_initializer())

#################
b3 = tf.Variable(tf.random_normal([256]), name = "bias3")

_layer3 = tf.nn.relu(tf.matmul(L2, W3) + b3)
layer3 = tf.nn.dropout(_layer3,keep_prob = drop_rate)

W4 = tf.get_variable("d4",shape=[256,256], initializer=tf.contrib.layers.xavier_initializer())
b4 = tf.Variable(tf.random_normal([256]), name = "bias4")

_layer4 = tf.nn.relu(tf.matmul(layer3,W4) + b4)
layer4 = tf.nn.dropout(_layer4,keep_prob = drop_rate)

W5 = tf.get_variable("e5",shape=[256,10], initializer=tf.contrib.layers.xavier_initializer())
b5 = tf.Variable(tf.random_normal([10]), name = "bias5")
# Hypothesis
logits = tf.matmul(layer4,W5) + b5
H = tf.nn.relu(logits)
# # 나온 logits의 각각의 확률을 구함

# # cost function
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits_v2(logits = logits, labels = Y))

# # train node 생성
optimizer = tf.train.AdamOptimizer(learning_rate = 0.001)
train = optimizer.minimize(cost)

# # session & 초기화
sess = tf.Session()
sess.run(tf.global_variables_initializer())

# # 사용하는 데이터의 크기가 상당히 커요
# # 데이터의 크기에 상관없이 학습하는 방식이 필요
# # epoch : traing data를 1번 학습시키는것

# # 학습진행
training_epoch = 6
batch_size = 128 # 55000개의 행을 다 읽어들이는게 아니라
                 # 100개의 행을 읽어서 학습

for step in range(training_epoch):
    num_of_iter = mnist.train.num_examples // batch_size
    cost_val = 0
    for i in range(num_of_iter):
        batch_x, batch_y = mnist.train.next_batch(batch_size)
        _, cost_val = sess.run([train, cost], feed_dict = {X : batch_x, Y : batch_y, drop_rate : 0.7})
    if step % 1 == 0:
        print(cost_val)
# Accuracy 측정
predict = tf.argmax(H, 1)
correct = tf.equal(predict, tf.argmax(Y, 1))
accuracy = tf.reduce_mean(tf.cast(correct, dtype = tf.float32))

result = sess.run(accuracy, feed_dict = {X : mnist.test.images, Y : mnist.test.labels, drop_rate : 1.0})
print("정확도 : {}".format(result))
```

```python
# 결과값

0.37440896
0.22493961
0.36278492
0.14796194
0.21794069
0.14423889
정확도 : 0.9757999777793884
```

------

- CNN 자세히 분할

```python
# 필요한 module import
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data
```

```python
## 1. Data Loading
mnist = input_data.read_data_sets("./data/mnist", one_hot = True)
```

```python
## 2. Model 정의 (Tensorlow graph 생성)
tf.reset_default_graph() # tensorflow graph 초기화

## 2.1 placeholder
X = tf.placeholder(shape = (None, 784), dtype = tf.float32)
Y = tf.placeholder(shape = (None, 10), dtype = tf.float32)

drop_rate = tf.placeholder(dtype = tf.float32)

# 2.2 Convolution
# CNN은 이미지 학습에 최적화된 deep learning방법
# 입력받은 이미지의 형태가 4차원 배열
# (이미지의 개수, 이미지의 width, 이미지의 height, 이미지의 color)
X_img = tf.reshape(X, [-1, 28, 28, 1])

## 2.3 Convolution Layer1
# 1번째 방법
#####################################################################################################
## filter 정의 => shape (width, height, color, filter 수)
# filter1 = tf.Variable(tf.random_normal([3, 3, 1, 32]))

# ## filter를 이용해서 Convolution image를 생성
# L1 = tf.nn.conv2d(X_img, filter1, strides = [1, 1, 1, 1], padding = "SAME") # strides : filter의 움직임

# ## 만들어진 Convolution에 relu를 적용
# L1 = tf.nn.relu(L1)

# ## pooling 작업(resize, sampling 작업) => optional
# L1 = tf.nn.max_pool(L1, ksize = [1, 2, 2, 1], strides = [1, 2, 2, 1], padding = "SAME")
#####################################################################################################

# 2번째 방법 ( 1번째 방법을 한줄에 처리)
#####################################################################################################

L1 = tf.layers.conv2d(inputs = X_img, filters = 32, kernel_size = [3, 3], 
                      padding = "SAME", strides = 1, activation = tf.nn.relu)
L1 = tf.layers.max_pooling2d(inputs = L1, pool_size = [2, 2], padding = "SAME", strides = 2)

print("L1 shaep : {}".format(L1.shape))
# L1 shaep : (?, 14, 14, 32)

## Convolution Layer 2
L2 = tf.layers.conv2d(inputs = L1, filters = 64, kernel_size = [3, 3], 
                      padding = "SAME", strides = 1, activation = tf.nn.relu)
L2 = tf.layers.max_pooling2d(inputs = L2, pool_size = [2, 2], padding = "SAME", strides = 2)

print("L2 shape : {}".format(L2.shape))
# L2 shape : (?, 7, 7, 64)

# 2.4 Neural Network
## Convolution의 결과를 Neural Network의 입력으로 사용하기 위해 shape를 변경
L2 = tf.reshape(L2, [-1, 7 * 7 * 64])

W1 = tf.get_variable("weight1", shape=[7 * 7 * 64, 256], initializer = tf.contrib.layers.xavier_initializer())
b1 = tf.Variable(tf.random_normal([256]), name = "bias1")
_layer1 = tf.nn.relu(tf.matmul(L2, W1) + b1)
# overfitting 방지 dropout
layer1 = tf.nn.dropout(_layer1, keep_prob = drop_rate)

W2 = tf.get_variable("weight2", shape=[256, 10], initializer = tf.contrib.layers.xavier_initializer())
b2 = tf.Variable(tf.random_normal([10]), name = "bias2")

# Hypothesis
logits = tf.matmul(layer1, W2) + b2
H = tf.nn.relu(logits)

# cost function
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits_v2(logits = logits, labels = Y))

## train
optimizer = tf.train.AdamOptimizer(learning_rate = 0.001)
train = optimizer.minimize(cost)

# session, 초기화
sess = tf.Session()
sess.run(tf.global_variables_initializer())

# 학습 진행(batch 처리)
training_epoch = 6
batch_size = 128 

for step in range(training_epoch):
    num_of_iter = int(mnist.train.num_examples / batch_size)
    cost_val = 0
    for i in range(num_of_iter):
        batch_x, batch_y = mnist.train.next_batch(batch_size)
        _, cost_val = sess.run([train, cost], feed_dict = {X : batch_x, Y : batch_y, drop_rate : 0.7})
    if step % 1 == 0:
        print(cost_val)

# Accuracy 측정
predict = tf.argmax(H, 1)
correct = tf.equal(predict, tf.argmax(Y, 1))
accuracy = tf.reduce_mean(tf.cast(correct, dtype = tf.float32))

result = sess.run(accuracy, feed_dict = {X : mnist.test.images, Y : mnist.test.labels, drop_rate : 1.0})
print("정확도 : {}".format(result))
```

- 앙상블

  ```python
  ### 결국 우리 MNIST 예제는 입력한 이미지 1개에 대해 예측한 결과가 H의 값으로 도출
  ### [0.5, 0.8, 0.99, 0.12, 0.34, ...] 총 10개
  
  ### 앙상블은 이런 model이 여러개 존재
  ### H1 => [0.5, 0.8, 0.99, 0.12, 0.34, ...]
  ### H2 => [0.2, 0.3, 0.94, 0.5, 0.1, ...]
  ### H3 => [0.7, 0.1, 0.3, 0.2, 0.12, ...]
  ### H4 => [0.26, 0.23, 0.194, 0.54, 0.31, ...]
  
  ### SUM => [1.66, 1.43, 2.4, 1.3, 1.2, ...]
  ### 최종 prediction은 SUM한 결과값을 가지고 예측
  ```
  ```python
  ## 앙상블을 사용하기 위해서는 클래스를 이용해 각각의 개체를 만들어 생성해야 함
  ## Class
  ## 1. 객체 모델링의 수단
  ## 2. ADT(abstract data type) - 추상적 데이터 타입

  ## class안에 정의되는 내용은 크게 3가지
  ## 상태값(field, member variable, property)
  ## 수행하는 작업을 위한 함수(method, member function, method function)
  ## 정의된 class정보를 바탕으로 일정 메모리를 확보
  ## => 이런 확보된 메모리 공간 -> 인스턴스, 객체
  ## 이런 객체를 생성하기 위해 생성자가 호출되어야 함

  ## 학생이 3명 존재
  ## 각 학생이 가지는 정보는 국어, 영어, 수학 점수
  ## 평균 ,총점 결과를 알고 싶다면

  class Student:
    
    # constructor
    def __init__(self, s_name, s_kor, s_eng, s_math):
        # property를 생성하려면 self를 이용
        self.student_name = s_name
        self.kor = s_kor
        self.eng = s_eng
        self.math = s_math
        
    # method
    def get_total(self):
        self.total = self.kor + self.eng + self.math
        return self.total

  stu1 = Student("홍길동", 10 , 20, 30)
  print(stu1.get_total())  
  ```
  ```python
  import tensorflow as tf
  import pandas as pd
  import datetime
  import numpy as np
  import math
  from sklearn.preprocessing import MinMaxScaler

  ## 1. Data Loading
  df_tr = pd.read_csv("./data/digit/train.csv", sep = ",")
  df  _tr_x = df_tr.drop("label", axis = 1, inplace = False)
  df_tr_y = pd.DataFrame(data={"label" : df_tr["label"]})

  sess = tf.Session()

  x_data = df_tr_x
  y_data = pd.DataFrame(sess.run(tf.one_hot(df_tr["label"], 10)))

  nx_data = MinMaxScaler().fit_transform(x_data.values)

  x_tr_data = nx_data[:29400]
  x_test_data = nx_data[29400:]

  y_tr_data = y_data[:29400]
  y_test_data = y_data[29400:]

  test_df = pd.read_csv("./data/digit/test.csv",sep=",")
  ```
  ```python
  tf.reset_default_graph()

  class CnnModel:

    def __init__(self, sess, name):
        self.sess = sess
        self.name = name
        self._build_net()

    def _build_net(self):
        with tf.variable_scope(self.name):
            # dropout (keep_prob) rate  0.7~0.5 on training, but should be 1
            # for testing
            self.training = tf.placeholder(tf.bool)

            # input place holders
            self.X = tf.placeholder(tf.float32, [None, 784])

            # img 28x28x1 (black/white), Input Layer
            X_img = tf.reshape(self.X, [-1, 28, 28, 1])
            self.Y = tf.placeholder(tf.float32, [None, 10])

            # Convolutional Layer #1
            conv1 = tf.layers.conv2d(inputs=X_img, filters=32, kernel_size=[3, 3], padding="SAME", activation=tf.nn.relu)
            # Pooling Layer #1
            pool1 = tf.layers.max_pooling2d(inputs=conv1, pool_size=[2, 2], padding="SAME", strides=2)
            dropout1 = tf.layers.dropout(inputs=pool1, rate=0.3, training=self.training)

            # Convolutional Layer #2 and Pooling Layer #2
            conv2 = tf.layers.conv2d(inputs=dropout1, filters=64, kernel_size=[3, 3], padding="SAME", activation=tf.nn.relu)
            pool2 = tf.layers.max_pooling2d(inputs=conv2, pool_size=[2, 2], padding="SAME", strides=2)
            dropout2 = tf.layers.dropout(inputs=pool2, rate=0.3, training=self.training)

            # Convolutional Layer #3 and Pooling Layer #3
            conv3 = tf.layers.conv2d(inputs=dropout2, filters=128, kernel_size=[3, 3], padding="SAME", activation=tf.nn.relu)
            pool3 = tf.layers.max_pooling2d(inputs=conv3, pool_size=[2, 2], padding="SAME", strides=2)
            dropout3 = tf.layers.dropout(inputs=pool3, rate=0.3, training=self.training)

            # Dense Layer with Relu
            flat = tf.reshape(dropout3, [-1, 128 * 4 * 4])
            dense4 = tf.layers.dense(inputs=flat, units=625, activation=tf.nn.relu)
            dropout4 = tf.layers.dropout(inputs=dense4, rate=0.5, training=self.training)

            # Logits (no activation) Layer: L5 Final FC 625 inputs -> 10 outputs
            self.logits = tf.layers.dense(inputs=dropout4, units=10)

        # define cost/loss & optimizer
        self.cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=self.logits, labels=self.Y))
        self.optimizer = tf.train.AdamOptimizer(learning_rate=0.001).minimize(self.cost)

        correct_prediction = tf.equal(tf.argmax(self.logits, 1), tf.argmax(self.Y, 1))
        self.accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

    def predict(self, x_test, training=False):
        return self.sess.run(self.logits, feed_dict={self.X: x_test, self.training: training})

    def get_accuracy(self, x_test, y_test, training=False):
        return self.sess.run(self.accuracy, feed_dict={self.X: x_test, self.Y: y_test, self.training: training})

    def train(self, x_data, y_data, training=True):
        return self.sess.run([self.cost, self.optimizer], feed_dict={self.X: x_data, self.Y: y_data, self.training: training})

    
  # initialize
  sess = tf.Session()

  #모델을 담아낼 리스트를 만들고 몇 개의 모델을 할지 정한다.
  models = []
  num_of_models = 5
  cnn_models = [CnnModel(sess, "Model" + str(x)) for x in range(num_of_models)]

  sess.run(tf.global_variables_initializer())

  print('Learning Started!')

  training_epochs = 5
  batch_size = 128

  # train my model
  # 에폭마다 필요한 배치 사이즈(x, y)를 불러와서 학습을 시킨다.
  print("***** leaning start ******")
  l_s_time = datetime.datetime.now()
  for epoch in range(training_epochs):
    
    avg_cost_list = np.zeros(len(cnn_models))
    num_of_iter = int(math.ceil(len(x_tr_data) / batch_size))
    start_time = datetime.datetime.now()
    for i in range(num_of_iter):
        print("epoch : {}, time : {},  {} %".format(epoch, datetime.datetime.now(), (i / num_of_iter)*100))
        batch_x = x_tr_data[i * batch_size : batch_size * (i + 1)]
        batch_y = y_tr_data[i * batch_size : batch_size * (i + 1)]
        # train each model. 기존에는 모델 하나만 학습시켰던 반면 앙상블 에서는 각각의 모델에 대해 학습을 시킨다.
        for x_idx, x in enumerate(cnn_models):
            c, _ = x.train(batch_x, batch_y)
            avg_cost_list[x_idx] += c / num_of_iter # 각각의 모델별로 cost를 구해준다.
            print("c : {}".format(c))
    end_time = datetime.datetime.now()
    print("time : {}, epoch: {}, cost : {}".format(datetime.datetime.now(), epoch, avg_cost_list))
    print("{} epoch 소요 시간 : {}, 현재까지 소요시간 : {}".format(epoch, end_time - start_time, end_time - l_s_time))
  print("****** learning end ******")
  l_e_time = datetime.datetime.now()
  print("##### 최종 소요 시간 : {} #####".format(l_e_time - l_s_time))
  # Test model and check accuracy
  test_size = len(x_test_data)
  predictions = np.zeros([test_size, 10])
  for x_idx, x in enumerate(cnn_models):
    print(x_idx, 'Accuracy:', x.get_accuracy(x_test_data, y_test_data))
    p = x.predict(x_test_data) # 각각의 모델에 대해서 예측
    predictions += p # 예측 한 것들의 합을 구한다.

  ensemble_correct_prediction = tf.equal(tf.argmax(predictions, 1), tf.argmax(y_test_data, 1)) 
  # predict_correct = tf.equal(tf.argmax(H, 1), tf.argmax(Y, 1))

  ensemble_accuracy = tf.reduce_mean(tf.cast(ensemble_correct_prediction, tf.float32))
  # accuracy = tf.reduce_mean(tf.cast(predict_correct, dtype = tf.float32))

  print('Ensemble accuracy:', sess.run(ensemble_accuracy))

  # print("정확도 : {}".format(sess.run(accuracy, feed_dict = {X : x_test_data, Y : y_test_data, drop_rate : 1.0}))
  ```
  
  

  
