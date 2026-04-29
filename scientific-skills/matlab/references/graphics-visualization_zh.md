# 图形与可视化参考

## 目录
1. [二维绘图](#2d-plotting)
2. [三维绘图](#3d-plotting)
3. [专业图表](#specialized-plots)
4. [图形管理](#figure-management)
5. [自定义设置](#customization)
6. [导出与保存](#exporting-and-saving)

## 二维绘图

### 折线图

```matlab
% 基础折线图
plot(y);                        % 绘制y与索引的关系
plot(x, y);                     % 绘制y与x的关系
plot(x, y, 'r-');               % 红色实线
plot(x, y, 'b--o');             % 蓝色虚线带圆圈标记

% 线条规范: [颜色][标记][线型]
% 颜色: r g b c m y k w (红, 绿, 蓝, 青, 品红, 黄, 黑, 白)
% 标记: o + * . x s d ^ v > < p h
% 线型: - -- : -.

% 多组数据
plot(x1, y1, x2, y2, x3, y3);
plot(x, [y1; y2; y3]');         % 将列作为独立线条绘制

% 设置属性
plot(x, y, 'LineWidth', 2, 'Color', [0.5 0.5 0.5]);
plot(x, y, 'Marker', 'o', 'MarkerSize', 8, 'MarkerFaceColor', 'r');

% 获取句柄用于后续修改
h = plot(x, y);
h.LineWidth = 2;
h.Color = 'red';
```

### 散点图

```matlab
scatter(x, y);                  % 基础散点图
scatter(x, y, sz);              % 设置标记尺寸
scatter(x, y, sz, c);           % 设置颜色
scatter(x, y, sz, c, 'filled'); % 填充标记

% sz: 标量或向量 (标记尺寸)
% c: 颜色标识符, 标量, 向量 (颜色映射) 或 RGB 矩阵

% 属性设置
scatter(x, y, 'MarkerEdgeColor', 'b', 'MarkerFaceColor', 'r');
```

### 条形图

```matlab
bar(y);                         % 垂直条形图
bar(x, y);                      % 指定x位置
barh(y);                        % 水平条形图

% 分组与堆叠
bar(Y);                         % 每列为一组
bar(Y, 'stacked');              % 堆叠条形图

% 属性设置
bar(y, 'FaceColor', 'b', 'EdgeColor', 'k', 'LineWidth', 1.5);
bar(y, 0.5);                    % 条形宽度 (0到1之间)
```

### 面积图

```matlab
area(y);                        % 曲线下填充区域
area(x, y);
area(Y);                        % 堆叠面积图
area(Y, 'FaceAlpha', 0.5);      % 半透明效果
```

### 直方图

```matlab
histogram(x);                   % 自动分箱
histogram(x, nbins);            % 指定箱数
histogram(x, edges);            % 指定边界
histogram(x, 'BinWidth', w);    % 指定箱宽

% 归一化
histogram(x, 'Normalization', 'probability');
histogram(x, 'Normalization', 'pdf');
histogram(x, 'Normalization', 'count');  % 默认

% 二维直方图
histogram2(x, y);
histogram2(x, y, 'DisplayStyle', 'tile');
histogram2(x, y, 'FaceColor', 'flat');
```

### 误差线

```matlab
errorbar(x, y, err);            % 对称误差
errorbar(x, y, neg, pos);       % 非对称误差
errorbar(x, y, yneg, ypos, xneg, xpos);  % X和Y方向误差

% 水平误差
errorbar(x, y, err, 'horizontal');

% 带线型设置
errorbar(x, y, err, 'o-', 'LineWidth', 1.5);
```

### 对数坐标图

```matlab
semilogy(x, y);                 % Y轴对数坐标
semilogx(x, y);                 % X轴对数坐标
loglog(x, y);                   % 双对数坐标
```

### 极坐标图

```matlab
polarplot(theta, rho);          % 极坐标系
polarplot(theta, rho, 'r-o');   % 带线型规范

% 自定义极坐标轴
pax = polaraxes;
pax.ThetaDir = 'clockwise';
pax.ThetaZeroLocation = 'top';
```

## 三维绘图

### 线条与散点

```matlab
% 三维线图
plot3(x, y, z);
plot3(x, y, z, 'r-', 'LineWidth', 2);

% 三维散点
scatter3(x, y, z);
scatter3(x, y, z, sz, c, 'filled');
```

### 曲面图

```matlab
% 先创建网格
[X, Y] = meshgrid(-2:0.1:2, -2:0.1:2);
Z = X.^2 + Y.^2;

% 曲面图
surf(X, Y, Z);                  % 带边缘的曲面
surf(Z);                        % 使用索引作为X,Y

% 曲面属性
surf(X, Y, Z, 'FaceColor', 'interp', 'EdgeColor', 'none');
surf(X, Y, Z, 'FaceAlpha', 0.5);  % 半透明

% 网格图 (线框)
mesh(X, Y, Z);
mesh(X, Y, Z, 'FaceColor', 'none');

% 带下方等高线的曲面
surfc(X, Y, Z);
meshc(X, Y, Z);
```

### 等高线图

```matlab
contour(X, Y, Z);               % 二维等高线
contour(X, Y, Z, n);            % n条等高线
contour(X, Y, Z, levels);       % 指定层级
contourf(X, Y, Z);              % 填充等高线

[C, h] = contour(X, Y, Z);
clabel(C, h);                   % 添加标签

% 三维等高线
contour3(X, Y, Z);
```

### 其他三维图表

```matlab
% 三维条形图
bar3(Z);                        
bar3(Z, 'stacked');

% 三维饼图
pie3(X);                        

% 瀑布图
waterfall(X, Y, Z);             % 类似无背线的网格图

% 带状图
ribbon(Y);                      % 三维带状图

% 三维针状图
stem3(x, y, z);                 
```

### 视角与光照

```matlab
% 设置视角
view(az, el);                   % 方位角, 仰角
view(2);                        % 俯视图 (二维视角)
view(3);                        % 默认三维视角
view([1 1 1]);                  % 指定方向视角

% 光照
light;                          % 添加光源
light('Position', [1 0 1]);
lighting gouraud;               % 平滑光照
lighting flat;                  % 平面着色
lighting none;                  % 无光照

% 材质属性
material shiny;
material dull;
material metal;

% 着色模式
shading flat;                   % 每面单色
shading interp;                 % 插值着色
shading faceted;                % 带边缘 (默认)
```

## 专业图表

### 统计图表

```matlab
% 箱线图
boxplot(data);
boxplot(data, groups);          % 分组
boxplot(data, 'Notch', 'on');   % 带凹口

% 小提琴图 (R2023b+)
violinplot(data);

% 热力图
heatmap(data);
heatmap(xLabels, yLabels, data);
heatmap(T, 'XVariable', 'Col1', 'YVariable', 'Col2', 'ColorVariable', 'Val');

% 平行坐标图
parallelplot(data);
```

### 图像显示

```matlab
% 显示图像
imshow(img);                    % 自动缩放
imshow(img, []);                % 全范围缩放
imshow(img, [low high]);        % 指定显示范围

% 图像作为图表
image(C);                       % 直接索引颜色
imagesc(data);                  % 缩放颜色
imagesc(data, [cmin cmax]);     % 指定颜色范围

% 为imagesc设置颜色映射
imagesc(data);
colorbar;
colormap(jet);
```

### 箭头与流线图

```matlab
% 矢量场
[X, Y] = meshgrid(-2:0.5:2);
U = -Y;
V = X;
quiver(X, Y, U, V);             % 二维箭头
quiver3(X, Y, Z, U, V, W);      % 三维箭头

% 流线图
streamline(X, Y, U, V, startx, starty);
```

### 饼图与环形图

```matlab
pie(X);                         % 饼图
pie(X, explode);                % 突出切片 (逻辑值)
pie(X, labels);                 % 带标签

% 环形图 (使用patch或变通方法)
pie(X);
% 在中心添加白色圆形实现环形效果
```

## 图形管理

### 创建图形窗口

```matlab
figure;                         % 新建图形窗口
figure(n);                      % 指定编号的图形
fig = figure;                   % 获取句柄
fig = figure('Name', '我的图形', 'Position', [100 100 800 600]);

% 图形属性
fig.Color = 'white';
fig.Units = 'pixels';
fig.Position = [左 下 宽 高];
```

### 子图布局

```matlab
subplot(m, n, p);               % m×n网格, 位置p
subplot(2, 2, 1);               % 2×2网格左上角

% 跨位置布局
subplot(2, 2, [1 2]);           % 顶部整行

% 带间距控制
tiledlayout(2, 2);              % 现代替代方案
nexttile;
plot(x1, y1);
nexttile;
plot(x2, y2);

% 跨图块布局
nexttile([1 2]);                % 跨越2列
```

### 图形叠加

```matlab
hold on;                        % 保留现有图形，添加新图
plot(x1, y1);
plot(x2, y2);
hold off;                       % 释放

% 替代方法
hold(ax, 'on');
hold(ax, 'off');
```

### 多坐标轴

```matlab
% 双Y轴
yyaxis left;
plot(x, y1);
ylabel('左侧Y轴');
yyaxis right;
plot(x, y2);
ylabel('右侧Y轴');

% 联动坐标轴
ax1 = subplot(2,1,1); plot(x, y1);
ax2 = subplot(2,1,2); plot(x, y2);
linkaxes([ax1, ax2], 'x');      % 联动X轴
```

### 当前对象

```matlab
gcf;                            % 当前图形句柄
gca;                            % 当前坐标轴句柄
gco;                            % 当前对象句柄

% 设置当前对象
figure(fig);
axes(ax);
```

## 自定义设置

### 标签与标题

```matlab
title('我的标题');
title('我的标题', 'FontSize', 14, 'FontWeight', 'bold');

xlabel('X轴标签');
ylabel('Y轴标签');
zlabel('Z轴标签');              % 三维图形专用

% 带解释器
title('$$\int_0^1 x^2 dx$$', 'Interpreter', 'latex');
xlabel('时间(秒)', 'Interpreter', 'none');
```

### 图例

```matlab
legend('系列1', '系列2');
legend({'系列1', '系列2'});
legend('Location', 'best');     % 自动定位
legend('Location', 'northeast');
legend('Location', 'northeastoutside');

% 指定图形对象
h1 = plot(x1, y1);
h2 = plot(x2, y2);
legend([h1, h2], {'数据1', '数据2'});

legend('off');                  % 移除图例
legend('boxoff');               % 移除图例框
```

### 坐标轴控制

```matlab
axis([xmin xmax ymin ymax]);    % 设置范围
axis([xmin xmax ymin ymax zmin zmax]);  % 三维范围
xlim([xmin xmax]);
ylim([ymin ymax]);
zlim([zmin zmax]);

axis equal;                     % 等比例坐标
axis square;                    % 方形坐标
axis tight;                     % 紧贴数据
axis auto;                      % 自动调整
axis off;                       % 隐藏坐标轴
axis on;                        % 显示坐标轴

% 反转方向
set(gca, 'YDir', 'reverse');
set(gca, 'XDir', 'reverse');
```

### 网格与边框

```matlab
grid on;
grid off;
grid minor;                     % 次要网格线

box on;                         % 显示边框
box off;                        % 隐藏边框
```

### 刻度设置

```matlab
xticks([0 1 2 3 4 5]);
yticks(0:0.5:3);

xticklabels({'A', 'B', 'C', 'D', 'E', 'F'});
yticklabels({'低', '中', '高'});

xtickangle(45);                 % 旋转标签
ytickformat('%.2f');            % 数值格式
xtickformat('usd');             % 货币格式
```

### 颜色与颜色映射

```matlab
% 预定义颜色映射
colormap(jet);
colormap(parula);               % 默认
colormap(hot);
colormap(cool);
colormap(gray);
colormap(bone);
colormap(hsv);
colormap(turbo);
colormap(viridis);

% 颜色条
colorbar;
colorbar('Location', 'eastoutside');
caxis([cmin cmax]);             % 颜色范围
clim([cmin cmax]);              % R2022a+语法

% 自定义颜色映射
cmap = [1 0 0; 0 1 0; 0 0 1];   % 红, 绿, 蓝
colormap(cmap);

% 线条颜色顺序
colororder(colors);             % R2019b+
```

### 文本与标注

```matlab
% 添加文本
text(x, y, '标签');
text(x, y, z, '标签');         % 三维
text(x, y, '标签', 'FontSize', 12, 'Color', 'red');
text(x, y, '标签', 'HorizontalAlignment', 'center');

% 标注工具
annotation('arrow', [x1 x2], [y1 y2]);
annotation('textarrow', [x1 x2], [y1 y2], 'String', '峰值');
annotation('ellipse', [x y w h]);
annotation('rectangle', [x y w h]);
annotation('line', [x1 x2], [y1 y2]);

% LaTeX文本
text(x, y, '$$\alpha = \beta^2$$', 'Interpreter', 'latex');
```

### 参考线与形状

```matlab
% 参考线
xline(5);                       % X=5处的垂直线
yline(10);                      % Y=10处的水平线
xline(5, '--r', '阈值');        % 带标签

% 形状
rectangle('Position', [x y w h]);
rectangle('Position', [x y w h], 'Curvature', [0.2 0.2]);  % 圆角矩形

% 多边形填充
patch(xv, yv, 'blue');
patch(xv, yv, zv, 'blue');      % 三维
```

## 导出与保存

### 保存图形

```matlab
saveas(gcf, 'figure.png');
saveas(gcf, 'figure.fig');      % MATLAB图形文件
saveas(gcf, 'figure.pdf');
saveas(gcf, 'figure.eps');
```

### 打印命令

```matlab
print('-dpng', 'figure.png');
print('-dpng', '-r300', 'figure.png');  % 300 DPI
print('-dpdf', 'figure.pdf');
print('-dsvg', 'figure.svg');
print('-deps', 'figure.eps');
print('-depsc', 'figure.eps');  % 彩色EPS

% 出版级矢量格式
print('-dpdf', '-painters', '

```matlab
exportgraphics(gcf, 'figure.eps');    % 用于LaTeX
```

### 复制到剪贴板

```matlab
copygraphics(gcf);              % 复制当前图形
copygraphics(gca);              % 复制当前坐标轴
copygraphics(gcf, 'ContentType', 'vector'); % 指定内容类型为矢量
```

### 纸张尺寸（用于打印）

```matlab
set(gcf, 'PaperUnits', 'inches'); % 设置单位为英寸
set(gcf, 'PaperPosition', [0 0 6 4]); % 设置位置和尺寸[宽 高]
set(gcf, 'PaperSize', [6 4]);     % 设置纸张尺寸
set(gcf, 'PaperPositionMode', 'auto'); % 启用自动定位模式
```
