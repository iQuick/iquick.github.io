---
layout:		post
title:		Studio 签名大法
category:	Android
tags:		Studio
published:	true
---
# Studio 签名大法
---

Android Studio 上对 apk 进行签名方法无外乎两种

## 第一种
通过任务栏上的 Build -> Generate Signed Apk 进行签名

此方法简单，但不适于批量打包签名


## 第二种
在 build.gradle 添加 task 

```gralde
signingConfigs {
    release {
        storeFile file("Android")
        storePassword "Android"
        keyAlias "Android"
        keyPassword "Android"
    }
    debug {
        storeFile file("Android")
        storePassword "Android"
        keyAlias "Android"
        keyPassword "Android"
    }
}
```
此方法简单，也可以适用于批量打包签名，只是在上传到的我 git 仓库的时候遇到了问题

我不想将我的签名信息也上传到仓库中，这时上传到 git 仓库时就需要修改我的 build.gradle 文件，这无疑又变的麻烦了


<!--break-->

## 新方法
在新方法中，我将签名文件独立成了一个新的文件 sign.config ，然后在 .gitignore 中将其过滤掉

**sign.config**

```json
{
    "storeFile": "C:/Users/Em/Desktop/Newsme/key.jks",
    "storePassword": "android",
    "keyAlias": "android",
    "keyPassword": "android"
}
```

**build.gralde**

```gralde
~~

class SignInfo {
    public String storeFile;
    public String storePassword;
    public String keyAlias;
    public String keyPassword;
}

ext.signinfo = null;

def SignInfo parseSignInfo() {
    if (signinfo == null) {
        String str = GFileUtils.readFile(file("sign.config"));
        signinfo = new Gson().fromJson(str, SignInfo.class);
    }
    return signinfo;
}

def getMyStoreFile() {
    return parseSignInfo().storeFile;
}

def getMyStorePassword() {
    return parseSignInfo().storePassword;
}

def getMyKeyAlias() {
    return parseSignInfo().keyAlias;
}

def getMyKeyPassword() {
    return parseSignInfo().keyPassword;
}


android {
	~~

    signingConfigs {
        release {
            storeFile file(getMyStoreFile())
            storePassword getMyStorePassword()
            keyAlias getMyKeyAlias()
            keyPassword getMyKeyPassword()
        }
        debug {
            storeFile file(getMyStoreFile())
            storePassword getMyStorePassword()
            keyAlias getMyKeyAlias()
            keyPassword getMyKeyPassword()
        }
    }

    buildTypes {

        debug {
        }

        release {
            minifyEnabled true
            shrinkResources false

            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release

        }
    }
    ~~
}

~~

```

至此，大功告成