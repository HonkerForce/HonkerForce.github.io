## Unity学习笔记

### 1. 脚本封装为链接库

Unity的C#脚本可以统一封装入一个链接库，通过引用（using）链接库的方式调用脚本中定义的类。可以在创建C#项目的时候选择项目类型为“类库”。

![image-20220914115440887](./Unity学习笔记/imageset/image-20220914115440887.png)

在创建的类库项目的引用中添加需要依赖的程序集。

![image-20221115104535315](./Unity学习笔记/imageset/image-20221115104535315.png)

![image-20221115104554567](./Unity学习笔记/imageset/image-20221115104554567.png)

之后在此项目（同名Namespace）中定义的程序，就会编译进一个与项目同名的链接库中，将链接库设置为Unity脚本项目的依赖库后，便可在Unity项目中导入使用该链接库中定义的程序。

```c#
// 链接库
using System;
using UnityEngine;

namespace DLLTest
{
	public class MyUtilities
	{
		public int c;

		public void AddValues(int a, int b)
		{
			c = a + b;
		}

		public static void GenerateRandom(int min, int max)
		{
			Debug.Log("GenerateRandom调用！！！");
		}
	}
}
```

```c#
// Unity脚本项目
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using DLLTest;		// 导入dll链接库

public class Test : MonoBehaviour
{
	void Start()
	{
		MyUtilities utils = new MyUtilities();		// 实例化链接库中定义的类
		utils.AddValues(2, 3);		// 调用链接库中定义的类的成员函数
		Debug.Log("2 + 3 = " + utils.c);		// 访问链接库中类的实例的publi成员
	}

	void Update()
	{
		MyUtilities.GenerateRandom(0, 100);		// 调用链接库中定义的类的静态成员函数
	}
}

```

![image-20221115104916332](./Unity学习笔记/imageset/image-20221115104916332.png)

除了导入C#脚本的链接库，还可以将C#链接库中包含的脚本类直接作为Script组件使用（**前提是链接库中供Unity使用的脚本类必须继承自MonoBehaviour**）：

![image-20220914150948523](./Unity学习笔记/imageset/image-20220914150948523.png)

```c#
// 可供Unity直接导入使用的DLL链接库
using System;
using UnityEngine;

namespace DLLTest
{
	public class MyUtilities : MonoBehaviour
	{
		void Start()
		{
			Debug.Log("只有继承MonoBehaviour的子类才能被Unity作为script组件！");
		}

		void Update()
		{
			Debug.Log("我爱Unity！");
		}
	}
}
```

### 2.多客户端测试

​	除了Build项目后，启动多个客户端程序去测试（测试岗一般方法），在自己本机上为了模拟多个客户端，可以通过Windows指令mklink创建项目文件夹的联接文件夹，类似于复制了一个会根据原项目文件夹变化的项目文件夹，这样就可以通过Unity同时打开两个名字不相同的项目，其中两个项目的核心文件都是会自动同步的。批处理程序如下：

```bat
@echo off

set dirSrc=E:\Rocket20220902\Bin\Client\Game\
set dirTarget=E:\Rocket20220902_Copy\Bin\Client\Game\

if not exist %dirSrc% ( md %dirSrc%)
if not exist %dirTarget% ( md %dirTarget%)

mklink /J %dirTarget%Assets %dirSrc%Assets

mklink /J %dirTarget%ProjectSettings %dirSrc%ProjectSettings

pause
```

### 3.Unity导入Protobuf

#### 下载protobuf库和编译工具

​	首先在GitHub上，下载最新版的Protobuf项目。

​	[Protobuf下载地址](https://github.com/protocolbuffers/protobuf/releases/tag/v3.20.2)

![image-20221012121904805](imageset/image-20221012121904805.png)

​	下载csharp语言的protobuf库的压缩包，并下载对应平台的protoc编译工具。

#### 项目导入protobuf

![image-20220928172327822](imageset/image-20220928172327822.png)

​	解压进入src目录，使用VS打开Google.Protobuf解决方案。

![image-20220928172730607](imageset/image-20220928172730607.png)

​	选择Release配置，生成Google.Protobuf程序集，编译成功后，生成的库文件在项目文件的bin\Release下。**PS：如果生成失败，请检查VS支持的.net sdk是否与下载的protobuf项目使用的一致。例如使用VS2022编译Protobuf3.19.2会报缺少.NET SDK的错误和警告，更换Protobuf3.20.1后就可以编译成功了。**

> NETSDK1141		Unable to resolve the .NET SDK version as specified in the global.json located at F:\protobuf\protobuf-3.19.2\global.json.
>
> 警告		无法找到 global.json 指定的 .NET SDK，请检查是否安装了指定的版本。

​	因为编译使用动态链接库需要保证Protobuf库、VS环境、Unity后台脚本编译器上的.net版本兼容，所以可以选择不使用Protobuf库，而是直接导入Protobuf源码的方式去使用Protobuf。

​	即只需要将下载的Protobuf项目中的csharp/src/Google.Protobuf文件夹复制粘贴到Unity项目的Asset目录中（具体路径可根据项目设计微调，但必须在Asset目录下）。

​	**PS：Protobuf 3.13以上版本，不可直接使用源码导入**

---

![image-20220928175048922](imageset/image-20220928175048922.png)

​	使用链接库方式导入：将**net45**文件夹下的库文件都复制粘贴到项目Assets目录下。**PS：Google.Protobuf库文件不在Unity项目Asset目录下会报找不到Google命名空间的错误。**

#### 根据proto文件生成cs脚本

![image-20221012123429675](imageset/image-20221012123429675.png)

​	创建Assets/Scripts/Protobuf文件夹，将.proto文件和protoc.exe放在Assets/Scripts/Proto文件夹中，使用Unity菜单栏工具（Tool/Protobuf/Compile）编译生成proto的cs脚本，cs脚本会自动生成在Assets/Scripts/Protobuf文件夹下。

![image-20221012092741294](imageset/image-20221012092741294.png)

### 4.Unity自动生成的配置文件

​	场景文件(.unity)：记录了所有场景中的GameObject上挂载的各个组件，以及各个组件的参数设置

​	资源配置(.meta)：记录了各种资源（预制体、美术资源、音频资源、文件夹资源、链接库资源）的参数设置

### 5.Unity中常用的宏定义

| 宏                     | 定义                          |
| :--------------------- | :---------------------------- |
| UNITY_STANDALONE       | PC(包括Windows、MacOS、Linux) |
| UNITY_STANDALONE_WIN   | Windows                       |
| UNITY_STANDALONE_OSX   | MacOS                         |
| UNITY_STANDALONE_LINUX | Linux                         |
| UNITY_ANDROID          | 安卓                          |
| UNITY_IOS              | IOS                           |
| UNITY_WEBGL            | WebGL                         |
| UNITY_WP_8_1           | Windows Phone 8.1             |
| UNITY_PS4              | PlayStation 4                 |

### 6.ReSharper For VS 常用快捷键

| 快捷键                        | 作用                                             |
| ----------------------------- | ------------------------------------------------ |
| Alt + F7                      | 查找引用                                         |
| Ctrl + N                      | Go To Everything 定位到任何，非常强大            |
| Ctrl + Shift + N              | Go To File 定位到文件                            |
| Ctrl + F12                    | Go To File Member 在当前类中查找                 |
| F2                            | 重命名任何东西，重构利器                         |
| Ctrl + Tab                    | 活动文件之间切换，当前打开的所有文件             |
| Ctrl + Shift + Alt +向上/向下 | 上下行代码交换位置                               |
| Ctrl + [Shift] + W            | 快速选中[/取消选中]整个/一块单词                 |
| Ctrl + Alt + F                | Clean Code                                       |
| Ctrl + Alt + J                | 快速添加语句块，如if,for,try catch,using,#region |
| Alt + F12                     | 显示下一个Error                                  |
| Ctrl + E                      | 显示最近编辑的文件                               |

![img](./Unity学习笔记/imageset/v2-1f4486a295fdd938aacb918afc8f3925_r.jpg)

### 7.Application几个关键的Path

Unity的Application有几个关键的Path：Application.dataPath、Application.streamingAssetsPath、Application.persistentDataPath、Application.temporaryCachePath。 

在个平台下的具体路径如下：

|                    | Application.dataPath                                         | Application.streamingAssetsPath                              | Application.persistentDataPath                             | Application.temporaryCachePath                               |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------ |
| iOS                | Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data | Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data/Raw | Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents | Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Library/Caches |
| Android            | /data/app/xxx.xxx.xxx.apk                                    | jar:file:///data/app/xxx.xxx.xxx.apk/!/assets                | /data/data/xxx.xxx.xxx/files                               | /data/data/xxx.xxx.xxx/cache                                 |
| Windows            | /Assets                                                      | /Assets/StreamingAssets                                      | C:/Users/xxxx/AppData/LocalLow/CompanyName/ProductName     | C:/Users/xxxx/AppData/Local/Temp/CompanyName/ProductName     |
| Mac                | /Assets                                                      | /Assets/StreamingAssets                                      | /Users/xxxx/Library/Caches/CompanyName/Product Name        | /var/folders/57/6b4_9w8113x2fsmzx_yhrhvh0000gn/T/CompanyName/Product Name |
| Windows Web Player | file:///D:/MyGame/WebPlayer (即导包后保存的文件夹，html文件所在文件夹) |                                                              |                                                            |                                                              |

### 8.WWW加载资源

WWW是异步加载所以执行加载命令式不能直接执行读取解析操作，要等待

``` csharp
WWW www = new WWW(filePath);
yield return www; // while (!www.isDone) {}
result = www.text;
```

Android之所以不支持C# IO流 方式读取StreamingAssets下的文件，是因为Android手机中，StreamingAssets下的文件都包含在压缩的.jar文件中（这基本上与标准的zip压缩文件的格式相同）。不能直接用读取文件的函数去读，而要用WWW方式。具体做法如下： 
1.把你要读取的文件放在Unity项目的Assets/StreamingAssets文件夹下面，没有这个文件夹的话自己建一个。 
2.读取的代码(假设名为"文件.txt") 

```csharp
//用来存储读入的数据
byte[] bytes;       
//判断当前程序是否运行在安卓下                                                                                                   
if (Application.platform == RuntimePlatform.Android)                                            
{ 
    

    string fpath= "jar:file://" + Application.dataPath + "!/assets/" + "文件.txt"; 
    //等价于string fpath = Application.StreamingAssetPath+"/文件.txt"; 
    
    WWW www = new WWW(fpath);                                                                
    //WWW是异步读取，所以要用循环来等待 
    while(!www.isDone){}                                                                       
    //存到字节数组里 
    bytes= www.bytes;                                                                           

 

} 
else 
{ 
    //其他平台的读取代码 
}
```

### 9.各目录权限

#### 1.StreamingAssets文件夹（只读）

``` csharp
#if UNITY_EDITOR
    string filepath = Application.dataPath +"/StreamingAssets/"+"my.xml";
#elif UNITY_IOS
    string filepath = Application.dataPath +"/Raw/"+"/my.xml";
#elif UNITY_ANDROID
    string filepath = "jar:file://" + Application.dataPath + "!/assets/"+"my.xml;
#endif
```

#### 2.Resources文件夹（只读）

可以使用Resources.Load("名字"); 把文件夹中的对象加载出来

#### 3.Application.dataPath文件夹（只读）
可以使用Application.dataPath进行读操作

Application.dataPath： 只可读不可写，放置一些资源数据

#### 4.Application.persistentDataPath文件夹（读写）
iOS与android平台都可以使用这个目录下进行读写操作，可以存放各种配置文件进行修改之类的。

在PC上的地址是：C:\Users\用户名 \AppData\LocalLow\DefaultCompany\test

#### 5.总结

##### (1)在项目根目录中创建Resources文件夹来保存文件。

可以使用Resources.Load("文件名字，注：不包括文件后缀名");把文件夹中的对象加载出来。
**注：此方可实现对文件实施“增删查改”等操作，但打包后不可以更改了。**

##### (2)直接放在项目根路径下来保存文件

在直接使用Application.dataPath来读取文件进行操作。
**注：移动端是没有访问权限的。**

##### (3)在项目根目录中创建StreamingAssets文件夹来保存文件。

a.可使用Application.dataPath来读取文件进行操作。

```csharp
#if UNITY_EDITOR
    string filepath = Application.dataPath +"/StreamingAssets/"+"my.xml";
#elif UNITY_IOS
    string filepath = Application.dataPath +"/Raw/"+"my.xml";
#elif UNITY_ANDROID
    string filepath = "jar:file://" + Application.dataPath + "!/assets/"+"my.xml;
#endif
// 上面三个平台各自判断，其实不用那么麻烦，直接统一使用Application.streamingAssets即可
// string filepath = Application.streamingAssets + "/my.xml";
```


b.直接使用Application.streamingAssetsPath来读取文件进行操作。
**注：此方法在pc/Mac电脑中可实现对文件实施“增删查改”等操作，但在移动端只支持读取操作。**

##### (4)使用Application.persistentDataPath来操作文件（荐）

该文件存在手机沙盒中，因此不能直接存放文件，

1. 通过服务器直接下载保存到该位置，也可以通过md5码比对下载更新新的资源
2. 没有服务器的，只有间接通过文件流的方式从本地读取并写入Application.persistentDataPath文件下，然后再通过Application.persistentDataPath来读取操作。
**注：在Windows / Mac电脑 以及Android、iPad、iPhone都可对文件进行任意操作，另外在iOS上该目录下的东西可以被iCloud自动备份。**
