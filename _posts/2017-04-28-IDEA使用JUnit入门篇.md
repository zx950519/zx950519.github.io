---
layout:     post
title:      IDEA中使用JUnit入门篇
subtitle:   JUnit搭建环境&测试
date:       2017-04-28
author:     Alitria
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - IDEA
    - Java
    - JUnit
---

## 下载&配置

1. 在官网下载插件:https://plugins.jetbrains.com/plugin/3064-junitgenerator-v2-0  
2. 根据路径File-->settings-->Plguins-->Install plugin from disk加载上一步骤下载的文件
3. 修改配置  
将Output Path改为：${SOURCEPATH}/../../test/java/${PACKAGE}/${FILENAME}  
Default Template选择JUnit 4
![Markdown](http://i4.bvimg.com/643127/35045a861ca09f15.jpg)  
模板中生成的package的包名需去掉test
![Markdown](http://i4.bvimg.com/643127/d790f1033811e9f4.png)

## 实例
#### 1. 新建被测试的类MyClass，如下
```
public class MyClass {
    private int radius;
    private double pai = 3.14;
    MyClass (int radiusValue) {
        radius = radiusValue;
    }
    public double getArea () {
        return pai * radius * radius;
    }
    public double getCircle () {
        return pai * 2 * radius;
    }
}
```
#### 2. 生成测试类  
按Alt+Shift+T，选择已有的测试类或新建测试类  
![Markdown](http://i4.bvimg.com/643127/5de0b8785e76464e.png)  
注意勾选要进行测试的方法，最终自动生成的代码如下  
```
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import static org.junit.Assert.*;

public class MyClassTest2 {
    @Before
    public void setUp() throws Exception {
    }

    @After
    public void tearDown() throws Exception {
    }

    @Test
    public void getArea() {
    }

    @Test
    public void getCircle() {
    }
}
```  
#### 3. 编写测试逻辑
```
@Test
public void getCircle() {
    assertEquals(6.28, new MyClass(1).getCircle(), 0.00001);
    // assertEquals(6.28124, new MyClass(1).getCircle(), 0.00001);
}  
```  

#### 4. 运行  
![Markdown](http://i4.bvimg.com/643127/7679a829f3a1ed94.png)  

#### 5. 运行结果  
```
Process finished with exit code 0  
或  
java.lang.AssertionError: 
Expected :6.28124
Actual   :6.28
 <Click to see difference>
```  
#### 6. 工程结构图  
![Markdown](http://i4.bvimg.com/643127/bd12e86ac28df6c7.png)
  
## 声明  
考虑到本文作者水平有限，如果你对本文章有意见，欢迎联系讨论，726710192@qq.com
![Markdown](https://github.com/zx950519/zx950519.github.io/tree/master/img/alitria.png)
