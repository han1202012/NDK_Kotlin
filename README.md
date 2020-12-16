


<br>
<br>
<br>
<br>

# 一、创建支持 Kotlin 的 NDK 项目 

---

<br>

点击 <font color=blue>菜单栏 / File / New / New Project / Create New Project</font> , 弹出以下对话框 , 选择 <font color=red>Native C++</font> 项目 , 点击 Next 按钮 ; 


![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216143744803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbjEyMDIwMTI=,size_16,color_FFFFFF,t_70)

在后续对话框中 , 使用默认的 Kotlin 语言 , 即可生成 Kotlin 中使用 NDK 的代码 ; 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216143929802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbjEyMDIwMTI=,size_16,color_FFFFFF,t_70)

默认 C++ 标准即可 ; 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216144044166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbjEyMDIwMTI=,size_16,color_FFFFFF,t_70)


<br>
<br>
<br>
<br>

# 二、Kotlin 语言中使用 NDK 要点 

---

<br>



<br>
<br>

## 1、加载动态库 

---

<br>


Kotlin 中在类的 <font color=blue>companion object 伴生对象 </font>中加载动态库 , 类似于 Java 的静态代码块 ; 


```kotlin
    companion object {
        // Used to load the 'native-lib' library on application startup.
        init {
            System.loadLibrary("native-lib")
        }
    }
```

<br>
<br>

## 2、声明 ndk 方法 

---

<br>

Java 中使用 native 声明 ndk 方法 , 在 Kotlin 中 , 使用 <font color=red>external</font> 声明 ndk 方法 ; 

```kotlin
    /**
     * A native method that is implemented by the 'native-lib' native library,
     * which is packaged with this application.
     */
    external fun stringFromJNI(): String
```

ndk 方法对应的 C++ 方法 ; 

```cpp
#include <jni.h>
#include <string>

extern "C" JNIEXPORT jstring JNICALL
Java_kim_hsl_ndk_1kotlin_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```


<br>
<br>

## 3、Project 下的 build.gradle 配置 

---

<br>

需要配置 Kotlin 版本号 , 和 Kotlin 插件版本号 ; 

```java
buildscript {
    ext.kotlin_version = "1.4.10"
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
```


<br>
<br>

## 4、Module 下的 build.gradle 配置 

---

<br>

在 Module 下的 build.gradle 中 , 

kotlin-android 是必须配置的 , 

kotlin-android-extensions 是扩展 ,  选择性配置 , 配置了之后 , 可以很方便地使用视图绑定 ; 

kotlin-kapt 也是选择性配置 , 配置使用注解 ; 

```csharp
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-android-extensions'
    id 'kotlin-kapt'
}
```





<br>
<br>
<br>
<br>

# 三、代码示例 

---

<br>

<br>
<br>

## 1、Java 代码 

---

<br>


```kotlin
package kim.hsl.ndk_kotlin

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.TextView

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Example of a call to a native method
        findViewById<TextView>(R.id.sample_text).text = stringFromJNI()
    }

    /**
     * A native method that is implemented by the 'native-lib' native library,
     * which is packaged with this application.
     */
    external fun stringFromJNI(): String

    companion object {
        // Used to load the 'native-lib' library on application startup.
        init {
            System.loadLibrary("native-lib")
        }
    }
}
```


<br>
<br>

## 2、C++ 代码 

---

<br>


```cpp
#include <jni.h>
#include <string>

extern "C" JNIEXPORT jstring JNICALL
Java_kim_hsl_ndk_1kotlin_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```


<br>
<br>

## 3、Project 下的 build.gradle 

---

<br>

```csharp
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    ext.kotlin_version = "1.4.10"
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath "com.android.tools.build:gradle:4.1.0"
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

<br>
<br>

## 4、Module 下的 build.gradle 

---

<br>

```csharp
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-android-extensions'
    id 'kotlin-kapt'
}

android {
    compileSdkVersion 29
    buildToolsVersion "30.0.2"

    defaultConfig {
        applicationId "kim.hsl.ndk_kotlin"
        minSdkVersion 18
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags ""
            }
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {

    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation 'androidx.core:core-ktx:1.3.2'
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'com.google.android.material:material:1.2.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.3.0'
}
```


<br>
<br>

## 5、执行效果 

---

<br>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216145546944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbjEyMDIwMTI=,size_16,color_FFFFFF,t_70)



<br>
<br>
<br>
<br>

# 四、GitHub 地址  

---

<br>


[https://github.com/han1202012/NDK_Kotlin](https://github.com/han1202012/NDK_Kotlin)

