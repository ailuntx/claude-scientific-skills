# GNU Octave 兼容性参考指南

## 目录
1. [概述](#概述)
2. [语法差异](#语法差异)
3. [运算符差异](#运算符差异)
4. [函数差异](#函数差异)
5. [Octave 独有功能](#octave-独有功能)
6. [Octave 缺失功能](#octave-缺失功能)
7. [编写兼容代码](#编写兼容代码)
8. [Octave 扩展包](#octave-扩展包)

## 概述

GNU Octave 是 MATLAB 的高兼容性免费开源替代品。大多数 MATLAB 脚本无需修改或仅需少量调整即可在 Octave 中运行，但仍需注意部分差异。

### 安装方法

```bash
# macOS (Homebrew)
brew install octave

# Ubuntu/Debian
sudo apt install octave

# Fedora
sudo dnf install octave

# Windows
# 从 https://octave.org/download 下载安装程序
```

### 运行 Octave

```bash
# 交互模式
octave

# 运行脚本
octave script.m
octave --eval "disp('Hello')"

# 图形界面模式
octave --gui

# 纯命令行模式（无图形）
octave --no-gui
octave-cli
```

## 语法差异

### 注释

```matlab
% MATLAB 风格（两者通用）
% 这是注释

# Octave 风格（仅限 Octave）
# 在 Octave 中也是注释

% 为保持兼容性，始终使用 %
```

### 字符串引号

```matlab
% MATLAB：仅支持单引号（字符数组）
str = 'Hello';              % 字符数组
str = "Hello";              % 字符串（R2017a+）

% Octave：两者皆可但行为不同
str1 = 'Hello';             % 字符数组，不解析转义符
str2 = "Hello\n";           % 将 \n 解析为换行符

% 为保持兼容性，字符数组使用单引号
% 避免使用含转义序列的双引号
```

### 续行符

```matlab
% MATLAB 风格（两者通用）
x = 1 + 2 + 3 + ...
    4 + 5;

% Octave 也接受反斜杠
x = 1 + 2 + 3 + \
    4 + 5;

% 为保持兼容性，使用 ...
```

### 块终止符

```matlab
% MATLAB 风格（两者通用）
if condition
    % 代码
end

for i = 1:10
    % 代码
end

% Octave 也支持特定终止符
if condition
    # 代码
endif

for i = 1:10
    # 代码
endfor

while condition
    # 代码
endwhile

% 为保持兼容性，始终使用 'end'
```

### 函数定义

```matlab
% MATLAB 要求函数必须定义在同名文件中
% Octave 支持命令行函数定义

% Octave 命令行函数
function y = f(x)
    y = x^2;
endfunction

% 为保持兼容性，请在 .m 文件中定义函数
```

## 运算符差异

### 自增/自减运算符

```matlab
% Octave 支持 C 风格运算符（MATLAB 不支持）
x++;                        % x = x + 1
x--;                        % x = x - 1
++x;                        % 前置自增
--x;                        % 前置自减

% 为保持兼容性，请显式赋值
x = x + 1;
x = x - 1;
```

### 复合赋值

```matlab
% Octave 支持（MATLAB 不支持）
x += 5;                     % x = x + 5
x -= 3;                     % x = x - 3
x *= 2;                     % x = x * 2
x /= 4;                     % x = x / 4
x ^= 2;                     % x = x ^ 2

% 逐元素版本
x .+= y;
x .-= y;
x .*= y;
x ./= y;
x .^= y;

% 为保持兼容性，请显式赋值
x = x + 5;
x = x .* y;
```

### 逻辑运算符

```matlab
% 两者均支持
& | ~ && ||

% 短路行为差异：
% MATLAB：在 if/while 条件中 & 和 | 会短路
% Octave：仅 && 和 || 会短路

% 为获得可预测行为，请使用：
% && || 用于标量短路逻辑
% & | 用于逐元素运算
```

### 表达式后索引

```matlab
% Octave 允许直接在表达式后索引
result = sin(x)(1:10);      % 获取 sin(x) 的前10个元素
value = func(arg).field;    % 访问返回结构体的字段

% MATLAB 需要中间变量
temp = sin(x);
result = temp(1:10);

temp = func(arg);
value = temp.field;

% 为保持兼容性，请使用中间变量
```

## 函数差异

### 内置函数

大多数基础函数兼容。部分差异：

```matlab
% 函数名称差异
% MATLAB          Octave 替代方案
% ------          ------------------
% inputname       （不可用）
% inputParser     （部分支持）
% validateattributes  （部分支持）

% 边界情况行为差异
% 请查阅具体函数的文档
```

### 随机数生成

```matlab
% 两者默认均使用梅森旋转算法
% 种子设置方式类似
rng(42);                    % MATLAB
rand('seed', 42);           % Octave（也接受 rng 语法）

% 兼容性方案
rng(42);                    % 现代 Octave 支持
```

### 图形功能

```matlab
% 基础绘图兼容
plot(x, y);
xlabel('X'); ylabel('Y');
title('标题');
legend('数据');

% 部分高级功能存在差异
% - Octave 使用 gnuplot 或 Qt 图形引擎
% - 部分属性名称可能不同
% - 动画/GUI 功能各异

% 请在两种环境中测试图形代码
```

### 文件输入输出

```matlab
% 基础 I/O 兼容
save('file.mat', 'x', 'y');
load('file.mat');
dlmread('file.txt');
dlmwrite('file.txt', data);

% MAT 文件版本
save('file.mat', '-v7');    % 兼容格式
save('file.mat', '-v7.3');  % HDF5 格式（Octave 部分支持）

% 为保持兼容性，请使用 -v7 或 -v6
```

## Octave 独有功能

### do-until 循环

```matlab
% 仅限 Octave
do
    x = x + 1;
until (x > 10)

% 等效的 MATLAB/兼容代码
x = x + 1;
while x <= 10
    x = x + 1;
end
```

### unwind_protect

```matlab
% 仅限 Octave - 确保资源清理
unwind_protect
    % 可能出错的操作
    result = risky_operation();
unwind_protect_cleanup
    % 始终执行（类似 finally）
    cleanup();
end_unwind_protect

% MATLAB 等效方案
try
    result = risky_operation();
catch
end
cleanup();  % 未捕获错误时不保证执行
```

### 内置文档系统

```matlab
% Octave 支持函数内 Texinfo 文档
function y = myfunction(x)
    %% -*- texinfo -*-
    %% @deftypefn {Function File} {@var{y} =} myfunction (@var{x})
    %% myfunction 功能描述
    %% @end deftypefn
    y = x.^2;
endfunction
```

### 包管理系统

```matlab
% Octave Forge 扩展包
pkg install -forge control
pkg load control

% 列出已安装包
pkg list

% 为保持 MATLAB 兼容性，请使用等效工具箱
% 或直接集成包功能
```

## Octave 缺失功能

### Simulink

```matlab
% 无等效替代方案
% Simulink 模型（.slx, .mdl）无法在 Octave 运行
```

### MATLAB 工具箱

```matlab
% 多数工具箱函数不可用
% 部分存在 Octave Forge 替代包：

% MATLAB 工具箱        Octave Forge 包
% ---------------       --------------------
% Control System        control
% Signal Processing     signal
% Image Processing      image
% Statistics            statistics
% Optimization          optim

% 请通过 pkg list 查看可用包
```

### App Designer / GUIDE

```matlab
% MATLAB GUI 工具在 Octave 不可用
% Octave 提供基础 UI 函数：
uicontrol, uimenu, figure 属性

% 跨平台 GUI 替代方案：
% - 基于 Web 的界面
% - Qt（通过 Octave 的 Qt 图形）
```

### 面向对象编程

```matlab
% Octave 提供部分 classdef 支持
% 部分功能缺失或行为不同：
% - Handle 类事件
% - 属性验证
% - 部分访问修饰符

% 为保持兼容性，请使用简单 OOP 模式
% 或基于结构体的方案
```

### 实时脚本

```matlab
% .mlx 文件仅限 MATLAB
% 兼容方案请使用常规 .m 脚本
```

## 编写兼容代码

### 环境检测

```matlab
function tf = isOctave()
    tf = exist('OCTAVE_VERSION', 'builtin') ~= 0;
end

% 用于条件代码
if isOctave()
    % Octave 专用代码
else
    % MATLAB 专用代码
end
```

### 最佳实践

```matlab
% 1. 使用 % 而非 # 注释
% 推荐
% 这是注释

% 避免
# 这是注释（仅限 Octave）

% 2. 使用 ... 续行
% 推荐
x = 1 + 2 + 3 + ...
    4 + 5;

% 避免
x = 1 + 2 + 3 + \
    4 + 5;

% 3. 所有块使用 'end' 终止
% 推荐
if condition
    code
end

% 避免
if condition
    code
endif

% 4. 避免复合运算符
% 推荐
x = x + 1;

% 避免
x++;
x += 1;

% 5. 字符串使用单引号
% 推荐
str = 'Hello World';

% 避免（转义序列问题）
str = "Hello\nWorld";

% 6. 索引使用中间变量
% 推荐
temp = func(arg);
result = temp(1:10);

% 避免（仅限 Octave）
result = func(arg)(1:10);

% 7. 使用兼容格式保存 MAT 文件
save('data.mat', 'x', 'y', '-v7');
```

### 兼容性测试

```bash
# 双环境测试
matlab -nodisplay -nosplash -r "run('test_script.m'); exit;"
octave --no-gui test_script.m

# 创建测试脚本
# test_script.m:
# try
#     main_function();
#     disp('测试通过');
# catch ME
#     disp(['测试失败: ' ME.message]);
# end
```

## Octave 扩展包

### 安装扩展包

```matlab
% 从 Octave Forge 安装
pkg install -forge package_name

% 从文件安装
pkg install package_file.tar.gz

% 从 URL 安装
pkg install 'http://example.com/package.tar.gz'

% 卸载
pkg uninstall package_name
```

### 使用扩展包

```matlab
% 加载包（使用前必需）
pkg load control
pkg load signal
pkg load image

% 启动时加载（添加到 .octaverc）
pkg load control

% 列出已加载包
pkg list

% 卸载包
pkg unload control
```

### 常用扩展包

| 包名称 | 描述 |
|---------|-------------|
| control | 控制系统设计 |
| signal | 信号处理 |
| image | 图像处理 |
| statistics | 统计函数 |
| optim | 优化算法 |
| io | 输入输出函数 |
| struct | 结构体操作 |
| symbolic | 符号计算（通过 SymPy） |
| parallel | 并行计算 |
| netcdf | NetCDF 文件支持 |

### 包管理

```matlab
% 更新所有包
pkg update

% 获取包描述
pkg describe package_name

% 检查更新
pkg list  % 与 Octave Forge 网站对比
```
