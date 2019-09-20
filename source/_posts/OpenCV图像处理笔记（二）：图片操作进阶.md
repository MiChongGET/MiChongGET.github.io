---
title: OpenCV图像处理笔记（二）：图片操作进阶
date: 2019-07-11 16:12:22
tags:
- C++
- OpenCV
categories:
- 计算机视觉
---
#### 一、图像模糊
##### 1、模糊原理
- Smooth/Blur 是图像处理中最简单和常用的操作之一
- 使用该操作的原因之一就为了给图像预处理时候减低噪声
- 使用Smooth/Blur操作其背后是数学的卷积计算

<html>
<a href="https://www.codecogs.com/eqnedit.php?latex=g\left&space;(&space;i,j&space;\right&space;)=\sum&space;_{k,j}f\left&space;(&space;i&plus;k,j&plus;l&space;\right&space;)h\left&space;(&space;k,l&space;\right&space;)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?g\left&space;(&space;i,j&space;\right&space;)=\sum&space;_{k,j}f\left&space;(&space;i&plus;k,j&plus;l&space;\right&space;)h\left&space;(&space;k,l&space;\right&space;)" title="g\left ( i,j \right )=\sum _{k,j}f\left ( i+k,j+l \right )h\left ( k,l \right )" /></a>
</html>


- 通常这些卷积算子计算都是线性操作，所以又叫线性滤波

###### 举例

```
假设有6x6的图像像素点矩阵。

卷积过程：6x6上面是个3x3的窗口，从左向右，从上向下移动，黄色的每个像个像素点值之和取平均值赋给中心红色像素作为它卷积处理之后新的像素值。每次移动一个像素格。
```

![像素点矩阵](http://myfile.buildworld.cn/像素点矩阵.png)


##### 2、归一化盒子滤波（均值滤波）
<a href="https://www.codecogs.com/eqnedit.php?latex=\dpi{100}&space;\LARGE&space;K=\frac{1}{K_{width}&space;.K_{height}}\begin{bmatrix}&space;1&space;&&space;1&&space;1&&space;...&&space;1\\&space;1&space;&&space;1&&space;1&&space;...&&space;1\\&space;.&&space;.&&space;.&&space;...&&space;1\\&space;.&&space;.&&space;.&&space;...&&space;1\\&space;1&space;&&space;1&&space;1&&space;...&&space;1&space;\end{bmatrix}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\dpi{100}&space;\LARGE&space;K=\frac{1}{K_{width}&space;.K_{height}}\begin{bmatrix}&space;1&space;&&space;1&&space;1&&space;...&&space;1\\&space;1&space;&&space;1&&space;1&&space;...&&space;1\\&space;.&&space;.&&space;.&&space;...&&space;1\\&space;.&&space;.&&space;.&&space;...&&space;1\\&space;1&space;&&space;1&&space;1&&space;...&&space;1&space;\end{bmatrix}" title="\LARGE K=\frac{1}{K_{width} .K_{height}}\begin{bmatrix} 1 & 1& 1& ...& 1\\ 1 & 1& 1& ...& 1\\ .& .& .& ...& 1\\ .& .& .& ...& 1\\ 1 & 1& 1& ...& 1 \end{bmatrix}" /></a>

###### 相关的API

```
- blur(Mat src, Mat dst, Size(xradius, yradius), Point(-1,-1));
```


##### 3、高斯滤波
<html>
<a  href="https://www.codecogs.com/eqnedit.php?latex=\dpi{100}&space;\LARGE&space;G_{0}\left&space;(&space;x,y&space;\right&space;)=Ae^{\frac{-(x-\mu&space;_{x})^{2}}{2\sigma&space;_{x}^{2}}&plus;\frac{-(y-\mu&space;_{y})^{2}}{2\sigma&space;_{y}^{2}}}" target="_blank"><img src="https://latex.codecogs.com/png.latex?\dpi{100}&space;\LARGE&space;G_{0}\left&space;(&space;x,y&space;\right&space;)=Ae^{\frac{-(x-\mu&space;_{x})^{2}}{2\sigma&space;_{x}^{2}}&plus;\frac{-(y-\mu&space;_{y})^{2}}{2\sigma&space;_{y}^{2}}}" title="\LARGE G_{0}\left ( x,y \right )=Ae^{\frac{-(x-\mu _{x})^{2}}{2\sigma _{x}^{2}}+\frac{-(y-\mu _{y})^{2}}{2\sigma _{y}^{2}}}" /></a>
</html>

###### 相关的API

```
- GaussianBlur(Mat src, Mat dst, Size(11, 11), sigmax, sigmay);
其中Size（x, y）, x, y 必须是正数而且是奇数
```

##### 4、中值滤波
- 统计排序滤波器
- 中值对椒盐噪声有很好的抑制作用

![中值滤波原理图](http://myfile.buildworld.cn/中值滤波.png)
###### 相关的API
```C++
中值模糊 medianBlur（Mat src, Mat dest, ksize）
中值模糊的ksize大小必须是大于1而且必须是奇数。

```

##### 5、双边滤波
- 均值模糊无法克服边缘像素信息丢失缺陷。原因是均值滤波是基于平均权重
- 高斯模糊部分克服了该缺陷，但是无法完全避免，因为没有考虑像素值的不同
- 高斯双边模糊 – 是边缘保留的滤波方法，避免了边缘信息丢失，保留了图像轮廓不变
###### 相关的API

```C++
双边模糊 bilateralFilter(src, dest, d=15, 150, 3);

	 - 15 –计算的半径，半径之内的像数都会被纳入计算，如果提供-1 则根据sigma space参数取值
	 - 150 – sigma color 决定多少差值之内的像素会被计算
 	 - 3 – sigma space 如果d的值大于0则声明无效，否则根据它来计算d值
```

#### 二、膨胀与腐蚀

##### 1、形态学操作简介

- 图像形态学操作 – 基于形状的一系列图像处理操作的合集，主要是基于集合论基础上的形态学数学
- 形态学有四个基本操作：腐蚀、膨胀、开、闭
- 膨胀与腐蚀是图像处理中最常用的形态学操作手段

##### 2、形态学操作(morphology operators)-膨胀
> 跟卷积操作类似，假设有图像A和结构元素B，结构元素B在A上面移动，其中B定义其中心为锚点，
计算B覆盖下A的最大像素值用来替换锚点的像素，其中B作为结构体可以是任意形状
###### 二值图像与灰度图像上的膨胀操作
![膨胀操作](http://myfile.buildworld.cn/膨胀操作.png)

##### 3、形态学操作-腐蚀
> 腐蚀跟膨胀操作的过程类似，唯一不同的是以最小值替换锚点重叠下图像的像素值
###### 二值图像与灰度图像上的膨胀操作
![二值图像与灰度图像上的腐蚀操作](http://myfile.buildworld.cn/腐蚀操作.png)

###### 膨胀腐蚀示例代码
```C++
int element_size = 3;
int max_size = 21;
Mat src, dst, gauss_dst;
void CallBack_Demo(int, void*);

string input_title = "input img";
string output_title = "output img";
string output2gauss_title = "output2gauss img";

int main(int argc, char ** argv) {

	
	src = imread("C:\\Users\\Administrator\\Pictures\\车牌.jpg");

	if (!src.data)
	{
		cout << "read img error" << endl;
		return -1;
	}

	imshow(input_title, src);

	namedWindow(output_title, CV_WINDOW_AUTOSIZE);
	createTrackbar("Element Size", output_title, &element_size, max_size, CallBack_Demo);

	waitKey(0);
	return 0;
}

void CallBack_Demo(int, void*) {
	int s = element_size * 2 + 1;
	
	/*getStructuringElement(int shape, Size ksize, Point anchor)
         - 形状 (MORPH_RECT \MORPH_CROSS \MORPH_ELLIPSE)
         - 大小
         - 锚点 默认是Point(-1, -1)意思就是中心像素*/
	Mat structureElement = getStructuringElement(MORPH_RECT, Size(s, s), Point(-1, -1));
	
	dilate(src, dst, structureElement,Point(-1,-1),1); //膨胀
	//erode(src, dst, structureElement, Point(-1, -1), 1); //腐蚀
	imshow(output_title, dst);
	return;
}
```
###### 动态调整结构元素大小

```
TrackBar – createTrackbar(const String & trackbarname, const String winName,  int* value, int count, Trackbarcallback func, void* userdata=0)
其中最中要的是 callback 函数功能。如果设置为NULL就是说只有值update，但是不会调用callback的函数。
```
##### 4、开操作- open
- 先腐蚀后膨胀
- 可以去掉小的对象，假设对象是前景色，背景是黑色

##### 5、闭操作-close

- 先膨胀后腐蚀（bin2）
- 可以填充小的洞（fill hole），假设对象是前景色，背景是黑色

##### 6、形态学梯度- Morphological Gradient
- 膨胀减去腐蚀
- 又称为基本梯度（其它还包括-内部梯度、方向梯度）

##### 7、顶帽 – top hat
> 顶帽 是原图像与开操作之间的差值图像

##### 8、黑帽
> 黑帽是闭操作图像与源图像的差值图像


##### 相关的API
```
morphologyEx(src, dest, CV_MOP_BLACKHAT, kernel);
 - Mat src – 输入图像
 - Mat dest – 输出结果
 - int OPT – CV_MOP_OPEN/ CV_MOP_CLOSE/ CV_MOP_GRADIENT / CV_MOP_TOPHAT/ CV_MOP_BLACKHAT 形态学操作类型
Mat kernel 结构元素
int Iteration 迭代次数，默认是1

```
#### 三、形态学操作应用-提取水平与垂直线

##### 1、原理方法

> 图像形态学操作时候，可以通过自定义的结构元素实现结构元素
对输入图像一些对象敏感、另外一些对象不敏感，这样就会让敏
感的对象改变而不敏感的对象保留输出。通过使用两个最基本的
形态学操作 – 膨胀与腐蚀，使用不同的结构元素实现对输入图像
的操作、得到想要的结果。

 - 膨胀，输出的像素值是结构元素覆盖下输入图像的最大像素值
 - 腐蚀，输出的像素值是结构元素覆盖下输入图像的最小像素值

##### 2、提取步骤
- 输入图像彩色图像 imread
- 转换为灰度图像 – cvtColor
- 转换为二值图像 – adaptiveThreshold
- 定义结构元素
- 开操作 （腐蚀+膨胀）提取 水平与垂直线

##### 示例代码

```C++
Mat src, dst;
	src = imread("C:\\Users\\15646\\Pictures\\验证码.jpg");
	
	if (!src.data)
	{
		cout << "not found img" << endl;
		waitKey(0);
		return -1;
	}
	//输出原图
	imshow("input img", src);
	//转化为灰度图像
	Mat gray_src;
	cvtColor(src, gray_src, CV_BGR2GRAY);
	imshow("gray img", gray_src);

	//转化为二值图像
	/*adaptiveThreshold(
		Mat src, // 输入的灰度图像
		Mat dest, // 二值图像
		double maxValue, // 二值图像最大值
		int adaptiveMethod // 自适应方法，只能其中之一 – 
						   // ADAPTIVE_THRESH_MEAN_C ， ADAPTIVE_THRESH_GAUSSIAN_C 
		int thresholdType,// 阈值类型
		int blockSize, // 块大小
		double C // 常量C 可以是正数，0，负数
	)*/

	Mat binImg;
	adaptiveThreshold(~gray_src, binImg, 255, ADAPTIVE_THRESH_MEAN_C, THRESH_BINARY, 15, -2);
	imshow("binary img", binImg);

	//定义结构元素
	//水平结构元素
	Mat hline = getStructuringElement(MORPH_RECT, Size(src.cols / 16, 1), Point(-1, -1));
	//垂直结构元素
	Mat vline = getStructuringElement(MORPH_RECT, Size(1, src.cols / 16), Point(-1, -1));

	//开操作 （腐蚀+膨胀）提取 水平线
	//Mat temp;
	//erode(binImg, temp, hline);//腐蚀
	//dilate(temp, dst, hline);//膨胀
	//直接使用开操作的api即可
	morphologyEx(binImg, dst, CV_MOP_OPEN, vline);
	bitwise_not(dst, dst);
	blur(dst, dst, Size(3, 3), Point(-1, -1));

	imshow("res", dst);

	waitKey(0);
	return 0;
```
#### 四、图像上采样和降采样
##### 1、图像金字塔概念
- 1. 我们在图像处理中常常会调整图像大小，最常见的就是放大(zoom in)和缩小（zoom out），尽管几何变换也可以实现图像放大和缩小，但是这里我们介绍图像金字塔
- 2. 一个图像金字塔式一系列的图像组成，最底下一张是图像尺寸最大，最上方的图像尺寸最小，从空间上从上向下看就想一个古代的金字塔。

##### 2、高斯金字塔
- 高斯金子塔是从底向上，逐层降采样得到。
- 降采样之后图像大小是原图像MxN的M/2 x N/2 ,就是对原图像删除偶数行与列，即得到降采样之后上一层的图片。
- 高斯金子塔的生成过程分为两步：
  	- 对当前层进行高斯模糊
  	- 删除当前层的偶数行与列
- 	即可得到上一层的图像，这样上一层跟下一层相比，都只有它的1/4大小。


##### 3、高斯不同(Difference of Gaussian-DOG)
- 定义：就是把同一张图像在不同的参数下做高斯模糊之后的结果相减，得到的输出图像。称为高斯不同(DOG)
- 高斯不同是图像的内在特征，在灰度图像增强、角点检测中经常用到。

##### 4、采样相关API

- 上采样(cv::pyrUp) – zoom in 放大
- 降采样 (cv::pyrDown) – zoom out 缩小

```
pyrUp(Mat src, Mat dst, Size(src.cols*2, src.rows*2)) 
生成的图像是原图在宽与高各放大两倍
pyrDown(Mat src, Mat dst, Size(src.cols/2, src.rows/2))
生成的图像是原图在宽与高各缩小1/2
```

##### 示例代码

```C++
    //上采样
	pyrUp(src, dst, Size(src.cols * 2, src.rows * 2));
	imshow("pyrUp img",dst);
	
	//降采样
	pyrDown(src, dst, Size(src.cols / 2, src.rows / 2));
	imshow("pyrDown img", dst);

	//DOG
	Mat gray_src, g1, g2, dogImg;
	cvtColor(src, gray_src, CV_BGR2GRAY);
	GaussianBlur(gray_src, g1, Size(3, 3), 0, 0);
	GaussianBlur(g1, g2, Size(3, 3), 0, 0);
	subtract(g1, g2, dogImg, Mat());

	//归一化显示
	normalize(dogImg, dogImg, 255, 0, NORM_MINMAX);
	imshow("dog img", dogImg);
```

#### 五、基本阈值操作
##### 1、图像阈值（threshold）
> 阈值 是什么？简单点说是把图像分割的标尺，这个标尺是根据什么产生的，阈值产生算法？阈值类型。（Binary segmentation）

##### 2、阈值类型一阈值二值化(threshold binary)
- 左下方的图表示图像像素点Src(x,y)值分布情况，蓝色水平线表示阈值 
![image](http://myfile.buildworld.cn/阈值二值化.jpg)

##### 3、阈值类型一阈值反二值化(threshold binary Inverted)
- 左下方的图表示图像像素点Src(x,y)值分布情况，蓝色水平线表示阈值 
![image](http://myfile.buildworld.cn/阈值反二值化.jpg)

##### 4、阈值类型一截断 (truncate)
- 左下方的图表示图像像素点Src(x,y)值分布情况，蓝色水平线表示阈值 
![image](http://myfile.buildworld.cn/截断.jpg)

##### 5、阈值类型一阈值取零 (threshold to zero)
- 左下方的图表示图像像素点Src(x,y)值分布情况，蓝色水平线表示阈值 
![image](http://myfile.buildworld.cn/阈值取零.jpg)

##### 5、阈值类型一阈值反取零 (threshold to zero inverted)
- 左下方的图表示图像像素点Src(x,y)值分布情况，蓝色水平线表示阈值 
![image](http://myfile.buildworld.cn/阈值反取零.jpg)
##### 6、相关的API
![image](http://myfile.buildworld.cn/阈值API.png)
##### 示例代码
```c++
Mat src, dst, gray_src;
int threshold_value = 127;
int threshold_max = 255;
int type_value = 2;
int type_max = 4;
string output_title = "output img";
void Threshold_Demo(int, void *);


int main(int argc, char ** argv) {

	src = imread("C:\\Users\\15646\\Pictures\\雷军.jpg");

	if (!src.data)
	{
		cout << "not found img" << endl;
		waitKey(0);
		return -1;
	}

	namedWindow(output_title, CV_WINDOW_AUTOSIZE);
	createTrackbar("Threshold Value", output_title, &threshold_value, threshold_max, Threshold_Demo);
	createTrackbar("Type Value", output_title, &type_value, type_max, Threshold_Demo);
	Threshold_Demo(0, 0);

	waitKey(0);
	return 0;
}

void Threshold_Demo(int, void *) {
	cvtColor(src, gray_src, CV_BGR2GRAY);
	//设置阈值
	threshold(gray_src, dst, threshold_value, threshold_max,type_value);
	imshow(output_title, dst);
}

```
#### 六、自定义线性滤波
##### 1、卷积概念
- 卷积是图像处理中一个操作，是kernel在图像的每个像素上的操作。
- Kernel本质上一个固定大小的矩阵数组，其中心点称为锚点(anchor point)

##### 2、卷积如何工作
> 把kernel放到像素数组之上，求锚点周围覆盖的像素乘积之和（包括锚点），用来替换锚点覆盖下像素点值称为卷积处理。数学表达如下：
<html>
<a href="https://www.codecogs.com/eqnedit.php?latex=H(x,y)=\sum_{i=0}^{M_{i-1}}\sum_{i=0}^{M_{j-1}}I\left&space;(&space;x&plus;i-a_{i}&space;,y&plus;j-a_{j}\right&space;)K(i,j)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?H(x,y)=\sum_{i=0}^{M_{i-1}}\sum_{i=0}^{M_{j-1}}I\left&space;(&space;x&plus;i-a_{i}&space;,y&plus;j-a_{j}\right&space;)K(i,j)" title="H(x,y)=\sum_{i=0}^{M_{i-1}}\sum_{i=0}^{M_{j-1}}I\left ( x+i-a_{i} ,y+j-a_{j}\right )K(i,j)" /></a>
</html>

###### 工作原理示意图
![image](http://myfile.buildworld.cn/卷积.png)

##### 3、常见算子

###### Robert算子
+1 | 0   
---|---   
0 | -1   

0| +1   
---|---   
-1 | 0  


###### Sobel算子
-1 | 0 |1
---|---|---
-2 | 0 |2
-1 | 0 |1

-1 | -2 |-1
---|---|---
0 | 0 | 0
1 | 2 | 1

###### 拉普拉多算子
0 | -1 |0
---|---|---
-1 |  4 |-1
0  |-1|0

##### 示例代码

```C++
    //Robert算子 X方向
	Mat kernel_x = (Mat_<int>(2, 2) << 1, 0, 0, -1);
	filter2D(src, dst, -1, kernel_x, Point(-1, -1), 0, 0);
	imshow("Robert X", dst);

	//Robert算子 Y方向
	Mat kernel_y = (Mat_<int>(2, 2) << 0,1,-1,0);
	filter2D(src, dst, -1, kernel_y, Point(-1, -1), 0, 0);
```
##### 4、自定义卷积模糊

```
ilter2D方法filter2D(
Mat src, //输入图像
Mat dst, // 模糊图像
int depth, // 图像深度32/8
Mat kernel, // 卷积核/模板
Point anchor, // 锚点位置
double delta // 计算出来的像素+delta
)
其中 kernel是可以自定义的卷积核

```
###### 图片逐渐模糊代码

```C++
	int c = 0;
	int index = 0;
	int ksize = 0;
	while (true)
	{
		c = waitKey(500);
		if ((char)c == 27)
		{
			break;
		}
		ksize = 4 + (index % 5) * 2;
		Mat kernel = Mat::ones(Size(ksize, ksize), CV_32F) / (float)(ksize*ksize);
		filter2D(src, dst, -1, kernel, Point(-1, -1));
		index++;
		imshow("show", dst);
	}
```
#### 七、处理边缘

##### 1、卷积边界问题
> 图像卷积的时候边界像素，不能被卷积操作，原因在于边界像素没有完全跟kernel重叠，所以当3x3滤波时候有1个像素的边缘没有被处理，5x5滤波的时候有2个像素的边缘没有被处理。

##### 2、处理边缘
> 在卷积开始之前增加边缘像素，填充的像素值为0或者RGB黑色，比如3x3在
四周各填充1个像素的边缘，这样就确保图像的边缘被处理，在卷积处理之
后再去掉这些边缘。openCV中默认的处理方法是： BORDER_DEFAULT，此外
常用的还有如下几种：
 - BORDER_CONSTANT – 填充边缘用指定像素值
 - BORDER_REPLICATE – 填充边缘像素用已知的边缘像素值。
 - BORDER_WRAP – 用另外一边的像素来补偿填充
##### 填充示例代码

```C++
Mat src, dst;
	src = imread("C:\\Users\\15646\\Pictures\\雷军.jpg");

	int top = (int)(0.05*src.rows);
	int bottom = (int)(0.05*src.rows);
	int left = (int)(0.05*src.cols);
	int right = (int)(0.05*src.cols);

	RNG rng;
	Scalar color = Scalar(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255));
	/*copyMakeBorder（
		- Mat src, // 输入图像
		-Mat dst, // 添加边缘图像
		-int top, // 边缘长度，一般上下左右都取相同值，
		-int bottom,
		-int left,
		-int right,
		-int borderType // 边缘类型
		- Scalar value
		）
		*/
	copyMakeBorder(src, dst, top, bottom, left, right, BORDER_CONSTANT,color);
	imshow("output img", dst);
```
#### 八、图像边缘提取--Sobel算子
##### 1、卷积应用-图像边缘提取
- 边缘是什么 – 是像素值发生跃迁的地方，是图像的显著特征之一，在图像特征提取、对象检测、模式识别等方面都有重要的作用。
- 如何捕捉/提取边缘 – 对图像求它的一阶导数
- 	  delta =  f(x) – f(x-1), delta越大，说明像素在X方向变化越大，边缘信号越强，
##### 2、Sobel算子
- 是离散微分算子（discrete differentiation operator），用来计算图像灰度的近似梯度
- Soble算子功能集合高斯平滑和微分求导
- 又被称为一阶微分算子，求导算子，在水平和垂直两个方向上求导，得到图像X方向与Y方向梯度图像
###### 原理图
![image](http://myfile.buildworld.cn/sobel算子.jpg)
##### 示例代码

```C++
Mat src, dst;
	src = imread("C:\\Users\\15646\\Pictures\\雷军.jpg");

	Mat gray_dst;
	GaussianBlur(src, dst, Size(3, 3), 0, 0);
	cvtColor(dst, gray_dst, CV_BGR2GRAY);

	//图像边缘提取
	Mat xgrad, ygrad;
	//获取x,y方向上的梯度图像
	/*cv::Sobel(
		InputArray Src // 输入图像
		OutputArray dst// 输出图像，大小与输入图像一致
		int depth // 输出图像深度. 
		Int dx.  // X方向，几阶导数
		int dy // Y方向，几阶导数. 
		int ksize, SOBEL算子kernel大小，必须是1、3、5、7、
		double scale = 1
		double delta = 0
		int borderType = BORDER_DEFAULT
	)*/

	Sobel(gray_dst, xgrad, CV_16S, 1, 0, 3);
	Sobel(gray_dst, ygrad, CV_16S, 0, 1, 3);
	convertScaleAbs(xgrad, xgrad); //计算图像像素的绝对值再输出到此图像
	convertScaleAbs(ygrad, ygrad);
	//合成x,y方向上的梯度图像
	Mat res;
	addWeighted(xgrad, 0.5, ygrad, 0, 0, res);
	imshow("res img", res);

	//手动合成
	Mat xygrad = Mat(xgrad.size(), xgrad.type());
	int height = ygrad.rows;
	int width = xgrad.cols;

	for (int row = 0; row < height; row++)
	{
		for (int col = 0; col < width; col++)
		{
			//将水平梯度和垂直梯度的值相加获取我们需要的值，合成图像
			xygrad.at<uchar>(row, col) = saturate_cast<uchar>(xgrad.at<uchar>(row, col) + ygrad.at<uchar>(row, col));
		}
	}
	namedWindow("xygrad", CV_WINDOW_AUTOSIZE);
	imshow("xygrad", xygrad);
```
###### 在OpenCV里面包含了增强的Scharr方法

```C++
cv::Scharr (
InputArray Src // 输入图像
OutputArray dst// 输出图像，大小与输入图像一致
int depth // 输出图像深度. 
Int dx.  // X方向，几阶导数
int dy // Y方向，几阶导数. 
double scale  = 1
double delta = 0
int borderType = BORDER_DEFAULT
)
```
#### 九、图像边缘提取-Laplance算子(拉普拉斯算子)
##### 1、处理流程
- 高斯模糊 – 去噪声GaussianBlur()
- 转换为灰度图像cvtColor()
- 拉普拉斯 – 二阶导数计算Laplacian()
- 取绝对值convertScaleAbs()
- 显示结果

##### 2、API使用cv::Laplacian

```C++
    Laplacian(
    InputArray src,
    OutputArray dst,
    int depth, //深度CV_16S
    int kisze, // 3
    double scale = 1,
    double delta =0.0,
    int borderType = 4
    )
```
###### 示例代码
```C++
    Mat src, dst, edge_img;
	src = imread("C:\\Users\\15646\\Pictures\\雷军.jpg");

	Mat gray_dst;
	GaussianBlur(src, dst, Size(3, 3), 0, 0);
	cvtColor(dst, gray_dst, CV_BGR2GRAY);

	threshold(edge_img, edge_img, 0, 255, THRESH_OTSU | THRESH_BINARY);
	Laplacian(gray_dst, edge_img, CV_16S, 3);
	convertScaleAbs(edge_img, edge_img);
	imshow("laplance img", edge_img);
```

#### 十、Canny边缘检测
##### 1、Canny算法介绍
- Canny是边缘检测算法，在1986年提出的。
- 是一个很好的边缘检测器
- 很常用也很实用的图像处理方法

##### 2、Canny算法使用
- 高斯模糊 - GaussianBlur
- 灰度转换 - cvtColor
- 计算梯度 – Sobel/Scharr
- 非最大信号抑制
- 高低阈值输出二值图像

##### 3、Canny算法介绍 - 非最大信号抑制
![image](http://myfile.buildworld.cn/Canny算法-非最大信号抑制.jpg)

##### 4、Canny算法介绍-高低阈值输出二值图像 
- T1， T2为阈值，凡是高于T2的都保留，凡是小于T1都丢弃，从高于T2的像素出发，凡是大于T1而且相互连接的，都保留。最终得到一个输出二值图像。
- 推荐的高低阈值比值为 T2: T1 = 3:1/2:1其中T2为高阈值，T1为低阈值

##### 示例代码

```C++
Mat gray_dst, src;
int t1_value = 50;
int max_value = 255;
string output_title = "output img";

void CannyDemo(int, void*);
int  main(int argc, char* argv) {

	src = imread("C:\\Users\\15646\\Pictures\\雷军.jpg");

	namedWindow(output_title, CV_WINDOW_AUTOSIZE);
	//灰度转换
	cvtColor(src, gray_dst, CV_BGR2GRAY);
	createTrackbar("Threshold Value", output_title, &t1_value, max_value, CannyDemo);
	CannyDemo(0, 0);

	waitKey(0);
	return 0;
}
void CannyDemo(int, void*) {
	Mat edge_dst;
	//高斯模糊
	GaussianBlur(gray_dst, edge_dst, Size(3, 3), 0,0);
	/*Canny（
		InputArray src, // 8-bit的输入图像
		OutputArray edges,// 输出边缘图像， 一般都是二值图像，背景是黑色
		double threshold1,// 低阈值，常取高阈值的1/2或者1/3
		double threshold2,// 高阈值
		int aptertureSize,// Soble算子的size，通常3x3，取值3
		bool L2gradient // 选择 true表示是L2来归一化，否则用L1归一化,一般用L1
		）*/

	Canny(gray_dst, edge_dst, t1_value, t1_value * 2, 3, false);
	imshow(output_title, edge_dst);
}
```

