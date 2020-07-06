# 一些内容

https://www.zhihu.com/question/26493945
https://blog.csdn.net/xiao__run/article/details/84374959
https://blog.csdn.net/varyshare/article/details/97642209

Kalman-and-Bayesian-Filters-in-Python 是一本书，许多关于卡尔曼和贝叶斯的内容

keras-yolov3-KF-objectTracking 修改的一个机遇KF的跟踪算法，目前在看：它的卡尔曼滤波器太简单了

opencv-master github上opencv的源码

Multitarget-tracker-master 使用了多个算法的一个多目标跟踪项目，编译出来了但是跑着有问题，原因是ffmpeg*.dll

windows的dll搜索路径包括path，可能需要重启生效
LBSP局部二值相似模式，前景减除算法SuBSENSE似乎用到了这个
http://fcv2011.ulsan.ac.kr/files/announcement/496/seminar_june.pdf
PAWCS也可以考虑看看

记录一下运行仓库Multitarget-tracker-master的过程，首先是github的opencv，然后给CMake生成，然后编译，可能会出现有ffmpeg、ippicv下载失败的情况，找到提示里给的CMakeDownload.log，然后根据log挂梯子下载需要的东西，然后找到.cache，没下载完的东西都在这里，然后将手动下载的东西修改名字之后放好
注意编译opencv的时候要选择Release版本，这个可以在一个json中进行设置
在编译Multitarget-tracker-master的时候，要提前设置其json的一个OpenCV的路径，设置为刚刚编译的那个，反正找一下就知道了，另外windows的话，还需要将opencv的dll的路径添加到path中，然后类似的，Multitarget-tracker-master也需要编译Release版本，编完之后运行终于正常了

yuv400转avi
ffmpeg -s 320*240 -pix_fmt gray -i highway_320x240.yuv -vcodec mpeg4 output.avi
avi转yuv
ffmpeg -i input.avi output.yuv
如果需要裁减
ffmpeg -i input.avi -vf crop=320:180 output.yuv
如果需要缩放
ffmpeg -i input.avi -vf scale=320:180 output.yuv

ubuntu上编译opencv和Multitarget-tracker-master，参考https://blog.csdn.net/u011897411/article/details/89743448和https://www.cnblogs.com/chenzhen0530/p/12109868.html

https://blog.csdn.net/Johnlee2019/article/details/99987007

==============================================
交叉编译opencv记录
https://blog.csdn.net/zdyueguanyun/article/details/51322295
https://worthsen.blog.csdn.net/article/details/77964315
$ aarch64-himix100-linux-gcc -print-sysroot
/home/ippfcox/aarch64_toolchain/aarch64-himix100-linux/aarch64-himix100-linux/host_bin/../aarch64-linux-gnu/libc

使用cmake-gui进行配置

先configure，配置交叉编译器gcc、g++的路径和搜索路径（搜索路径后面还可以在下面的框框中改）

选中
BUILD_JASPER,
BUILD_JPEG,
BUILD_ZLLIB,
BUILD_WITH_DEBUG_INFO, 
BUILD_opencv_world  (只編譯成一個大的函數庫)
取消选中
BUILD_DOCS,
BUILD_PERF_TESTS
BUILD_TESTS,
BUILD_FAT_JAVA_LIB,
BUIlD_openvc_app
BUILD_SHARED_LIBS,
WITH_CUDA
WITH_PNG,
WITH_TIFF,
WITH_WEBP

修改 CMAKE_INSTALL_PREFIX = 为我们想要的目标路径

勾选Advanced
设置
CMAKE_EXE_LINKER_FLAGS= -lpthread -ldl -lrt
CMAKE_C_FLAGS = -DWITH_PARALLEL_PF=OFF
CMAKE_CXX_FALGS = -DWITH_PARALLEL_PF=OFF

错误No suitable threading library available
定位到文件，添加HAVE_PTHREAD宏（这个感觉很奇怪，是不是搜索路径有问题）
错误grfmt_jpeg.cpp中的几个类型转换问题
定位到文件强转即可

原有的opencv的路径
/usr/local/lib/cmake/opencv4
原有CMAKE_CXX_FLAGS_DEBUG -O0 -g -march=native -mtune=native -Wall -DDEBUG

原有CMAKE_CXX_FLAGS_RELEASE -O3 -g -march=native -mtune=native -funroll-loops -Wall -DNDEBUG -DBOOST_DISABLE_ASSERTS

VS2019调试Cmake项目，https://docs.microsoft.com/zh-cn/visualstudio/ide/customize-build-and-debug-tasks-in-visual-studio?view=vs-2019，添加命令行参数时，配置launch.vs.json，重要的是添加args项目

# Eigen

使用时将下载的包里面的Eigen目录复制出来即可，包括src目录

```c++
// A simple quickref for Eigen. Add anything that's missing.
// Main author: Keir Mierle

#include <Eigen/Dense>

Matrix<double, 3, 3> A;               // Fixed rows and cols. Same as Matrix3d. 
Matrix<double, 3, Dynamic> B;         // Fixed rows, dynamic cols. 动态列数的默认值为0
Matrix<double, Dynamic, Dynamic> C;   // Full dynamic. Same as MatrixXd.
Matrix<double, 3, 3, RowMajor> E;     // Row major; default is column-major.
Matrix3f P, Q, R;                     // 3x3 float matrix.
Vector3f x, y, z;                     // 3x1 float matrix.
RowVector3f a, b, c;                  // 1x3 float matrix.
VectorXd v;                           // Dynamic column vector of doubles，这里的d是double
double s;                            

// Basic usage
// Eigen          // Matlab           // comments
x.size()          // length(x)        // vector size  元素总长度
C.rows()          // size(C,1)        // number of rows
C.cols()          // size(C,2)        // number of columns
x(i)              // x(i+1)           // Matlab is 1-based
C(i,j)            // C(i+1,j+1)       //

A.resize(4, 4);   // Runtime error if assertions are on.
B.resize(4, 9);   // Runtime error if assertions are on.
A.resize(3, 3);   // Ok; size didn't change.
B.resize(3, 9);   // Ok; only dynamic cols changed. resize时只能改动长度为Dynamic维的，3x3改成1x9也不行，3x3改成2x3或3x2也不行
                  
A << 1, 2, 3,     // Initialize A. The elements can also be
     4, 5, 6,     // matrices, which are stacked along cols
     7, 8, 9;     // and then the rows are stacked.
B << A, A, A;     // B is three horizontally stacked A's.
A.fill(10);       // Fill A with all 10's.

// Eigen                                    // Matlab
MatrixXd::Identity(rows,cols)               // eye(rows,cols)
C.setIdentity(rows,cols)                    // C = eye(rows,cols)
MatrixXd::Zero(rows,cols)                   // zeros(rows,cols)
C.setZero(rows,cols)                        // C = zeros(rows,cols)
MatrixXd::Ones(rows,cols)                   // ones(rows,cols)
C.setOnes(rows,cols)                        // C = ones(rows,cols)
MatrixXd::Random(rows,cols)                 // rand(rows,cols)*2-1            // MatrixXd::Random returns uniform random numbers in (-1, 1).
C.setRandom(rows,cols)                      // C = rand(rows,cols)*2-1
VectorXd::LinSpaced(size,low,high)          // linspace(low,high,size)'
v.setLinSpaced(size,low,high)               // v = linspace(low,high,size)'
VectorXi::LinSpaced(((hi-low)/step)+1,      // low:step:hi
                    low,low+step*(size-1))  //


// Matrix slicing and blocks. All expressions listed here are read/write.
// Templated size versions are faster. Note that Matlab is 1-based (a size N
// vector is x(1)...x(N)).
// Eigen                           // Matlab
x.head(n)                          // x(1:n)
x.head<n>()                        // x(1:n)
x.tail(n)                          // x(end - n + 1: end)
x.tail<n>()                        // x(end - n + 1: end)
x.segment(i, n)                    // x(i+1 : i+n)
x.segment<n>(i)                    // x(i+1 : i+n)
P.block(i, j, rows, cols)          // P(i+1 : i+rows, j+1 : j+cols)
P.block<rows, cols>(i, j)          // P(i+1 : i+rows, j+1 : j+cols)
P.row(i)                           // P(i+1, :)
P.col(j)                           // P(:, j+1)
P.leftCols<cols>()                 // P(:, 1:cols)
P.leftCols(cols)                   // P(:, 1:cols)
P.middleCols<cols>(j)              // P(:, j+1:j+cols)
P.middleCols(j, cols)              // P(:, j+1:j+cols)
P.rightCols<cols>()                // P(:, end-cols+1:end)
P.rightCols(cols)                  // P(:, end-cols+1:end)
P.topRows<rows>()                  // P(1:rows, :)
P.topRows(rows)                    // P(1:rows, :)
P.middleRows<rows>(i)              // P(i+1:i+rows, :)
P.middleRows(i, rows)              // P(i+1:i+rows, :)
P.bottomRows<rows>()               // P(end-rows+1:end, :)
P.bottomRows(rows)                 // P(end-rows+1:end, :)
P.topLeftCorner(rows, cols)        // P(1:rows, 1:cols)
P.topRightCorner(rows, cols)       // P(1:rows, end-cols+1:end)
P.bottomLeftCorner(rows, cols)     // P(end-rows+1:end, 1:cols)
P.bottomRightCorner(rows, cols)    // P(end-rows+1:end, end-cols+1:end)
P.topLeftCorner<rows,cols>()       // P(1:rows, 1:cols)
P.topRightCorner<rows,cols>()      // P(1:rows, end-cols+1:end)
P.bottomLeftCorner<rows,cols>()    // P(end-rows+1:end, 1:cols)
P.bottomRightCorner<rows,cols>()   // P(end-rows+1:end, end-cols+1:end)

// Of particular note is Eigen's swap function which is highly optimized.
// Eigen                           // Matlab
R.row(i) = P.col(j);               // R(i, :) = P(:, j)
R.col(j1).swap(mat1.col(j2));      // R(:, [j1 j2]) = R(:, [j2, j1])

// Views, transpose, etc;
// Eigen                           // Matlab
R.adjoint()                        // R'
R.transpose()                      // R.' or conj(R')       // Read-write
R.diagonal()                       // diag(R)               // Read-write
x.asDiagonal()                     // diag(x)
R.transpose().colwise().reverse()  // rot90(R)              // Read-write
R.rowwise().reverse()              // fliplr(R)
R.colwise().reverse()              // flipud(R)
R.replicate(i,j)                   // repmat(P,i,j)


// All the same as Matlab, but matlab doesn't have *= style operators.
// Matrix-vector.  Matrix-matrix.   Matrix-scalar.
y  = M*x;          R  = P*Q;        R  = P*s;
a  = b*M;          R  = P - Q;      R  = s*P;
a *= M;            R  = P + Q;      R  = P/s;
                   R *= Q;          R  = s*P;
                   R += Q;          R *= s;
                   R -= Q;          R /= s;

// Vectorized operations on each element independently
// Eigen                       // Matlab
R = P.cwiseProduct(Q);         // R = P .* Q
R = P.array() * s.array();     // R = P .* s
R = P.cwiseQuotient(Q);        // R = P ./ Q
R = P.array() / Q.array();     // R = P ./ Q
R = P.array() + s.array();     // R = P + s
R = P.array() - s.array();     // R = P - s
R.array() += s;                // R = R + s
R.array() -= s;                // R = R - s
R.array() < Q.array();         // R < Q
R.array() <= Q.array();        // R <= Q
R.cwiseInverse();              // 1 ./ P
R.array().inverse();           // 1 ./ P
R.array().sin()                // sin(P)
R.array().cos()                // cos(P)
R.array().pow(s)               // P .^ s
R.array().square()             // P .^ 2
R.array().cube()               // P .^ 3
R.cwiseSqrt()                  // sqrt(P)
R.array().sqrt()               // sqrt(P)
R.array().exp()                // exp(P)
R.array().log()                // log(P)
R.cwiseMax(P)                  // max(R, P)
R.array().max(P.array())       // max(R, P)
R.cwiseMin(P)                  // min(R, P)
R.array().min(P.array())       // min(R, P)
R.cwiseAbs()                   // abs(P)
R.array().abs()                // abs(P)
R.cwiseAbs2()                  // abs(P.^2)
R.array().abs2()               // abs(P.^2)
(R.array() < s).select(P,Q );  // (R < s ? P : Q)
R = (Q.array()==0).select(P,R) // R(Q==0) = P(Q==0)
R = P.unaryExpr(ptr_fun(func)) // R = arrayfun(func, P)   // with: scalar func(const scalar &x);


// Reductions.
int r, c;
// Eigen                  // Matlab
R.minCoeff()              // min(R(:))
R.maxCoeff()              // max(R(:))
s = R.minCoeff(&r, &c)    // [s, i] = min(R(:)); [r, c] = ind2sub(size(R), i);
s = R.maxCoeff(&r, &c)    // [s, i] = max(R(:)); [r, c] = ind2sub(size(R), i);
R.sum()                   // sum(R(:))
R.colwise().sum()         // sum(R)
R.rowwise().sum()         // sum(R, 2) or sum(R')'
R.prod()                  // prod(R(:))
R.colwise().prod()        // prod(R)
R.rowwise().prod()        // prod(R, 2) or prod(R')'
R.trace()                 // trace(R)
R.all()                   // all(R(:))
R.colwise().all()         // all(R)
R.rowwise().all()         // all(R, 2)
R.any()                   // any(R(:))
R.colwise().any()         // any(R)
R.rowwise().any()         // any(R, 2)

// Dot products, norms, etc.
// Eigen                  // Matlab
x.norm()                  // norm(x).    Note that norm(R) doesn't work in Eigen.
x.squaredNorm()           // dot(x, x)   Note the equivalence is not true for complex
x.dot(y)                  // dot(x, y)
x.cross(y)                // cross(x, y) Requires #include <Eigen/Geometry>

//// Type conversion
// Eigen                  // Matlab
A.cast<double>();         // double(A)
A.cast<float>();          // single(A)
A.cast<int>();            // int32(A)
A.real();                 // real(A)
A.imag();                 // imag(A)
// if the original type equals destination type, no work is done

// Note that for most operations Eigen requires all operands to have the same type:
MatrixXf F = MatrixXf::Zero(3,3);
A += F;                // illegal in Eigen. In Matlab A = A+F is allowed
A += F.cast<double>(); // F converted to double and then added (generally, conversion happens on-the-fly)

// Eigen can map existing memory into Eigen matrices.
float array[3];
Vector3f::Map(array).fill(10);            // create a temporary Map over array and sets entries to 10
int data[4] = {1, 2, 3, 4};
Matrix2i mat2x2(data);                    // copies data into mat2x2
Matrix2i::Map(data) = 2*mat2x2;           // overwrite elements of data with 2*mat2x2
MatrixXi::Map(data, 2, 2) += mat2x2;      // adds mat2x2 to elements of data (alternative syntax if size is not know at compile time)

// Solve Ax = b. Result stored in x. Matlab: x = A \ b.
x = A.ldlt().solve(b));  // A sym. p.s.d.    #include <Eigen/Cholesky>
x = A.llt() .solve(b));  // A sym. p.d.      #include <Eigen/Cholesky>
x = A.lu()  .solve(b));  // Stable and fast. #include <Eigen/LU>
x = A.qr()  .solve(b));  // No pivoting.     #include <Eigen/QR>
x = A.svd() .solve(b));  // Stable, slowest. #include <Eigen/SVD>
// .ldlt() -> .matrixL() and .matrixD()
// .llt()  -> .matrixL()
// .lu()   -> .matrixL() and .matrixU()
// .qr()   -> .matrixQ() and .matrixR()
// .svd()  -> .matrixU(), .singularValues(), and .matrixV()

// Eigenvalue problems
// Eigen                          // Matlab
A.eigenvalues();                  // eig(A);
EigenSolver<Matrix3d> eig(A);     // [vec val] = eig(A)
eig.eigenvalues();                // diag(val)
eig.eigenvectors();               // vec
// For self-adjoint matrices use SelfAdjointEigenSolver<>
```

# npm

windows下安装，安装node.js

linux下安装

```bash
sudo apt-get install npm
```

下来看版本号

```bash
npm -v
```

换淘宝源

```bash
npm config set registry https://registry.npm.taobao.org  # 永久使用
npm config get registry
npm info express
npm --registry https://registry.npm.taobao.org install express  # 临时使用
```

# 遇到一个现象C++/queue

### 现象

定义结构体obj，里面有个队列q（不是指针）

使用时malloc结构体大小的内存，然后obj->q.push(xx)，第一个元素没问题，第二个元素报错，段错误，应该是访问不该访问的地方了

### 修改

队列中q改为指针，使用时q = new queue<>()申请内存，此时再obj->q->push()，多个都不会出错了

### 猜测

给结构体申请内存时就申请了一小块队列的内存（会不会是当时队列长度？），往里push多个的时候就会出错，但是不确定，印象中之前是用过这种用法的，也并没有错，找机会深究一下

# 信号捕获问题

可以使用signal(signo, sig_handle)捕获信号，有些信号时可以捕获的，但有些是捕获不了的，比如ctrl-c，是SIGINT(2)，可以捕获；kill pid，是SIGTERM(15)，可以捕获；kill -9 pid，是SIGKILL(9)，不能捕获

# c++的split

c++自己是没有split的，也不算完全没有，但是……，这个是比较优雅的实现：

```cpp
static void split(const std::string& s, std::vector<std::string>& tokens, const std::string& delimiters = " ")
{
        std::string::size_type lastPos = s.find_first_not_of(delimiters, 0);
        std::string::size_type pos = s.find_first_of(delimiters, lastPos);
        while (std::string::npos != pos || std::string::npos != lastPos) {
                tokens.push_back(s.substr(lastPos, pos - lastPos));
                lastPos = s.find_first_not_of(delimiters, pos);
                pos = s.find_first_of(delimiters, lastPos);
        }
}
```

c++的字符串转数字

```cpp
std::stoi();
std::stol();
std::stoll();
```

# 运算符重载

```cpp
class Point
{
public:
    Point(int x, int y) : m_x(x), m_y(y){}
    
    friend std::ostream & operator<< (std::ostream &out, Point &p)
    {
        out << "(" << p.m_x << ", " << p.m_y <<")";
    }

private:
    int m_x;
    int m_y;
};
```

似乎得写成这个样子才能正常使用

