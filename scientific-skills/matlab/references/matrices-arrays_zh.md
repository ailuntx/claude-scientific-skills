# 矩阵与数组参考

## 目录
1. [数组创建](#数组创建)
2. [索引与下标](#索引与下标)
3. [数组操作](#数组操作)
4. [连接与重塑](#连接与重塑)
5. [数组信息](#数组信息)
6. [排序与搜索](#排序与搜索)

## 数组创建

### 基本创建

```matlab
% 直接指定
A = [1 2 3; 4 5 6; 7 8 9];    % 3x3 矩阵（行间用 ; 分隔）
v = [1, 2, 3, 4, 5];           % 行向量
v = [1; 2; 3; 4; 5];           % 列向量

% 范围运算符
v = 1:10;                       % 1 到 10，步长 1
v = 0:0.5:5;                    % 0 到 5，步长 0.5
v = 10:-1:1;                    % 10 递减到 1

% 线性/对数等距
v = linspace(0, 1, 100);        % 100 个点，0 到 1
v = logspace(0, 3, 50);         % 50 个点，10^0 到 10^3
```

### 特殊矩阵

```matlab
% 常见模式
I = eye(n);                     % n×n 单位矩阵
I = eye(m, n);                  % m×n 单位矩阵
Z = zeros(m, n);                % m×n 零矩阵
O = ones(m, n);                 % m×n 全一矩阵
D = diag([1 2 3]);              % 向量创建对角阵
d = diag(A);                    % 提取矩阵对角线

% 随机矩阵
R = rand(m, n);                 % 均匀分布 [0,1]
R = randn(m, n);                % 正态分布 (均值=0, 标准差=1)
R = randi([a b], m, n);         % [a,b] 区间随机整数
R = randperm(n);                % 1:n 的随机排列

% 逻辑数组
T = true(m, n);                 % 全真
F = false(m, n);                % 全假

% 二维/三维网格
[X, Y] = meshgrid(x, y);        % 二维向量网格
[X, Y, Z] = meshgrid(x, y, z);  % 三维网格
[X, Y] = ndgrid(x, y);          % 替代方案（不同方向）
```

### 基于现有数组创建

```matlab
A_like = zeros(size(B));        % 与 B 同尺寸
A_like = ones(size(B), 'like', B);  % 与 B 同尺寸和类型
A_copy = A;                     % 复制（值复制，非引用）
```

## 索引与下标

### 基本索引

```matlab
% 单元素（基于1的索引）
elem = A(2, 3);                 % 第2行第3列
elem = A(5);                    % 线性索引（列优先顺序）

% 范围索引
row = A(2, :);                  % 整行第2行
col = A(:, 3);                  % 整列第3列
sub = A(1:2, 2:3);              % 1-2行，2-3列

% end 关键字
last = A(end, :);               % 最后一行
last3 = A(end-2:end, :);        % 最后3行
```

### 逻辑索引

```matlab
% 查找满足条件的元素
idx = A > 5;                    % 逻辑数组
elements = A(A > 5);            % 提取 >5 的元素
A(A < 0) = 0;                   % 负元素置零

% 组合条件
idx = (A > 0) & (A < 10);       % 与
idx = (A < 0) | (A > 10);       % 或
idx = ~(A == 0);                % 非
```

### 线性索引

```matlab
% 线性索引与下标互转
[row, col] = ind2sub(size(A), linearIdx);
linearIdx = sub2ind(size(A), row, col);

% 查找非零/条件索引
idx = find(A > 5);              % A>5 的线性索引
idx = find(A > 5, k);           % 前 k 个索引
idx = find(A > 5, k, 'last');   % 后 k 个索引
[row, col] = find(A > 5);       % 下标索引
```

### 高级索引

```matlab
% 数组索引
rows = [1 3 5];
cols = [2 4];
sub = A(rows, cols);            % 子矩阵

% 逻辑数组索引
B = A(logical_mask);            % 掩码为真的元素

% 索引赋值
A(1:2, 1:2) = [10 20; 30 40];   % 子矩阵赋值
A(:) = 1:numel(A);              % 全元素赋值（列优先）
```

## 数组操作

### 逐元素操作

```matlab
% 算术运算（逐元素需加 . 前缀）
C = A + B;                      % 加法
C = A - B;                      % 减法
C = A .* B;                     % 逐元素乘法
C = A ./ B;                     % 逐元素除法
C = A .\ B;                     % 逐元素左除 (B./A)
C = A .^ n;                     % 逐元素幂

% 比较运算（逐元素）
C = A == B;                     % 相等
C = A ~= B;                     % 不等
C = A < B;                      % 小于
C = A <= B;                     % 小于等于
C = A > B;                      % 大于
C = A >= B;                     % 大于等于
```

### 矩阵运算

```matlab
% 矩阵算术
C = A * B;                      % 矩阵乘法
C = A ^ n;                      % 矩阵幂
C = A';                         % 共轭转置
C = A.';                        % 转置（非共轭）

% 矩阵函数
B = inv(A);                     % 逆矩阵
B = pinv(A);                    % 伪逆
d = det(A);                     % 行列式
t = trace(A);                   % 迹（对角线之和）
r = rank(A);                    % 秩
n = norm(A);                    % 矩阵/向量范数
n = norm(A, 'fro');             % Frobenius 范数

% 解线性系统
x = A \ b;                      % 解 Ax = b
x = b' / A';                    % 解 xA = b
```

### 常用函数

```matlab
% 逐元素应用
B = abs(A);                     % 绝对值
B = sqrt(A);                    % 平方根
B = exp(A);                     % 指数
B = log(A);                     % 自然对数
B = log10(A);                   % 常用对数
B = sin(A);                     % 正弦（弧度）
B = sind(A);                    % 正弦（角度）
B = round(A);                   % 四舍五入
B = floor(A);                   % 向下取整
B = ceil(A);                    % 向上取整
B = real(A);                    % 实部
B = imag(A);                    % 虚部
B = conj(A);                    % 复共轭
```

## 连接与重塑

### 连接

```matlab
% 水平连接（并排）
C = [A B];                      % 列连接
C = [A, B];                     % 同上
C = horzcat(A, B);              % 函数形式
C = cat(2, A, B);               % 沿维度2连接

% 垂直连接（堆叠）
C = [A; B];                     % 行连接
C = vertcat(A, B);              % 函数形式
C = cat(1, A, B);               % 沿维度1连接

% 块对角
C = blkdiag(A, B, C);           % 块对角矩阵
```

### 重塑

```matlab
% 重塑形状
B = reshape(A, m, n);           % 重塑为 m×n（总元素数不变）
B = reshape(A, [], n);          % 自动计算行数
v = A(:);                       % 展平为列向量

% 转置与置换维度
B = A';                         % 二维转置
B = permute(A, [2 1 3]);        % 维度置换
B = ipermute(A, [2 1 3]);       % 逆置换

% 增减维度
B = squeeze(A);                 % 移除单一维度
B = shiftdim(A, n);             % 维度移位

% 复制
B = repmat(A, m, n);            % 平铺 m×n 次
B = repelem(A, m, n);           % 元素重复
```

### 翻转与旋转

```matlab
B = flip(A);                    % 沿首个非单一维度翻转
B = flip(A, dim);               % 沿指定维度翻转
B = fliplr(A);                  % 左右翻转（列）
B = flipud(A);                  % 上下翻转（行）
B = rot90(A);                   % 逆时针旋转90°
B = rot90(A, k);                % 旋转 k×90°
B = circshift(A, k);            % 循环移位
```

## 数组信息

### 大小与维度

```matlab
[m, n] = size(A);               % 行数与列数
m = size(A, 1);                 % 行数
n = size(A, 2);                 % 列数
sz = size(A);                   % 尺寸向量
len = length(A);                % 最大维度长度
num = numel(A);                 % 元素总数
ndim = ndims(A);                % 维度数量
```

### 类型检查

```matlab
tf = isempty(A);                % 是否为空？
tf = isscalar(A);               % 是否为标量（1×1）？
tf = isvector(A);               % 是否为向量（1×n 或 n×1）？
tf = isrow(A);                  % 是否为行向量？
tf = iscolumn(A);               % 是否为列向量？
tf = ismatrix(A);               % 是否为二维矩阵？
tf = isnumeric(A);              % 是否为数值型？
tf = isreal(A);                 % 是否为实数（无虚部）？
tf = islogical(A);              % 是否为逻辑型？
tf = isnan(A);                  % 哪些元素是 NaN？
tf = isinf(A);                  % 哪些元素是 Inf？
tf = isfinite(A);               % 哪些元素是有限值？
```

### 比较

```matlab
tf = isequal(A, B);             % 数组是否相等？
tf = isequaln(A, B);            % 相等（视 NaN 为相等）？
tf = all(A);                    % 是否全非零/真？
tf = any(A);                    % 是否有非零/真？
tf = all(A, dim);               % 沿维度全为真
tf = any(A, dim);               % 沿维度存在真值
```

## 排序与搜索

### 排序

```matlab
B = sort(A);                    % 列升序排序
B = sort(A, 'descend');         % 降序排序
B = sort(A, dim);               % 沿维度排序
[B, idx] = sort(A);             % 同时返回原始索引
B = sortrows(A);                % 按首列排序行
B = sortrows(A, col);           % 按指定列排序
B = sortrows(A, col, 'descend');
```

### 唯一值与集合操作

```matlab
B = unique(A);                  % 唯一元素
[B, ia, ic] = unique(A);        % 带索引信息
B = unique(A, 'rows');          % 唯一行

% 集合操作
C = union(A, B);                % 并集
C = intersect(A, B);            % 交集
C = setdiff(A, B);              % 差集（在 A 不在 B）
C = setxor(A, B);               % 对称差集
tf = ismember(A, B);            % A 的元素是否在 B 中？
```

### 最小值/最大值

```matlab
m = min(A);                     % 列最小值
m = min(A, [], 'all');          % 全局最小值
[m, idx] = min(A);              % 带索引
m = min(A, B);                  % 逐元素最小值

M = max(A);                     % 列最大值
M = max(A, [], 'all');          % 全局最大值
[M, idx] = max(A);              % 带索引

[minVal, minIdx] = min(A(:));   % 全局最小值（线性索引）
[maxVal, maxIdx] = max(A(:));   % 全局最大值（线性索引）

% k 个最小/最大
B = mink(A, k);                 % k 个最小元素
B = maxk(A, k);                 % k 个最大元素
```

### 求和与乘积

```matlab
s = sum(A);                     % 列求和
s = sum(A, 'all');              % 总和
s = sum(A, dim);                % 沿维度求和
s = cumsum(A);                  % 累积和

p = prod(A);                    % 列求积
p = prod(A, 'all');             % 总乘积
p = cumprod(A);                 % 累积积
```
