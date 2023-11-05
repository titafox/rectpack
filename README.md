# rectpack [![Build Status](https://travis-ci.org/secnot/rectpack.svg?branch=master)](https://travis-ci.org/secnot/rectpack)


Rectpack是一组用于解决2D背包问题的启发式算法，也称为装箱问题。本质上是将一组矩形装入最少数量的箱中。

![alt tag](docs/maxrects.png)


## 安装

下载该软件包或克隆存储库，然后使用以下方式安装：

```bash
python setup.py install
```

或者 使用 pypi:

```bash
pip install rectpack
```

## 基本用法

将矩形装入一定数量的箱子非常简单：

```python
from rectpack import newPacker

rectangles = [(100, 30), (40, 60), (30, 30),(70, 70), (100, 50), (30, 30)]
bins = [(300, 450), (80, 40), (200, 150)]

packer = newPacker()

# 将矩形添加到打包队列
for r in rectangles:
	packer.add_rect(*r)

# 添加矩形将放置的箱子
for b in bins:
	packer.add_bin(*b)

# 开始装箱
packer.pack()
```

一旦矩形被装入箱子，可以逐个访问结果。

```python
# 获取用于装箱的箱子数量。
nbins = len(packer)

# 索引第一个箱子
abin = packer[0]

# 箱子尺寸（箱子在装箱过程中可以重新排序）
width, height = abin.width, abin.height

# 装入第一个箱子的矩形数量
nrect = len(packer[0])

# 第二个箱子的第一个矩形
rect = packer[1][0]

# `rect` 是一个矩形对象。
x = rect.x # rectangle bottom-left x coordinate
y = rect.y # rectangle bottom-left y coordinate
w = rect.width
h = rect.height
```

遍历它们所有

```python
for abin in packer:
  print(abin.bid) # 如果有的话，箱子的ID
  for rect in abin:
    print(rect)
```

or using **rect_list()**

```python
# 完整的矩形列表
all_rects = packer.rect_list()
for rect in all_rects:
	b, x, y, w, h, rid = rect

# b - 箱子索引
# x - 矩形左下角的 x 坐标
# y - 矩形左下角的 y 坐标
# w - 矩形宽度
# h - 矩形高度
# rid - 用户分配的矩形ID或无（None）
```

最后，所有的尺寸（箱子和矩形）必须是整数或小数，以避免由浮点数舍入引起的碰撞。如果您的数据是浮点数，请使用 float2dec 将浮点值转换为小数（请参见下面的 float）。


## API

以下是API调用的更详细描述：

* class **newPacker**([, mode][, bin_algo][, pack_algo][, sort_algo][, rotation])  
  返回一个新的装箱器对象
  * mode: 操作模式
    * PackingMode.Offline: 预先知道矩形集，装箱不会开始，直到调用 *pack()* 方法为止。
    * PackingMode.Online: 开始时未知矩形，它们将在添加时立即进行装箱。
  * bin_algo: 箱子选择启发式算法
    * PackingBin.BNF: (箱子下一个适合) 如果一个矩形不适合当前的箱子，关闭它并尝试下一个。
    * PackingBin.BFF: (箱子首个适合) 将矩形装入第一个适合它的箱子（不关闭）。
    * PackingBin.BBF: (箱子最佳适合) 将矩形装入提供最佳适应度的箱子。
    * PackingBin.Global: 对于每个箱子，直到装满为止，都将矩形装入适应度最佳的箱子，然后继续下一个箱子。
  * pack_algo: 支持的装箱算法之一（请参见下面的列表）
  * sort_algo: 装箱之前的矩形排序顺序（仅适用于离线模式）
    * SORT_NONE: 矩形保持未排序。
    * SORT_AREA: 按面积降序排序。
    * SORT_PERI: 按周长降序排序。
    * SORT_DIFF: 按矩形边长差异排序。
    * SORT_SSIDE: 按最短边排序。
    * SORT_LSIDE: 按最长边排序。
    * SORT_RATIO: 按边长比例排序。
  * rotation: 启用或禁用矩形旋转。
    
* packer.**add_bin**(width, height[, count][, bid])  
  向装箱器添加一个或多个空箱
  * width: 箱子的宽度
  * height: 箱子的高度
  * count: 要添加的箱子数量，默认为1。也可以通过 *count=float("inf")* 添加无限数量的箱子。
  * bid: 可选的箱子标识符


* packer.**add_rect**(width, height[, rid])  
  将矩形添加到装箱队列
  * width: 矩形的宽度
  * height: 矩形的高度
  * rid: 用户分配的矩形ID


* packer.**pack**():  
  启动装箱过程（仅适用于离线模式）。

* packer.**rect_list**():  
  返回装箱后的矩形列表，每个矩形用元组 (b, x, y, w, h, rid) 表示，其中：
  * b: 矩形装入的箱子的索引
  * x: 矩形左下角的 x 坐标
  * y: 矩形左下角的 y 坐标
  * w: 矩形宽度
  * h: 矩形高度
  * rid: 用户提供的ID或无（None）


## 支持的算法

该库实现了[1]中描述的三种算法：Skyline、Maxrects 和 Guillotine，具有以下变种：

* MaxRects
  * MaxRectsBl
  * MaxRectsBssf
  * MaxRectsBaf
  * MaxRectsBlsf

* Skyline
  * SkylineBl
  * SkylineBlWm
  * SkylineMwf
  * SkylineMwfl
  * SkylineMwfWm
  * SkylineMwflWm

* Guillotine
  * GuillotineBssfSas
  * GuillotineBssfLas
  * GuillotineBssfSlas
  * GuillotineBssfLlas
  * GuillotineBssfMaxas
  * GuillotineBssfMinas
  * GuillotineBlsfSas
  * GuillotineBlsfLas
  * GuillotineBlsfSlas
  * GuillotineBlsfLlas
  * GuillotineBlsfMaxas
  * GuillotineBlsfMinas
  * GuillotineBafSas
  * GuillotineBafLas
  * GuillotineBafSlas
  * GuillotineBafLlas
  * GuillotineBafMaxas
  * GuillotineBafMinas

我建议使用默认算法，除非装箱速度太慢，这种情况下可以切换到 Guillotine 的一个变种，例如 *GuillotineBssfSas*。您可以在[1]中了解更多关于这些算法的信息。

## 测试

Rectpack 经过了全面的测试，可以使用以下命令运行测试：

```bash
python setup.py test
```

或者

```bash
python -m unittest discover
```

## 浮点数

如果需要使用浮点数，只需将其转换为定点数，使用 Decimal 类型，注意四舍五入，使实际矩形尺寸始终小于转换值。Rectpack 提供了帮助函数 **float2dec** 来执行此任务，它接受一个数字和要四舍五入的小数位数，并返回四舍五入的 Decimal。

```python
from rectpack import float2dec, newPacker

float_rects = [...]
dec_rects = [(float2dec(r[0], 3), float2dec(r[1], 3)) for r in float_rects]

p = newPacker()
...
```

## 参考文献

[1] Jukka Jylang - A Thousand Ways to Pack the Bin - A Practical Approach to Two-Dimensional Rectangle Bin Packing (2010)

[2] Huang, E. Korf - Optimal Rectangle Packing: An Absolute Placement Approach (2013)
