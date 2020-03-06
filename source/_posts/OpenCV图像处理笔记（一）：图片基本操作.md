---
title: OpenCV图像处理笔记（一）：图片基本操作
date: 2019-07-08 14:01:45
tags:
- C++
- OpenCV
categories:
- 计算机视觉
cover: https://file.buildworld.cn/img/opencv.png

---

#### 一、基本介绍
##### 1、简介
- OpenCV是计算机视觉开源库，主要算法涉及图像处理和机器学习相关方法。
- 是Intel公司贡献出来的，俄罗斯工程师贡献大部分C/C++带代码。
- 在多数图像处理相关的应用程序中被采用,BSD许可，可以免费应用在商业和研究领域
- 最新版本是OpenCV 3.1.0，当前SDK支持语言包括了Java、Python、IOS和Android版本。
- 官方主页： http://opencv.org/opencv-3-1.html
- 其它Matlab、Halcon
  ![image](http://myfile.buildworld.cn/opencv.png)

##### 2、核心模块
- HighGUI部分
- Image Process
- 2D Feature
- Camera Calibration and 3D reconstruction
- Video Analysis
- Object Detection
- Machine Learning
- GPU加速

##### 3、安装（vs2015环境 && openCV 3.x）
[点击博客地址](https://blog.csdn.net/duwangthefirst/article/details/79452314)

==如果有报无法找到opencv_world343.dll的Error，请把C:\opencv\build\x64\vc14\bin下的opencv_world343.dll文件复制到C:\Windows 目录下即可==


#### 二、图像处理
##### 1、加载、修改、保存图像

###### 加载图像（用cv::imread）

> imread功能是加载图像文件成为一个Mat对象，其中第一个参数表示图像文件名称

> 第二个参数，表示加载的图像是什么类型，支持常见的三个参数值

- IMREAD_UNCHANGED (<0) 表示加载原图，不做任何改变
- IMREAD_GRAYSCALE ( 0)表示把原图作为灰度图像加载进来
- IMREAD_COLOR (>0) 表示把原图作为RGB图像加载进来

==注意：== OpenCV支持JPG、PNG、TIFF等常见格式图像文件加载

###### 显示图像 (cv::namedWindos 与cv::imshow)
- namedWindos功能是创建一个OpenCV窗口，它是由OpenCV自动创建与释放，你无需取销毁它。

- 常见用法namedWindow("Window Title", WINDOW_AUTOSIZE)

- WINDOW_AUTOSIZE会自动根据图像大小，显示窗口大小，不能人为改变窗口大小

- WINDOW_NORMAL,跟QT集成的时候会使用，允许修改窗口大小。

- imshow根据窗口名称显示图像到指定的窗口上去，第一个参数是窗口名称，第二参数是Mat对象

###### 修改图像 (cv::cvtColor)
- cvtColor的功能是把图像从一个彩色空间转换到另外一个色彩空间，有三个参数，第一个参数表示源图像、第二参数表示色彩空间转换之后的图像、第三个参数表示源和目标色彩空间如：COLOR_BGR2HLS 、COLOR_BGR2GRAY 等
- cvtColor( image, gray_image, COLOR_BGR2GRAY );

###### 保存图像(cv::imwrite)
- 保存图像文件到指定目录路径
- 只有8位、16位的PNG、JPG、Tiff文件格式而且是单通道或者三通道的BGR的图像才可以通过这种方式保存
- 保存PNG格式的时候可以保存透明通道的图片
- 可以指定压缩参数

##### 2、矩阵的掩膜操作
![image](http://myfile.buildworld.cn/矩阵掩膜.png)
######  获取图像像素指针
- CV_Assert(myImage.depth() == CV_8U); 
- Mat.ptr<uchar>(int i=0) 获取像素矩阵的指针，索引i表示第几行，从0开始计行数。
- 获得当前行指针const uchar*  current= myImage.ptr<uchar>(row );
- 获取当前像素点P(row, col)的像素值 p(row, col) =current[col]

###### 像素范围处理saturate_cast<uchar>
- saturate_cast<uchar>（-100），返回 0。
- saturate_cast<uchar>（288），返回255
- saturate_cast<uchar>（100），返回100
- 这个函数的功能是确保RGB值得范围在0~255之间

```C++
#include<opencv2/opencv.hpp>
#include<iostream>
#include<math.h>
using namespace cv;

int main(int argc, char ** argv) {
	Mat src, dst;
	src = imread("C:\\Users\\15646\\Pictures\\雷军.jpg");

	if (!src.data)
	{
		printf("no image\n");
		return -1;
	}
	namedWindow("input img", CV_WINDOW_AUTOSIZE);
	imshow("input img", src);
	
	int cols = (src.cols-1)* src.channels();
	int offsetx = src.channels();
	int rows = src.rows;
	dst = Mat(src.size(), src.type());
	for (int row = 1; row < rows-1; row++)
	{
		const uchar* current = src.ptr<uchar>(row);
		const uchar* previous = src.ptr<uchar>(row - 1);
		const uchar* next = src.ptr<uchar>(row);
		uchar* output = dst.ptr<uchar>(row);
		for (int col = offsetx; col<cols;col++)
		{
			output[col] = saturate_cast<uchar>(5 * current[col] - (current[col - offsetx] +
				current[col + offsetx] + previous[col] + next[col]));
		}
	}

	namedWindow("contrast img ", CV_WINDOW_AUTOSIZE);
	imshow("contrast img ", dst);
	waitKey(0);
	return 0;
}
```
###### 函数调用filter2D功能
- 定义掩膜：Mat kernel = (Mat_<char>(3,3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);
- filter2D( src, dst, src.depth(), kernel );其中src与dst是Mat类型变量、src.depth表示位图深度，有32、24、8等。

```C++
    Mat kernel = (Mat_<char>(3, 3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);
	filter2D(src,dst,src.depth(),kernel);
```
##### 3、Mat对象
###### Mat对象与IplImage对象
- Mat对象OpenCV2.0之后引进的图像数据结构、自动分配内存、不存在内存泄漏的问题，是面向对象的数据结构。分了两个部分，头部与数据部分
- IplImage是从2001年OpenCV发布之后就一直存在，是C语言风格的数据结构，需要开发者自己分配与管理内存，对大的程序使用它容易导致内存泄漏问题

==常用方法：==


```C++
void copyTo(Mat mat)    克隆
void convertTo(Mat dst, int type) 
Mat clone() 克隆
int channels()  获取通道
int depth()
bool empty();
uchar* ptr(i=0)  获取指针
```

###### 复制
- 部分复制：一般情况下只会复制Mat对象的头和指针部分，不会复制数据部分

```C++
    Mat A= imread(imgFilePath);
    Mat B(A)  // 只复制
```

- 完全复制：如果想把Mat对象的头部和数据部分一起复制，可以通过如下两个API实现

```C++
    Mat F = A.clone(); 或 Mat G; A.copyTo(G);
```
###### 四个要点
- 输出图像的内存是自动分配的
- 使用OpenCV的C++接口，不需要考虑内存分配问题
- 赋值操作和拷贝构造函数只会复制头部分
- 使用clone与copyTo两个函数实现数

###### Mat对象的创建

```C++
cv::Mat::Mat构造函数
	Mat M(2,2,CV_8UC3, Scalar(0,0,255))
	其中前两个参数分别表示行(row)跟列(column)、第三个CV_8UC3中的8表示每个通道占8位、U表示无符号、C表示Char类型、3表示通道数目是3，
	第四个参数是向量表示初始化每个像素值是多少，向量长度对应通道数目一致

创建多维数组cv::Mat::create
	 int sz[3] = {2,2,2};     
	Mat  L(3,sz, CV_8UC1, Scalar::all(0));
```

###### 定义小数组

```C++
Mat C = (Mat_<double>(3,3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);     
```
###### MATLAB风格写法

```C++
    Mat m2;
	m2 = Mat::zeros(2, 2, CV_8UC1);
	imshow("demo2", m2);
```
##### 4、图像像素操作
###### 读写像素
- 读一个GRAY像素点的像素值（CV_8UC1）

```C++
Scalar intensity = img.at<uchar>(y, x); 
或者 Scalar intensity = img.at<uchar>(Point(x, y));
```


- 读一个RGB像素点的像素值

```C++
Vec3f intensity = img.at<Vec3f>(y, x); 
float blue = intensity.val[0]; 
float green = intensity.val[1]; 
float red = intensity.val[2];
```

###### 修改像素值

- 灰度图像

```C++
img.at<uchar>(y, x) = 128;
```
- RGB三通道图像
```C++
img.at<Vec3b>(y,x)[0]=128; // blue
img.at<Vec3b>(y,x)[1]=128; // green
img.at<Vec3b>(y,x)[2]=128; // red
```
- 空白图像赋值
```C++
img = Scalar(0);
```
- ROI选择
```C++
Rect r(10, 10, 100, 100); 
Mat smallImg = img(r);
```

###### 示例代码
```C++
int main(int argc, char ** argo) {

	Mat src, gray_src;
	src = imread("C:\\Users\\Administrator\\Pictures\\girl.jpg");
	if (src.empty())
	{
		cout <<" read img error!"<< endl ;
	}

	namedWindow("src img", CV_WINDOW_AUTOSIZE);
	imshow("src img", src);

	cvtColor(src, gray_src, CV_BGR2GRAY);
	//namedWindow("gray_src img", CV_WINDOW_AUTOSIZE);
	//imshow("gray_src img", gray_src);


	int height = gray_src.rows;
	int width = gray_src.cols;

	for (int row =0;row<height;row++)
	{
		for (int col = 0;  col< width;col ++)
		{
			//获取像素值
			int gray = gray_src.at<uchar>(row, col);
			gray_src.at<uchar>(row, col) = 255 - gray;
		}
	}

	namedWindow("gray_src_change img", CV_WINDOW_AUTOSIZE);
	imshow("gray_src_change img", gray_src);

	//Mat操作
	Mat dst;
	dst.create(src.size(), src.type());
	height = src.rows;
	width = src.cols;
	//获取图片通道值
	int nc = src.channels();

	/*for (int row = 0; row < height; row++)
	{
		for (int col = 0; col < width; col++)
		{
			if (nc ==1)
			{
				int gray = gray_src.at<uchar>(row, col);
				gray_src.at<uchar>(row, col) = 255 - gray;
			}
			else if(nc == 3)
			{
				int b = src.at<Vec3b>(row, col)[0];
				int g = src.at<Vec3b>(row, col)[1];
				int r = src.at<Vec3b>(row, col)[2];
				dst.at<Vec3b>(row, col)[0] = 255 - b;
				dst.at<Vec3b>(row, col)[1] = 255 - g;
				dst.at<Vec3b>(row, col)[2] = 255 - r; 
			}
		}
	}*/

	//上面的转换代码可以替换,效果相同
	bitwise_not(src, dst);
	imshow("gray_src_change_by_mat img", dst);

	cout << "enter anything" << endl;
	waitKey(0);
	return 0;
}
```

##### 5、图像混合
###### 理论-线性混合操作
<a href="https://www.codecogs.com/eqnedit.php?latex=g\left&space;(&space;x&space;\right&space;)=\left&space;(&space;1-\alpha&space;\right&space;)f_{0}\left&space;(&space;x&space;\right&space;)&plus;\alpha&space;f_{1}\left&space;(&space;x&space;\right&space;)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?g\left&space;(&space;x&space;\right&space;)=\left&space;(&space;1-\alpha&space;\right&space;)f_{0}\left&space;(&space;x&space;\right&space;)&plus;\alpha&space;f_{1}\left&space;(&space;x&space;\right&space;)" title="g\left ( x \right )=\left ( 1-\alpha \right )f_{0}\left ( x \right )+\alpha f_{1}\left ( x \right )" /></a>
> 其中a的取值范围为0~1之间
###### 相关API (addWeighted)

```C++
void addWeighted(InputArray src1, 
                    double alpha, 
                    InputArray src2,
                    double beta, 
                    double gamma, 
                    OutputArray dst, 
                    int dtype = -1);

参数1：输入图像Mat – src1
参数2：输入图像src1的alpha值
参数3：输入图像Mat – src2
参数4：输入图像src2的alpha值
参数5：gamma值
参数6：输出混合图像

注意点：两张图像的大小和类型必须一致才可以

```
###### 示例代码
```C++
//设置权重
	double alpha = 0.5;
	if (src1.rows==src2.rows && src1.cols==src2.cols && src1.type() == src2.type())
	{
		addWeighted(src1, alpha, src2, (1.0 - alpha), 0.0, dst);
		//multiply(src1, src2, dst, 1.0); 图像相乘
		imshow("dst", dst);
	}

```

##### 6、调整图像亮度与对比度


- 图像变换可以看作如下：
    - 像素变换 – 点操作
    - 邻域操作 – 区域
- 调整图像亮度和对比度属于像素变换-点操作

<a href="https://www.codecogs.com/eqnedit.php?latex=g\left&space;(&space;i,j\right&space;)=\alpha&space;f\left&space;(&space;i,j&space;\right&space;)&plus;\beta" target="_blank"><img src="https://latex.codecogs.com/gif.latex?g\left&space;(&space;i,j\right&space;)=\alpha&space;f\left&space;(&space;i,j&space;\right&space;)&plus;\beta" title="g\left ( i,j\right )=\alpha f\left ( i,j \right )+\beta" /></a>

###### 重要的API

```C++
Mat new_image = Mat::zeros( image.size(), image.type() );  创建一张跟原图像大小和类型一致的空白图像、像素值初始化为0

saturate_cast<uchar>(value)确保值大小范围为0~255之间

Mat.at<Vec3b>(y,x)[index]=value 给每个像素点每个通道赋值
```
###### 示例代码

```C++
int height = src1.rows; //高度
	int width = src1.cols; //宽度
	double alpha = 0.8; 
	int beta = 30;

	//生成一个空的大小和src1一样大的图
	output = Mat::zeros(src1.size(), src1.type());
	for (int row = 0; row < height; row++)
	{
		for (int col = 0; col < width; col++)
		{
			if (src1.channels()==3)
			{
				//像素点变换，换值,达到调整亮度和对比度的效果
				float b = src1.at<Vec3b>(row, col)[0]; //blue
				float g = src1.at<Vec3b>(row, col)[1]; //green
				float r = src1.at<Vec3b>(row, col)[2]; //red

				output.at<Vec3b>(row, col)[0] = saturate_cast<uchar>(alpha*b + beta);
				output.at<Vec3b>(row, col)[1] = saturate_cast<uchar>(alpha*g + beta);
				output.at<Vec3b>(row, col)[2] = saturate_cast<uchar>(alpha*r + beta);
			}
			else if(src1.channels()==1)
			{
				output.at<uchar>(row, col) = saturate_cast<uchar>(alpha*src1.at<uchar>(row, col)+ beta);
			}
			
		}
	}
    
```
##### 7、绘制形状与文字
###### 使用cv::Point与cv::Scalar

```C++
Point表示2D平面上一个点x,y
	Point p;
	p.x = 10;
	p.y = 8;
 	or
	p = Pont(10,8);
	
Scalar表示四个元素的向量
	Scalar(a, b, c);// a = blue, b = green, c = red表示RGB三个通道
```
###### 绘制线、矩形、园、椭圆等基本几何形状

```
画线 cv::line （LINE_4\LINE_8\LINE_AA）
画椭圆cv::ellipse
画矩形cv::rectangle
画圆cv::circle
画填充cv::fillPoly

```
==示例代码==

```C++
#include<opencv2\opencv.hpp>
#include<iostream>
 
using namespace std;
using namespace cv;
Mat bgImage;
void Myline();
void MyRectangle();
void MyEllipse();
void MyCircle();
void MyPolygon();

int main() {

	bgImage = imread("C:\\Users\\Administrator\\Pictures\\girl2.jpg");
	if (!bgImage.data)
	{
		return -1;
	}

	Myline();
	MyRectangle();
	MyEllipse();
	MyCircle();
	MyPolygon();

	imshow("bgImage", bgImage);

	waitKey(0);
	return 0;
}
//画线
void Myline() {
	Point p1 = Point(20, 30);
	Point p2;

	p2.x = 300;
	p2.y = 300;

	//设置颜色
	Scalar color = Scalar(0, 0, 255);
	line(bgImage, p1, p2, color, 1, LINE_8);
}

//画矩形
void MyRectangle() {
	Rect rect = Rect(150, 500, 300, 300);
	Scalar color = Scalar(255, 0, 0);
	rectangle(bgImage, rect, color, 2, LINE_8);
}

//绘制椭圆
void MyEllipse() {
	Scalar color = Scalar(0, 255, 0);
	ellipse(bgImage, 
		Point(bgImage.cols / 2, bgImage.rows / 2), 
		Size(bgImage.cols / 4, bgImage.rows / 8),
		90,0,360,color,2,LINE_8);
}

//绘制圆
void MyCircle() {
	Scalar color = Scalar(0, 255, 0);
	Point center = Point(bgImage.cols / 2, bgImage.rows / 2);
	circle(bgImage, center, 150, color, 2, LINE_8);
}

//绘制多边形
void MyPolygon() {
	Point pts[1][5];
	pts[0][0] = Point(100, 100);
	pts[0][1] = Point(100, 200);
	pts[0][2] = Point(200, 200);
	pts[0][3] = Point(200, 100);
	pts[0][4] = Point(100, 100);

	const Point* ppts[] = {pts[0]};
	int npt[] = { 5 };
	Scalar color = Scalar(255, 140, 0);

	fillPoly(bgImage, ppts, npt, 1, color, LINE_8);
}
```

###### 随机数生成cv::RNG

```
生成高斯随机数gaussian (double sigma)
生成正态分布随机数uniform (int a, int b)
```
==随机画线代码==

```c++
void RandomLineDemo() {
	RNG rng(12345);
	Point pt1, pt2;
	Mat new_img = Mat::zeros(bgImage.size(), bgImage.type());
	for (int i = 0; i < 100000;i++)
	{
		//随机生成的两个端点
		pt1.x = rng.uniform(0, bgImage.cols);
		pt2.x = rng.uniform(0, bgImage.cols);
		pt1.y = rng.uniform(0, bgImage.rows);
		pt2.y = rng.uniform(0, bgImage.rows);

		int num = rng.uniform(0, 255);
		line(new_img, pt1, pt2, CV_RGB(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255)), 1, LINE_8);

		//延迟50ms
		if (waitKey(50)>0)
		{
			break;
		}
		imshow("随机生成图片", new_img);
	}	
}
```


###### 绘制添加文字

```c++
putText函数 中设置fontFace(cv::HersheyFonts), 

putText(bgImage, "Hello World", Point(300, 300), CV_FONT_BLACK, 1.0, CV_RGB(255, 69, 0), 1, LINE_8);

```