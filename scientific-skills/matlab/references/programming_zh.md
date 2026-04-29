# 编程参考

## 目录
1. [脚本与函数](#scripts-and-functions)
2. [控制流](#control-flow)
3. [函数类型](#function-types)
4. [错误处理](#error-handling)
5. [性能与调试](#performance-and-debugging)
6. [面向对象编程](#object-oriented-programming)

## 脚本与函数

### 脚本

```matlab
% 脚本是包含MATLAB命令的.m文件
% 在基础工作区运行（共享变量）

% 示例：myscript.m
% 这是注释
x = 1:10;
y = x.^2;
plot(x, y);
title('我的绘图');

% 运行脚本
myscript;           % 或：run('myscript.m')
```

### 函数

```matlab
% 函数拥有独立工作区
% 保存在与函数同名的文件中

% 示例：myfunction.m
function y = myfunction(x)
%MYFUNCTION 函数简要描述
%   Y = MYFUNCTION(X) 详细描述
%
%   示例：
%       y = myfunction(5);
%
%   另见 OTHERFUNCTION
    y = x.^2;
end

% 多输出
function [result1, result2] = multioutput(x)
    result1 = x.^2;
    result2 = x.^3;
end

% 可变参数
function varargout = flexfun(varargin)
    % varargin 是输入元胞数组
    % varargout 是输出元胞数组
    n = nargin;          % 输入数量
    m = nargout;         % 输出数量
end
```

### 输入验证

```matlab
function result = validatedinput(x, options)
    arguments
        x (1,:) double {mustBePositive}
        options.Normalize (1,1) logical = false
        options.Scale (1,1) double {mustBePositive} = 1
    end

    result = x * options.Scale;
    if options.Normalize
        result = result / max(result);
    end
end

% 用法
y = validatedinput([1 2 3], 'Normalize', true, 'Scale', 2);

% 常用验证器
% mustBePositive, mustBeNegative, mustBeNonzero
% mustBeInteger, mustBeNumeric, mustBeFinite
% mustBeNonNaN, mustBeReal, mustBeNonempty
% mustBeMember, mustBeInRange, mustBeGreaterThan
```

### 局部函数

```matlab
% 局部函数位于主函数之后
% 仅在同一文件内可访问

function result = mainfunction(x)
    intermediate = helper1(x);
    result = helper2(intermediate);
end

function y = helper1(x)
    y = x.^2;
end

function y = helper2(x)
    y = sqrt(x);
end
```

## 控制流

### 条件语句

```matlab
% if-elseif-else
if condition1
    % 语句
elseif condition2
    % 语句
else
    % 语句
end

% 逻辑运算符
%   &  - 与（逐元素）
%   |  - 或（逐元素）
%   ~  - 非
%   && - 与（短路，标量）
%   || - 或（短路，标量）
%   == - 等于
%   ~= - 不等于
%   <, <=, >, >= - 比较

% 示例
if x > 0 && y > 0
    quadrant = 1;
elseif x < 0 && y > 0
    quadrant = 2;
elseif x < 0 && y < 0
    quadrant = 3;
else
    quadrant = 4;
end
```

### Switch语句

```matlab
switch expression
    case value1
        % 语句
    case {value2, value3}  % 多值匹配
        % 语句
    otherwise
        % 默认语句
end

% 示例
switch dayOfWeek
    case {'Saturday', 'Sunday'}
        dayType = '周末';
    case {'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'}
        dayType = '工作日';
    otherwise
        dayType = '未知';
end
```

### For循环

```matlab
% 基础for循环
for i = 1:10
    % 使用i的语句
end

% 自定义步长
for i = 10:-1:1
    % 倒计时
end

% 遍历向量
for val = [1 3 5 7 9]
    % val依次取值
end

% 遍历矩阵列
for col = A
    % col是列向量
end

% 遍历元胞数组
for i = 1:length(C)
    item = C{i};
end
```

### While循环

```matlab
% 基础while循环
while condition
    % 语句
    % 更新条件
end

% 示例
count = 0;
while count < 10
    count = count + 1;
    % 执行操作
end
```

### 循环控制

```matlab
% Break - 立即退出循环
for i = 1:100
    if someCondition
        break;
    end
end

% Continue - 跳过当前迭代
for i = 1:100
    if skipCondition
        continue;
    end
    % 处理i
end

% Return - 退出函数
function y = myfunction(x)
    if x < 0
        y = NaN;
        return;
    end
    y = sqrt(x);
end
```

## 函数类型

### 匿名函数

```matlab
% 创建内联函数
f = @(x) x.^2 + 2*x + 1;
g = @(x, y) x.^2 + y.^2;

% 使用
y = f(5);           % 36
z = g(3, 4);        % 25

% 捕获变量
a = 2;
h = @(x) a * x;     % 捕获a的当前值
y = h(5);           % 10
a = 3;              % 修改a不影响h
y = h(5);           % 仍为10

% 无参函数
now_fn = @() datestr(now);
timestamp = now_fn();

% 传递给其他函数
result = integral(f, 0, 1);
```

### 嵌套函数

```matlab
function result = outerfunction(x)
    y = x.^2;           % 嵌套函数共享变量

    function z = nestedfunction(a)
        z = y + a;      % 可访问外部作用域的y
    end

    result = nestedfunction(10);
end
```

### 函数句柄

```matlab
% 创建现有函数的句柄
h = @sin;
y = h(pi/2);        % 1

% 从字符串创建
h = str2func('cos');

% 获取函数名
name = func2str(h);

% 获取局部函数句柄
handles = localfunctions;

% 函数信息
info = functions(h);
```

### 回调函数

```matlab
% 使用函数句柄作为回调

% 定时器示例
t = timer('TimerFcn', @myCallback, 'Period', 1);
start(t);

function myCallback(~, ~)
    disp(['时间: ' datestr(now)]);
end

% 使用匿名函数
t = timer('TimerFcn', @(~,~) disp('滴答'), 'Period', 1);

% GUI回调
uicontrol('Style', 'pushbutton', 'Callback', @buttonPressed);
```

## 错误处理

### Try-Catch

```matlab
try
    % 可能出错的代码
    result = riskyOperation();
catch ME
    % 处理错误
    disp(['错误: ' ME.message]);
    disp(['标识符: ' ME.identifier]);

    % 可选重新抛出
    rethrow(ME);
end

% 捕获特定错误
try
    result = operation();
catch ME
    switch ME.identifier
        case 'MATLAB:divideByZero'
            result = Inf;
        case 'MATLAB:nomem'
            rethrow(ME);
        otherwise
            result = NaN;
    end
end
```

### 抛出错误

```matlab
% 简单错误
error('发生错误');

% 带标识符
error('MyPkg:InvalidInput', '输入必须为正数');

% 带格式化
error('MyPkg:OutOfRange', '值 %f 超出范围 [%f, %f]', val, lo, hi);

% 创建并抛出异常
ME = MException('MyPkg:Error', '错误信息');
throw(ME);

% 断言
assert(condition, '条件为假时的消息');
assert(x > 0, 'MyPkg:NotPositive', 'x必须为正数');
```

### 警告

```matlab
% 发出警告
warning('这可能存在问题');
warning('MyPkg:Warning', '警告信息');

% 控制警告
warning('off', 'MyPkg:Warning');    % 禁用特定警告
warning('on', 'MyPkg:Warning');     % 启用
warning('off', 'all');              % 禁用所有
warning('on', 'all');               % 启用所有

% 查询警告状态
s = warning('query', 'MyPkg:Warning');

% 临时禁用
origState = warning('off', 'MATLAB:nearlySingularMatrix');
% ... 代码 ...
warning(origState);
```

## 性能与调试

### 计时

```matlab
% 简单计时
tic;
% ... 代码 ...
elapsed = toc;

% 多计时器
t1 = tic;
% ... 代码 ...
elapsed1 = toc(t1);

% CPU时间
t = cputime;
% ... 代码 ...
cpuElapsed = cputime - t;

% 性能分析器
profile on;
myfunction();
profile viewer;     % 分析结果的GUI界面
p = profile('info'); % 获取程序化结果
profile off;
```

### 内存

```matlab
% 内存信息
[user, sys] = memory;   % 仅限Windows
whos;                   % 变量大小

% 清除变量
clear x y z;
clear all;              % 所有变量（谨慎使用）
clearvars -except x y;  % 保留指定变量
```

### 调试

```matlab
% 设置断点（在编辑器或程序化）
dbstop in myfunction at 10
dbstop if error
dbstop if warning
dbstop if naninf          % 在NaN或Inf时停止

% 单步调试
dbstep                    % 下一行
dbstep in                 % 进入函数
dbstep out                % 跳出函数
dbcont                    % 继续执行
dbquit                    % 退出调试

% 清除断点
dbclear all

% 检查状态
dbstack                   % 调用栈
whos                      % 变量信息
```

### 向量化技巧

```matlab
% 尽可能避免循环
% 慢速：
for i = 1:n
    y(i) = x(i)^2;
end

% 快速：
y = x.^2;

% 逐元素运算（使用.前缀）
y = a .* b;             % 逐元素乘法
y = a ./ b;             % 逐元素除法
y = a .^ b;             % 逐元素幂

% 内置函数支持数组操作
y = sin(x);             % 作用于所有元素
s = sum(x);             % 求和
m = max(x);             % 最大值

% 使用逻辑索引替代find
% 慢速：
idx = find(x > 0);
y = x(idx);

% 快速：
y = x(x > 0);

% 预分配数组
% 慢速：
y = [];
for i = 1:n
    y(i) = compute(i);
end

% 快速：
y = zeros(1, n);
for i = 1:n
    y(i) = compute(i);
end
```

### 并行计算

```matlab
% 并行for循环
parfor i = 1:n
    results(i) = compute(i);
end

% 注意：parfor有约束
% - 迭代必须独立
% - 变量分类（切片、广播等）

% 启动并行池
pool = parpool;         % 默认集群
pool = parpool(4);      % 4个工作进程

% 删除并行池
delete(gcp('nocreate'));

% 并行数组操作
spmd
    % 每个工作进程执行此代码块
    localData = myData(labindex);
    result = process(localData);
end
```

## 面向对象编程

### 类定义

```matlab
% 文件 MyClass.m
classdef MyClass
    properties
        PublicProp
    end

    properties (Access = private)
        PrivateProp
    end

    properties (Constant)
        ConstProp = 42
    end

    methods
        % 构造函数
        function obj = MyClass(value)
            obj.PublicProp = value;
        end

        % 实例方法
        function result = compute(obj, x)
            result = obj.PublicProp * x;
        end
    end

    methods (Static)
        function result = staticMethod(x)
            result = x.^2;
        end
    end
end
```

### 使用类

```matlab
% 创建对象
obj = MyClass(10);

% 访问属性
val = obj.PublicProp;
obj.PublicProp = 20;

% 调用方法
result = obj.compute(5);
result = compute(obj, 5);   % 等效形式

% 静态方法
result = MyClass.staticMethod(3);

% 常量属性
val = MyClass.ConstProp;
```

### 继承

```matlab
classdef DerivedClass < BaseClass
    properties
        ExtraProp
    end

    methods
        function obj = DerivedClass(baseVal, extraVal)
            % 调用超类构造函数
            obj@BaseClass(baseVal);
            obj.ExtraProp = extraVal;
        end

        % 重写方法
        function result = compute(obj, x)
            % 调用超类方法
            baseResult = compute@BaseClass(obj, x);
            result = baseResult + obj.ExtraProp;
        end
    end
end
```

### 句柄类与值类

```matlab
% 值类（默认）- 复制语义
classdef ValueClass
    properties
        Data
    end
end

a = ValueClass();
a.Data = 1;
b = a;          % b是副本
b.Data = 2;     % a.Data仍为1

% 句柄类 - 引用语义
classdef HandleClass < handle
    properties
        Data
    end
end

a = HandleClass();
a.Data = 1;
b = a;          % b引用同一对象
b.Data = 2;     % a.Data变为2
```

### 事件与监听器

```matlab
classdef EventClass < handle
    events
        DataChanged
    end

    properties
        Data
    end

    methods
        function set.Data(obj, value)
            obj.Data = value;
            notify(obj, 'DataChanged');
        end
    end
end

% 用法
obj = EventClass();
listener = addlistener(obj, 'DataChanged', @(src, evt) disp('数据已变更！'));
obj.Data = 42;  % 触发事件
```
