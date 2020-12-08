
---
title: swc2obj-- 报告总结及展望
---


## 关于技术保证
- 无论是从视觉效果还是从mesh显示均看不出拼接结果。 
> 证明1: mesh 显示结果（可由第三方人工检查）
> ![picture 1](images/b88548851cf2a2abc1dc345a0d05d3def572e48a0a5ea95010b68b34ffc4ae14.png)  
> 证明2: 参见我主机的80端口



## 关于padding

+ 1. 需要生成等值面，而且是较为平滑的等值面，所以，必须首先对体数据进行卷积，那么这个过程中卷积n次，每个维度加2*n个padding,这个padding参与计算，但最终截断不要。
+ 2. 不能断，众所周知，marching cube面的生成，至少需要2层8个体数据，如果说，体数据的分块仅仅重合一层，那么负责人的讲，这一层会不会导致歧义（理论上不会），但这个marching cube不是我写的，我不了解其内部细节。而我发现，如果仅仅重合一层，还是会断裂（一半连上了，一半没连上）,所以我选择重合2层，达到了老师要求的视觉结果。 

## 最终结果隐患

1. 非流行非严格水密：由于我重合了2层,所以三角形的数目会更多,在交界处多了一倍，所以这不是一个流形，目前很多软件检测结果都是非流形，meshlab不支持非流形的水密性检测。所以老师是否要求非常硬的就是要一个水密性检测的结果，这样我会想办法去除重复三角形。
2. 缺少技术: 从胞体出发，很快就会收敛到0.37左右,如果强制不能增加半径，这样的话就可能导致后面obj无法支持医学用途，因为不再精准。

## 代码过程总结

### swc2vol:读swc文件转化为分块体数据
+ 主要做三件事。1. radius为0的swc根据前后两个不为0的radius进行赋值 2.从胞体开始向外增大的改为不变。 3. 分块体数据转换并保存。 

> 对应cmake中的SWC2obj对象，命令行调用方式如下所示：
```C++
范式：
SWC2Obj.exe -s the_path_to_swc_file -o the_path_to_save_vol_data -b block_size
示例:
C:/Users/lidan/Desktop/SWC2Obj/cmake-build-release/SWC2Obj.exe -s C:/Users/lidan/Desktop/brain/14193_30neurons/N001.swc -o C:/Users/lidan/Desktop/SWC2Obj/newResult/ -b 256
```


### marchingcube:读取多个分块体数据marching cube并拼接为一个obj

> 环境说明：因为何博说cuda marchingcube对分块marchingcube的不支持, 所以建议我基于paraview二次开发出python脚本，并使用paraview封装的python解释器进行执行。我从善如流。我将paraview封装的解释器直接重命名为python4,希望paraview可以帮助到大家。

> 对应python脚本为vol2obj, 命令行调用方式如下所示：
```C++
范式:
python4.obj volume2obj/main.py -i the_path_to_save_vol_data -o the_path_to_save_obj 
示例：
"D:/software/ParaView 5.8.1-Windows-Python3.7-msvc2015-64bit/bin/python4.exe" C:/Users/lidan/PycharmProjects/volume2obj/main.py -i C:/Users/lidan/Desktop/SWC2Obj/newResult/N001.swc -o C:/Users/lidan/Desktop/SWC2Obj/newResult/N001_use.obj 
```

### mergeOBJ:读取obj,去重，合并相同节点，实现space坐标映射等。

> 只合并了相同的节点，所以某一个三角形可能会出现两次。 
> 现实中，每个体素对应的并不是一个正方形，而是一个长方形，因此我们在该代码中使用space(0.32,0.32,2)进行坐标映射。
> 针对坐标映射，我们考虑几种情况:
> - 1. 直接在体数据就开始坐标映射，发现会导致体数据marching cube的点的精度问题，导致偏移。 
> - 2. 在最后除以space，然后加offset，和swc转成的线obj关系吻合最好（目前采用）
> - 3. 最后处理坐标映射（开放问题） 


对应cmake对象为mergeOBJ, 命令行调用方式如下所示：
```C++
范式：
mergeOBJ.exe -i the_path_to_save_obj -o the_path_to_save_obj_without_repeat_vertex
示例：
C:/Users/lidan/Desktop/SWC2Obj/cmake-build-release/ObjMerge/mergeOBJ.exe -i  C:/Users/lidan/Desktop/SWC2Obj/newResult/N001_use.obj -o C:/Users/lidan/Desktop/SWC2Obj/newResult/N001_new_use.obj
```

### swc2lobj:读取swc，生成线框obj等。

> 这部分我基本没写，完全把何博代码抄过来，添加cmake即可。 


对应cmake对象为swc2lobj, 命令行调用方式如下所示：
```C++
范式：
swc2Lobj.exe -s the_path_to_swc_file -o the_path_to_save_line_obj
示例：
C:\Users\lidan\Desktop\SWC2Obj\cmake-build-release\swc2lobj\swc2Lobj.exe -s C:/Users/lidan/Desktop/brain/14193_30neurons/N030.swc -o C:/Users/lidan/Desktop/brain/N30Line.obj
```


## 最终命令行拼接

```Shell
C:/Users/lidan/Desktop/SWC2Obj/cmake-build-release/SWC2Obj.exe -s C:/Users/lidan/Desktop/brain/14193_30neurons/N001.swc -o C:/Users/lidan/Desktop/SWC2Obj/newResult/ -b 256 &&"D:/software/ParaView 5.8.1-Windows-Python3.7-msvc2015-64bit/bin/python4.exe" C:/Users/lidan/PycharmProjects/volume2obj/main.py -i C:/Users/lidan/Desktop/SWC2Obj/newResult/N001.swc -o C:/Users/lidan/Desktop/SWC2Obj/newResult/N001_use.obj &&  C:/Users/lidan/Desktop/SWC2Obj/cmake-build-release/ObjMerge/mergeOBJ.exe -i  C:/Users/lidan/Desktop/SWC2Obj/newResult/N001_use.obj -o C:/Users/lidan/Desktop/SWC2Obj/newResult/N001_new_use.obj
```

## 平滑
采用tubuin(经过尝试为最好平滑)，第二个负参数设为-0.1,即可得到经验的最好的平滑值。

## 代码未来展望
- 将tubuin 平滑集成到我的代码中。
- 智能指针的转换尚未完成
- paraview解释器的docker环境还未配置好


## 最终结果
参见10.186.152.213/webgl/