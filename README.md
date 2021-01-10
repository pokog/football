# football



#include<opencv2/opencv.hpp>
#include<opencv2/highgui/highgui.hpp>
#include<opencv2/imgproc/imgproc.hpp>
using namespace std;
using namespace cv;

int main()
{
	VideoCapture video;
	Mat frame,tempframe;
	int framenum = 0;
	int inputKey;
	Mat previousframe, currentframe;
	Mat threshold_output;
	vector<vector<Point>>contours;
	vector<Vec4i>hierarchy;
	Mat element = getStructuringElement(MORPH_RECT, Size(4, 4));
	frame = video.open("test.avi");
	if (!video.isOpened())
	{
		printf("can not open ...\n");
		return -1;
	}

	while (video.read(frame))
	{
		framenum++;
		if (framenum == 110)
		{
			cvtColor(frame, previousframe, CV_BGR2GRAY);
		}
		if (framenum > 110)
		{
			//差帧法处理
			cvtColor(frame, currentframe, CV_BGR2GRAY);
			absdiff(previousframe, currentframe, currentframe);

			//平滑处理
			threshold(currentframe, currentframe, 20, 255.0, CV_THRESH_BINARY);
			morphologyEx(currentframe, currentframe, MORPH_OPEN, element);
			
			//边缘检测
			threshold(currentframe, threshold_output, 27, 255, THRESH_BINARY);
			findContours(threshold_output, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE, Point(0, 0));

			//获取图形边界
			int x;
			double maxArea = 0, area;
			vector<vector<Point>>contours_poly(contours.size());
			vector<Rect>boundRect(contours.size());
			for (unsigned int i = 0; i < contours.size(); i++)
			{
				approxPolyDP(Mat(contours[i]), contours_poly[i], 3, true);//多边形边界
				boundRect[i] = boundingRect(Mat(contours_poly[i]));//矩形边界
				area = contourArea(contours[i], false);
				if (maxArea < area)
				{
					x = i;
					maxArea = area;
				}
			}

			//绘制边框
			Mat drawing = Mat::zeros(threshold_output.size(), CV_8SC3);
			Scalar color = Scalar(0, 150, 0);
			for (int unsigned i = 0; i < contours.size(); i++)
			{
				drawContours(drawing, contours_poly, i, color, 1, 8, vector<Vec4i>(), 0, Point());
			}
			
			//绘制方框
			rectangle(frame, boundRect[x].tl(), boundRect[x].br(), color, 2, 8, 0);
			imshow("drawing", drawing);
			imshow("frame", frame);
			waitKey(0);
		}

	}
	return 0;
}
