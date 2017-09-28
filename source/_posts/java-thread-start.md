---
title: java线程启动之start源码阅读
date: 2017-09-20 20:40:59
tags: java线程
categories: java基础
---

#### 问题：  

我们知道java实现线程有两种方式：

- 继承Thread类，重写run方法
- 实现Runnable接口，实现run方法

那么启动线程的方法为什么不是重写/实现的run方法，而是start方法？？？

#### 解决：

那么我们就来看看start方法到底“做”了什么？！

##### Thread.java

```java
public
class Thread implements Runnable {    
    public synchronized void start() {
        /**
         * 每个新建线程实例都会有一个threadStatus状态，默认为0，即"NEW"
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /** 
         * 通知线程组这个线程将要被启动，从而这个线程被加入到线程组中，
         * 并且线程组中的未启动的线程数会递减
         */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    /**
     * start调用的本地方法
     */
    private native void start0();
}

```
Thread类的start方法通过JNI调用了start0方法，继续：  
这里下载jvm的源码，openjdk下载网址：http://download.java.net/openjdk/jdk8

##### /jdk/src/share/native/java/lang/Thread.c
```c
// 本地方法
static JNINativeMethod methods[] = {
    {"start0",           "()V",        (void *)&JVM_StartThread},
    {"stop0",            "(" OBJ ")V", (void *)&JVM_StopThread},
    {"isAlive",          "()Z",        (void *)&JVM_IsThreadAlive},
    {"suspend0",         "()V",        (void *)&JVM_SuspendThread},
    {"resume0",          "()V",        (void *)&JVM_ResumeThread},
    {"setPriority0",     "(I)V",       (void *)&JVM_SetThreadPriority},
    {"yield",            "()V",        (void *)&JVM_Yield},
    {"sleep",            "(J)V",       (void *)&JVM_Sleep},
    {"currentThread",    "()" THD,     (void *)&JVM_CurrentThread},
    {"countStackFrames", "()I",        (void *)&JVM_CountStackFrames},
    {"interrupt0",       "()V",        (void *)&JVM_Interrupt},
    {"isInterrupted",    "(Z)Z",       (void *)&JVM_IsInterrupted},
    {"holdsLock",        "(" OBJ ")Z", (void *)&JVM_HoldsLock},
    {"getThreads",        "()[" THD,   (void *)&JVM_GetAllThreads},
    {"dumpThreads",      "([" THD ")[[" STE, (void *)&JVM_DumpThreads},
    {"setNativeName",    "(" STR ")V", (void *)&JVM_SetNativeThreadName},
};

// JNI把本地方法注册到jvm中
JNIEXPORT void JNICALL
Java_java_lang_Thread_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls, methods, ARRAY_LENGTH(methods));
}
```
本地方法start0会调用JVM_StartThread，继续：  

##### /hotspot/src/share/vm/prims/jvm.cpp
```c
static void thread_entry(JavaThread* thread, TRAPS) {
  HandleMark hm(THREAD);
  Handle obj(THREAD, thread->threadObj());
  JavaValue result(T_VOID);
  JavaCalls::call_virtual(&result,
                          obj,
                          KlassHandle(THREAD, SystemDictionary::Thread_klass()),
                          vmSymbols::run_method_name(),
                          vmSymbols::void_method_signature(),
                          THREAD);
}

JVM_ENTRY(void, JVM_StartThread(JNIEnv* env, jobject jthread))
  JVMWrapper("JVM_StartThread");
  JavaThread *native_thread = NULL; 

      ...

      native_thread = new JavaThread(&thread_entry, sz);

  ...

  Thread::start(native_thread);

JVM_END
```
在JVM_StartThread函数内创建了真正的平台相关的本地线程native_thread，其线程函数是thread_entry，jvm.cpp中也有thread_entry的定义；通过JavaCalls的call_virtual去调用java中的run方法，继续：  

##### /hotspot/src/share/vm/classfile/vmSymbols.hpp
```c
template(run_method_name, "run") 
```
在call_virtual方法的参数中传入run，调用我们实现的run方法；

**start方法来启动线程，真正实现了多线程运行；run方法是当作普通方法的方式调用。**