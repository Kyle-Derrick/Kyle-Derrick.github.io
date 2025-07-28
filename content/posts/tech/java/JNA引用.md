---
title: 'JNA引用相关细节记录'
author: ['kyle']
date: '2025-07-23T14:25:55+08:00'
description: "记录来源于AI回答"
tags:
- Java

keywords:
- Java
---

在 Java JNI 开发中，获取或创建 Java 对象时，虚拟机会通过**引用计数机制**管理对象的生命周期，避免对象被垃圾回收（GC）过早回收。开发者需根据引用类型（局部、全局、弱全局）主动管理引用，而非直接操作引用计数。以下是核心机制和最佳实践：

---

### 🔄 **一、JNI 引用类型与生命周期**
#### 1. **局部引用（Local Reference）**  
- **创建方式**：默认由 JNI 函数返回（如 `FindClass`、`NewObject`、`GetObjectClass`）。  
- **生命周期**：仅在当前 Native 方法执行期间有效，方法返回时**自动释放**，引用计数减一。  
- **手动释放场景**：  
  - 循环中创建大量对象（避免局部引用表溢出，默认容量 512-1024）。  
  - 长时间运行的方法（如后台线程回调）。  
  - 需提前释放大对象（如大数组或字符串）。  
  **代码示例**：  
  ```cpp
  for (int i = 0; i < 1000; i++) {
      jstring str = env->NewStringUTF("test");
      // 使用后立即释放
      env->DeleteLocalRef(str);
  }
  ```

#### 2. **全局引用（Global Reference）**  
- **创建方式**：显式调用 `NewGlobalRef`。  
- **生命周期**：**需手动释放**（`DeleteGlobalRef`），否则对象永远不被回收，导致内存泄漏。  
- **用途**：缓存类（`jclass`）、方法 ID（`jmethodID`）或跨多次 Native 调用的对象。  
  **代码示例**：  
  ```cpp
  static jclass g_myClass = nullptr;
  
  JNIEXPORT void JNICALL Java_MyClass_init(JNIEnv* env, jobject obj) {
      jclass localCls = jclass localCls = env->FindClass("com/example/MyClass");
      if (localCls == nullptr) {
          // 处理异常
          return;
      }
      g_myClass = (jclass)env->NewGlobalRef(localCls); // 提升为全局引用
      env->DeleteLocalRef(localCls); // 释放局部引用
  }
  
  // 退出时释放
  JNIEXPORT void JNI_OnUnload(JavaVM* vm, void* reserved) {
      JNIEnv* env;
      vm->GetEnv((void**)&env, JNI_VERSION_1_6);
      env->DeleteGlobalRef(g_myClass);
  }
  ```

#### 3. **弱全局引用（Weak Global Reference）**  
- **创建方式**：`NewWeakGlobalRef`。  
- **特点**：不阻止 GC 回收对象，需通过 `IsSameObject(ref, NULL)` 检查对象是否存活。  
- **用途**：缓存可能被回收的对象（如 Activity Context）。  
  **代码示例**：  
  ```cpp
  static jweak g_weakRef = nullptr;
  
  void useWeakRef(JNIEnv* env) {
      if (env->IsSameObject(g_weakRef, nullptr)) {
          // 对象已被回收
      } else {
          jobject obj = env->NewLocalRef(g_weakRef); // 转为局部引用再使用
      }
  }
  ```

---

### 🧠 **二、引用管理的核心原则**
1. **避免直接缓存局部引用**  
   局部引用在方法返回后失效，若缓存到静态变量中会变成野指针，引发崩溃。  
   **错误示例**：  
   ```cpp
   static jclass g_cls; // 错误！缓存局部引用
   void init(JNIEnv* env) {
       g_cls = env->FindClass("java/lang/String"); // 方法返回后 g_cls 失效
   }
   ```

2. **及时释放全局引用**  
   全局引用必须成对使用 `NewGlobalRef`/`DeleteGlobalRef`，否则永久泄漏内存。

3. **优化大量局部引用**  
   使用 `PushLocalFrame` 和 `PopLocalFrame` 批量管理局部引用，避免手动释放每个引用：  
   ```cpp
   env->PushLocalFrame(256); // 创建局部引用作用域
   // 创建多个临时对象
   env->PopLocalFrame(nullptr); // 释放作用域内所有局部引用
   ```

---

### 🛠️ **三、关键场景实践**
#### 1. **缓存类/方法 ID**  
- **推荐方式**：在 `JNI_OnLoad` 中将 `jclass` 提升为全局引用，缓存方法 ID（无需释放）：  
  > “jmethodID 由 JVM 自动管理，无需手动释放。”
  > 
  
  ```cpp
  static jclass g_myClass;
  static jmethodID g_myMethod;
  
  JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) {
      JNIEnv* env;
      vm->GetEnv((void**)&env, JNI_VERSION_1_6);
      jclass localCls = env->FindClass("com/example/MyClass");
      g_myClass = (jclass)env->NewGlobalRef(localCls);
      g_myMethod = env->GetMethodID(g_myClass, "myMethod", "()V");
      env->DeleteLocalRef(localCls);
      return JNI_VERSION_1_6;
  }
  ```

#### 2. **跨线程传递对象**  
- 通过 `JavaVM` 附加线程获取 `JNIEnv`，再创建全局引用传递对象：  
  ```cpp
  void worker_thread(JavaVM* jvm) {
      JNIEnv* env;
      jvm->AttachCurrentThread(&env, nullptr);
      jobject globalObj = env->NewGlobalRef(obj); // 跨线程使用
      // ...
      env->DeleteGlobalRef(globalObj);
      jvm->DetachCurrentThread();
  }
  ```

---

### ⚠️ **四、常见内存泄漏问题**
| **问题**                | **原因**                          | **解决方案**                     |
|--------------------------|-----------------------------------|----------------------------------|
| 局部引用缓存为静态变量   | 方法返回后引用失效                | 改用全局引用或弱全局引用         |
| 未释放全局引用           | 忘记调用 `DeleteGlobalRef`       | 在 `JNI_OnUnload` 或销毁时释放  |
| 循环中未释放局部引用     | 局部引用表溢出（默认上限 512）   | 循环内手动调用 `DeleteLocalRef` |
| 跨线程使用未附加的 `JNIEnv` | `JNIEnv` 线程绑定               | 通过 `JavaVM` 获取当前线程 `JNIEnv` |

---

### 💎 **总结**
- **虚拟机自动管理引用计数**：开发者通过创建/释放引用间接控制计数，无需直接操作。  
- **关键操作**：  
  - 局部引用 → 及时 `DeleteLocalRef`（尤其在循环中）。  
  - 全局引用 → 必须配对 `NewGlobalRef`/`DeleteGlobalRef`。  
  - 弱全局引用 → 使用前检查 `IsSameObject`。  
- **最佳实践**：缓存类/方法 ID 用全局引用，临时对象用局部引用并主动释放，跨线程对象传递用全局引用+线程附着。