---
layout:      post
title:       JNI_Invocation_API
subtitle: 
date:        2019-04-05
author:      Haiden
header-img:   
catalog:     true
category:    JNI
tags: 
    - JNI开发
---

Invocation API允许软件提供商在原生程序中内嵌Java虚拟机。因此可以不需要链接任何Java虚拟机代码来提供Java-enabled的应用程序。

### 一、库和版本管理

在 JDK/JRE 1.2，每个ClassLoader都有自己的一组本地库。**一个本地库一旦被一个ClassLoader加载后，则不允许再被其他ClassLoader重复加载了。**否则会抛出 ``UnsatisfiedLinkError`` 异常。

#### 1.1 JNI_OnLoad

Java虚拟机在加载本地库（native library）时（即调用 `System.loadLibrary()` ）后，在加载本地库到内存之后，会寻找其内部的 `JNI_OnLoad` 函数，并执行它。这个函数必须返回本地库使用的 JNI版本号。

#### 1.2 JNI_OnUnload

虚拟机在本地库被垃圾回收前，调用其 `JNI_OnUnload` 函数。这个函数用来执行一个清理操作。

### 二、Invocation API函数

#### 2.1 JNI_GetDefaultJavaVMInitArgs

返回Java虚拟机的默认配置。

```c++
jint JNI_GetDefaultJavaVMInitArgs(void *vm_args);
---------------------
//参数：
vm_args ：JavaVMIntArgs结构体的指针，包含虚拟机默认配置。

//返回值：
成功返回 JNI_OK ，失败返回负数。
```

#### 2.2 JNI_GetCreatedJavaVMs

返回所有已经创建过的Java虚拟机。

```c++
jint JNI_GetCreatedJavaVMs(JavaVM **p_vm, jsize, jsize*);
--------------------------------
//参数：
p_vm：指向 JavaVM的指针。
jsize：指向整数的指针。

//返回值：
成功返回 JNI_OK ，失败返回负数。
```

#### 2.3 JNI_CreateJavaVM

加载和初始化一个Java虚拟机，当前线程作为主线程（main thread）。

在 JDK/JRE 1.2，不允许在同一个进程创建多个Java虚拟机。

```c++
jint JNI_CreateJavaVM(JavaVM **p_vm, void **p_env, void *vm_args);
---------------------
//参数：
p_vm：指向 JavaVM的指针。
p_env：指向 JNIEnv指针的指针。
vm_args: 虚拟机的参数。

//返回值：
成功返回 JNI_OK ，失败返回负数。
```

`vm_args` 的结构体为：

```c++
typedef struct JavaVMInitArgs {
    jint version;

    jint nOptions;
    JavaVMOption *options;
    jboolean ignoreUnrecognized;
} JavaVMInitArgs;
----------------------
version 必须大于等于 JNI_VERSION_1_2,
nOptions 为 options 的数量.
ignoreUnrecognized 设置为 JNI_TRUE ，则会忽视所有不被识别的以 -X 或 _ 开头的参数字符串，如果设置为 JNI_FALSE ，则遇到不被识别的参数时JNI_CreateJavaVM 函数会返回 JNI_ERR
```

#### 2.4 DestoryJavaVM

卸载一个Java虚拟机，并收回它拥有的资源。

```c++
jint DestroyJavaVM(JavaVM *vm);
-------------------------
//参数：
vm：需要被销毁的虚拟机。

//返回值：
成功返回 JNI_OK ，失败返回负数。
```

#### 2.5 AttachCurrentThread

attach当前线程到Java虚拟机，返回JNI接口指针 `JNIEnv` 。一个本地线程不能同时attach到两个不同的Java虚拟机。

```c++
jint AttachCurrentThread(JavaVM *vm, void **p_env, void *thr_args);
------------------------
//参数：
vm：需要被attach到的虚拟机。
p_env ：返回的当前线程的JNI接口指针。
thr_args ：JavaVMAttachArgs 结构体来指定附加信息，或传入 NULL

//返回值：
成功返回 JNI_OK ，失败返回负数。
```

#### 2.6 AttachCurrentThreadAsDaemon

和 `AttachCurrentThread` 类似，只是新创建的 `java.lang.Thread` 被设置为守护线程

```c++
jint AttachCurrentThreadAsDaemon(JavaVM* vm, void** penv, void* args);
---------------------
//参数：
vm：需要被attach到的虚拟机。
penv ：返回的当前线程的JNI接口指针。
args ：JavaVMAttachArgs 结构体来指定附加信息，或传入 NULL

//返回值：
成功返回 JNI_OK ，失败返回负数。
```

#### 2.7 DetachCurrentThread

从java虚拟机detach当前线程。所有这个线程持有的Java监视区(monitor)都会被释放。

```c++
jint DetachCurrentThread(JavaVM *vm);
---------------------
//参数：
vm：需要detach的虚拟机。

//返回值：
成功返回 JNI_OK ，失败返回负数。
```

#### 2.8 GetEnv

获取当前线程的JNI接口指针 `JNIEnv`

```c++
jint GetEnv(JavaVM *vm, void **env, jint version);
-----------------------
//参数：
vm：虚拟机实例。
env：放置返回的当前线程的JNI接口指针。
version：JNI版本。

//返回值：
如果当前线程还没有attach到虚拟机，则设置 *env 为 NULL ，并返回 JNI_EDETACHED 。如果指定的JNI版本不被支持，则也设置 *env 为 NULL ，并且返回 JNI_EVERSION。否则设置 *env 为正常的接口，并返回 JNI_OK 。
```

