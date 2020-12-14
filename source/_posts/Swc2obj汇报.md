
---
title: swc2obj-- 项目问题
---


# 实验结果
以比较大的004.obj为例
![picture 1](/images/28a776b63101d5c47df5911d9f3fcdd1b7fb1e3d074a23aa47ca7e017f0d47a9.png)  
> 所有obj已经打包发送到老师的qq，所以老师直接点击obj即可打开。

## 实验问题
- 其实有两种Padding

- [x] 为了防止断裂的padding
```C++
1. 由于我们marching cube是基于某一个文件的，所以第一个体数据从0开始到127结束，共128个。那么第二个体数据
同样必须从127开始254结束，共128个。也即重复一个。
```
>如果没有重复就会导致：
![picture 2](/images/0051a00e6191f9d0da89208d21db06c205b32fe9f2a22ad597c807f0ca6f92c9.png)  

- [x] 为了防止不能耦合的padding 
```C++
2. 由于我们需要contour等值线，所以我们必须对体数据进行average,分块之后，average有多少你就必须加多少为average服务的padding。
```
> 如果不加这个average的padding的话：
![picture 3](/images/b52b37679e2c036dc966500cd5b42b57eaba37d2b85756e1c4756ce9823bd79c.png)  


我们加了以上两个padding,同时加大加粗防止断裂的padding，直接设成6个，每一边三个。在这种情况下，达到了如下效果：
![picture 4](/images/4bf39bbe6eebb89967acba85e532b3d67143fa2b8190d2eabea808c380c46be3.png)  
![![picture 4](/images/4bf39bbe6eebb89967acba85e532b3d67143fa2b8190d2eabea808c380c46be3.png)  
 1](/images/91dffc44914a72fcc03d016c36aaaa0a5fa4fc8ec50460b137f56534cfe02616.png)  
肉眼已经很难分辨出是否闭合了，即使放的非常近，也无法用肉眼看出了。但是水密性问题并未得到验证。

## path插值
***由于数据中出现了大量的0.0000的半径，所以我们必须插值。 
插值想法是，找到最近的10个半径不为0的节点，进行平均，作为目前的这个0半径节点的半径。前面5个，后面5个。 
如果一个方向不足5个，由另一个方向补充，反正一定要10个。 （这个算法如果您觉得不合适，请告诉我大概如何插值会比较好。 ***


"D:/software/ParaView 5.8.1-Windows-Python3.7-msvc2015-64bit/bin/python4.exe" C:/Users/lidan/PycharmProjects/volume2obj/main.py -i C:/Users/lidan/Desktop/SWC2Obj/newResult/N022.swc -o C:/Users/lidan/Desktop/SWC2Obj/newResult/N022_use.obj &&  C:/Users/lidan/Desktop/SWC2Obj/cmake-build-release/ObjMerge/mergeOBJ.exe -i  C:/Users/lidan/Desktop/SWC2Obj/newResult/N022_use.obj -o C:/Users/lidan/Desktop/SWC2Obj/newResult/N022_new_use.obj



## 关于padding

+ 1. 需要生成等值面，而且是较为平滑的等值面，所以，必须首先对体数据进行卷积，那么这个过程中卷积n次，每个维度加2*n个padding,这个padding参与计算，但最终截断不要。
+ 2. 不能断，众所周知，marching cube面的生成，至少需要2层8个体数据，如果说，体数据的分块仅仅重合一层，那么负责人的讲，这一层会不会导致歧义（理论上不会），但这个marching cube不是我写的，我不了解其内部细节。而我发现，如果仅仅重合一层，还是会断裂（一半连上了，一半没连上）,所以我选择重合2层，达到了老师要求的视觉结果。 

## 最终结果隐患

由于我重合了2层,所以三角形的数目会更多,在交界处多了一倍，所以这不是一个流形，目前很多软件检测结果都是非流形，meshlab不支持非流形的水密性检测。所以老师是否要求非常硬的就是要一个水密性检测的结果，这样我会想办法去除重复三角形。