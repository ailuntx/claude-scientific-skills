# 数学参考

## 目录
1. [线性代数](#线性代数)
2. [初等数学](#初等数学)
3. [微积分与积分](#微积分与积分)
4. [微分方程](#微分方程)
5. [优化](#优化)
6. [统计学](#统计学)
7. [信号处理](#信号处理)
8. [插值与拟合](#插值与拟合)

## 线性代数

### 求解线性系统

```matlab
% Ax = b
x = A \ b;                      % 首选方法 (mldivide)
x = linsolve(A, b);             % 带选项
x = inv(A) * b;                 % 效率较低，避免使用

% linsolve选项
opts.LT = true;                 % 下三角矩阵
opts.UT = true;                 % 上三角矩阵
opts.SYM = true;                % 对称矩阵
opts.POSDEF = true;             % 正定矩阵
x = linsolve(A, b, opts);

% xA = b
x = b / A;                      % mrdivide

% 最小二乘（超定系统）
x = A \ b;                      % 最小范数解
x = lsqminnorm(A, b);           % 显式最小范数解

% 非负最小二乘
x = lsqnonneg(A, b);            % x >= 0 约束
```

### 矩阵分解

```matlab
% LU分解: A = L*U 或 P*A = L*U
[L, U] = lu(A);                 % L可能不是下三角矩阵
[L, U, P] = lu(A);              % P*A = L*U

% QR分解: A = Q*R
[Q, R] = qr(A);                 % 完全分解
[Q, R] = qr(A, 0);              % 经济尺寸
[Q, R, P] = qr(A);              % 列主元: A*P = Q*R

% Cholesky分解: A = R'*R (对称正定)
R = chol(A);                    % 上三角矩阵
L = chol(A, 'lower');           % 下三角矩阵

% LDL'分解: A = L*D*L' (对称矩阵)
[L, D] = ldl(A);

% Schur分解: A = U*T*U'
[U, T] = schur(A);              % T是拟三角矩阵
[U, T] = schur(A, 'complex');   % T是三角矩阵
```

### 特征值与特征向量

```matlab
% 特征值
e = eig(A);                     % 仅特征值
[V, D] = eig(A);                % V: 特征向量, D: 对角特征值
                                % A*V = V*D

% 广义特征值: A*v = lambda*B*v
e = eig(A, B);
[V, D] = eig(A, B);

% 稀疏/大型矩阵（特征值子集）
e = eigs(A, k);                 % k个最大幅值特征值
e = eigs(A, k, 'smallestabs');  % k个最小幅值特征值
[V, D] = eigs(A, k, 'largestreal');
```

### 奇异值分解

```matlab
% SVD: A = U*S*V'
[U, S, V] = svd(A);             % 完全分解
[U, S, V] = svd(A, 'econ');     % 经济尺寸
s = svd(A);                     % 仅奇异值

% 稀疏/大型矩阵
[U, S, V] = svds(A, k);         % k个最大奇异值

% 应用
r = rank(A);                    % 秩（非零奇异值计数）
p = pinv(A);                    % 伪逆（通过SVD）
n = norm(A, 2);                 % 2-范数 = 最大奇异值
c = cond(A);                    % 条件数 = 最大/最小奇异值之比
```

### 矩阵属性

```matlab
d = det(A);                     % 行列式
t = trace(A);                   % 迹（对角线元素之和）
r = rank(A);                    % 秩
n = norm(A);                    % 2-范数（默认）
n = norm(A, 1);                 % 1-范数（最大列和）
n = norm(A, inf);               % 无穷范数（最大行和）
n = norm(A, 'fro');             % Frobenius范数
c = cond(A);                    % 条件数
c = rcond(A);                   % 倒数条件数（快速估计）
```

## 初等数学

### 三角函数

```matlab
% 弧度制
y = sin(x);   y = cos(x);   y = tan(x);
y = asin(x);  y = acos(x);  y = atan(x);
y = atan2(y, x);            % 四象限反正切

% 角度制
y = sind(x);  y = cosd(x);  y = tand(x);
y = asind(x); y = acosd(x); y = atand(x);

% 双曲函数
y = sinh(x);  y = cosh(x);  y = tanh(x);
y = asinh(x); y = acosh(x); y = atanh(x);

% 正割、余割、余切
y = sec(x);   y = csc(x);   y = cot(x);
```

### 指数与对数

```matlab
y = exp(x);                     % e^x
y = log(x);                     % 自然对数 (ln)
y = log10(x);                   % 以10为底对数
y = log2(x);                    % 以2为底对数
y = log1p(x);                   % log(1+x)，对小x精确
[F, E] = log2(x);               % F * 2^E = x

y = sqrt(x);                    % 平方根
y = nthroot(x, n);              % 实数n次方根
y = realsqrt(x);                % 实数平方根（x<0时报错）

y = pow2(x);                    % 2^x
y = x .^ y;                     % 逐元素幂运算
```

### 复数

```matlab
z = complex(a, b);              % a + bi
z = 3 + 4i;                     % 直接创建

r = real(z);                    % 实部
i = imag(z);                    % 虚部
m = abs(z);                     % 模
p = angle(z);                   % 相位角（弧度）
c = conj(z);                    % 共轭复数

[theta, rho] = cart2pol(x, y);  % 笛卡尔坐标转极坐标
[x, y] = pol2cart(theta, rho);  % 极坐标转笛卡尔坐标
```

### 舍入与余数

```matlab
y = round(x);                   % 四舍五入取整
y = round(x, n);                % 保留n位小数
y = floor(x);                   % 向负无穷舍入
y = ceil(x);                    % 向正无穷舍入
y = fix(x);                     % 向零舍入

y = mod(x, m);                  % 取模（符号同m）
y = rem(x, m);                  % 取余（符号同x）
[q, r] = deconv(x, m);          % 商与余数

y = sign(x);                    % 符号函数（-1, 0, 1）
y = abs(x);                     % 绝对值
```

### 特殊函数

```matlab
y = gamma(x);                   % Gamma函数
y = gammaln(x);                 % Gamma对数（避免溢出）
y = factorial(n);               % n阶乘
y = nchoosek(n, k);             % 二项式系数

y = erf(x);                     % 误差函数
y = erfc(x);                    % 互补误差函数
y = erfcinv(x);                 % 逆互补误差函数

y = besselj(nu, x);             % 贝塞尔J函数
y = bessely(nu, x);             % 贝塞尔Y函数
y = besseli(nu, x);             % 修正贝塞尔I函数
y = besselk(nu, x);             % 修正贝塞尔K函数

y = legendre(n, x);             % 勒让德多项式
```

## 微积分与积分

### 数值积分

```matlab
% 定积分
q = integral(fun, a, b);        % 对fun从a到b积分
q = integral(@(x) x.^2, 0, 1);  % 示例：x^2的积分

% 选项
q = integral(fun, a, b, 'AbsTol', 1e-10);
q = integral(fun, a, b, 'RelTol', 1e-6);

% 反常积分
q = integral(fun, 0, Inf);      % 积分至无穷
q = integral(fun, -Inf, Inf);   % 全实数轴积分

% 多维积分
q = integral2(fun, xa, xb, ya, yb);  % 二重积分
q = integral3(fun, xa, xb, ya, yb, za, zb);  % 三重积分

% 离散数据积分
q = trapz(x, y);                % 梯形法则
q = trapz(y);                   % 单位间距
q = cumtrapz(x, y);             % 累积积分
```

### 数值微分

```matlab
% 有限差分
dy = diff(y);                   % 一阶差分
dy = diff(y, n);                % n阶差分
dy = diff(y, n, dim);           % 沿维度差分

% 梯度（数值导数）
g = gradient(y);                % dy/dx，单位间距
g = gradient(y, h);             % dy/dx，间距h
[gx, gy] = gradient(Z, hx, hy); % 二维数据梯度
```

## 微分方程

### ODE求解器

```matlab
% 标准形式: dy/dt = f(t, y)
odefun = @(t, y) -2*y;          % 示例: dy/dt = -2y
[t, y] = ode45(odefun, tspan, y0);

% 求解器选择:
% ode45  - 非刚性，中等精度（默认选择）
% ode23  - 非刚性，低精度
% ode113 - 非刚性，变阶
% ode15s - 刚性，变阶（当ode45缓慢时尝试）
% ode23s - 刚性，低阶
% ode23t - 中等刚性，梯形法
% ode23tb - 刚性，TR-BDF2法

% 带选项
options = odeset('RelTol', 1e-6, 'AbsTol', 1e-9);
options = odeset('MaxStep', 0.1);
options = odeset('Events', @myEventFcn);  % 停止条件
[t, y] = ode45(odefun, tspan, y0, options);
```

### 高阶ODE

```matlab
% y'' + 2y' + y = 0, y(0) = 1, y'(0) = 0
% 转换为系统: y1 = y, y2 = y'
% y1' = y2
% y2' = -2*y2 - y1

odefun = @(t, y) [y(2); -2*y(2) - y(1)];
y0 = [1; 0];                    % [y(0); y'(0)]
[t, y] = ode45(odefun, [0 10], y0);
plot(t, y(:,1));                % 绘制y（第一分量）
```

### 边值问题

```matlab
% y'' + |y| = 0, y(0) = 0, y(4) = -2
solinit = bvpinit(linspace(0, 4, 5), [0; 0]);
sol = bvp4c(@odefun, @bcfun, solinit);

function dydx = odefun(x, y)
    dydx = [y(2); -abs(y(1))];
end

function res = bcfun(ya, yb)
    res = [ya(1); yb(1) + 2];   % y(0) = 0, y(4) = -2
end
```

## 优化

### 无约束优化

```matlab
% 单变量，有界
[x, fval] = fminbnd(fun, x1, x2);
[x, fval] = fminbnd(@(x) x.^2 - 4*x, 0, 5);

% 多变量，无约束
[x, fval] = fminsearch(fun, x0);
options = optimset('TolX', 1e-8, 'TolFun', 1e-8);
[x, fval] = fminsearch(fun, x0, options);

% 显示迭代过程
options = optimset('Display', 'iter');
```

### 求根

```matlab
% 求f(x) = 0的解
x = fzero(fun, x0);             % 在x0附近
x = fzero(fun, [x1 x2]);        % 在区间[x1, x2]内
x = fzero(@(x) cos(x) - x, 0.5);

% 多项式求根
r = roots([1 0 -4]);            % x^2 - 4 = 0的根
                                % 返回[2; -2]
```

### 最小二乘

```matlab
% 线性最小二乘: 最小化 ||Ax - b||
x = A \ b;                      % 标准解
x = lsqminnorm(A, b);           % 最小范数解

% 非负最小二乘
x = lsqnonneg(A, b);            % x >= 0

% 非线性最小二乘
x = lsqnonlin(fun, x0);         % 最小化 sum(fun(x).^2)
x = lsqcurvefit(fun, x0, xdata, ydata);  % 曲线拟合
```

## 统计学

### 描述性统计

```matlab
% 集中趋势
m = mean(x);                    % 算术平均数
m = mean(x, 'all');             % 所有元素的均值
m = mean(x, dim);               % 沿维度均值
m = mean(x, 'omitnan');         % 忽略NaN值
gm = geomean(x);                % 几何平均数
hm = harmmean(x);               % 调和平均数
med = median(x);                % 中位数
mo = mode(x);                   % 众数

% 离散程度
s = std(x);                     % 标准差 (N-1)
s = std(x, 1);                  % 总体标准差 (N)
v = var(x);                     % 方差
r = range(x);                   % 极差 (最大值-最小值)
iqr_val = iqr(x);               % 四分位距

% 极值
[minv, mini] = min(x);
[maxv, maxi] = max(x);
[lo, hi] = bounds(x);           % 同时获取最小值和最大值
```

### 相关性与协方差

```matlab
% 相关性
R = corrcoef(X, Y);             % 相关矩阵
r = corrcoef(x, y);             % 相关系数

% 协方差
C = cov(X, Y);                  % 协方差矩阵
c = cov(x, y);                  % 协方差

% 互相关（信号处理）
[r, lags] = xcorr(x, y);        % 互相关
[r, lags] = xcorr(x, y, 'coeff');  % 归一化互相关
```

### 百分位数与分位数

```matlab
p = prctile(x, [25 50 75]);     % 百分位数
q = quantile(x, [0.25 0.5 0.75]);  % 分位数
```

### 移动统计

```matlab
y = movmean(x, k);              % k点移动平均
y = movmedian(x, k);            % 移动中位数
y = movstd(x, k);               % 移动标准差
y = movvar(x, k);               % 移动方差
y = movmin(x, k);               % 移动最小值
y = movmax(x, k);               % 移动最大值
y = movsum(x, k);               % 移动求和

% 窗口选项

```markdown
Y = fft(x, n);                  % n点FFT（补零/截断）
Y = fft2(X);                    % 二维FFT
Y = fftn(X);                    % N维FFT

% 逆FFT
x = ifft(Y);
X = ifft2(Y);
X = ifftn(Y);

% 零频移至中心
Y_shifted = fftshift(Y);
Y = ifftshift(Y_shifted);

% 频率轴
n = length(x);
fs = 1000;                      % 采样频率
f = (0:n-1) * fs / n;           % 频率向量
f = (-n/2:n/2-1) * fs / n;      % 中心化频率向量
```

### 滤波

```matlab
% 一维滤波
y = filter(b, a, x);            % 应用IIR/FIR滤波器
y = filtfilt(b, a, x);          % 零相位滤波

% 简单移动平均
b = ones(1, k) / k;
y = filter(b, 1, x);

% 卷积
y = conv(x, h);                 % 完全卷积
y = conv(x, h, 'same');         % 与x同尺寸
y = conv(x, h, 'valid');        % 仅有效部分

% 解卷积
[q, r] = deconv(y, h);          % y = conv(q, h) + r

% 二维滤波
Y = filter2(H, X);              % 二维滤波器
Y = conv2(X, H, 'same');        % 二维卷积
```

## 插值与拟合

### 插值

```matlab
% 一维插值
yi = interp1(x, y, xi);         % 线性（默认）
yi = interp1(x, y, xi, 'spline');  % 样条
yi = interp1(x, y, xi, 'pchip');   % 分段三次
yi = interp1(x, y, xi, 'nearest'); % 最近邻

% 二维插值
zi = interp2(X, Y, Z, xi, yi);
zi = interp2(X, Y, Z, xi, yi, 'spline');

% 三维插值
vi = interp3(X, Y, Z, V, xi, yi, zi);

% 散点数据
F = scatteredInterpolant(x, y, v);
vi = F(xi, yi);
```

### 多项式拟合

```matlab
% 多项式拟合
p = polyfit(x, y, n);           % 拟合n阶多项式
                                % p = [p1, p2, ..., pn+1]
                                % y = p1*x^n + p2*x^(n-1) + ... + pn+1

% 多项式求值
yi = polyval(p, xi);

% 带拟合质量评估
[p, S] = polyfit(x, y, n);
[yi, delta] = polyval(p, xi, S);  % delta = 误差估计

% 多项式运算
r = roots(p);                   % 求根
p = poly(r);                    % 由根构造多项式
q = polyder(p);                 % 导数
q = polyint(p);                 % 积分
c = conv(p1, p2);               % 多项式乘法
[q, r] = deconv(p1, p2);        % 多项式除法
```

### 曲线拟合

```matlab
% 使用fit函数（需曲线拟合工具箱或基础形式）
% 线性：y = a*x + b
p = polyfit(x, y, 1);
a = p(1); b = p(2);

% 指数：y = a*exp(b*x)
% 线性化：log(y) = log(a) + b*x
p = polyfit(x, log(y), 1);
b = p(1); a = exp(p(2));

% 幂函数：y = a*x^b
% 线性化：log(y) = log(a) + b*log(x)
p = polyfit(log(x), log(y), 1);
b = p(1); a = exp(p(2));

% 非线性最小二乘拟合
model = @(p, x) p(1)*exp(-p(2)*x);  % 示例：a*exp(-b*x)
p0 = [1, 1];                        % 初始猜测
p = lsqcurvefit(model, p0, xdata, ydata);
```
