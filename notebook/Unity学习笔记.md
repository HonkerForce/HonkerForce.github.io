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

### 10.MVC架构个人理解

​	MVC（Model-View-Control）的本质：通过Control监控(使用事件/消息等注册的委托回调实现)View的用户交互和画面变化，编写逻辑修改Model中的数据，并通知View根据Model中的数据更新View控制的画面显示。

### 11.tolua插件的使用

​	tolua插件是Unity引擎的开源lua热更新第三方库，可以针对特定C#类型生成wrap文件，将C#中的类型快速反射到lua虚拟机中，通过lua进行逻辑显示层的编写。也可通过tolua库在C#中的全局LuaState对象，直接获取lua虚拟机中的所有表数据，实现C#与lua之间逻辑和数据的同步，并可借助lua实现刷脚本热更新客户端。

#### tolua库的C# API语法

LuaState封装了对lua 主要数据结构 lua_State 指针的各种堆栈操作。
一般对于客户端，推荐只创建一个LuaState对象。如果要使用多State需要在Unity中设置全局宏 MULTI_STATE

- LuaState.Start 需要在tolua代码加载到内存后调用。如果使用assetbunblde加载lua文件，调用Start()之前assetbundle必须加载好
- LuaState.DoString 执行一段lua代码,除了例子,比较少用这种方式加载代码,无法避免代码重复加载覆盖等,需调用者自己保证。第二个参数用于调试信息,或者error消息(用于提示出错代码所在文件名称)
- LuaState.CheckTop 检查是否堆栈是否平衡，一般放于update中，c#中任何使用lua堆栈操作，都需要调用者自己平衡堆栈（参考LuaFunction以及LuaTable代码）, 当CheckTop出现警告时其实早已经离开了堆栈操作范围，这是需自行review代码。
- LuaState.Dispose 释放LuaState 以及其资源。

tolua#DoFile函数,跟lua保持一致行为,能多次执行一个文件。tolua#加入了新的Require函数,无论c#和lua谁先require一个lua文件, 都能保证加载唯一性

- LuaState.AddSearchPath 增加搜索目录, 这样DoFile跟Require函数可以只用文件名,无需写全路径
- LuaState.DoFile 加载一个lua文件, 注意dofile需要扩展名, 可反复执行, 后面的变量会覆盖之前的DoFile加载的变量
- LuaState.Require 同lua require(modname)操作, 加载指定模块并且把结果写入到package.loaded中,如果modname存在, 则直接返回package.loaded[modname]
- LuaState.Collect 垃圾回收, 对于被自动gc的LuaFunction, LuaTable, 以及委托减掉的LuaFunction, 延迟删除的Object之类。等等需要延迟处理的回收, 都在这里自动执行

tolua# 简化了lua函数的操作，通过LuaFunction封装(并缓存)一个lua函数，并提供各种操作, 建议频繁调用函数使用无GC方式。

- LuaState.GetLuaFunction 获取并缓存一个lua函数, 此函数支持串式操作, 如"test.luaFunc"代表test表中的luaFunc函数。
- LuaState.Invoke 临时调用一个lua function并返回一个值，这个操作并不缓存lua function，适合频率非常低的函数调用。
- LuaFunction.Call() 不需要返回值的函数调用操作
- LuaFunction.Invoke() 有一个返回值的函数调用操作
- LuaFunction.BeginPCall() 开始函数调用
- LuaFunction.Push() 压入函数调用需要的参数，通过众多的重载函数来解决参数转换gc问题
- LuaFunction.PCall() 调用lua函数
- LuaFunction.CheckNumber() 提取函数返回值, 并检查返回值为lua number类型
- LuaFunction.EndPCall() 结束lua函数调用, 清楚函数调用造成的堆栈变化
- LuaFunction.Dispose() 释放LuaFunction, 递减引用计数，如果引用计数为0, 则从_R表删除该函数

> **注意:** 无论Call还是PCall只相当于lua中的函数'.'调用。
> 请注意':'这种语法糖 self:call(...) == self.call(self, ...）
> c# 中需要按后面方式调用, 即必须主动传入第一个参数self

- luaState["Objs2Spawn"] LuaState通过重载this操作符，访问lua _G表中的变量Objs2Spawn
- LuaState.GetTable 从lua中获取一个lua table, 可以串式访问比如lua.GetTable("varTable.map.name") 等于 varTable->map->name
- LuaTable 支持this操作符，但此this不支持串式访问。比如table["map.name"] "map.name" 只是一个key，不是table->map->name
- LuaTable.GetMetaTable() 可以获取当前table的metatable
- LuaTable.ToArray() 获取数组表中的所有对象存入到object[]表中
- LuaTable.AddTable(name) 在当前的table表中添加一个名字为name的表
- LuaTable.GetTable(key) 获取t[key]值到c#, 类似于 lua_gettable
- LuaTable.SetTable(key, value) 等价于t[k] = v的操作, 类似于lua_settable
- LuaTable.RawGet(key) 获取t[key]值到c#, 类似于 lua_rawget
- LuaTable.RawSet(key, value) 等价于t[k] = v的操作, 类似于lua_rawset

- 必须启动LuaLooper驱动协程，这里将一个lua的半双工协程装换为类似unity的全双工协程
- coroutine.start 启动一个lua协程
- coroutine.wait 协程中等待一段时间，单位:秒
- coroutine.step 协程中等待一帧.
- coroutine.www 等待一个WWW完成.
- tolua.tolstring 转换byte数组为lua字符串缓冲
- coroutine.stop 停止一个正在lua将要执行的协程

#### tolua库自定义数据

​	主要需要自定义的是需要生成wrap并绑定到lua虚拟机的C#类型，在CustomSetting.cs文件中的customTypeList表里添加要注册的类型，还有就是wrap文件的生成路径等相关配置，在LuaConst.cs文件中定义。

#### tolua生成wrap文件

​	在Unity的菜单栏中找到"Lua"，使用"Lua\Clear wrap files"清空之前生成的wrap文件，之后使用"Lua\Gen Lua Wrap Files"生成Wrap文件，_针对CustomSetting.cs中注释标记的wrap文件还原为SVN上的版本（解决tolua转换.net7.2+系统库API导致的语法错误问题）_，然后使用"Lua\Gen Lua Delegates"和"Lua\Gen LuaBinder File"，就可成功导入C# API到Lua了。

#### Lua中C#命名空间的映射关系

​	C#全局命名空间中的类，转换后在Lua中都会被放在UnityEngine包中，默认使用的全局C#类，可以不写UnityEngine包名。对于在其他命名空间中定义的类，比如其他第三方库生成导入的类，在导入Lua时会将每个命名空间转化为Lua中的模块(包)，在Lua中使用非UnityEngine命名空间的类时，需要通过``特定的模块(命名空间)名.类名``去访问。
