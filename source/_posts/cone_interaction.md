# swc数据结构和boxpen碰撞检测

## 问题简述 
swc数据就是一个节点一个球，两个节点之间是一个圆台，圆台的两个面就是
两个节点所对应半径的半球。而体数据就是一个box。要判定二者是否相交。

## 问题规约
问题可以规约为一个圆台和两个半球，同一个box之间的求交问题。进一步的，问题可以规约为2个圆锥和2个球同一个box之间的求交问题。之所以这样规约是因为最基础的判定相交的代价很低。 

## 问题核心（圆锥和box相交）
首先我想到了vtk，但是事实上vtk无法做到这一点。 
>http://vtk.1045678.n5.nabble.com/collision-detection-with-vtk-td5722845.html#a5722985

This is my Code 
'''C++
#include <vtkActor.h>
#include <vtkIntersectionPolyDataFilter.h>
#include <vtkPolyDataMapper.h>
#include <vtkProperty.h>
#include <vtkRenderer.h>
#include <vtkRenderWindow.h>
#include <vtkRenderWindowInteractor.h>
#include <vtkSmartPointer.h>
#include <vtkSphereSource.h>
#include <vtkConeSource.h>
#include <vtkCubeSource.h>

int main(int, char *[])
{
    vtkSmartPointer<vtkConeSource> coneSource1 = vtkSmartPointer<vtkConeSource>::New() ;
//    coneSource1->SetRadius(5) ;
    coneSource1->SetCenter(0.5,0.5,0.5) ;
//    coneSource1->SetDirection(1,2,3);
//    coneSource1->SetHeight(10) ;
//    coneSource1->SetResolution(1000) ;

    vtkSmartPointer<vtkPolyDataMapper> coneMapper = vtkSmartPointer<vtkPolyDataMapper>::New();
    coneMapper->SetInputConnection( coneSource1->GetOutputPort() );
    coneMapper->ScalarVisibilityOff();

    vtkSmartPointer<vtkActor> coneActor = vtkSmartPointer<vtkActor>::New();
    coneActor->SetMapper( coneMapper );
    coneActor->GetProperty()->SetColor(1,0,0);


    vtkSmartPointer<vtkCubeSource> cubeSource1 = vtkSmartPointer<vtkCubeSource>::New() ;
//    cubeSource1->SetCenter(5,5,5) ;
//    cubeSource1->SetBounds(0,10,0,10,0,10) ;


    vtkSmartPointer<vtkPolyDataMapper> cube = vtkSmartPointer<vtkPolyDataMapper>::New();
    cube->SetInputConnection( cubeSource1->GetOutputPort() );
    cube->ScalarVisibilityOff();

    vtkSmartPointer<vtkActor> cubeActor = vtkSmartPointer<vtkActor>::New();
    cubeActor->SetMapper( cube );
    cubeActor->GetProperty()->SetColor(0,1,0);








//    vtkSmartPointer<vtkSphereSource> sphereSource1 = vtkSmartPointer<vtkSphereSource>::New();
//    sphereSource1->SetCenter(0.0, 0.0, 0.0);
//    sphereSource1->SetRadius(2.0f);
//    sphereSource1->Update();


    vtkSmartPointer<vtkIntersectionPolyDataFilter> intersectionPolyDataFilter =
            vtkSmartPointer<vtkIntersectionPolyDataFilter>::New();
    intersectionPolyDataFilter->SetInputConnection( 0, coneSource1->GetOutputPort() );
    intersectionPolyDataFilter->SetInputConnection( 1, cubeSource1->GetOutputPort() );
    intersectionPolyDataFilter->Update();

    vtkSmartPointer<vtkPolyDataMapper> intersectionMapper = vtkSmartPointer<vtkPolyDataMapper>::New();
    intersectionMapper->SetInputConnection( intersectionPolyDataFilter->GetOutputPort() );
    intersectionMapper->ScalarVisibilityOff();


    vtkSmartPointer<vtkActor> intersectionActor = vtkSmartPointer<vtkActor>::New();
    intersectionActor->SetMapper( intersectionMapper );
    intersectionActor->GetProperty()->SetColor(0,0,1);

    vtkSmartPointer<vtkRenderer> renderer = vtkSmartPointer<vtkRenderer>::New();
    renderer->AddViewProp(cubeActor);
    renderer->AddViewProp(coneActor);
    renderer->AddViewProp(intersectionActor);

    vtkSmartPointer<vtkRenderWindow> renderWindow = vtkSmartPointer<vtkRenderWindow>::New();
    renderWindow->AddRenderer( renderer );

    vtkSmartPointer<vtkRenderWindowInteractor> renWinInteractor =
            vtkSmartPointer<vtkRenderWindowInteractor>::New();
    renWinInteractor->SetRenderWindow( renderWindow );

    renderWindow->Render();
    renWinInteractor->Start();

    return EXIT_SUCCESS;
}
'''

这段代码会出现以下问题：
> vtkPointLocator (0x7f917de07a60): No points to subdivide
> No Intersection between objects
即使物体之间相交，但是还是没有办法判定。听说vtkCollisionDetectionFilter可以，但是我看了它的代码是基于点作的碰撞检测，这代价太多，而且不是严格的，很可能导致意外。

## 学长之前的思路
学长之前的思路类似于metaball，而且不支持分块，所以没办法作长方体和圆台的快速相交判定。学长的思路是每隔一个step，找一个球心，根据这个球生成体数据，step取得足够小，那么体数据就会足够接近圆台，只需要mesh平滑就可以了。

## 我的思路
1.如果这个任务是一个时间短的任务，那么我们直接也是step，每一个小球和box求交，记录下需要marchingcube的体数据的块。
2.如果这个任务是一个时间长的任务，我已经写了部分代码，是圆台和box求交的算法。

我尝试了思路2，但是计算的鲁棒性不能得到保证，因此，我暂时放弃。如果思路1的效率不能满足，那么我将会尝试思路2.

## 算法描述

1. 整体的体数据加一个padding。 
2. 然后基于这个padding之后的体数据进行划分，stride为126，size为128，这样每个块的左右都重合了2个pad，建立所有的box
3. 然后基于box建立BVH。
4. 根据step圆球，去和BVH求交，找到所需要的box，转化为box坐标，更新体数据
5. 然后marchingcube，更改marching cube算法。


