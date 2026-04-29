---
name: matlab
description: MATLAB和GNU Octave数值计算工具，用于矩阵运算、数据分析、可视化和科学计算。适用于编写MATLAB/Octave脚本处理线性代数、信号处理、图像处理、微分方程、优化、统计或创建科学可视化场景。当用户需要MATLAB语法帮助、函数指导或需在MATLAB与Python代码间转换时也可使用。脚本可通过MATLAB或开源GNU Octave解释器执行。
license: MATLAB许可证(https://www.mathworks.com/pricing-licensing.html)，Octave采用GNU通用公共许可证第3版
compatibility: 测试需安装MATLAB或Octave，仅生成脚本则无需安装
metadata:
    skill-author: K-Dense Inc.
---

# MATLAB/Octave科学计算

MATLAB是专为矩阵运算和科学计算优化的数值计算环境。GNU Octave是具备高度MATLAB兼容性的免费开源替代方案。

## 快速入门

**运行MATLAB脚本：**
```bash
# MATLAB (商业版)
matlab -nodisplay -nosplash -r "run('script.m'); exit;"

# GNU Octave (免费开源版)
octave script.m
```

**安装GNU Octave：**
```bash
# macOS
brew install octave

# Ubuntu/Debian
sudo apt install octave

# Windows - 从 https://octave.org/download 下载
```

## 核心功能

### 1. 矩阵运算

MATLAB基础操作围绕矩阵和数组展开：

```matlab
% 创建矩阵
A = [1 2 3; 4 5 6; 7 8 9];  % 3x3矩阵
v = 1:10;                     % 1到10的行向量
v = linspace(0, 1, 100);      % 0到1的100个等距点

% 特殊矩阵
I = eye(3);          % 单位矩阵
Z = zeros(3, 4);     % 3x4零矩阵
O = ones(2, 3);      % 2x3全1矩阵
R = rand(3, 3);      % 均匀分布随机矩阵
N = randn(3, 3);     % 正态分布随机矩阵

% 矩阵运算
B = A';              % 转置
C = A * B;           % 矩阵乘法
D = A .* B;          % 逐元素乘法
E = A \ b;           % 解线性方程组 Ax = b
F = inv(A);          % 矩阵求逆
```

完整矩阵运算参考[references/matrices-arrays.md](references/matrices-arrays.md)。

### 2. 线性代数

```matlab
% 特征值与特征向量
[V, D] = eig(A);     % V: 特征向量, D: 特征值对角阵

% 奇异值分解
[U, S, V] = svd(A);

% 矩阵分解
[L, U] = lu(A);      % LU分解
[Q, R] = qr(A);      % QR分解
R = chol(A);         % Cholesky分解(对称正定矩阵)

% 解线性方程组
x = A \ b;           % 推荐方法
x = linsolve(A, b);  % 带选项求解
x = inv(A) * b;      % 效率较低
```

完整线性代数参考[references/mathematics.md](references/mathematics.md)。

### 3. 绘图与可视化

```matlab
% 二维绘图
x = 0:0.1:2*pi;
y = sin(x);
plot(x, y, 'b-', 'LineWidth', 2);
xlabel('x'); ylabel('sin(x)');
title('正弦波形');
grid on;

% 多图叠加
hold on;
plot(x, cos(x), 'r--');
legend('sin', 'cos');
hold off;

% 三维曲面
[X, Y] = meshgrid(-2:0.1:2, -2:0.1:2);
Z = X.^2 + Y.^2;
surf(X, Y, Z);
colorbar;

% 保存图形
saveas(gcf, 'plot.png');
print('-dpdf', 'plot.pdf');
```

完整可视化指南参考[references/graphics-visualization.md](references/graphics-visualization.md)。

### 4. 数据导入导出

```matlab
% 读取表格数据
T = readtable('data.csv');
M = readmatrix('data.csv');

% 写入数据
writetable(T, 'output.csv');
writematrix(M, 'output.csv');

% MAT文件(MATLAB原生格式)
save('data.mat', 'A', 'B', 'C');  % 保存变量
load('data.mat');                   % 全部加载
S = load('data.mat', 'A');         % 加载指定变量

% 图像处理
img = imread('image.png');
imwrite(img, 'output.jpg');
```

完整I/O指南参考[references/data-import-export.md](references/data-import-export.md)。

### 5. 控制流与函数

```matlab
% 条件语句
if x > 0
    disp('正数');
elseif x < 0
    disp('负数');
else
    disp('零');
end

% 循环语句
for i = 1:10
    disp(i);
end

while x > 0
    x = x - 1;
end

% 函数(独立.m文件或同文件)
function y = myfunction(x, n)
    y = x.^n;
end

% 匿名函数
f = @(x) x.^2 + 2*x + 1;
result = f(5);  % 36
```

完整编程指南参考[references/programming.md](references/programming.md)。

### 6. 统计与数据分析

```matlab
% 描述性统计
m = mean(data);
s = std(data);
v = var(data);
med = median(data);
[minVal, minIdx] = min(data);
[maxVal, maxIdx] = max(data);

% 相关性分析
R = corrcoef(X, Y);
C = cov(X, Y);

% 线性回归
p = polyfit(x, y, 1);  % 线性拟合
y_fit = polyval(p, x);

% 移动统计
y_smooth = movmean(y, 5);  % 5点移动平均
```

统计参考见[references/mathematics.md](references/mathematics.md)。

### 7. 微分方程

```matlab
% 常微分方程求解
% dy/dt = -2y, y(0) = 1
f = @(t, y) -2*y;
[t, y] = ode45(f, [0 5], 1);
plot(t, y);

% 高阶方程: y'' + 2y' + y = 0
% 转换系统: y1' = y2, y2' = -2*y2 - y1
f = @(t, y) [y(2); -2*y(2) - y(1)];
[t, y] = ode45(f, [0 10], [1; 0]);
```

ODE求解器指南参考[references/mathematics.md](references/mathematics.md)。

### 8. 信号处理

```matlab
% 快速傅里叶变换
Y = fft(signal);
f = (0:length(Y)-1) * fs / length(Y);
plot(f, abs(Y));

% 滤波处理
b = fir1(50, 0.3);           % FIR滤波器设计
y_filtered = filter(b, 1, signal);

% 卷积运算
y = conv(x, h, 'same');
```

信号处理参考[references/mathematics.md](references/mathematics.md)。

## 常用模式

### 模式1：数据分析流程

```matlab
% 加载数据
data = readtable('experiment.csv');

% 数据清洗
data = rmmissing(data);  % 移除缺失值

% 分析处理
grouped = groupsummary(data, 'Category', 'mean', 'Value');

% 可视化
figure;
bar(grouped.Category, grouped.mean_Value);
xlabel('类别'); ylabel('均值');
title('按类别统计结果');

% 保存
writetable(grouped, 'results.csv');
saveas(gcf, 'results.png');
```

### 模式2：数值模拟

```matlab
% 参数设置
L = 1; N = 100; T = 10; dt = 0.01;
x = linspace(0, L, N);
dx = x(2) - x(1);

% 初始条件
u = sin(pi * x);

% 时间步进(热传导方程)
for t = 0:dt:T
    u_new = u;
    for i = 2:N-1
        u_new(i) = u(i) + dt/(dx^2) * (u(i+1) - 2*u(i) + u(i-1));
    end
    u = u_new;
end

plot(x, u);
```

### 模式3：批处理

```matlab
% 多文件处理
files = dir('data/*.csv');
results = cell(length(files), 1);

for i = 1:length(files)
    data = readtable(fullfile(files(i).folder, files(i).name));
    results{i} = analyze(data);  % 自定义分析函数
end

% 结果合并
all_results = vertcat(results{:});
```

## 参考文件

- **[matrices-arrays.md](references/matrices-arrays.md)** - 矩阵创建、索引、操作与运算
- **[mathematics.md](references/mathematics.md)** - 线性代数、微积分、常微分方程、优化、统计
- **[graphics-visualization.md](references/graphics-visualization.md)** - 2D/3D绘图、定制化、导出
- **[data-import-export.md](references/data-import-export.md)** - 文件I/O、表格、数据格式
- **[programming.md](references/programming.md)** - 函数、脚本、控制流、面向对象
- **[python-integration.md](references/python-integration.md)** - MATLAB与Python互调用
- **[octave-compatibility.md](references/octave-compatibility.md)** - MATLAB与GNU Octave差异
- **[executing-scripts.md](references/executing-scripts.md)** - 脚本执行与测试指南

## GNU Octave兼容性

GNU Octave高度兼容MATLAB，多数脚本无需修改即可运行。主要差异：

- 注释符可用`#`或`%`(MATLAB仅支持`%`)
- Octave支持`++`、`--`、`+=`运算符
- 部分工具箱函数在Octave中不可用
- Octave包管理使用`pkg load`命令

完整兼容性指南参考[references/octave-compatibility.md](references/octave-compatibility.md)。

## 最佳实践

1. **向量化运算** - 尽可能避免循环：
   ```matlab
   % 低效方式
   for i = 1:1000
       y(i) = sin(x(i));
   end

   % 高效方式
   y = sin(x);
   ```

2. **预分配数组** - 避免循环中动态扩展数组：
   ```matlab
   % 低效方式
   for i = 1:1000
       y(i) = i^2;
   end

   % 高效方式
   y = zeros(1, 1000);
   for i = 1:1000
       y(i) = i^2;
   end
   ```

3. **选用合适数据类型** - 表格处理混合数据，矩阵处理数值数据：
   ```matlab
   % 数值数据
   M = readmatrix('numbers.csv');

   % 带标题混合数据
   T = readtable('mixed.csv');
   ```

4. **注释与文档** - 使用函数帮助说明：
   ```matlab
   function y = myfunction(x)
   %MYFUNCTION 功能简述
   %   Y = MYFUNCTION(X) 详细说明
   %
   %   示例:
   %       y = myfunction(5);
       y = x.^2;
   end
   ```

## 扩展资源

- MATLAB文档: https://www.mathworks.com/help/matlab/
- GNU Octave手册: https://docs.octave.org/latest/
- MATLAB入门教程(免费): https://www.mathworks.com/learn/tutorials/matlab-onramp.html
- 文件交换中心: https://www.mathworks.com/matlabcentral/fileexchange/
