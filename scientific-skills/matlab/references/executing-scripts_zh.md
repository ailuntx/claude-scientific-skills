# 从Bash运行MATLAB和GNU Octave脚本

本文档展示了在Bash环境中使用MATLAB（MathWorks）和GNU Octave执行MATLAB风格`.m`脚本的常用方法。内容涵盖交互式使用、非交互式批处理运行、传递参数、捕获输出以及用于自动化和CI的实用模式。

## 目录

- 环境要求
- 快速对比
- 从Bash运行MATLAB脚本
  - 交互模式
  - 非交互式运行脚本
  - 带参数运行函数
  - 运行单行命令
  - 工作目录与路径处理
  - 捕获输出和退出码
  - 常用MATLAB脚本标志
- 从Bash运行Octave脚本
  - 交互模式
  - 非交互式运行脚本
  - 带参数运行函数
  - 运行单行命令
  - 使`.m`文件可执行（shebang）
  - 工作目录与路径处理
  - 捕获输出和退出码
  - 常用Octave脚本标志
- 跨平台兼容性提示（MATLAB + Octave）
- 示例：便携式运行脚本
- 故障排除

## 环境要求

### MATLAB
- 必须安装MATLAB
- `matlab`可执行文件需在PATH中，或使用完整路径引用
- 需要有效许可证

验证：
```bash
matlab -help | head
```

### GNU Octave

* 必须安装Octave
* `octave`可执行文件需在PATH中

验证：

```bash
octave --version
```

## 快速对比

| 任务                          | MATLAB                            | Octave                   |
| ----------------------------- | --------------------------------- | ------------------------ |
| 交互式shell                   | `matlab` (默认启动GUI)            | `octave`                 |
| 无头运行(CI)                  | `matlab -batch "cmd"` (推荐)      | `octave --eval "cmd"`    |
| 运行脚本文件                  | `matlab -batch "run('file.m')"`   | `octave --no-gui file.m` |
| 带退出码结束                  | `exit(n)`                         | `exit(n)`                |
| 使`.m`直接可执行              | 不常见                            | 常用shebang实现          |

## 从Bash运行MATLAB脚本

### 1) 交互模式

启动MATLAB。根据平台和安装方式，可能启动GUI界面。

```bash
matlab
```

纯终端使用推荐添加`-nodesktop`，可选`-nosplash`：

```bash
matlab -nodesktop -nosplash
```

### 2) 非交互式运行脚本

推荐现代方法：`-batch`。执行命令后自动退出。

使用`run()`运行脚本：

```bash
matlab -batch "run('myscript.m')"
```

若脚本依赖所在目录，需先设置工作目录：

```bash
matlab -batch "cd('/path/to/project'); run('myscript.m')"
```

传统替代方案：`-r`（自动化中可靠性较低，需手动确保退出）：

```bash
matlab -nodisplay -nosplash -r "run('myscript.m'); exit"
```

### 3) 带参数运行函数

若文件定义函数，直接调用。推荐`-batch`：

```bash
matlab -batch "myfunc(123, 'abc')"
```

传递Bash变量值：

```bash
matlab -batch "myfunc(${N}, '${NAME}')"
```

若参数含引号或空格，建议编写MATLAB包装函数读取环境变量。

### 4) 运行单行命令

```bash
matlab -batch "disp(2+2)"
```

多语句执行：

```bash
matlab -batch "a=1; b=2; fprintf('%d\n', a+b)"
```

### 5) 工作目录与路径处理

常用选项：

* 启动时切换目录：

```bash
matlab -batch "cd('/path/to/project'); myfunc()"
```

* 添加代码目录到MATLAB路径：

```bash
matlab -batch "addpath('/path/to/lib'); myfunc()"
```

包含子目录：

```bash
matlab -batch "addpath(genpath('/path/to/project')); myfunc()"
```

### 6) 捕获输出和退出码

捕获标准输出/错误：

```bash
matlab -batch "run('myscript.m')" > matlab.out 2>&1
```

检查退出码：

```bash
matlab -batch "run('myscript.m')"
echo $?
```

显式返回失败状态，使用`exit(1)`。示例模式：

```matlab
try
  run('myscript.m');
catch ME
  disp(getReport(ME));
  exit(1);
end
exit(0);
```

执行：

```bash
matlab -batch "try, run('myscript.m'); catch ME, disp(getReport(ME)); exit(1); end; exit(0);"
```

### 7) MATLAB脚本常用标志

实用选项：

* `-batch "cmd"`：执行命令后返回进程退出码并退出
* `-nodisplay`：无显示（适用于无头系统）
* `-nodesktop`：无桌面GUI
* `-nosplash`：无启动画面
* `-r "cmd"`：执行命令；需包含`exit`才能终止

具体可用性因MATLAB版本而异，请使用`matlab -help`查看当前版本支持。

## 从Bash运行GNU Octave脚本

### 1) 交互模式

```bash
octave
```

静默模式：

```bash
octave --quiet
```

### 2) 非交互式运行脚本

执行文件后退出：

```bash
octave --no-gui myscript.m
```

静默执行：

```bash
octave --quiet --no-gui myscript.m
```

某些环境使用：

```bash
octave --no-window-system myscript.m
```

### 3) 带参数运行函数

若`myfunc.m`定义了函数`myfunc`，通过`--eval`调用：

```bash
octave --quiet --eval "myfunc(123, 'abc')"
```

若函数不在Octave路径，需先添加路径：

```bash
octave --quiet --eval "addpath('/path/to/project'); myfunc()"
```

### 4) 运行单行命令

```bash
octave --quiet --eval "disp(2+2)"
```

多语句执行：

```bash
octave --quiet --eval "a=1; b=2; printf('%d\n', a+b);"
```

### 5) 使`.m`文件可执行（shebang）

这是Octave中常见的"独立脚本"模式。

创建`myscript.m`：

```matlab
#!/usr/bin/env octave
disp("你好，来自Octave");
```

添加可执行权限：

```bash
chmod +x myscript.m
```

运行：

```bash
./myscript.m
```

若需标志（如静默、无GUI），建议使用包装脚本，因shebang行在跨平台时通常只支持有限参数。

### 6) 工作目录与路径处理

运行前在shell中切换目录：

```bash
cd /path/to/project
octave --quiet --no-gui myscript.m
```

或在Octave内切换目录：

```bash
octave --quiet --eval "cd('/path/to/project'); run('myscript.m');"
```

添加路径：

```bash
octave --quiet --eval "addpath('/path/to/lib'); run('myscript.m');"
```

### 7) 捕获输出和退出码

捕获标准输出/错误：

```bash
octave --quiet --no-gui myscript.m > octave.out 2>&1
```

退出码：

```bash
octave --quiet --no-gui myscript.m
echo $?
```

错误时强制返回非零退出码：

```matlab
try
  run('myscript.m');
catch err
  disp(err.message);
  exit(1);
end
exit(0);
```

执行：

```bash
octave --quiet --eval "try, run('myscript.m'); catch err, disp(err.message); exit(1); end; exit(0);"
```

### 8) Octave脚本常用标志

实用选项：

* `--eval "cmd"`：执行命令字符串
* `--quiet`：抑制启动信息
* `--no-gui`：禁用GUI
* `--no-window-system`：某些安装中的类似无头模式
* `--persist`：执行后保持Octave运行（与批处理行为相反）

验证：

```bash
octave --help | head -n 50
```

## 跨平台兼容性提示（MATLAB和Octave）

1. 自动化场景优先使用函数而非脚本  
   函数提供更清晰的参数传递和命名空间管理。

2. 需要兼容性时避免工具箱特定调用  
   许多MATLAB工具箱在Octave中无等效实现。

3. 注意字符串和引号处理  
   MATLAB和Octave均支持`'单引号'`字符串，新版MATLAB支持`"双引号"`。为最大限度兼容，除非确认Octave版本支持所需双引号用法，否则优先使用单引号。

4. 使用`fprintf`或`disp`输出  
   在CI日志中保持输出简洁且确定。

5. 确保退出码反映执行状态  
   两种环境中，`exit(0)`表示成功，`exit(1)`表示失败。

## 示例：便携式Bash运行器

此脚本优先尝试MATLAB，不可用时转Octave。

创建`run_mfile.sh`：

```bash
#!/usr/bin/env bash
set -euo pipefail

FILE="${1:?用法: run_mfile.sh 脚本/函数路径.m}"
CMD="${2:-}"  # 可选命令覆盖

if command -v matlab >/dev/null 2>&1; then
  if [[ -n "$CMD" ]]; then
    matlab -batch "$CMD"
  else
    matlab -batch "run('${FILE}')"
  fi
elif command -v octave >/dev/null 2>&1; then
  if [[ -n "$CMD" ]]; then
    octave --quiet --no-gui --eval "$CMD"
  else
    octave --quiet --no-gui "$FILE"
  fi
else
  echo "在PATH中未找到matlab或octave" >&2
  exit 127
fi
```

添加可执行权限：

```bash
chmod +x run_mfile.sh
```

运行：

```bash
./run_mfile.sh myscript.m
```

或运行函数调用：

```bash
./run_mfile.sh myfunc.m "myfunc(1, 'abc')"
```

## 故障排除

### MATLAB: 命令未找到

* 将MATLAB加入PATH，或使用完整路径调用，例如：

```bash
/Applications/MATLAB_R202x?.app/bin/matlab -batch "disp('ok')"
```

### Octave: 服务器上GUI问题

* 使用`--no-gui`或`--no-window-system`。

### 脚本依赖相对路径

* 启动前`cd`到脚本目录，或在MATLAB/Octave内先执行`cd()`再调用`run()`。

### 传递字符串时引号问题

* 避免在`--eval`或`-batch`中使用复杂引号。
* 当输入复杂时，使用环境变量并在MATLAB/Octave内部读取。

### MATLAB与Octave行为差异

* 检查是否存在不支持的函数或工具箱调用。
* 使用`--eval`或`-batch`运行最小复现步骤以隔离兼容性问题。
