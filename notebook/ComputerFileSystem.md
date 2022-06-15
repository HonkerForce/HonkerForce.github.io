# 计算机文件系统

## 文件的定义

​	文件是操作系统中**数据**的<u>最基本载体</u>和<u>操作单位</u>，对数据的所有操作都是通过对其所在的文件进行的。

>文件按其逻辑结构可分为：记录文件和流式文件。而记录文件又可分为：顺序文件、索引文件、索引顺序文件及散列文件等。
>
>流式文件是以字节为单位，对流式文件的访问一般采用穷举搜索的方式，效率不高，故一般需频繁访问的较大数据不适宜采用流式文件逻辑结构。但由于流式文件管理简单，用户可以较方便地对文件进行相关操作。

## 流的概念

​	流是对所有不同标准的I/O设备的抽象化概念。

​	由于不同设备的访问标准不同，计算机制订了标准I/O系统，在标准I/O系统中，会将所有的I/O设备都抽象为流，并对流的I/O操作使用一套标准的操作，进而解决对不同I/O设备的兼容性问题。

> **流** 按*方向*可分为``输入流``和``输出流``。
>
> 对文件的输入可以理解为对输入流的操作，对文件的输出可以理解为对输出流的操作。

## 文件类型

> **文件** 按照*数据存储方式* 可分为``文本文件`` 和``二进制文件``。

	### 文本文件

> **文本文件** 的数据存储单位是 **字节**（8位二进制数），也就是一个char的内存大小。

### 二进制文件

> **二进制文件** 的数据存储单位是 **整数**(int) ，视操作系统的类型确定大小。如果是32位的系统，则一个整数是4个字节（32位二进制数）；如果是64位的系统，则一个整数是8个字节（64位二进制数）。

## C语言文件操作

### 文件操作模式

| 模式 | 含 义      | 说 明                                                        |
| ---- | ---------- | ------------------------------------------------------------ |
| r    | 只读       | 文件必须存在，否则打开失败                                   |
| w    | 只写       | 若文件存在，则清除原文件内容后写入；否则，新建文件后写入     |
| a    | 追加只写   | 若文件存在，则位置指针移到文件末尾，在文件尾部追加写人，故该方式不 删除原文件数据；若文件不存在，则打开失败 |
| r+   | 读写       | 文件必须存在。在只读 r 的基础上加 '+' 表示增加可写的功能。下同 |
| w+   | 读写       | 新建一个文件，先向该文件中写人数据，然后可从该文件中读取数据 |
| a+   | 读写       | 在” a”模式的基础上，增加可读功能                             |
| rb   | 二进制读   | 功能同模式”r”，区别：b表示以二进制模式打开。下同             |
| wb   | 二进制写   | 功能同模式“w”。二进制模式                                    |
| ab   | 二进制追加 | 功能同模式”a”。二进制模式                                    |
| rb+  | 二进制读写 | 功能同模式"r+”。二进制模式                                   |
| wb+  | 二进制读写 | 功能同模式”w+”。二进制模式                                   |
| ab+  | 二进制读写 | 功能同模式”a+”。二进制模式                                   |

### 打开文件和关闭文件

> 头文件：`#include<stdio.h>`
>
> **FILE * fopen(char *filename, char *mode);** 	返回打开的文件指针，打开失败返回NULL。
>
> **int fclose(FILE *fp);** 	正常关闭返回0，否则返回EOF（-1）。

```cpp
// 打开文件示例：
#include <iostream>
#include <stdio.h>
#include <string>

using namespace std;

int main()
{
	FILE* pImgFile = fopen("TestFile.png", "rb");
	if(pImgFile == NULL)
	{
		cout << "The file is not found." << endl;
		return 0;
	}
	else
	{
		cout << "The file open successfully." << endl;
	}
	
	fclose(pImgFile);

    return 0;
}
```

###  文件输入和文件输出

#### 字符文件输入输出方式

> 头文件：`#include<stdio.h>`
>
> **int fgetc (FILE *fp);** 	返回打开的文件指针，打开失败返回NULL。
>
> **char * fgets (char *s, int size, FILE * fp);**	成功返回缓存数据内存的地址，失败返回NULL。
>
> **int fscanf (文件指针，格式控制串，不定长数据地址（数量与格式控制串中的数量一致）);**	成功返回字符数量，失败返回负数。
>
> **int fputc (int c, FILE *fp);** 	成功关闭返回0，否则返回EOF（-1）。
>
> **int fputs (const char *str, FILE *fp);**	成功返回非负数，失败返回EOF（-1）。
>
> **int fprintf (文件指针，格式控制串，输出表列)；**	成功返回输出的字符数量，失败返回负数。

```cpp
// 输入输出字符文件示例：

#include <iostream>
#include <stdio.h>
#include <cstring>

using namespace std;

int main()
{
	FILE* pTxtFile = fopen("TestTxt.txt", "r+");

	if(pTxtFile == NULL)
	{
		cout << "The file is not found." << endl;
		return 0;
	}
	else
	{
		cout << "The file open successfully." << endl;
	}

	char strTxt[128] = {0};
	// fscanf(pTxtFile, "%s", strTxt);
	if(fgets(strTxt, sizeof(strTxt), pTxtFile) == NULL)
	{
		cout << "读取字符文本文件失败！" << endl;
	}
	else
	{
		cout << "TestTxt.txt：" << endl << strTxt << endl;
	}

	strcpy(strTxt, "你好，世界！");
	
	// fprintf(pTxtFile, "%s", "你好，世界！");
	if(fputs(strTxt, pTxtFile) == EOF)
	{
		cout << "写入字符文本文件失败！" << endl;
	}
	else
	{
		cout << "输出成功！" << endl;
	}
	
	fclose(pTxtFile);
    return 0;
}
```

#### 二进制文件输入输出方式

> 头文件：`#include<stdio.h>`
>
> **unsigned fread (void \*buf, unsigned size, unsigned count, FILE\* fp);**	返回读取到的整数个数，如果返回值比count小，则说明已经读取到文件结尾（用feof()判断）或存在读取错误（用ferror()判断）。
>
> 参数：
>
> - buf：指向存放数据块的内存空间，该内存可以是数组空间，也可以是动态分配的内存。void类型指针，故可存放各种类型的数据，包括基本类型及自定义类型等。
>- size：每个数据块所占的字节数。
> - count：预读取的数据块最大个数。
> - fp：文件指针，指向所读取的文件。
> 
> **unsigned fwrite (const void \*bufAunsigned size,unsigned count,FILE\* fp);**	返回写入的整数个数，如果返回值比count小，则说明需要写入的数据已写完（用feof()判断）或存在写入错误（用ferror()判断）。
>
> 参数：
>
> - buf：前加const的含义是buf所指的内存空间的数据块只读属性，避免程序中有意或无意的修改。
>- size：每个数据块所占的字节数。
> - count：预写入的数据块最大个数。
> - fp：文件指针，指向所读取的文件。

### 文件的随机读写

> 头文件：`#include<stdio.h>`
>
> **void rewind (FILE *fp);**	将文件操作指针复位。
>
> **int fseek(FILE *fp, long offset, int origin);**	把文件读写指针调整到从 origin 基点开始偏移 offset 处，即把文件读写指针移动到 origin+offset 处。（offset为字节数）
>
> | origin（第三个参数） | 位置                                     |
> | -------------------- | ---------------------------------------- |
> | SEEK_SET             | 文件开头，即第一个有效数据的起始位置     |
> | SEEK_CUR             | 当前位置                                 |
> | SEEK_END             | 文件结尾，即最后一个有效数据之后的位置。 |
>
> **long ftell (FILE *fp);**	获取当前文件读写指针相对于文件头的偏移字节数。

```cpp
// 输入输出二进制文件、文件随机读写示例：

#include <iostream>
#include <cstring>
#include <stdio.h>

using namespace std;

int main()
{
	FILE* pPngFile = fopen("TestFile.png", "rb");
	if (pPngFile == NULL)
	{
		cout << "打开文件失败！" << endl;
		return 0;
	}

	fseek(pPngFile, 0, SEEK_SET);

	const int nReadSize = 64;		// 读取的字节大小
	unsigned char strPng[nReadSize] = {0};
	if (fread(strPng, sizeof(char), nReadSize / sizeof(char), pPngFile) < nReadSize && ferror(pPngFile))
	{
		cout << "读取前64字节出错！" << endl;
		return 0;
	}
	
	cout << "前64字节：" << endl;
	for(int i = 0; i < nReadSize; ++i)
	{
		if (i && i % 16 == 0)
		{
			printf("\n");
		}

		printf("%02X ", strPng[i]);
	}

	cout << endl;

	fseek(pPngFile, -64, SEEK_END);
	memset(strPng, 0, sizeof(strPng));

	if (fread(strPng, sizeof(char), nReadSize / sizeof(char), pPngFile) < nReadSize && ferror(pPngFile))
	{
		cout << "读取后64字节出错！" << endl;
		return 0;
	}

	cout << "后64字节：" << endl;
	for(int i = 0; i < nReadSize; ++i)
	{
		if (i && i % 16 == 0)
		{
			printf("\n");
		}
		
		printf("%02X ", strPng[i]);
	}
	
	fclose(pPngFile);
    return 0;
}
```
