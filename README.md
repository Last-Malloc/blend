***以最简单的方式实现几个C语言课程设计的常见题目，适合大一或者刚学习C语言的同学学习参考。使用Code::Blocks编译器创建的纯C项目，将其中的源码粘贴进其他编译器或C++项目也可直接运行。因为部分同学没有学习过数据结构，所以尽量使用传统的数组进行存储，规避没有学习过的知识点，但鼓励大家自己改进。为了使得程序更加简单方便阅读，基本上没有进行对用户输入的容错，可以自己添加。***

Code::Blocks安装和使用 https://blog.csdn.net/qq_42283621/article/details/124055391?spm=1001.2014.3001.5501

项目源码会在文章末尾给出，因为里边有两个bitmap图可能大家会用到，所以项目同步放置在github	https://github.com/Last-Malloc/blend

[TOC]

# 需求分析

功能非常简单，混合两个bitmap图片的像素为一个bitmap图片，通过指令来实现，指令格式为

blend src1.bmp 80% src2.bmp dst.bmp 

blend：为可执行程序名称；
src1.bmp src2.bmp： 原始图像文件名；
80%：混合百分比，即取得src1.mbp像素的80%和src2.bmp像素的20%
dst.bmp： 生成的新的文件的文件名
**除了blend外，其他的参数都可以进行修改**

先看一下效果

![image-20220419162413152](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419162413152.png)

![image-20220419162419986](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419162419986.png)

![image-20220419162428402](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419162428402.png)

# 基础讲解

## 可执行文件exe

我们用code::blocks创建一个C项目名为blend，点击构建运行后，你可以看到 blend/bin/Debug目录下有一个blend.exe，这是一个和项目同名的可执行程序。

![image-20220419220238050](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419220238050.png)

![image-20220419220050474](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419220050474.png)

在这个界面上，按住shift键然后点击鼠标右键，选择“在此处运行PowerShell窗口”

![image-20220419220153620](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419220153620.png)

输入 ./blend.exe即可运行这个程序，即除了在编译器里点击运行按钮来运行程序，还可以使用命令行来运行程序。

![image-20220419220335937](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419220335937.png)

***十分注意的是，如果你用Code::Blocks里面的运行按钮（绿色三角），那么根目录为main.cpp所在的地方；但如果你用上面的命令行方式运行，那么根目录为blend.exe所在的地方，也就是说，你应该把两个原始图片放在blend.exe的旁边，然后生成的新的图片也会出现在blend.exe的旁边***

![image-20220420102345696](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420102345696.png)

## main函数带参数

上面我们看到的main.c主函数为int main()，这是我们最常见的类型。实际上main函数可以带参数，即int main(int argc, char* argv[])，argc是参数数量，argv是argc个参数。我们可以对这argc个参数进行输出。

可以看到，它输出了一个参数，是这个blend.exe的完整路径。你从Code::blocks等编译器里点击构建运行代码，默认有一个参数为可执行文件的路径。

![image-20220419220816756](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419220816756.png)

用之前的方式，打开Powershell窗口，然后

输入.\blend.exe，输出blend.exe的完整路径，即只有一个默认的参数blend.exe的完整路径
输入.\blend.exe aaa bbb ccc ddd，输出5个参数，除了默认的参数外，还有我们输入的4个参数即aaa bbb ccc ddd

![image-20220419221127796](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220419221127796.png)

## Bitmap文件

https://blog.csdn.net/swordjun/article/details/108667926
https://blog.csdn.net/weixin_41336592/article/details/109710440

### BMP文件构成

有4个部分：
	 位图文件头(bitmap-file header)	14字节
	 位图信息头(bitmap-informationheader)	40字节
	 颜色表(color table)		**24位真彩图没有该项**
	 颜色点阵数据(bits data)	

**本文只讲解24位真彩图的使用**

位图文件头：
![image-20220420091032019](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420091032019.png)

位图信息头：
![image-20220420091101707](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420091101707.png)

可以定义两个结构体，对文件头和信息头进行读取。对于24位真彩图，前54位（0-53）是文件信息（如上两图所示），从第54字节（下标从0开始数）开始，是颜色点阵数据。颜色点阵数据的格式为，BGR，行从左向右，列从下向上，这样说可能比较模糊，我们可以看一下下面的例子。例如，4\*4像素的真彩图，其像素点个数应该为4\*4\*3=48位，图1是我们直观认为的图片格式，图2是它实际上在文件中存储的格式。
![image-20220420100039828](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420100039828.png)

```c
//位图文件头
struct bfHead
{
    char bfType[2]; //检查读入的文件是否是"BM"类型
	long bfSize;
	long bfReserved;
	long bfOffBits;
};

//位图信息头
struct biHead
{
	long biSize;
	long biWidth;   //图的宽度
	long biHeight;  //图的高度
	short biPlanes;
	short biBitCount;   //像素点位深度，24则为24位真彩图
	long biCompression;
	long biSizeImage;
	long biXPelsPerMeter;
	long biYpelsPerMeter;
	long biClrUsed;
	long biClrImportant;
};
```

**在后面用到的位图文件头和位图信息头中有用的字段即在上面的代码中有注释的字段**

# 功能实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

//存放图片的颜色点阵数据RGB
char src1[1000][1000][3];
char src2[1000][1000][3];
char dst[1000][1000][3];

//位图文件头
struct bfHead
{
    char bfType[2]; //检查读入的文件是否是"BM"类型
	long bfSize;
	long bfReserved;
	long bfOffBits;
};

//位图信息头
struct biHead
{
	long biSize;
	long biWidth;   //图的宽度
	long biHeight;  //图的高度
	short biPlanes;
	short biBitCount;   //像素点位深度，24则为24位真彩图
	long biCompression;
	long biSizeImage;
	long biXPelsPerMeter;
	long biYpelsPerMeter;
	long biClrUsed;
	long biClrImportant;
};

int main(int argc, char* argv[])
{
    struct bfHead bfSrc1, bfSrc2;
    struct biHead biSrc1, biSrc2;

    //打开2个原始文件 供读取数据
    FILE *fp1 = fopen(argv[1], "r+b");
    FILE *fp2 = fopen(argv[3], "r+b");

    //混合的百分比，例如输入 80% 则percent为0.8
    double percent = 0;
    for (int i = 0; i < strlen(argv[2]) - 1; ++i)
        percent = percent * 10 + (argv[2][i] - '0');
    percent /= 100;

    //打开目标文件 供写入数据
    //这个文件可能是没有的，fopen会创建一个新的文件；如果该文件已经存在，fopen会将其覆盖掉
    FILE *fp3 = fopen(argv[4], "w+b");

    //读取位图文件头 位图信息头
    fread(&bfSrc1, 14, 1, fp1);
    fread(&bfSrc2, 14, 1, fp2);
    fread(&biSrc1, 40, 1, fp1);
    fread(&biSrc2, 40, 1, fp2);

    //检查是否为BM类型 是否为24位真彩图 照片尺寸是否相等
    if (bfSrc1.bfType[0]=='B'&& bfSrc1.bfType[1]=='M' && bfSrc2.bfType[0]=='B' && bfSrc2.bfType[1]=='M'
        && biSrc1.biBitCount == 24 && biSrc2.biBitCount == 24
        && biSrc1.biWidth == biSrc2.biWidth && biSrc1.biHeight == biSrc2.biHeight)
    {
        //读取颜色点阵数据
        //遍历列 从下向上
        for (int y = biSrc1.biHeight - 1; y >= 0; --y)
        {
            //遍历行 从前向后
            for (int x = 0; x < biSrc1.biWidth; ++x)
            {
                //文件数据流中按照BGR的顺序存储，我们以RGB的顺序存储
                for (int k = 2; k >= 0; --k)
                {
                    fread(&src1[x][y][k], 1, 1, fp1);
                    fread(&src2[x][y][k], 1, 1, fp2);
                }
            }
        }

        //混合像素到目标文件数据
        for (int y = biSrc1.biHeight - 1; y >= 0; --y)
        {
            for (int x = 0; x<biSrc1.biWidth; ++x)
            {
                for (int k = 2; k >= 0; --k)
                    dst[x][y][k] = src1[x][y][k] * percent + src2[x][y][k] * (1 - percent);
            }
        }

        //写目标文件
        fwrite(&bfSrc1,14,1,fp3);
        fwrite(&biSrc1,40,1,fp3);
        for (int y = biSrc1.biHeight - 1; y >= 0; --y)
        {
            for (int x = 0; x<biSrc1.biWidth; ++x)
            {
                for (int k = 2; k >= 0; --k)
                    fwrite(&dst[x][y][k], 1, 1, fp3);
            }
        }

        printf("执行成功\n");
    }

    //关闭文件
    fclose(fp1);
    fclose(fp2);
    fclose(fp3);

    return 0;
}
```

# 总体效果

保存之后，点击构建并运行，**发现返回值错误，这是正常的，不用管**

![image-20220420102639377](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420102639377.png)

blend/bin/Debug目录下，按住shift键然后点击鼠标右键，选择“在此处运行PowerShell窗口”，然后向下面这样输入：

![image-20220420102918653](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420102918653.png)

指向之前和执行之后，文件夹下的文件如下：

![image-20220420102821564](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420102821564.png)

![image-20220420102828269](https://image-bad-for-malloc.oss-cn-qingdao.aliyuncs.com/img/image-20220420102828269.png)

