# Numpy

```python
a = np.random.randint(5, size=(5,5))  # 返回一个5*5的{0,1,2,3,4}的随机数二维数组
a = np.random.randn(10000) * 0.1 + 5  # 生成均值为5，标准差为0.1的正态分布随机数10000个
np.where(a < 4)  # 返回一个元组，两个元素分别对应横纵坐标
a.shape  # a是np.ndarray，shape是一个属性，为其高宽的长度
np.arange(10, 30, 5)  # 10到30，左闭右开，步进为5，如果为浮点数时尽量不要用这个，元素数目不一定是我们想要的（浮点数精度问题）
np.linspace(0, 2, 9)  # 即0到2取9个数，用这个来代替前述
```

# OpenCV

```python
cv::Mat inversedMat = 255 - originalMat  # 对8位图像，计算反色
retval, labels, stats, centroids = cv2.connectedComponentsWithStats(src_gray)  # 计算连通区，并给出连通区信息
```

gemm：下面两式等价

```python
d = cv2.gemm(a, b, alpha, b, beta, flags=cv2.GEMM_2_T)
e = alpha * np.dot(a, b.T) + beta * c
```

要注意gemm的输入类型只能是float