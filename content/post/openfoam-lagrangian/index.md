---
title: OpenFOAM lagrangian求解器 **cell, tetFace and tetPt search failure** 错误
description: OpenFOAM lagrangian
date: 2024-07-20 00:00:00+0000
image: cover.png
categories:
  - Development
tags:
  - OpenFOAM
  - Lagrangian
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

## 问题描述
在使用OpenFOAM的lagrangian求解器（例如`sprayFoam`）时，并行计算完成后，`reconstructPar`出现如下错误：
```log
--> FOAM FATAL ERROR: 
cell, tetFace and tetPt search failure at position (-0.0378579 0.000214455 0.0202072)
for requested cell 6154

    From function void Foam::particle::initCellFacePt()
    in file /public1/home/sc50027/OpenFOAM/OpenFOAM-2.3.1/src/lagrangian/basic/lnInclude/particleI.H at line 758.

FOAM aborting

Aborted (core dumped)
```
## 问题原因
这个错误是由于在并行计算过程中，粒子的位置在某个分区的边界上，导致了粒子的位置无法被正确的定位到某个cell上。

可以查阅相关的[Post](https://www.cfd-online.com/Forums/openfoam-bugs/235178-cell-tetface-tetpt-search-failure-position.html)与[Bug Report](https://bugs.openfoam.org/view.php?id=1341)。

但是似乎并没有一个很好的解决方案，对于本文的问题，我们发现计算过程和结果并没有受到影响，所以我们想要的是`reconstructPar`来得到最终的计算结果。


## 解决方案
定位是那个分区`processor`出现了问题，如果该分区的粒子结果不重要，可以考虑直接舍弃掉出问题的`processor`中的粒子结果。

- 在`reconstruct.ini`中指定时间步和要舍弃的`processor` ID，例如：
  ```ini
  0.15 (22,35,122)
  0.21 (30)
  ```
- 修改代码（见下一节），使得`reconstructPar`在出现错误时，不会终止，而是舍弃指定的`processor`并继续执行，直到所有的`processor`都被处理完毕。

  1. 显示当前处理的`processor` ID，当程序出现错误时，可以知道是哪个`processor`出现了问题
  2. 根据第1步的报错，写入配置文件`reconstruct.ini`
  3. 读入用户指定的配置文件，获取各个时间步下需要舍弃的`processor` ID，并在处理时跳过这些`processor` ID

### 代码示例
需要修改 `src/parallel/reconstruct/reconstruct/reconstructLagrangianPositions.C` 文件：
```cpp
// * * * * * * * * * * * * * * * Global Functions  * * * * * * * * * * * * * //

void Foam::reconstructLagrangianPositions
(
    const polyMesh& mesh,
    const word& cloudName, 
    const PtrList<fvMesh>& meshes,
    const PtrList<labelIOList>& faceProcAddressing,
    const PtrList<labelIOList>& cellProcAddressing
)
{
    passiveParticleCloud lagrangianPositions
    (
        mesh,
        cloudName,
        IDLList<passiveParticle>()
    );

    DynamicList<scalar> times(0);
    DynamicList<List<scalar> > regions(0);

    word fName = "reconstruct.ini";
    IFstream file(fName);

    if (!file.good())
    {
        Info << "File " << fName << " not found." << endl;
        return;
    }
    else
    {
        scalar t;
        scalarList r;
        while (file >> t)
        {
            times.append(t);
            file >> r;
            regions.append(r);
        }
    }

    forAll(meshes, i)
    {
        Info << "Mesh I = " << i << endl;
        bool skip = false;
        forAll(times, j)
        {
            if (mesh.time().timeName() == name(times[j]))
            {
                forAll(regions[j], k)
                {
                    if (i == regions[j][k])
                    {
                        skip = true;
                        break;
                    }
                }
                break;
            }
        }
        if (!skip)
        {
            const labelList& cellMap = cellProcAddressing[i];

            // faceProcAddressing not required currently.
            // const labelList& faceMap = faceProcAddressing[i];

            Cloud<passiveParticle> lpi(meshes[i], cloudName, false);

            forAllConstIter(Cloud<passiveParticle>, lpi, iter)
            {
                const passiveParticle& ppi = iter();

                // // Inverting sign if necessary and subtracting 1 from
                // // faceProcAddressing
                // label mappedTetFace = mag(faceMap[ppi.tetFace()]) - 1;

                lagrangianPositions.append
                (
                    new passiveParticle
                    (
                        mesh,
                        ppi.position(),
                        cellMap[ppi.cell()],
                        false
                    )
                );
            }
        }
    }
    IOPosition<Cloud<passiveParticle> >(lagrangianPositions).write();

}
```
