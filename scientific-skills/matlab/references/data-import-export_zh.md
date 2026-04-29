# 数据导入导出参考指南

## 目录
1. [文本与CSV文件](#文本与csv文件)
2. [电子表格](#电子表格)
3. [MAT文件](#mat文件)
4. [图像](#图像)
5. [表格与数据类型](#表格与数据类型)
6. [底层文件I/O](#底层文件io)

## 文本与CSV文件

### 读取文本文件

```matlab
% 推荐的高级函数
T = readtable('data.csv');          % 读取为表格（混合类型）
M = readmatrix('data.csv');         % 读取为数值矩阵
C = readcell('data.csv');           % 读取为元胞数组
S = readlines('data.txt');          % 读取为字符串数组（按行）
str = fileread('data.txt');         % 读取整个文件为字符串

% 带选项读取
T = readtable('data.csv', 'ReadVariableNames', true);
T = readtable('data.csv', 'Delimiter', ',');
T = readtable('data.csv', 'NumHeaderLines', 2);
M = readmatrix('data.csv', 'Range', 'B2:D100');

% 检测导入选项
opts = detectImportOptions('data.csv');
opts.VariableNames = {'Col1', 'Col2', 'Col3'};
opts.VariableTypes = {'double', 'string', 'double'};
opts.SelectedVariableNames = {'Col1', 'Col3'};
T = readtable('data.csv', opts);
```

### 写入文本文件

```matlab
% 高级函数
writetable(T, 'output.csv');
writematrix(M, 'output.csv');
writecell(C, 'output.csv');
writelines(S, 'output.txt');

% 带选项写入
writetable(T, 'output.csv', 'Delimiter', '\t');
writetable(T, 'output.csv', 'WriteVariableNames', false);
writematrix(M, 'output.csv', 'Delimiter', ',');
```

### 制表符分隔文件

```matlab
% 读取
T = readtable('data.tsv', 'Delimiter', '\t');
T = readtable('data.txt', 'FileType', 'text', 'Delimiter', '\t');

% 写入
writetable(T, 'output.tsv', 'Delimiter', '\t');
writetable(T, 'output.txt', 'FileType', 'text', 'Delimiter', '\t');
```

## 电子表格

### 读取Excel文件

```matlab
% 基础读取
T = readtable('data.xlsx');
M = readmatrix('data.xlsx');
C = readcell('data.xlsx');

% 指定工作表
T = readtable('data.xlsx', 'Sheet', 'Sheet2');
T = readtable('data.xlsx', 'Sheet', 2);

% 指定范围
M = readmatrix('data.xlsx', 'Range', 'B2:D100');
M = readmatrix('data.xlsx', 'Sheet', 2, 'Range', 'A1:F50');

% 带选项读取
opts = detectImportOptions('data.xlsx');
opts.Sheet = 'Data';
opts.DataRange = 'A2';
preview(opts.VariableNames)     % 检查列名
T = readtable('data.xlsx', opts);

% 获取工作表名称
[~, sheets] = xlsfinfo('data.xlsx');
```

### 写入Excel文件

```matlab
% 基础写入
writetable(T, 'output.xlsx');
writematrix(M, 'output.xlsx');
writecell(C, 'output.xlsx');

% 指定工作表和范围
writetable(T, 'output.xlsx', 'Sheet', 'Results');
writetable(T, 'output.xlsx', 'Sheet', 'Data', 'Range', 'B2');
writematrix(M, 'output.xlsx', 'Sheet', 2, 'Range', 'A1');

% 追加到现有工作表（使用Range指定起始位置）
writetable(T2, 'output.xlsx', 'Sheet', 'Data', 'WriteMode', 'append');
```

## MAT文件

### 保存变量

```matlab
% 保存所有工作区变量
save('data.mat');

% 保存特定变量
save('data.mat', 'x', 'y', 'results');

% 带选项保存
save('data.mat', 'x', 'y', '-v7.3');    % 大文件(>2GB)
save('data.mat', 'x', '-append');        % 追加到现有文件
save('data.mat', '-struct', 's');        % 将结构体字段保存为变量

% 压缩选项
save('data.mat', 'x', '-v7');            % 压缩（默认）
save('data.mat', 'x', '-v6');            % 未压缩，速度更快
```

### 加载变量

```matlab
% 加载所有变量
load('data.mat');

% 加载特定变量
load('data.mat', 'x', 'y');

% 加载到结构体
S = load('data.mat');
S = load('data.mat', 'x', 'y');
x = S.x;
y = S.y;

% 列出内容而不加载
whos('-file', 'data.mat');
vars = who('-file', 'data.mat');
```

### MAT文件对象（大文件）

```matlab
% 创建MAT文件对象用于部分访问
m = matfile('data.mat');
m.Properties.Writable = true;

% 读取部分数据
x = m.bigArray(1:100, :);       % 仅前100行

% 写入部分数据
m.bigArray(1:100, :) = newData;

% 获取变量信息
sz = size(m, 'bigArray');
```

## 图像

### 读取图像

```matlab
% 读取图像
img = imread('image.png');
img = imread('image.jpg');
img = imread('image.tiff');

% 获取图像信息
info = imfinfo('image.png');
info.Width
info.Height
info.ColorType
info.BitDepth

% 读取特定帧（多页TIFF、GIF）
img = imread('animation.gif', 3);  % 第3帧
[img, map] = imread('indexed.gif');  % 带颜色映射的索引图像
```

### 写入图像

```matlab
% 写入图像
imwrite(img, 'output.png');
imwrite(img, 'output.jpg');
imwrite(img, 'output.tiff');

% 带选项写入
imwrite(img, 'output.jpg', 'Quality', 95);
imwrite(img, 'output.png', 'BitDepth', 16);
imwrite(img, 'output.tiff', 'Compression', 'lzw');

% 写入带颜色映射的索引图像
imwrite(X, map, 'indexed.gif');

% 追加到多页TIFF
imwrite(img1, 'multipage.tiff');
imwrite(img2, 'multipage.tiff', 'WriteMode', 'append');
```

### 图像格式

```matlab
% 支持的格式（部分列表）
% BMP  - Windows位图
% GIF  - 图形交换格式
% JPEG - 联合图像专家组
% PNG  - 便携式网络图形
% TIFF - 标签图像文件格式
% PBM, PGM, PPM - 便携式位图格式

% 检查支持的格式
formats = imformats;
```

## 表格与数据类型

### 创建表格

```matlab
% 从变量创建
T = table(var1, var2, var3);
T = table(var1, var2, 'VariableNames', {'Col1', 'Col2'});

% 从数组创建
T = array2table(M);
T = array2table(M, 'VariableNames', {'A', 'B', 'C'});

% 从元胞数组创建
T = cell2table(C);
T = cell2table(C, 'VariableNames', {'Name', 'Value'});

% 从结构体创建
T = struct2table(S);
```

### 访问表格数据

```matlab
% 按变量名访问
col = T.VariableName;
col = T.('VariableName');
col = T{:, 'VariableName'};

% 按索引访问
row = T(5, :);              % 第5行
col = T(:, 3);              % 第3列（表格形式）
data = T{:, 3};             % 第3列（数组形式）
subset = T(1:10, 2:4);      % 子集（表格形式）
data = T{1:10, 2:4};        % 子集（数组形式）

% 逻辑索引
subset = T(T.Value > 5, :);
```

### 修改表格

```matlab
% 添加变量
T.NewVar = newData;
T = addvars(T, newData, 'NewName', 'Col4');
T = addvars(T, newData, 'Before', 'ExistingCol');

% 删除变量
T.OldVar = [];
T = removevars(T, 'OldVar');
T = removevars(T, {'Col1', 'Col2'});

% 重命名变量
T = renamevars(T, 'OldName', 'NewName');
T.Properties.VariableNames{'OldName'} = 'NewName';

% 重排序变量
T = movevars(T, 'Col3', 'Before', 'Col1');
T = T(:, {'Col2', 'Col1', 'Col3'});
```

### 表格操作

```matlab
% 排序
T = sortrows(T, 'Column');
T = sortrows(T, 'Column', 'descend');
T = sortrows(T, {'Col1', 'Col2'}, {'ascend', 'descend'});

% 唯一行
T = unique(T);
T = unique(T, 'rows');

% 连接表格
T = join(T1, T2);                   % 内连接（基于共同键）
T = join(T1, T2, 'Keys', 'ID');
T = innerjoin(T1, T2);
T = outerjoin(T1, T2);

% 堆叠/解堆叠
T = stack(T, {'Var1', 'Var2'});
T = unstack(T, 'Values', 'Keys');

% 分组操作
G = groupsummary(T, 'GroupVar', 'mean', 'ValueVar');
G = groupsummary(T, 'GroupVar', {'mean', 'std'}, 'ValueVar');
```

### 元胞数组

```matlab
% 创建元胞数组
C = {1, 'text', [1 2 3]};
C = cell(m, n);             % 空m×n元胞数组

% 访问内容
contents = C{1, 2};         % 元胞(1,2)的内容
subset = C(1:2, :);         % 元胞子集（仍为元胞数组）

% 转换
A = cell2mat(C);            % 转换为矩阵（若兼容）
T = cell2table(C);          % 转换为表格
S = cell2struct(C, fields); % 转换为结构体
```

### 结构体

```matlab
% 创建结构体
S.field1 = value1;
S.field2 = value2;
S = struct('field1', value1, 'field2', value2);

% 访问字段
val = S.field1;
val = S.('field1');

% 字段名
names = fieldnames(S);
tf = isfield(S, 'field1');

% 结构体数组
S(1).name = 'Alice';
S(2).name = 'Bob';
names = {S.name};           % 提取所有名称
```

## 底层文件I/O

### 打开与关闭文件

```matlab
% 打开文件
fid = fopen('file.txt', 'r');   % 读取
fid = fopen('file.txt', 'w');   % 写入（覆盖）
fid = fopen('file.txt', 'a');   % 追加
fid = fopen('file.bin', 'rb');  % 读取二进制
fid = fopen('file.bin', 'wb');  % 写入二进制

% 错误检查
if fid == -1
    error('无法打开文件');
end

% 关闭文件
fclose(fid);
fclose('all');              % 关闭所有文件
```

### 文本文件I/O

```matlab
% 读取格式化数据
data = fscanf(fid, '%f');           % 读取浮点数
data = fscanf(fid, '%f %f', [2 Inf]);  % 两列数据
C = textscan(fid, '%f %s %f');      % 混合类型

% 读取行
line = fgetl(fid);          % 单行（不含换行符）
line = fgets(fid);          % 单行（含换行符）

% 写入格式化数据
fprintf(fid, '%d, %f, %s\n', intVal, floatVal, strVal);
fprintf(fid, '%6.2f\n', data);

% 读写字符串
str = fscanf(fid, '%s');
fprintf(fid, '%s', str);
```

### 二进制文件I/O

```matlab
% 读取二进制数据
data = fread(fid, n, 'double');     % n个双精度数
data = fread(fid, [m n], 'int32');  % m×n个int32
data = fread(fid, Inf, 'uint8');    % 所有字节

% 写入二进制数据
fwrite(fid, data, 'double');
fwrite(fid, data, 'int32');

% 数据类型: 'int8', 'uint8', 'int16', 'uint16', 'int32', 'uint32',
%             'int64', 'uint64', 'single', 'double', 'char'
```

### 文件位置

```matlab
% 获取位置
pos = ftell(fid);

% 设置位置
fseek(fid, 0, 'bof');       % 文件起始位置
fseek(fid, 0, 'eof');       % 文件结束位置
fseek(fid, offset, 'cof'); % 当前位置+偏移量

% 重置到起始位置
frewind(fid);

% 检查文件结束
tf = feof(fid);
```

### 文件与目录操作

```matlab
% 检查存在性
tf = exist('file.txt', 'file');
tf = exist('folder', 'dir');
tf = isfile('file.txt');
tf = isfolder('folder');

% 列出文件
files = dir('*.csv');           % 结构体数组
files = dir('folder/*.mat');
names = {files.name};

% 文件信息
info = dir('file.txt');
info.name
info.bytes
info.date
info.datenum

% 文件操作
copyfile('src.txt', 'dst.txt');
movefile('src.txt', 'dst.txt');
delete('file.txt');

% 目录操作
mkdir('newfolder');
rmdir('folder');
rmdir('folder', 's');           % 递归删除
cd('path');
pwd                             % 当前目录
```

### 路径操作

```matlab
% 构建路径
fullpath = fullfile('folder', 'subfolder', 'file.txt');
fullpath = fullfile(pwd, 'file.txt');

% 解析路径
[path, name, ext] = fileparts('/path/to/file.txt');
% path = '/path/to', name = 'file', ext = '.txt'

% 临时文件/目录
tmpfile = tempname;
tmpdir = tempdir;
```
