---
layout:     post
title:      Kaggle Mnist上手指南
subtitle:   Keras深度学习练手系列
date:       2018-11-28
author:     ZX
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - 机器学习
    - Python
---

## 目录
- 1.题目简介
- 2.运行环境说明
- 3.理论介绍
- 4.代码
- 5.代码得分
- 6.总结
## 题目简介
&emsp;&emsp; MNIST ("Modified National Institute of Standards and Technology") is the de facto “hello world” dataset of computer vision. Since its release in 1999, this classic dataset of handwritten images has served as the basis for benchmarking classification algorithms. As new machine learning techniques emerge, MNIST remains a reliable resource for researchers and learners alike.In this competition, your goal is to correctly identify digits from a dataset of tens of thousands of handwritten images. We’ve curated a set of tutorial-style kernels which cover everything from regression to neural networks. We encourage you to experiment with different algorithms to learn first-hand what works well and how techniques compare.  

&emsp;&emsp; MNIST是计算机视觉中的入门级数据集.自1999年发布以来，这个手写图像的经典数据集作成为了基准分类算法的基础.随着新的机器学习技术的出现，MNIST仍然是研究人员和学习者的可靠资源.在这次竞赛中，你的目标是从成千上万张手写图像的数据集中正确识别数字.鼓励您使用不同的算法进行实验，以直接了解哪些算法工作良好，以及如何比较技术.  

## 运行环境说明
```
    OS-Centos7
    Environment-Python3.7
    Framework-Keras-cpu-2.2
    Editor-Pycharm
```

## 理论分析

## 代码
#### 项目结构
```
  Mnist
    |---main.py                     //主调函数
    |---Mnist-CNN.csv               //输出文件
    |---tmp_x_mnist_weihts.hdf5     //训练器权重文件
    |---train.csv                   //训练集
    |---test.csv                    //待预测数据
```
#### main.py
```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import seaborn as sns
import os
import itertools
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'

from sklearn.model_selection import train_test_split
from sklearn.metrics import confusion_matrix

from keras.layers.normalization import BatchNormalization
from keras.utils.np_utils import to_categorical # convert to one-hot-encoding
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten, Conv2D, MaxPool2D, MaxPooling2D
from keras.optimizers import RMSprop, Adam
from keras.optimizers import Adadelta
from keras.optimizers import Adamax
from keras.optimizers import SGD

from keras.callbacks import ModelCheckpoint
from keras.preprocessing.image import ImageDataGenerator
from keras.callbacks import ReduceLROnPlateau


def get_Model_1():
    model = Sequential()
    model.add(Conv2D(filters=32, kernel_size=(5, 5), padding='Same',
                     activation='relu', input_shape=(28, 28, 1)))
    model.add(Conv2D(filters=32, kernel_size=(5, 5), padding='Same',
                     activation='relu'))
    model.add(MaxPool2D(pool_size=(2, 2)))
    model.add(Dropout(0.25))
    model.add(Conv2D(filters=64, kernel_size=(3, 3), padding='Same',
                     activation='relu'))
    model.add(Conv2D(filters=64, kernel_size=(3, 3), padding='Same',
                     activation='relu'))
    model.add(MaxPool2D(pool_size=(2, 2), strides=(2, 2)))
    model.add(Dropout(0.25))
    model.add(Flatten())
    model.add(Dense(128, activation="relu"))
    model.add(Dropout(0.5))
    model.add(Dense(10, activation="softmax"))
    return model

def get_Model_3():
    model = Sequential()
    model.add(Conv2D(filters=16, kernel_size=(3, 3), activation='relu', input_shape=(28, 28, 1)))
    model.add(Conv2D(filters=16, kernel_size=(3, 3), activation='relu'))
    model.add(MaxPool2D(strides=(2, 2)))
    model.add(Dropout(0.25))

    model.add(Conv2D(filters=32, kernel_size=(3, 3), activation='relu'))
    model.add(BatchNormalization())
    model.add(Conv2D(filters=32, kernel_size=(3, 3), activation='relu'))
    model.add(BatchNormalization())
    model.add(MaxPool2D(strides=(2, 2)))
    model.add(Dropout(0.25))

    model.add(Flatten())
    model.add(Dense(512, activation="relu"))
    model.add(Dropout(0.25))
    model.add(Dense(1024, activation="relu"))
    model.add(Dropout(0.50))
    model.add(Dense(10, activation="softmax"))
    return model

def get_Model_2():
    model = Sequential()
    model.add(Conv2D(filters=32, kernel_size=(3, 3),
                     activation='relu', input_shape=(28, 28, 1)))
    model.add(BatchNormalization())
    model.add(Conv2D(filters=32, kernel_size=(5, 5), activation='relu'))
    model.add(BatchNormalization())
    model.add(Conv2D(filters=32, kernel_size=(5, 5), padding='Same',
                     activation='relu'))
    model.add(BatchNormalization())
    model.add(Dropout(0.25))

    model.add(Conv2D(filters=64, kernel_size=(3, 3), activation='relu'))
    model.add(BatchNormalization())
    model.add(Conv2D(filters=64, kernel_size=(3, 3), activation='relu'))
    model.add(BatchNormalization())
    model.add(Dropout(0.40))

    model.add(Conv2D(filters=64, kernel_size=(4, 4), padding='Same',
                     activation='relu'))
    model.add(BatchNormalization())
    model.add(Flatten())
    model.add(Dense(128, activation="relu"))
    model.add(Dropout(0.5))
    model.add(Dense(10, activation="softmax"))
    return model

if __name__=="__main__":

    # 加载数据
    train = pd.read_csv("./train.csv")
    test = pd.read_csv("./test.csv")
    Y_train = train["label"]
    # 丢弃标签所在的列
    X_train = train.drop(labels=["label"], axis=1)
    # 删除缓存
    del train

    # 归一化
    X_train /= 255.0
    test /= 255.0
    # 修改通道
    X_train = X_train.values.reshape(-1, 28, 28, 1)
    test = test.values.reshape(-1, 28, 28, 1)
    # 标签转向量
    Y_train = to_categorical(Y_train, num_classes=10)

    # 数据增强
    datagen = ImageDataGenerator(
        featurewise_center=False,  # set input mean to 0 over the dataset
        samplewise_center=False,  # set each sample mean to 0
        featurewise_std_normalization=False,  # divide inputs by std of the dataset
        samplewise_std_normalization=False,  # divide each input by its std
        zca_whitening=False,  # apply ZCA whitening
        rotation_range=10,  # randomly rotate images in the range (degrees, 0 to 180)
        zoom_range=0.1,  # Randomly zoom image
        width_shift_range=0.1,  # randomly shift images horizontally (fraction of total width)
        height_shift_range=0.1,  # randomly shift images vertically (fraction of total height)
        horizontal_flip=False,  # randomly flip images
        vertical_flip=False)  # randomly flip images
    datagen.fit(X_train)
    
    # 迭代次数
    epochs = 30
    # 批处理量
    batch_size = 128
    # 训练器数量
    nets = 18
    # 结果集
    res = np.zeros((test.shape[0], 10))

    for i in range(nets):

        print("Training the "+str(i+1)+" LM...")
        
        # Step_0:设定环境参数
        tmp_mnist_weight_path = "./tmp_" + str(i + 1) + "_mnist_weights.hdf5"
        X_train, X_val, Y_train, Y_val = train_test_split(X_train, Y_train, test_size=0.1)

        # Step_1:设定模型
        if i<6:
            model = get_Model_1()
        if i>=6 and i<12:
            model = get_Model_2()
        if i >= 12 and i < 18:
            model = get_Model_3()
        
        # Step_2:设定优化器
        checkpointer = ModelCheckpoint(monitor='val_acc', filepath=tmp_mnist_weight_path, verbose=1,
                                       save_best_only=True,
                                       period=1)
        learning_rate_reduction = ReduceLROnPlateau(monitor='val_acc', patience=3, verbose=1, factor=0.5,
                                                    min_lr=0.00001)
        callbacks_list = [checkpointer, learning_rate_reduction]
        if i%3==0:
            optimizer = RMSprop(lr=0.001, rho=0.90, epsilon=1e-08, decay=0.0)
        if i%3==1:
            optimizer = Adam(lr=0.001, beta_1=0.90, beta_2=0.999, epsilon=1e-8)
        if i%3==2:
            optimizer = Adadelta(lr=0.001, rho=0.95, epsilon=1e-08)
            
        # Step_3:编译模型
        model.compile(optimizer=optimizer, loss="categorical_crossentropy", metrics=["accuracy"])
        
        # Step_4:训练模型
        history = model.fit_generator(datagen.flow(X_train, Y_train, batch_size=batch_size),
                                      epochs=epochs, validation_data=(X_val, Y_val),
                                      verbose=2, steps_per_epoch=X_train.shape[0] // batch_size
                                      , callbacks=callbacks_list)
        # Step_5:加载最优权重并给出预测
        model.load_weights(tmp_mnist_weight_path)
        res = res + model.predict(test)

        print("Trained the " + str(i + 1) + " LM...\n\n")

    # 处理结果并输出
    res = np.argmax(res, axis=1)
    res = pd.Series(res, name="Label")
    submission = pd.concat([pd.Series(range(1, 28001), name="ImageId"), res], axis=1)
    submission.to_csv("MNIST-CNN-ENSEMBLE.csv", index=False)
```
