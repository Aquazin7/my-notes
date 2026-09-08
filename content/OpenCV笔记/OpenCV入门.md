

C++ Opencv 4.x的基本操作：图片创建，读取，显示，访问，保存：



### 准备工作：导入库和头文件

首先CMakeLists.txt导入Opencv库：

```cmake
cmake_minimum_required(VERSION 4.0)
project(DeepLearningCpp)
set(CMAKE_CXX_STANDARD 20)
if(MSVC)
    add_compile_options(/utf-8)
endif()

find_package(OpenCV CONFIG REQUIRED)
add_executable(Main main.cpp)

#预编译头技术，缓存构筑文件用于加快编译（可选）
#target_precompile_headers(Main PRIVATE <opencv2/opencv.hpp>)
target_link_libraries(Main PRIVATE ${OpenCV_LIBS})
```

需要包含核心头文件，并使用 `cv` 命名空间。

```cpp
#include <opencv2/opencv.hpp>  // 包含了OpenCV的主要模块
#include <iostream>
```



### 图片对象cv::Mat

在Opencv里最基本的图像对象是`cv::Mat`

cv::Mat的主要分为两部字段：

- 矩阵头：图片的信息，比如宽、高、通道数、数据类型、数据指针等等
- 图片数据：就是图片本身的像素数据



### 图片创建

你可以创建空图、指定大小的图、带颜色的图，或基于现有图像创建副本。

*   **创建空Mat对象**：只创建了一个空壳，没有实际数据。
```cpp
cv::Mat image; 
```

* **创建指定大小的图像**：创建一个480x640的彩色图像（三通道，8位无符号整数）。
```cpp
// 参数顺序：(行数, 列数, 数据类型)，即 (高度, 宽度, 数据类型宏)
cv::Mat image(480, 640, CV_8UC3); 
```
  
*   **创建指定颜色的图像**：创建一个480x640的红色图像。注意OpenCV默认通道顺序是 **BGR**（蓝-绿-红）。
```cpp
// Scalar(B, G, R)
cv::Mat redImage(480, 640, CV_8UC3, cv::Scalar(0, 0, 255)); 
```

*   **创建特殊矩阵（单位阵、全0、全1矩阵）**：
```cpp
cv::Mat E = Mat::eye(4, 4, CV_64F); // 4x4 单位矩阵
cv::Mat Z = Mat::zeros(3, 3, CV_8UC1); // 3x3 全0矩阵
cv::Mat O = Mat::ones(2, 2, CV_32F); // 2x2 全1矩阵
```

* **深拷贝与浅拷贝**：

  *   **浅拷贝**：复制矩阵头，**共享**数据，修改一个会影响另一个。
	```cpp
	cv::Mat src = cv::imread("image.jpg");
    cv::Mat dst = src; // 浅拷贝
	```

  
  *   **深拷贝**：复制矩阵头和独立的数据副本，**互不影响**。
```cpp
cv::Mat dst = src.clone(); // 深拷贝
      // 或
src.copyTo(dst); // 深拷贝
  
```

- **基于其他数据类型创建**：
  注意这种方式下，cv::Mat不负责管理数据的内存释放。
  ```c++
  //从std::array创建
  std::array< std::array<uint8_t, 128>, 128> arr{};
  cv::Mat img(128,128,CV_8UC1,arr.data()); // 把array的数据指针传给Mat
  ```

  

**补充：**

也可以用`cv::Mat_<T>`创建图像对象，这是个模板类继承自`cv::Mat`进行了简单封装，所以可以沿用cv::Mat的各种操作，同时也更安全。

```c++
cv::Mat_<uint8_t> img(480, 640); //相当于cv::Mat img(480, 640, CV_8UC1); 
cv::Mat_<cv::Vec3b> img2(480, 640); //相当于cv::Mat img2(480, 640, CV_8UC3);
```

**数据类型宏：**

OpenCV 中常见的数据类型包括：

| 数据类型   | 含义               |
| ---------- | ------------------ |
| `CV_8UC1`  | 8位无符号、单通道  |
| `CV_8UC3`  | 8位无符号、三通道  |
| `CV_8UC4`  | 8位无符号、四通道  |
| `CV_16UC1` | 16位无符号、单通道 |
| `CV_32FC1` | 32位浮点、单通道   |
| `CV_32FC3` | 32位浮点、三通道   |
| `CV_64FC1` | 64位浮点、单通道   |

以 `CV_8UC3` 为例，可以拆分为：

```text
CV_ + 8U + C3
```

其中：

- `8U` 表示每个通道使用 8 位无符号整数；
- `C3` 表示图像有三个通道。

彩色图像通常使用：`CV_8UC3`

灰度图通常使用：`CV_8UC1`

进行梯度计算、卷积或者坐标计算时，则经常使用：`CV_32F` 或 `CV_64F`



###  图片读取

使用 `imread()` 函数加载图片。

```cpp
// imread(const String& filename, int flags = IMREAD_COLOR)
Mat img = imread("path/to/your/image.jpg", IMREAD_COLOR);

// 检查是否读取成功
if (img.empty()) {
    cout << "Could not read the image." << endl;
    return -1;
}
```

*   **`flags` 参数**：指定读取方式。
    *   `IMREAD_COLOR` (>0)：加载为彩色图像，默认。
    *   `IMREAD_GRAYSCALE` (0)：加载为灰度图像。
    *   `IMREAD_UNCHANGED` (<0)：加载原图（包括alpha通道）。



### 图片显示 

显示图像需要创建窗口并在其中展示。

```cpp
// 1. 创建窗口（可选）
namedWindow("My Window", WINDOW_AUTOSIZE);
// 2. 在窗口中显示图像
imshow("My Window", img);

// 3. 等待键盘输入（会阻塞程序运行）
int key = waitKey(0); // 参数为0时无限等待按键
// 如果按下 's' 键，则保存图像
if (key == 's') {
    imwrite("saved_image.png", img);
}
// 4. 销毁所有窗口（可选）
destroyAllWindows();
```

*   **`namedWindow()` 的 `flags` 参数**：
    *   `WINDOW_AUTOSIZE`：窗口大小自适应图像。
    *   `WINDOW_FREERATIO`：窗口可自由调整比例。
    *   `WINDOW_FULLSCREEN`：全屏显示。
*   **`waitKey()` 函数**：参数为等待时间（毫秒），0 表示无限等待。返回值是按键的ASCII码。



### 图片访问 

对于cv::Mat对象像素点的访问方式

*   **使用 `at<Type>(row, col)` 方法**：安全且直观。
    
```cpp
    // 1. 访问灰度图（单通道）像素：at<像素数据类型>(行, 列);
    uchar intensity = gray_img.at<uchar>(y, x);
	gray_img.at<uchar>(y, x) = 128; // 修改像素值
    
    // 2. 访问彩色图（三通道）像素
    Vec3b pixel = color_img.at<Vec3b>(y, x);
    uchar blue = pixel[0];   // B通道
    uchar green = pixel[1];  // G通道
	uchar red = pixel[2];    // R通道
    
    // 修改彩色图像像素
    color_img.at<Vec3b>(y, x) = Vec3b(255, 0, 0); // 将该像素设为蓝色 (BGR)
    ```
```
    
*   **使用指针 `ptr<Type>(row)` 方法**：效率高，适用于遍历。
    ```cpp
    for (int r = 0; r < img.rows; r++) {
        Vec3b* ptr = img.ptr<Vec3b>(r); // 获取第r行首地址
        for (int c = 0; c < img.cols; c++) {
            // 通过 ptr[c] 访问当前行的第c列像素
            ptr[c][0] = 0; // 修改B通道
        }
    }
```

*   **获取图像属性**：
    ```cpp
    int height = img.rows;   // 图像高度（行数）
    int width = img.cols;    // 图像宽度（列数）
    int channels = img.channels(); // 通道数
    Size size = img.size();  // 图像尺寸 (width, height)
    ```



### 图片保存

使用 `imwrite()` 函数保存图像。

```cpp
// imwrite(const String& filename, InputArray img)
bool success = imwrite("output.jpg", img);
if (success) {
    cout << "Image saved successfully." << endl;
} else {
    cout << "Failed to save image." << endl;
}
```



### 总结与快速参考

为了方便快速查阅，核心API如下：

| 操作           | 核心函数/方法                      | 关键说明                                  |
| :------------- | :--------------------------------- | :---------------------------------------- |
| **包含头文件** | `#include <opencv2/opencv.hpp>`    | 包含所有主要模块                          |
| **读取图像**   | `imread(filename, flags)`          | `flags` 控制色彩模式，默认 `IMREAD_COLOR` |
| **显示图像**   | `imshow(winname, mat)`             | 需配合 `waitKey()` 使用                   |
| **创建窗口**   | `namedWindow(winname, flags)`      | `flags` 控制窗口特性，可选                |
| **等待按键**   | `waitKey(delay)`                   | `delay=0` 时无限等待                      |
| **保存图像**   | `imwrite(filename, mat)`           | 格式由文件扩展名决定                      |
| **创建图像**   | `Mat img(rows, cols, type)`        | `type` 如 `CV_8UC3`                       |
| **访问像素**   | `mat.at<Type>(row, col)`           | 安全访问，彩色图用 `Vec3b`                |
| **高效遍历**   | `mat.ptr<Type>(row)`               | 获取行指针，效率更高                      |
| **深拷贝**     | `mat.clone()` 或 `mat.copyTo(dst)` | 复制独立数据                              |

