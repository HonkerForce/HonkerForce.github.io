## Unity学习笔记

### 1. 脚本封装为链接库

Unity的C#脚本可以统一封装入一个链接库，通过引用（using）链接库的方式调用脚本中定义的类。可以在创建C#项目的时候选择项目类型为“类库”。

![创建类库](D:\HonkerForce.github.io\notebook\imageset\image-20220914115440887.png)

在创建的类库项目的引用中添加需要依赖的程序集。

![添加依赖的程序集](C:\Users\dongdeyu\AppData\Roaming\Typora\typora-user-images\image-20220914120305631.png)

![image-20220914120353261](C:\Users\dongdeyu\AppData\Roaming\Typora\typora-user-images\image-20220914120353261.png)

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

![image-20220914121051260](C:\Users\dongdeyu\AppData\Roaming\Typora\typora-user-images\image-20220914121051260.png)

除了导入C#脚本的链接库，还可以将C#链接库中包含的脚本类直接作为Script组件使用（**前提是链接库中供Unity使用的脚本类必须继承自MonoBehaviour**）：

![image-20220914150948523](imageset/image-20220914150948523.png)

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

