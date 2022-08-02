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

## C++文件操作

### 重定向输入输出流

> 头文件：`#include<stdio.h>`
>
> **FILE \* freopen(文件名, 文件打开模式, 输入流或输出流[stdin或stdout]);**

```cpp
// 重定向输入输出流示例：

#include<cstdio>
#include<iostream>
using namespace std;
int main()
{
	freopen("a+b.in","r",stdin);
	freopen("a+b.out","w",stdout);
	//以上是模板
	int a,b;
	cin>>a>>b;
	cout<<a+b<<endl;
	return 0;
}
```

### C++文件流

> 文件流一共有三种，文件输入流（ifstream），文件输出流（ofstream），文件输入输出流（fstream）。三种流对应三个类。
>
> 输入流只能以读取模式打开文件，输出流只能以写入模式打开文件，输入输出流以读取和写入模式打开文件。
>
> **每个文件流类的构造函数可以传入文件名，可以用缺省的模式打开文件。**
>
> 也可以调用类内的成员函数对文件进行操作 ：
>
> #### 打开文件
>
> **void open(const char* filename,int mode,int access);**
>
> 打开文件的方式在类ios(是所有流式I/O类的[基类](https://baike.baidu.com/item/基类/9589663))中定义，常用的值如下：
>
> ios::app：以追加的方式打开文件<br>ios::ate：文件打开后定位到文件尾<br>ios:app就包含有此属性<br>ios::binary：以二进制方式打开文件，缺省的方式是文本方式。<br>ios::in：文件以输入方式打开<br>ios::out：文件以输出方式打开<br>ios::nocreate：不建立文件，所以文件不存在时打开失败<br>ios::noreplace：不覆盖文件，所以保存文件时如果文件存在失败<br>ios::trunc：如果文件存在，把文件长度设为0
>
> 可以用“或”把以上属性连接起来，如 ios::out | ios::binary
>
> *注：新的[C++标准库](https://baike.baidu.com/item/C%2B%2B标准库/8795193)不支持nocreate和noreplace，以前的旧版本可以用.*
>
> 打开文件的属性取值是：
>
> 0：普通文件，打开访问<br>1：只读文件<br>2：隐含文件<br>4：系统文件
>
> 可以用“或”或者“+”把以上属性连接起来 ，如`3`或`1 | 2`就是以只读和隐含属性打开文件。
>
> *注：打开文件后可使用__bool is_open() const;__函数根据返回值判断是否打开成功。*
>
> #### 关闭文件
>
> **void close();**
>
> #### 读写文件
>
> 1. 文本文件<br>可直接使用插入器(<<)向文件输出；用析取器(>>)从文件输入。<br>**ofstream &put(char ch);**写入一个字符<br>**ifstream &get(char &ch);**读取一个字符到ch<br>**int get();**返回一个字符<br>**ifstream &get(char *buf,int num,char delim='n')；**把[字符](https://baike.baidu.com/item/字符)读入由 buf 指向的[数组](https://baike.baidu.com/item/数组)，直到读入了 num 个字符或遇到了由 delim 指定的字符，如果没使用 delim 这个参数，将使用缺省值[换行符](https://baike.baidu.com/item/换行符/1410821)'n'。
>
> 2. 二进制文件<br>
>
>    **read(unsigned char *buf,int num);**
>
>    **write(const unsigned char *buf,int num);**
>
>    参数：读取到的数据块存储的地址，读取的数据块总大小
>
> #### 文件定位
>
> **istream& seekg(streamoff offset,seek_dir origin);**文件操作指针跳转到origin + offset处
>
> **ostream&seekp(streamoff offset,seek_dir origin);**文件操作指针跳转到origin + offset处（同上）
>
> seek_dir 表示移动的基准位置，是一个有以下值的枚举：<br>	ios::beg：文件开头<br>	ios::cur：文件当前位置<br>	ios::end：文件结尾
>
> **pos_type tellp();**获得当前文件流操作指针的位置
>
> **pos_type tellg();**获得当前文件流操作指针的位置（同上）

```cpp
// C++文件流随机读写二进制文件示例：

#include <iostream>
#include <fstream>
#include <cstring>
#include <stdio.h>

using namespace std;

int main()
{
	ifstream PngFile("TestFile.png", ios::in | ios::binary);

	if (!PngFile.is_open())
	{
		cout << "打开文件失败！" << endl;
		return;
	}
	
	const int ReadSize = 64;
	char strRead[ReadSize] = {0};
	// PngFile.seekg(0, ios::beg);
	long long llSize = PngFile.readsome(strRead, ReadSize * sizeof(char));
	if(llSize < ReadSize * sizeof(char) && PngFile.fail())
	{
		cout << "读取文件1失败！" << endl;
		return;
	}

	cout << "前64位：" << endl;
	for(int i = 0; i < ReadSize; ++i)
	{
		if(i && i % 16 == 0)
		{
			cout << endl;
		}

		printf("%02X ", (unsigned char)strRead[i]);
	}

	memset(strRead, 0, sizeof(strRead));
	PngFile.seekg(-ReadSize, ios::end);
	llSize = PngFile.readsome(strRead, ReadSize * sizeof(char));
	if(llSize < ReadSize * sizeof(char) && PngFile.fail())
	{
		cout << "读取文件2错误！" << endl;
		return;
	}
	cout << endl << "后64位：" << endl;
	for(int i = 0; i < ReadSize; ++i)
	{
		if(i && i % 16 == 0)
		{
			cout << endl;
		}

		printf("%02X ", (unsigned char)strRead[i]);
	}

	PngFile.close();
	return 0;
}
```

## 内存映射文件

​	因为文件存储在硬盘，正常读取文件时，要先从硬盘读取到操作系统内核态的页缓存中，之后再由页缓存拷贝到读取该文件的用户进程内存中去，所以会经历两次拷贝。写入文件也同样是反方向的两次拷贝。而通过内存映射的方式打开文件，则可以建立用户进程内存空间到硬盘的映射，使文件的读写只需要经历一次拷贝就可以完成，所以相对标准文件I/O操作，使用操作系统内存映射的方式打开文件更加高效。

### Windows内存映射文件

> 打开文件：
>
> ​	HANDLE CreateFile(LPCTSTR lpFileName, DWORD dwDesiredAccess, DWORD dwShareMode, LPSECURITY_ATTRIBUTES lpSecurityAttributes, DWORD dwCreationDisposition, DWORD dwFlagsAndAttributes, HANDLE hTemplateFile);
>
> 创建文件映射区：
>
> ​	HANDLE CreateFileMapping(HANDLE hFile,
> LPSECURITY_ATTRIBUTES lpFileMappingAttributes,
> DWORD flProtect,
> DWORD dwMaximumSizeHigh,
> DWORD dwMaximumSizeLow,
> LPCTSTR lpName);
>
> 映射文件：
>
> ​	SYSTEM_INFO sinf;<br> GetSystemInfo(&sinf); <br>DWORD dwAllocationGranularity = sinf.dwAllocationGranularity;读取系统分配粒度
>
> ​	LPVOID MapViewOfFile(HANDLE hFileMappingObject,
> DWORD dwDesiredAccess,
> DWORD dwFileOffsetHigh,
> DWORD dwFileOffsetLow,
> DWORD dwNumberOfBytesToMap);
>
> 取消映射：
>
> ​	BOOL UnmapViewOfFile(LPCVOID lpBaseAddress);
>
> 关闭文件/映射区：
>
> ​	BOOL CloseHandle(HANDLE hObject);

```cpp
// Windows内存映射方式读写文件示例：
#include <iostream>
#include <Windows.h>

using namespace std;

bool OpenFileAndMappingFile(LPCSTR strFileName, LPVOID* ppData, PDWORD pDataSize)
{
	bool bRet = false;

	HANDLE hFile = CreateFile(strFileName, GENERIC_READ | GENERIC_WRITE, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
	if (hFile == INVALID_HANDLE_VALUE)
	{
		cout << "打开文件失败！" << endl;
		CloseHandle(hFile);
		hFile = NULL;
		return bRet;
	}

	HANDLE hFileMap = CreateFileMapping(hFile, NULL, PAGE_READWRITE, 0, 0, 0);
	if (hFileMap == INVALID_HANDLE_VALUE)
	{
		cout << "创建内存映射区失败！" << endl;
		CloseHandle(hFileMap);
		hFileMap = NULL;
		return bRet;
	}

	if (hFileMap > 0)
	{
		*ppData = MapViewOfFile(hFileMap, FILE_MAP_ALL_ACCESS, 0, 0, 0);
		if (*ppData != NULL)
		{
			*pDataSize = GetFileSize(hFile, NULL);
			bRet = true;
		}
	}

	if (hFile != NULL)
	{
		CloseHandle(hFile);
		hFile = NULL;
	}

	if (hFileMap != NULL)
	{
		CloseHandle(hFileMap);
		hFileMap = NULL;
	}

	return bRet;
}

int main() 
{
	LPCSTR szFileName = "TestFile.png";
	LPVOID pData = NULL;
	DWORD dwSize = 0;

	if (!OpenFileAndMappingFile(szFileName, &pData, &dwSize))
	{
		cout << "打开文件，创建内存映射失败！" << endl;
		return 0;
	}
	//cout << dwSize << endl;

	cout << "前64字节：" << endl;

	for (int i = 0; i < 64; ++i)
	{
		if (i && i % 16 == 0)
		{
			printf("\n");
		}
		printf("%02X ", ((unsigned char*)(pData))[i]);
	}

	cout << endl << "后64字节：" << endl;

	auto pData2 = (unsigned char*)pData + dwSize - 64;
	for (int i = 0; i < 64; ++i)
	{
		if (i && i % 16 == 0)
		{
			printf("\n");
		}
		printf("%02X ", pData2[i]);
	}
	
	return 0;
}

```



### Linux内存映射文件

> 打开文件：
>
>   int open(const char *pathname, int flags);
>
>   int open(const char *pathname, int flags, mode_t mode);
>
> 映射文件：
>
> ​	void *mmap(void *addr, size_t length, int prot, int flags,int fd, off_t offset);
>
> 同步映射修改：
>
> ​	进程在映射空间的对共享内容的改变并不直接写回到磁盘文件中，往往在调用[munmap](https://baike.baidu.com/item/munmap)()后才执行该操作。可以通过调用msync()函数来实现磁盘文件内容与共享内存区中的内容一致,即同步操作.
>
> ​	int msync(void *addr, size_t length, int flags);
>
> 关闭映射：
>
> ​	int munmap(void *addr, size_t length);
>
> 关闭文件：
>
> ​	int close(int fd);

```cpp
// Linux内存映射方式读写文件示例：
#include<stdio.h>
#include<stdlib.h>
#include<sys/mman.h>
#include<unistd.h>
#include<fcntl.h>
#define NumReconds 100
typedef struct
{
    int iNum;
    char sName[24];
} Recond;
int main(void)
{
    Recond recond,*mapped;
    int i,f;
    FILE *fp;
                                                                                                                                                         
    fp=fopen("recond.dat","w+");
    for( i=0; i < NumReconds; i++)
    {
        recond.iNum = i;
        sprintf(recond.sName,"Recond-%d\n",i);
        fwrite(&recond,sizeof(Recond),1,fp);
    }  
                                                                                                                                                         
    fclose(fp);
        //使用传统方式修改文件内容
    fp = fopen("recond.dat","r+");
    //获得要修改文件的位置
        fseek(fp,43*sizeof(recond),SEEK_SET);
    fread(&recond,sizeof(recond),1,fp);
    recond.iNum = 143;
    sprintf(recond.sName,"Recond-%d",recond.iNum);
    fwrite(&recond,sizeof(recond),1,fp);
    fclose(fp);
        //使用内存映射的方式打开文件，修改文件内存
    //注意这里是open打开不是fopen!!!!   
    f = open("recond.dat",O_RDWR);
        //获得磁盘文件的内存映射
    mapped = (Recond *) mmap(0 , NumReconds * sizeof(Recond) , PROT_READ|PROT_WRITE, MAP_SHARED, f, 0);
    mapped[43].iNum = 999;
        sprintf(mapped[43].sName,"Recond-%d",mapped[43].iNum);
        //将修改同步到磁盘中
    msync((void *)mapped,NumReconds*sizeof(recond),MS_ASYNC);
        //关闭内存映射
    munmap((void *)mapped,NumReconds*sizeof(recond));
    close(f);
    return 0;
}
```

