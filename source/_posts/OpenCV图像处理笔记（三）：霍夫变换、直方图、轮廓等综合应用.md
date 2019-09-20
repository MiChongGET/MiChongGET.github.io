---
title: OpenCV图像处理笔记（三）：霍夫变换、直方图、轮廓等综合应用
date: 2019-07-22 09:46:12
tags:
- C++
- OpenCV
categories:
- 计算机视觉
---
#### 一、霍夫直线变换

##### 1、霍夫直线变换
- Hough Line Transform用来做直线检测
- 前提条件 – 边缘检测已经完成
- 平面空间到极坐标空间转换

##### 2、霍夫直线变换介绍
- 对于任意一条直线上的所有点来说
- 变换到极坐标中，从[0~360]空间，可以得到r的大小
- 属于同一条直线上点在极坐标空(r, theta)必然在一个点上有最强的信号出现，根据此反算到平面坐标中就可以得到直线上各点的像素坐标。从而得到直线

##### 3、相关API
- 标准的霍夫变换 cv::HoughLines从平面坐标转换到霍夫空间，最终输出是               表示极坐标空间
- 霍夫变换直线概率 cv::HoughLinesP最终输出是直线的两个点


```C++
    cv::HoughLinesP(
    InputArray src, // 输入图像，必须8-bit的灰度图像
    OutputArray lines, // 输出的极坐标来表示直线
    double rho, // 生成极坐标时候的像素扫描步长
    double theta, //生成极坐标时候的角度步长，一般取值CV_PI/180
    int threshold, // 阈值，只有获得足够交点的极坐标点才被看成是直线
    double minLineLength=0;// 最小直线长度
    double maxLineGap=0;// 最大间隔
    )
```
##### 实例代码

```C++
    Mat src, src_gray,dst;
	src = imread("C:\\Users\\15646\\Pictures\\线条.jpg");

	//边缘检测
	Canny(src, src_gray, 150, 200);
	//灰度转彩色
	cvtColor(src_gray, dst, CV_GRAY2BGR);

	//霍夫直线检测
	vector<Vec4f> plines;
	HoughLinesP(src_gray, plines, 1, CV_PI / 180.0, 10, 0, 10);
	Scalar color = Scalar(0, 0, 255);
	for (size_t i = 0; i < plines.size(); i++)
	{
		Vec4f hline = plines[i];
		line(dst, Point(hline[i], hline[1]), Point(hline[2], hline[3]), color,3, LINE_AA);
	}

	imshow("dst img", dst);
```
#### 二、霍夫圆检测
##### 1、原理
![原理](http://myfile.buildworld.cn/霍夫圆检测原理.png)
![image](http://myfile.buildworld.cn/霍夫圆检测原理2.png)

##### 2、相关API cv::HoughCircles
- 因为霍夫圆检测对噪声比较敏感，所以首先要对图像做中值滤波。
- 基于效率考虑，Opencv中实现的霍夫变换圆检测是基于图像梯度的实现，分为两步：
	1. 检测边缘，发现可能的圆心
 	2. 基于第一步的基础上从候选圆心开始计算最佳半径大小

```C++
    HoughCircles(
    InputArray image, // 输入图像 ,必须是8位的单通道灰度图像
    OutputArray circles, // 输出结果，发现的圆信息
    Int method, // 方法 - HOUGH_GRADIENT
    Double dp, // dp = 1; 
    Double mindist, // 10 最短距离-可以分辨是两个圆的，否则认为是同心圆- src_gray.rows/8
    Double param1, // canny edge detection low threshold
    Double param2, // 中心点累加器阈值 – 候选圆心
    Int minradius, // 最小半径
    Int maxradius//最大半径 
    )

```
##### 示例代码


```C++
Mat src, dst;
	src = imread("C:\\Users\\Administrator\\Pictures\\霍夫圆检测4.jpg");
	imshow("src img", src);

	//中值滤波转灰度
	Mat mediaImg;
	medianBlur(src, mediaImg, 3);
	cvtColor(mediaImg, mediaImg, CV_BGR2GRAY);
	GaussianBlur(mediaImg,mediaImg, Size(5, 5), 0, 0);

	//霍夫圆检测
	vector<Vec3f> pcircle;
	HoughCircles(mediaImg, pcircle, CV_HOUGH_GRADIENT, 1, 10, 100, 30, 5, 50);
	src.copyTo(dst);
	for (size_t i = 0; i < pcircle.size(); i++)
	{
		Vec3f cc = pcircle[i];
		//圆形标注
		circle(dst, Point(cc[0], cc[1]), cc[2], Scalar(0, 0, 255), 2, LINE_AA);
		//圆心标注
		circle(dst, Point(cc[0], cc[1]), 2, Scalar(198, 50, 255), 2, LINE_AA);
	}

	imshow("hough img", dst);

```
#### 三、像素重映射
##### 1、原理
> 简单点说就是把输入图像中各个像素按照一定的规则映射到另外一张图像的对应位置上去，形成一张新的图像。

##### 2、API介绍cv::remap
```C++
    Remap(
    InputArray src,// 输入图像
    OutputArray dst,// 输出图像
    InputArray  map1,// x 映射表 CV_32FC1/CV_32FC2
    InputArray map2,// y 映射表
    int interpolation,// 选择的插值方法，常见线性插值，可选择立方等
    int borderMode,// BORDER_CONSTANT
    const Scalar borderValue// color 
    )
```
##### 示例代码

```C++
#include <opencv2/opencv.hpp>
#include <iostream>
#include <math.h>

using namespace cv;
Mat src, dst, map_x, map_y;
const char* OUTPUT_TITLE = "remap demo";
int index = 0;
void update_map(void);
int main(int argc, char** argv) {
	src = imread("C:\\Users\\Administrator\\Pictures\\girl2.jpg");
	if (!src.data) {
		printf("could not load image...\n");
		return -1;
	}
	char input_win[] = "input image";
	namedWindow(input_win, CV_WINDOW_AUTOSIZE);
	namedWindow(OUTPUT_TITLE, CV_WINDOW_AUTOSIZE);
	imshow(input_win, src);

	map_x.create(src.size(), CV_32FC1);
	map_y.create(src.size(), CV_32FC1);

	int c = 0;
	while (true) {
		c = waitKey(500);
		//按住esc退出
		if ((char)c == 27) {
			break;
		}

		//键盘输入1,2,3,4
		index = c % 4;
		update_map();
		remap(src, dst, map_x, map_y, INTER_LINEAR, BORDER_CONSTANT, Scalar(0, 255, 255));
		imshow(OUTPUT_TITLE, dst);
	}

	return 0;
}

void update_map(void) {
	for (int row = 0; row < src.rows; row++) {
		for (int col = 0; col < src.cols; col++) {
			switch (index) {
			//缩小一半
			case 0:
				if (col >(src.cols * 0.25) && col <= (src.cols*0.75) && row >(src.rows*0.25) && row <= (src.rows*0.75)) {
					map_x.at<float>(row, col) = 2 * (col - (src.cols*0.25));
					map_y.at<float>(row, col) = 2 * (row - (src.rows*0.25));
				}
				else {
					map_x.at<float>(row, col) = 0;
					map_y.at<float>(row, col) = 0;
				}
				break;
			//x方向对调
			case 1:
				map_x.at<float>(row, col) = (src.cols - col - 1);
				map_y.at<float>(row, col) = row;
				break;
			//y方向对调
			case 2:
				map_x.at<float>(row, col) = col;
				map_y.at<float>(row, col) = (src.rows - row - 1);
				break;
			//x,y方向都对调
			case 3:
				map_x.at<float>(row, col) = (src.cols - col - 1);
				map_y.at<float>(row, col) = (src.rows - row - 1);
				break;
			}

		}
	}
}
```
#### 四、直方图 (histogram)
##### 1、直方图均衡化概念
> 图像直方图，是指对整个图像像在灰度范围内的像素值(0~255)统计出现频率次数，据此生成的直方图，称为图像直方图-直方图。直方图反映了图像灰度的分布情况。是图像的统计学特征。

##### 2、直方图均衡化
> 是一种提高图像对比度的方法，拉伸图像灰度值范围。

> 如何实现，通过上一课中的remap我们知道可以将图像灰度分布从一个分布映射到另外一个分布，然后在得到映射后的像素值即可。

##### 3、直方图均衡化API说明cv::equalizeHist

```
equalizeHist(
InputArray src,//输入图像，必须是8-bit的单通道图像
OutputArray dst// 输出结果
)
```

##### 4、直方图计算
- 上述直方图概念是基于图像像素值，其实对图像梯度、每个像素的角度、等一切图像的属性值，我们都可以建立直方图。这个才是直方图的概念真正意义，不过是基于图像像素灰度直方图是最常见的。
- 直方图最常见的几个属性：
  - dims 表示维度，对灰度图像来说只有一个通道值dims=1
  - bins 表示在维度中子区域大小划分，bins=256，划分为256个级别
  - range 表示值得范围，灰度值范围为[0~255]之间

```C++
    split(// 把多通道图像分为多个单通道图像
    const Mat &src, //输入图像
    Mat* mvbegin）// 输出的通道图像数组
    
    calcHist(
     const Mat* images,//输入图像指针
    int images,// 图像数目
    const int* channels,// 通道数
    InputArray mask,// 输入mask，可选，不用
    OutputArray hist,//输出的直方图数据
    int dims,// 维数
    const int* histsize,// 直方图级数
    const float* ranges,// 值域范围
    bool uniform,// true by default
    bool accumulate// false by defaut
    )
```


##### 4、直方图比较方法-概述

> 对输入的两张图像计算得到直方图H1与H2，归一化到相同的尺度空间
然后可以通过计算H1与H2的之间的距离得到两个直方图的相似程度进
而比较图像本身的相似程度。

> Opencv提供的比较方法有四种：
- Correlation 相关性比较(CV_COMP_CORREL)
- Chi-Square 卡方比较(CV_COMP_CHISQR)
- Intersection 十字交叉性(CV_COMP_INTERSECT)
- Bhattacharyya distance 巴氏距离(CV_COMP_BHATTACHARYYA )
##### 相关API
- 首先把图像从RGB色彩空间转换到HSV色彩空间cvtColor
- 计算图像的直方图，然后归一化到[0~1]之间calcHist和normalize;
- 使用上述四种比较方法之一进行比较compareHist
###### 相关API cv::compareHist

```
compareHist(
InputArray h1, // 直方图数据，下同
InputArray H2,
int method// 比较方法，上述四种方法之一
)
```
###### 示例代码

```C++
Mat src, dst;
	src = imread("C:\\Users\\15646\\Pictures\\刘亦菲.jpg");
	
	//分通道显示
	vector<Mat> bgr_planes;
	split(src, bgr_planes);

	int histSize = 256;
	float range[] = { 0,256 };
	const float *histRange = { range };
	Mat b_hist, g_hist, r_hist;
	calcHist(&bgr_planes[0], 1, 0, Mat(), b_hist, 1, &histSize, &histRange, true, false);
	calcHist(&bgr_planes[1], 1, 0, Mat(), g_hist, 1, &histSize, &histRange, true, false);
	calcHist(&bgr_planes[2], 1, 0, Mat(), r_hist, 1, &histSize, &histRange, true, false);

	//归一化
	int hist_h = 200;
	int hist_w = 300;
	int bin_w = hist_w / histSize;
	Mat histImage(hist_w, hist_h, CV_8UC3, Scalar(0, 0, 0));
	normalize(b_hist, b_hist, 0, hist_h, NORM_MINMAX, -1, Mat());
	normalize(g_hist, g_hist, 0, hist_h, NORM_MINMAX, -1, Mat());
	normalize(r_hist, r_hist, 0, hist_h, NORM_MINMAX, -1, Mat());

	//绘制直方图
	for (int i = 0; i < histSize; i++)
	{
		line(histImage, Point((i - 1)*bin_w, hist_h - cvRound(b_hist.at<float>(i - 1))),
			Point((i)*bin_w, hist_h - cvRound(b_hist.at<float>(i))), Scalar(255, 0, 0), LINE_AA);
		line(histImage, Point((i - 1)*bin_w, hist_h - cvRound(g_hist.at<float>(i - 1))),
			Point((i)*bin_w, hist_h - cvRound(g_hist.at<float>(i))), Scalar(255, 0, 0), LINE_AA);
		line(histImage, Point((i - 1)*bin_w, hist_h - cvRound(r_hist.at<float>(i - 1))),
			Point((i)*bin_w, hist_h - cvRound(r_hist.at<float>(i))), Scalar(255, 0, 0), LINE_AA);

	}
	imshow("src img", histImage);
```

##### 5、直方图反向投影(Back Projection)
- 反向投影是反映直方图模型在目标图像中的分布情况
- 简单点说就是用直方图模型去目标图像中寻找是否有相似的对象。通常用HSV色彩空间的HS两个通道直方图模型

###### 反向投影 – 步骤
- 1.建立直方图模型
- 2.计算待测图像直方图并映射到模型中
- 3.从模型反向计算生成图像
 
```C++
加载图片imread
将图像从RGB色彩空间转换到HSV色彩空间cvtColor
计算直方图和归一化calcHist与normalize
Mat与MatND其中Mat表示二维数组，MatND表示三维或者多维数据，此处均可以用Mat表示。
计算反向投影图像 - calcBackProject

Mat src, hsv, hue;
int bins = 12;
string src_img = "src img";
void Hist_Add_Backproject(int, void*);

int main(int argc, char * argv) {

	src = imread("C:\\Users\\15646\\Pictures\\lab\\test1.png");
	namedWindow(src_img, WINDOW_AUTOSIZE);

	cvtColor(src, hsv, CV_BGR2HSV);
	hue.create(hsv.size(), hsv.depth());
	int nchannels[] = { 0,0 };
	mixChannels(&hsv, 1, &hue, 1, nchannels, 1);

	createTrackbar("Histogram Bins", src_img, &bins, 180, Hist_Add_Backproject);
	Hist_Add_Backproject(0, 0);

	imshow(src_img, src);
	waitKey(0);
	return 0;
}

void Hist_Add_Backproject(int, void*) {

	float range[] = { 0,180 };
	const float *histRanges = { range };
	Mat h_hist;
	calcHist(&hue, 1, 0, Mat(), h_hist, 1, &bins, &histRanges, true, false);
	normalize(h_hist, h_hist, 0, 255, NORM_MINMAX, -1, Mat());

	Mat backPrjImage;
	calcBackProject(&hue, 1, 0, h_hist, backPrjImage, &histRanges, 1, true);
	imshow("BackProj", backPrjImage);

	int hist_h = 400;
	int hist_w = 400;
	Mat histImage(hist_w, hist_h, CV_8UC3, Scalar(0, 0, 0));
	int bin_w = cvRound((double)hist_w / bins);
	for (int i = 1; i < bins; i++)
	{
		rectangle(histImage,
			Point((i - 1)*bin_w, (hist_h - cvRound(h_hist.at<float>(i - 1)*(400 / 255)))),
			Point(i*bin_w, hist_h),
			Scalar(0, 0, 255), -1);
	}
	imshow("Histogram", histImage);
}

```
#### 五、模板匹配(Template Match)
##### 1、模板匹配介绍
- 模板匹配就是在整个图像区域发现与给定子图像匹配的小块区域。
- 所以模板匹配首先需要一个模板图像T（给定的子图像）
- 另外需要一个待检测的图像-源图像S
- 工作方法，在带检测图像上，从左到右，从上向下计算模板图像与重叠子图像的匹配度，匹配程度越大，两者相同的可能性越大。

##### 2、模板匹配介绍 – 匹配算法介绍
![image](http://myfile.buildworld.cn/模板匹配算法.jpg)

##### 3、相关API介绍cv::matchTemplate

```C++
matchTemplate(
    InputArray image,// 源图像，必须是8-bit或者32-bit浮点数图像
    
    InputArray templ,// 模板图像，类型与输入图像一致
    
    OutputArray result,// 输出结果，必须是单通道32位浮点数，假设源图像WxH,模板图像wxh,
    	             则结果必须为W-w+1, H-h+1的大小。
    int method,//使用的匹配方法
    
    InputArray mask=noArray()//(optional)
)

```
##### 示例代码

```C++
#include<opencv2\opencv.hpp>
#include<iostream>

using namespace std;
using namespace cv;
Mat src, dst, temp;
int match_method = CV_TM_SQDIFF;
int max_track = 5;
string inputImg = "input img";
string outputImg = "output img";
string matchimg = "template match-demo";
void Match_Demo(int, void*);

int main(int argc, char * argv) {

	src = imread("C:\\Users\\15646\\Pictures\\雷军.jpg");
	temp = imread("C:\\Users\\15646\\Pictures\\雷军头像.jpg");

	namedWindow(inputImg, CV_WINDOW_AUTOSIZE);
	namedWindow(outputImg, CV_WINDOW_AUTOSIZE);
	namedWindow(matchimg, CV_WINDOW_AUTOSIZE);

	imshow(inputImg, src);
	createTrackbar("Macth Control", inputImg, &match_method, max_track, Match_Demo);

	Match_Demo(0, 0);
	waitKey(0);
	return 0;
}

void Match_Demo(int, void*) {
	int width = src.cols - temp.cols + 1;
	int height = src.rows - temp.rows + 1;
	Mat result(width, height, CV_32FC1);

	matchTemplate(src,temp,result,match_method,Mat());
	normalize(result, result, 0, 1, NORM_MINMAX, -1, Mat());

	Point minLoc;
	Point maxLoc;
	double min, max;
	src.copyTo(dst);
	Point tenLoc;
	minMaxLoc(result, &min, &max, &minLoc, &maxLoc, Mat());

	if (match_method==CV_TM_SQDIFF || match_method == CV_TM_SQDIFF_NORMED)
	{
		tenLoc = minLoc;
	}
	else {
		tenLoc = maxLoc;
	}

	//绘制矩形
	rectangle(dst, Rect(tenLoc.x, tenLoc.y, temp.cols, temp.rows), Scalar(0, 0, .255), 2, 8);
	rectangle(result, Rect(tenLoc.x, tenLoc.y, temp.cols, temp.rows), Scalar(0, 0, .255), 2, 8);

	imshow(outputImg, result);
	imshow(matchimg, dst);
}
```



#### 六、轮廓发现(find contour in your image)
- 轮廓发现是基于图像边缘提取的基础寻找对象轮廓的方法。
- 所以边缘提取的阈值选定会影响最终轮廓发现结果
- API介绍
    - findContours发现轮廓
    - drawContours绘制轮廓

##### 轮廓发现(find contour)

```
在二值图像上发现轮廓使用API cv::findContours(
InputOutputArray  binImg, // 输入图像，非0的像素被看成1,0的像素值保持不变，8-bit
     OutputArrayOfArrays  contours,//  全部发现的轮廓对象
    OutputArray,  hierachy// 图该的拓扑结构，可选，该轮廓发现算法正是基于图像拓扑结构实现。
    int mode, //  轮廓返回的模式
    int method,// 发现方法
    Point offset=Point()//  轮廓像素的位移，默认（0, 0）没有位移
)
```
##### 轮廓绘制(draw contour)

```C++
在二值图像上发现轮廓使用API cv::findContours之后对发现的轮廓数据进行绘制显示
drawContours(
    InputOutputArray  binImg, // 输出图像
     OutputArrayOfArrays  contours,//  全部发现的轮廓对象
    Int contourIdx// 轮廓索引号
    const Scalar & color,// 绘制时候颜色
    int  thickness,// 绘制线宽
    int  lineType ,// 线的类型LINE_8
    InputArray hierarchy,// 拓扑结构图
    int maxlevel,// 最大层数， 0只绘制当前的，1表示绘制绘制当前及其内嵌的轮廓
    Point offset=Point()// 轮廓位移，可选
)
```
##### 流程
- 输入图像转为灰度图像cvtColor
- 使用Canny进行边缘提取，得到二值图像
- 使用findContours寻找轮廓
- 使用drawContours绘制轮廓

##### 示例代码
```C++
using namespace std;
using namespace cv;
Mat src, dst;
int threshold_value = 100;
int threshold_max = 255;
RNG rng;
void Demo_Countours(int, void*);
string input_title = "input img";
string output_title = "output img";

int main(int argc, char * argv) {
	src = imread("C:\\Users\\15646\\Pictures\\lab\\山羊.jfif");
	namedWindow(input_title, CV_WINDOW_AUTOSIZE);
	namedWindow(output_title, CV_WINDOW_AUTOSIZE);
	imshow(input_title, src);
	//图像灰度化
	cvtColor(src, src, CV_BGR2GRAY);

	createTrackbar("control img", output_title, &threshold_value, threshold_max, Demo_Countours);
	Demo_Countours(0, 0);
	waitKey(0);
	return 0;
}
void Demo_Countours(int, void*) {
	Mat canny_output;
	vector<vector<Point>> contours;
	vector<Vec4i>hierachy;
	//图像边缘检测二值化
	Canny(src, canny_output, threshold_value, threshold_value * 2, 3, false);
	//轮廓发现
	findContours(canny_output, contours, hierachy, RETR_TREE, CHAIN_APPROX_SIMPLE, Point(0, 0));

	dst = Mat::zeros(src.size(), CV_8UC3);
	rng(12345);
	for (size_t i = 0; i < contours.size(); i++)
	{
		Scalar color = Scalar(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255));
		//轮廓绘制
		drawContours(dst, contours, i, color, 2, 8, hierachy, 0, Point(0, 0));
	}

	imshow(output_title, dst);
}
```

#### 七、凸包-Convex Hull
##### 1、定义
> 什么是凸包(Convex Hull)，在一个多变形边缘或者内部任                     意两个点的连线都包含在多边形边界或者内部。

###### 正式定义：
> 包含点集合S中所有点的最小凸多边形称为凸包


##### 2、概念介绍-Graham扫描算法
- 首先选择Y方向最低的点作为起始点p0
- 从p0开始极坐标扫描，依次添加p1….pn（排序顺序是根据极坐标的角度大小，逆时针方向）
- 对每个点pi来说，如果添加pi点到凸包中导致一个左转向（逆时针方法）则添加该点到凸包， 反之如果导致一个右转向（顺时针方向）删除该点从凸包中

##### 3、API说明cv::convexHull

```
convexHull(
    InputArray points,// 输入候选点，来自findContours
    OutputArray hull,// 凸包
    bool clockwise,// default true, 顺时针方向
    bool returnPoints）// true 表示返回点个数，如果第二个参数是			vector<Point>则自动忽略
)
```
##### 4、流程
- 首先把图像从RGB转为灰度
- 然后再转为二值图像
- 在通过发现轮廓得到候选点
- 凸包API调用
- 绘制显示。
##### 示例代码

```C++
#include<opencv2\opencv.hpp>
#include<iostream>

using namespace std;
using namespace cv;
Mat src,gray_src, dst;
int threshold_value = 100;
int threshold_max = 255;
RNG rng;
void Demo_Countours(int, void*);
string input_title = "input img";
string output_title = "output img";

int main(int argc, char * argv) {
	src = imread("C:\\Users\\15646\\Pictures\\lab\\test2.png");
	namedWindow(input_title, CV_WINDOW_AUTOSIZE);
	namedWindow(output_title, CV_WINDOW_AUTOSIZE);
	imshow(input_title, src);
	//图像灰度化
	cvtColor(src, gray_src, CV_BGR2GRAY);
	//模糊
	blur(gray_src, gray_src, Size(3, 3), Point(-1, -1), BORDER_DEFAULT);

	createTrackbar("control img", output_title, &threshold_value, threshold_max, Demo_Countours);
	Demo_Countours(0, 0);
	waitKey(0);
	return 0;
}
void Demo_Countours(int, void*) {
	Mat bin_output;
	vector<vector<Point>> contours;
	vector<Vec4i>hierachy;
	//设置阈值，阈值二值化
	threshold(gray_src, bin_output, threshold_value, threshold_max, THRESH_BINARY);
	//发现轮廓
	findContours(bin_output, contours, hierachy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE, Point(0, 0));

	vector<vector<Point>> convexs(contours.size());
	for (size_t i = 0; i < contours.size(); i++)
	{
		convexHull(contours[i], convexs[i], false, true);
	}

	//绘制
	dst = Mat::zeros(src.size(), CV_8UC3);
	vector<Vec4i>empty(0);
	for (size_t i = 0; i < contours.size(); i++)
	{
		Scalar color = Scalar(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255));
		drawContours(dst, contours, i, color,2, LINE_8, hierachy, 0, Point(0, 0));
		drawContours(dst, convexs, i, color, 2, LINE_8, empty, 0, Point(0, 0));
	}

	imshow(output_title, dst);
}

```


#### 八、轮廓周围绘制矩形框和圆形框
##### 1、轮廓周围绘制矩形 -API
- approxPolyDP(InputArray  curve, OutputArray approxCurve,  double  epsilon,  bool  closed)
- cv::boundingRect(InputArray points)得到轮廓周围最小矩形左上交点坐标和右下角点坐标，绘制一个矩形
- cv::minAreaRect(InputArray  points)得到一个旋转的矩形，返回旋转矩形

##### 2、轮廓周围绘制圆和椭圆-API
- cv::minEnclosingCircle(InputArray points, //得到最小区域圆形
    - Point2f& center, // 圆心位置
    - float& radius)// 圆的半径
- cv::fitEllipse(InputArray  points)得到最小椭圆

##### 3、流程
- 首先将图像变为二值图像
- 发现轮廓，找到图像轮廓
- 通过相关API在轮廓点上找到最小包含矩形和圆，旋转矩形与椭圆。
- 绘制它们。
##### 示例代码

```C++
#include <opencv2/opencv.hpp>
#include <iostream>
#include <math.h>

using namespace std;
using namespace cv;
Mat src, gray_src, drawImg;
int threshold_v = 170;
int threshold_max = 255;
const char* output_win = "rectangle-demo";
RNG rng(12345);
void Contours_Callback(int, void*);
int main(int argc, char** argv) {
	src = imread("C:\\Users\\15646\\Pictures\\lab\\热气球.jfif");
	if (!src.data) {
		printf("could not load image...\n");
		return -1;
	}
	cvtColor(src, gray_src, CV_BGR2GRAY);
	blur(gray_src, gray_src, Size(3, 3), Point(-1, -1));

	const char* source_win = "input image";
	namedWindow(source_win, CV_WINDOW_AUTOSIZE);
	namedWindow(output_win, CV_WINDOW_AUTOSIZE);
	imshow(source_win, src);

	createTrackbar("Threshold Value:", output_win, &threshold_v, threshold_max, Contours_Callback);
	Contours_Callback(0, 0);

	waitKey(0);
	return 0;
}

void Contours_Callback(int, void*) {
	Mat binary_output;
	vector<vector<Point>> contours;
	vector<Vec4i> hierachy;
	threshold(gray_src, binary_output, threshold_v, threshold_max, THRESH_BINARY);
	//imshow("binary image", binary_output);
	findContours(binary_output, contours, hierachy, RETR_TREE, CHAIN_APPROX_SIMPLE, Point(-1, -1));

	vector<vector<Point>> contours_ploy(contours.size());
	vector<Rect> ploy_rects(contours.size());
	vector<Point2f> ccs(contours.size());
	vector<float> radius(contours.size());

	vector<RotatedRect> minRects(contours.size());
	vector<RotatedRect> myellipse(contours.size());

	for (size_t i = 0; i < contours.size(); i++) {
		approxPolyDP(Mat(contours[i]), contours_ploy[i], 3, true);
		ploy_rects[i] = boundingRect(contours_ploy[i]);
		minEnclosingCircle(contours_ploy[i], ccs[i], radius[i]);
		if (contours_ploy[i].size() > 5) {
			myellipse[i] = fitEllipse(contours_ploy[i]);
			minRects[i] = minAreaRect(contours_ploy[i]);
		}
	}

	// draw it
	drawImg = Mat::zeros(src.size(), src.type());
	Point2f pts[4];
	for (size_t t = 0; t < contours.size(); t++) {
		Scalar color = Scalar(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255));
		//rectangle(drawImg, ploy_rects[t], color, 2, 8);
		//circle(drawImg, ccs[t], radius[t], color, 2, 8);
		if (contours_ploy[t].size() > 5) {
			ellipse(drawImg, myellipse[t], color, 1, 8);
			minRects[t].points(pts);
			for (int r = 0; r < 4; r++) {
				line(drawImg, pts[r], pts[(r + 1) % 4], color, 1, 8);
			}
		}
	}

	imshow(output_win, drawImg);
	return;
}

```



#### 九、图像矩(Image Moments)
##### 1、概念
![image](http://myfile.buildworld.cn/几何矩.jpg)
##### 2、图像中心Center(x0, y0)

<html>
<a href="https://www.codecogs.com/eqnedit.php?latex=\dpi{200}&space;\large&space;x_{0}=\frac{m_{10}}{m_{00}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\dpi{200}&space;\large&space;x_{0}=\frac{m_{10}}{m_{00}}" title="\large x_{0}=\frac{m_{10}}{m_{00}}" /></a>
</html>

<html>
<a href="https://www.codecogs.com/eqnedit.php?latex=\dpi{200}&space;\large&space;y_{0}=\frac{m_{01}}{m_{00}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\dpi{200}&space;\large&space;y_{0}=\frac{m_{01}}{m_{00}}" title="\large y_{0}=\frac{m_{01}}{m_{00}}" /></a>
</html>


##### 3、API介绍与使用-计算矩cv::moments

```
    moments(
    InputArray  array,//输入数据
    bool   binaryImage=false // 是否为二值图像
    )
    
    contourArea(
    InputArray  contour,//输入轮廓数据
    bool   oriented// 默认false、返回绝对值)
    )
    
    arcLength(
    InputArray  curve,//输入曲线数据
    bool   closed// 是否是封闭曲线)
    )
```
##### 4、过程
- 提取图像边缘
- 发现轮廓
- 计算每个轮廓对象的矩
- 计算每个对象的中心、弧长、面积

##### 示例代码

```C++
#include<opencv2\opencv.hpp>
#include<iostream>

using namespace std;
using namespace cv;
Mat src, dst,gray_src;
int threshold_value = 80;
int threshold_max = 255;
RNG rng;
void Demo_Countours(int, void*);
string input_title = "input img";
string output_title = "output img";

int main(int argc, char * argv) {
	src = imread("C:\\Users\\Administrator\\Pictures\\lab\\星球.jpg");
	namedWindow(input_title, CV_WINDOW_AUTOSIZE);
	namedWindow(output_title, CV_WINDOW_AUTOSIZE);
	imshow(input_title, src);
	//图像灰度化
	cvtColor(src, gray_src, CV_BGR2GRAY);
	//高斯模糊
	GaussianBlur(gray_src, gray_src, Size(3, 3), 0, 0);

	createTrackbar("control img", output_title, &threshold_value, threshold_max, Demo_Countours);
	Demo_Countours(0, 0);
	waitKey(0);
	return 0;
}
void Demo_Countours(int, void*) {
	Mat canny_out;
	vector<vector<Point>> contours;
	vector<Vec4i>hierachy; 

	//边缘检测
	Canny(gray_src, canny_out, threshold_value, threshold_value * 2,3, false);
	//轮廓发现
	findContours(canny_out, contours, hierachy, RETR_TREE, CHAIN_APPROX_SIMPLE, Point(0, 0));

	vector<Moments> contours_moments(contours.size());
	vector<Point2f> ccs(contours.size());

	for (size_t i = 0; i < contours.size(); i++)
	{
		//计算矩
		contours_moments[i] = moments(contours[i]);
		//计算图像中心，获取中心坐标
		ccs[i] = Point(static_cast<float>(contours_moments[i].m10 / contours_moments[i].m00), static_cast<float>(contours_moments[i].m01 / contours_moments[i].m00));
	}

	Mat drawImg;
	src.copyTo(drawImg);
	for (size_t i = 0; i < contours.size(); i++)
	{
		if (contours[i].size()<30)
		{
			continue;
		}

		//打印面积和长度
		printf("contours %d  area :%.2f and length :%.2f \n", i, contourArea(contours[i]), arcLength(contours[i], true));

		//随机颜色
		Scalar color = Scalar(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255));
		//轮廓绘制
		drawContours(drawImg, contours, i, color, 2, 8, hierachy, 0, Point(0, 0));
		//画圆
		circle(drawImg, ccs[i], 2, color, 2, 8);
	}

	imshow(output_title, drawImg);
}
```


#### 十、点多边形测试
##### 1、概念
> 测试一个点是否在给定的多边形内部，边缘或者外部

##### 2、API介绍 cv::pointPolygonTest

```
pointPolygonTest(
InputArray  contour,// 输入的轮廓
Point2f  pt, // 测试点
bool  measureDist // 是否返回距离值，如果是false，1表示在内面，0表示在边界上，-1表示在外部，true返回实际距离
)

返回数据是double类型
```
##### 3、步骤
- 构建一张400x400大小的图片， Mat::Zero(400, 400, CV_8UC1)
- 画上一个六边形的闭合区域line
- 发现轮廓
- 对图像中所有像素点做点 多边形测试，得到距离，归一化后显示。

##### 示例代码

```C++
#include<opencv2\opencv.hpp>
#include<iostream>

using namespace std;
using namespace cv;
Mat src, dst,gray_src;
int threshold_value = 80;
int threshold_max = 255;
RNG rng;
string input_title = "input img";
string output_title = "output img";
const int r = 100;
int main(int argc, char * argv) {

	src = Mat::zeros(r * 4, r * 4, CV_8UC1);

	vector<Point2f> vert(6);
	vert[0] = Point(3 * r / 2, static_cast<int>(1.34*r));
	vert[1] = Point(1 * r, static_cast<int>(2 * r));
	vert[2] = Point(3 * r / 2, static_cast<int>(2.886*r));
	vert[3] = Point(5 * r / 2, static_cast<int>(2.886*r));
	vert[4] = Point(3 * r, static_cast<int>(2 * r));
	vert[5] = Point(5 * r / 2, static_cast<int>(1.34*r));

	//画多边形
	for (int i = 0; i < 6; i++)
	{
		line(src, vert[i], vert[(i + 1) % 6], Scalar(255), 8, LINE_8, 0);
	}

	vector<vector<Point>> contours;
	vector<Vec4i> hierachy;
	Mat csrc;
	src.copyTo(csrc);
	//轮廓发现
	findContours(csrc, contours, RETR_TREE, CHAIN_APPROX_SIMPLE, Point(0, 0));

	Mat raw_dist = Mat::zeros(csrc.size(), CV_32FC1);
	for (int row = 0; row < raw_dist.rows; row++)
	{
		for (int col = 0; col < raw_dist.cols; col++)
		{
			//点多边形测试
			double dist = pointPolygonTest(contours[0], Point2f(static_cast<float>(col), static_cast<float>(row)), true);
			raw_dist.at<float>(row, col) = static_cast<float>(dist);
		}
	}

	double minValue, maxValue;
	//归一化处理
	minMaxLoc(raw_dist, &minValue, &maxValue, 0, 0, Mat());

	Mat drawImg = Mat::zeros(src.size(), CV_8UC3);
	for (int row = 0; row < drawImg.rows; row++)
	{
		for (int col = 0; col < drawImg.cols; col++)
		{
			float dist = raw_dist.at<float>(row, col);
			if (dist > 0) {
				drawImg.at<Vec3b>(row, col)[0] = (uchar)(abs(1.0 - (dist / maxValue)) * 255);
			}
			else if (dist < 0) {
				drawImg.at<Vec3b>(row, col)[2] = (uchar)(abs(1.0- (dist / maxValue)) * 255);
			}
			else {
				drawImg.at<Vec3b>(row, col)[0] = (uchar)(abs(255 - dist));
				drawImg.at<Vec3b>(row, col)[1] = (uchar)(abs(255 - dist));
				drawImg.at<Vec3b>(row, col)[2] = (uchar)(abs(255 - dist));
			}
		}
	}
		namedWindow(input_title, CV_WINDOW_AUTOSIZE);
		namedWindow(output_title, CV_WINDOW_AUTOSIZE);
		imshow(input_title, src);
		imshow(output_title, drawImg);

		waitKey(0);
		return 0;
}
```



#### 十一、基于距离变换与分水岭的图像分割
##### 1、概念
- 图像分割(Image Segmentation)是图像处理最重要的处理手段之一
- 图像分割的目标是将图像中像素根据一定的规则分为若干(N)个cluster集合，每个集合包含一类像素。
- 根据算法分为监督学习方法和无监督学习方法，图像分割的算法多数都是无监督学习方法 - KMeans

##### 2、距离变换与分水岭介绍
- 距离变换常见算法有两种
  - 不断膨胀/ 腐蚀得到
  - 基于倒角距离
 
- 分水岭变换常见的算法
  - 基于浸泡理论实现 

##### 3、相关API

```
cv::distanceTransform(InputArray  src, OutputArray dst,  OutputArray  labels,  int  distanceType,  int maskSize,  int labelType=DIST_LABEL_CCOMP)
        distanceType = DIST_L1/DIST_L2,
        maskSize = 3x3,最新的支持5x5，推荐3x3、
        labels离散维诺图输出
        dst输出8位或者32位的浮点数，单一通道，大小与输入图像一致

cv::watershed(InputArray image, InputOutputArray  markers)

```
##### 4、流程
- 1. 将白色背景变成黑色-目的是为后面的变换做准备
- 2. 使用filter2D与拉普拉斯算子实现图像对比度提高，sharp
- 3. 转为二值图像通过threshold
- 4. 距离变换
- 5. 对距离变换结果进行归一化到[0~1]之间
- 6. 使用阈值，再次二值化，得到标记
- 7. 腐蚀得到每个Peak - erode
- 8. 发现轮廓 – findContours
- 9. 绘制轮廓- drawContours
- 10. 分水岭变换 watershed
- 11. 对每个分割区域着色输出结果
- 
##### 示例代码

```C++
#include<opencv2\opencv.hpp>
#include<iostream>

using namespace std;
using namespace cv;
Mat src, dst;
int threshold_value = 100;
int threshold_max = 255;
RNG rng;
string input_title = "input img";
string output_title = "output img";

int main(int argc, char * argv) {
	src = imread("C:\\Users\\Administrator\\Pictures\\lab\\红桃.png");
	namedWindow(input_title, CV_WINDOW_AUTOSIZE);
	namedWindow(output_title, CV_WINDOW_AUTOSIZE);
	
	//修改背景颜色
	for (int row = 0; row < src.rows; row++)
	{
		for (int col = 0; col < src.cols; col++)
		{
			if (src.at<Vec3b>(row,col) == Vec3b(255,255,255))
			{
				src.at<Vec3b>(row, col)[0] = 0;
				src.at<Vec3b>(row, col)[1] = 0;
				src.at<Vec3b>(row, col)[2] = 0;
			}
		}
	}
	//膨胀（去除小黑点，噪点）
	Mat structureElement = getStructuringElement(MORPH_RECT, Size(5, 5), Point(-1, -1));
	dilate(src, src, structureElement, Point(-1, -1), 1);

	//锐化图像，shape
	Mat kernel = (Mat_<float>(3, 3) << 1, 1, 1, 1, -8, 1, 1, 1, 1);
	Mat imgLaplance;
	Mat shapenImg = src;
	filter2D(src, imgLaplance, CV_32F, kernel, Point(-1, -1), 0, BORDER_DEFAULT);
	src.convertTo(shapenImg, CV_32F);
	Mat resultImg = shapenImg - imgLaplance;
	resultImg.convertTo(resultImg, CV_8UC3);
	imgLaplance.convertTo(imgLaplance, CV_8UC3);

	//二值图像convert to binary
	Mat binaryImg;
	cvtColor(resultImg, resultImg, CV_BGR2GRAY);
	threshold(resultImg, binaryImg, 40, 255, CV_THRESH_BINARY | THRESH_OTSU);

	//距离变化
	Mat distImg;
	distanceTransform(binaryImg, distImg, DIST_L1, 3, 5);
	normalize(distImg, distImg, 0, 1, NORM_MINMAX);

	//距离变化之后再次二值化图像
	threshold(distImg, distImg, 0.4, 1, THRESH_BINARY);
	Mat kernel1 = Mat::zeros(13, 13, CV_8UC1);
	//腐蚀
	erode(distImg, distImg, kernel1);

	//markers标记
	Mat dist_8u;
	distImg.convertTo(dist_8u, CV_8U);
	vector<vector<Point>> contours;
	findContours(dist_8u, contours, RETR_TREE, CHAIN_APPROX_SIMPLE, Point(0, 0));

	//create makers
	Mat markers = Mat::zeros(src.size(), CV_32SC1);
	for (size_t i = 0; i < contours.size(); i++)
	{
		drawContours(markers, contours, static_cast<int>(i), Scalar::all (static_cast<int>(i) + 1),-1);
	}
	circle(markers, Point(5, 5), 3, Scalar(255, 255, 255), -1);


	//分水岭
	watershed(src, markers);
	Mat res = Mat::zeros(markers.size(), CV_8UC1);
	markers.convertTo(res, CV_8UC1);
	bitwise_not(res, res, Mat());

	//生成随机颜色
	vector<Vec3b> colors;
	for (size_t i = 0; i < contours.size(); i++)
	{
		int r = rng.uniform(0, 255);
		int g = rng.uniform(0, 255);
		int b = rng.uniform(0, 255);

		colors.push_back(Vec3b((uchar)b, (uchar)g, (uchar)r));
	}

	//填满颜色
	dst = Mat::zeros(markers.size(), CV_8UC3);
	for (int row = 0; row < src.rows; row++)
	{
		for (int col = 0; col < src.cols; col++)
		{
			int index = markers.at<int>(row, col);
			if (index>0 && index<=static_cast<int>(contours.size()))
			{
				dst.at<Vec3b>(row, col) = colors[index - 1];
			}
			else {
				dst.at<Vec3b>(row, col) = Vec3b(0, 0, 0);
			}
		}
	}


	imshow(input_title, src);
	imshow(output_title, dst);

	waitKey(0);
	return 0;
}
```
