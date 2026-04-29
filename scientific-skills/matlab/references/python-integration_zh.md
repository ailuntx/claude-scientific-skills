# Python 集成参考

## 目录
1. [从 MATLAB 调用 Python](#calling-python-from-matlab)
2. [数据类型转换](#data-type-conversion)
3. [处理 Python 对象](#working-with-python-objects)
4. [从 Python 调用 MATLAB](#calling-matlab-from-python)
5. [常见工作流程](#common-workflows)

## 从 MATLAB 调用 Python

### 环境设置

```matlab
% 检查 Python 配置
pyenv

% 设置 Python 版本（在调用任何 Python 前）
pyenv('Version', '/usr/bin/python3');
pyenv('Version', '3.10');

% 检查 Python 是否可用
pe = pyenv;
disp(pe.Version);
disp(pe.Executable);
```

### 基础 Python 调用

```matlab
% 使用 py. 前缀调用内置函数
result = py.len([1, 2, 3, 4]);  % 4
result = py.sum([1, 2, 3, 4]);  % 10
result = py.max([1, 2, 3, 4]);  % 4
result = py.abs(-5);            % 5

% 创建 Python 对象
pyList = py.list({1, 2, 3});
pyDict = py.dict(pyargs('a', 1, 'b', 2));
pySet = py.set({1, 2, 3});
pyTuple = py.tuple({1, 2, 3});

% 调用模块函数
result = py.math.sqrt(16);
result = py.os.getcwd();
wrapped = py.textwrap.wrap('This is a long string');
```

### 导入和使用模块

```matlab
% 导入模块
np = py.importlib.import_module('numpy');
pd = py.importlib.import_module('pandas');

% 使用模块
arr = np.array({1, 2, 3, 4, 5});
result = np.mean(arr);

% 替代方案：直接 py. 语法
arr = py.numpy.array({1, 2, 3, 4, 5});
result = py.numpy.mean(arr);
```

### 运行 Python 代码

```matlab
% 执行 Python 语句
pyrun("x = 5")
pyrun("y = x * 2")
result = pyrun("z = y + 1", "z");

% 运行 Python 文件
pyrunfile("script.py");
result = pyrunfile("script.py", "output_variable");

% 带输入变量运行
x = 10;
result = pyrun("y = x * 2", "y", x=x);
```

### 关键字参数

```matlab
% 使用 pyargs 传递关键字参数
result = py.sorted({3, 1, 4, 1, 5}, pyargs('reverse', true));

% 多个关键字参数
df = py.pandas.DataFrame(pyargs( ...
    'data', py.dict(pyargs('A', {1, 2, 3}, 'B', {4, 5, 6})), ...
    'index', {'x', 'y', 'z'}));
```

## 数据类型转换

### MATLAB 到 Python

| MATLAB 类型 | Python 类型 |
|-------------|-------------|
| double, single | float |
| int8, int16, int32, int64 | int |
| uint8, uint16, uint32, uint64 | int |
| logical | bool |
| char, string | str |
| cell 数组 | list |
| struct | dict |
| 数值数组 | numpy.ndarray (若 numpy 可用) |

```matlab
% 自动转换示例
py.print(3.14);         % float
py.print(int32(42));    % int
py.print(true);         % bool (True)
py.print("hello");      % str
py.print({'a', 'b'});   % list

% 显式转换为 Python 类型
pyInt = py.int(42);
pyFloat = py.float(3.14);
pyStr = py.str('hello');
pyList = py.list({1, 2, 3});
pyDict = py.dict(pyargs('key', 'value'));
```

### Python 到 MATLAB

```matlab
% 将 Python 类型转换为 MATLAB
matlabDouble = double(py.float(3.14));
matlabInt = int64(py.int(42));
matlabChar = char(py.str('hello'));
matlabString = string(py.str('hello'));
matlabCell = cell(py.list({1, 2, 3}));

% 转换 numpy 数组
pyArr = py.numpy.array({1, 2, 3, 4, 5});
matlabArr = double(pyArr);

% 将 pandas DataFrame 转换为 MATLAB 表格
pyDf = py.pandas.read_csv('data.csv');
matlabTable = table(pyDf);  % 需要 pandas2table 或类似工具

% 手动转换 DataFrame
colNames = cell(pyDf.columns.tolist());
data = cell(pyDf.values.tolist());
T = cell2table(data, 'VariableNames', colNames);
```

### 数组转换

```matlab
% MATLAB 数组转 numpy
matlabArr = [1 2 3; 4 5 6];
pyArr = py.numpy.array(matlabArr);

% numpy 转 MATLAB
pyArr = py.numpy.random.rand(int64(3), int64(4));
matlabArr = double(pyArr);

% 注意：numpy 使用行优先(C)顺序，MATLAB 使用列优先(Fortran)
% 可能需要转置以获得正确布局
```

## 处理 Python 对象

### 对象方法和属性

```matlab
% 调用方法
pyList = py.list({3, 1, 4, 1, 5});
pyList.append(9);
pyList.sort();

% 访问属性
pyStr = py.str('hello world');
upper = pyStr.upper();
words = pyStr.split();

% 检查属性
methods(pyStr)          % 列出方法
fieldnames(pyDict)      % 列出键名
```

### 遍历 Python 对象

```matlab
% 遍历 Python 列表
pyList = py.list({1, 2, 3, 4, 5});
for item = py.list(pyList)
    disp(item{1});
end

% 转换为 cell 数组后遍历
items = cell(pyList);
for i = 1:length(items)
    disp(items{i});
end

% 遍历字典键
pyDict = py.dict(pyargs('a', 1, 'b', 2, 'c', 3));
keys = cell(pyDict.keys());
for i = 1:length(keys)
    key = keys{i};
    value = pyDict{key};
    fprintf('%s: %d\n', char(key), int64(value));
end
```

### 错误处理

```matlab
try
    result = py.some_module.function_that_might_fail();
catch ME
    if isa(ME, 'matlab.exception.PyException')
        disp('发生 Python 错误:');
        disp(ME.message);
    else
        rethrow(ME);
    end
end
```

## 从 Python 调用 MATLAB

### 设置 MATLAB 引擎

```python
# 安装 MATLAB Engine API for Python
# 在 MATLAB 中: cd(fullfile(matlabroot,'extern','engines','python'))
# 然后执行: python setup.py install

import matlab.engine

# 启动 MATLAB 引擎
eng = matlab.engine.start_matlab()

# 或连接到共享会话 (MATLAB: matlab.engine.shareEngine)
eng = matlab.engine.connect_matlab()

# 列出可用会话
matlab.engine.find_matlab()
```

### 调用 MATLAB 函数

```python
import matlab.engine

eng = matlab.engine.start_matlab()

# 调用内置函数
result = eng.sqrt(16.0)
result = eng.sin(3.14159 / 2)

# 多输出处理
mean_val, std_val = eng.std([1, 2, 3, 4, 5], nargout=2)

# 矩阵运算
A = matlab.double([[1, 2], [3, 4]])
B = eng.inv(A)
C = eng.mtimes(A, B)  # 矩阵乘法

# 调用自定义函数（需在 MATLAB 路径中）
result = eng.myfunction(arg1, arg2)

# 清理
eng.quit()
```

### 数据类型转换（Python 到 MATLAB）

```python
import matlab.engine
import numpy as np

eng = matlab.engine.start_matlab()

# Python 到 MATLAB 类型转换
matlab_double = matlab.double([1.0, 2.0, 3.0])
matlab_int = matlab.int32([1, 2, 3])
matlab_complex = matlab.double([1+2j, 3+4j], is_complex=True)

# 二维数组
matlab_matrix = matlab.double([[1, 2, 3], [4, 5, 6]])

# numpy 转 MATLAB
np_array = np.array([[1, 2], [3, 4]], dtype=np.float64)
matlab_array = matlab.double(np_array.tolist())

# 使用 numpy 数据调用 MATLAB
result = eng.sum(matlab.double(np_array.flatten().tolist()))
```

### 异步调用

```python
import matlab.engine

eng = matlab.engine.start_matlab()

# 异步调用
future = eng.sqrt(16.0, background=True)

# 执行其他任务...

# 完成后获取结果
result = future.result()

# 检查是否完成
if future.done():
    result = future.result()

# 必要时取消
future.cancel()
```

## 常见工作流程

### 在 MATLAB 中使用 Python 库

```matlab
% 在 MATLAB 中使用 scikit-learn
sklearn = py.importlib.import_module('sklearn.linear_model');

% 准备数据
X = rand(100, 5);
y = X * [1; 2; 3; 4; 5] + randn(100, 1) * 0.1;

% 转换为 Python/numpy
X_py = py.numpy.array(X);
y_py = py.numpy.array(y);

% 训练模型
model = sklearn.LinearRegression();
model.fit(X_py, y_py);

% 获取系数
coefs = double(model.coef_);
intercept = double(model.intercept_);

% 预测
y_pred = double(model.predict(X_py));
```

### 在 Python 脚本中使用 MATLAB

```python
import matlab.engine
import numpy as np

# 启动 MATLAB
eng = matlab.engine.start_matlab()

# 使用 MATLAB 优化工具
def matlab_fmincon(objective, x0, A, b, Aeq, beq, lb, ub):
    """MATLAB fmincon 的封装"""
    # 转换为 MATLAB 类型
    x0_m = matlab.double(x0.tolist())
    A_m = matlab.double(A.tolist()) if A is not None else matlab.double([])
    b_m = matlab.double(b.tolist()) if b is not None else matlab.double([])

    # 调用 MATLAB（假设 objective 是 MATLAB 函数）
    x, fval = eng.fmincon(objective, x0_m, A_m, b_m, nargout=2)

    return np.array(x).flatten(), fval

# 使用 MATLAB 绘图
def matlab_plot(x, y, title_str):
    """使用 MATLAB 创建图表"""
    eng.figure(nargout=0)
    eng.plot(matlab.double(x.tolist()), matlab.double(y.tolist()), nargout=0)
    eng.title(title_str, nargout=0)
    eng.saveas(eng.gcf(), 'plot.png', nargout=0)

eng.quit()
```

### MATLAB 与 Python 共享数据

```matlab
% 保存数据供 Python 使用
data = rand(100, 10);
labels = randi([0 1], 100, 1);
save('data_for_python.mat', 'data', 'labels');

% 在 Python 中:
% import scipy.io
% mat = scipy.io.loadmat('data_for_python.mat')
% data = mat['data']
% labels = mat['labels']

% 加载 Python 保存的数据（使用 scipy.io.savemat）
loaded = load('data_from_python.mat');
data = loaded.data;
labels = loaded.labels;

% 替代方案：使用 CSV 进行简单数据交换
writematrix(data, 'data.csv');
% Python: pd.read_csv('data.csv')

% Python 写入: df.to_csv('results.csv')
results = readmatrix('results.csv');
```

### 使用 MATLAB 不可用的 Python 包

```matlab
% 示例：使用 Python 的 requests 库
requests = py.importlib.import_module('requests');

% 发起 HTTP 请求
response = requests.get('https://api.example.com/data');
status = int64(response.status_code);

if status == 200
    data = response.json();
    % 转换为 MATLAB 结构体
    dataStruct = struct(data);
end

% 示例：使用 Python 的 PIL/Pillow 进行高级图像处理
PIL = py.importlib.import_module('PIL.Image');

% 打开图像
img = PIL.open('image.png');

% 调整尺寸
img_resized = img.resize(py.tuple({int64(256), int64(256)}));

% 保存
img_resized.save('image_resized.png');
```
