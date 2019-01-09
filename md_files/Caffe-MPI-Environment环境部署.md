# Caffe-MPI-Environment
Caffe-MPI在Centos下的安装

## 目录
- 说明
- 快速安装
- 普通安装 
- 运行demo
- 致谢

## 说明

&emsp;&emsp;原案传送门:https://github.com/sailorsb/caffe-parallel  

&emsp;&emsp;Caffe-Parallel是一个更快的深度学习框架，它来自BVLC / caffe（主分支）。（https://github.com/BVLC/caffe，更多详情请访问 http://caffe.berkeleyvision.org）。该项目的主要成就是通过MPI实现数据并行。  

&emsp;&emsp;下面的安装过程必要的环境依赖于集群内所有机器可以互相免密ssh，请提前配置好。自动化脚本可以参考:https://github.com/Beckham007/b_keygen  
&emsp;&emsp;压缩包下载: 链接：https://pan.baidu.com/s/1h1NoOHkKitIZKxEt8TIXPw 密码：gb2s  

## 快速安装

&emsp;&emsp;在快速安装中，我们仅需将已经编译好的各个组件从压缩包中解压，然后配置本地环境即可，在我的环境中的目录结构为:  

```
-home
  |---其他用户
  |---hadoop
        |---其他文件夹
        |---environment
              |---boost(组件)
              |---caffe-parallel-master(主要)
              |---curl(组件)
              |---gflags(组件)
              |---glog(组件)
              |---glog-master(组件)
              |---hdf5(组件)
              |---jsoncpp(组件)
              |---leveldb(组件)
              |---lmdb(组件)
              |---openblas(低版本)
              |---openblas-0.2.20(组件)
              |---opencv-2.4.9(组件)
              |---openmpi-1.6.5(弃用)
              |---protobuf(组件)
              |---snappy(组件)
              |---myenv2profile.txt(环境变量配置文件)
              |---myinjectLib(需要注入的库的集合)
        |---environment.tar.gz
        |---openmpi-1.6.5(组件)
        |---openmpi-1.6.5.tar.gz
```

- 解压缩包后目录如上
- cd environment后su root登录root账户
- 执行：cat myenv2profile.txt >> /etc/profile将我们预先写好的环境变量(需要根据你自己的环境进行修改)直接进行追加，避免繁琐的手工配置  
```

#################################################################
#	Caffe Environment
#################################################################

#export PATH=$PATH:/home/hadoop/protobuf-2.5.0/bin
#export PKG_CONFIG_PATH=/usr/Env/opencv-2.4.9/unix-install:$PKG_CONFIG_PATH

# boost
#export PATH=$PATH:/home/hadoop/envirnoment/boost_1_56_0/boost
#export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/boost_1_56_0
export PATH=$PATH:/home/hadoop/envirnoment/boost_1640/include
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/boost_1640/lib

# opencv
export PATH=$PATH:/home/hadoop/environment/opencv-2.4.9/include
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/opencv-2.4.9/lib
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/hadoop/environment/opencv-2.4.9/lib/pkgconfig

# openblas
export PATH=$PATH:/home/hadoop/environment/OpenBLAS/include
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/OpenBLAS/lib

# gflags
export PATH=$PATH:/home/hadoop/environment/gflags-2.0/include

## ???
export PATH=$PATH:/usr/include

# ???
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

# HDF5
export HDF5_DIR=/home/hadoop/environment/hdf5
export PATH=$PATH:/home/hadoop/environment/hdf5/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/hdf5/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/hdf5/share

# CRUL
export PATH=$PATH:/home/hadoop/environment/curl/include
export PATH=$PATH:/home/hadoop/environment/curl/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/curl/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/curl/share

# gflag
export GFLAGS_ROOT_DIR=/home/hadoop/environment/gflags-2.0
export PATH=$PATH:/home/hadoop/environment/gflags-2.0/include
export PATH=$PATH:/home/hadoop/environment/gflags-2.0/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/gflags-2.0/lib

# glog
export GLOG_ROOT_DIR=/home/hadoop/environment/glog
export PATH=$PATH:/home/hadoop/environment/glog/include
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/glog/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/glog/share

# leveldb
export LEVELLDB_ROOT=/home/hadoop/environment/leveldb:$PATH
export PATH=$PATH://home/hadoop/environment/leveldb/include
export LD_LIBRARY_PATH=/home/hadoop/environment/leveldb/lib64:$LD_LIBRARY_PATH

# lmdb
export LMDB__DIR=/home/hadoop/environment/lmdb
export PATH=$PATH:/home/hadoop/environment/lmdb/include
export PATH=$PATH:/home/hadoop/environment/lmdb/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/environment/lmdb/lib

# protobuf
export PATH=/home/hadoop/environment/protobuf/include:$PATH
export PATH=/home/hadoop/environment/protobuf/bin:$PATH
export LD_LIBRARY_PATH=/home/hadoop/environment/protobuf/lib:$LD_LIBRARY_PATH


# openmpi
export PATH=/home/hadoop/openmpi-1.6.5/bin:$PATH
export PATH=/home/hadoop/openmpi-1.6.5/include:$PATH
export LD_LIBRARY_PATH=/home/hadoop/openmpi-1.6.5/lib:$LD_LIBRARY_PATH

# snappy

# json
export LD_LIBRARY_PATH=/home/hadoop/environment/jsoncpp/libs/linux-gcc-4.8.5:$LD_LIBRARY_PATH

```
- cd myinjectLib后执行：cp ./lib* /usr/lib将所有运行时需要的库注入到系统默认的动态链接库  

## 普通安装

&emsp;&emsp;依赖环境及其版本如下图所示，在各自官网下载即可，手动安装教程请参考度娘或Google。具体细节有待补充：  

![](http://ww1.sinaimg.cn/large/005L0VzSly1ftvg577c5oj30ff0f7wej.jpg)  


## 运行Demo
&emsp;&emsp;测试mpi是否安装好:mpirun -n 6 pwd

&emsp;&emsp;单机测试caffe-mpi运行:在/home/hadoop/environment/caffe-parallel-master/examples/mnist目录下；修改machinefile，填写自己机器hostname；在终端里输入 sh ./MyTrainLenet.sh即可；下图是我在一个从节点单机运行测试程序：

![](http://ww1.sinaimg.cn/large/005L0VzSly1ftvfvbf5xfj30iv09owhp.jpg)  

&emsp;&emsp;单机测试caffe-mpi运行:在/home/hadoop/environment/caffe-parallel-master/examples/mnist目录下；修改machinefile，填写参与计算的机器hostname；在终端里输入 sh ./MyTrainLenet.sh即可；下图是我在一个master节点并行运行测试程序：  

![](http://ww1.sinaimg.cn/large/005L0VzSly1ftvfzszvltj30ir06eabo.jpg)  

![](http://ww1.sinaimg.cn/large/005L0VzSly1ftvfzzosegj30ix05w0ud.jpg)  

## 致谢

&emsp;&emsp;感谢范禹辰同学的热心帮助！



