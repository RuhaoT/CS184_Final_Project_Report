# Dataset

Date: 2024/04/28
Author: Ruhao Tian

## Dataset Overview

数据集生成脚本位于`/src`目录下。
```
📦Dataset
 ┣ 📜Dataset.cpp
 ┗ 📜Dataset.h
```

主要功能是记录SeedPath和BlockPath的生成过程，并把最终结果按照json格式保存。

## 编译

所有的代码都定义在`DATASET_ENABLE`宏下，cmake使用`-DENABLE_DATASET=ON`来开启。
根目录下的`compiler_dataset.sh`提供了一个示范。

## 使用

每次一个SeedPath生成完毕后，如果SeedPath合法，就会被**添加到**执行程序同目录下的`seedpath.json`中，**如果这个文件已经存在，不会覆写其中的数据**。如果BlockPath合法，就会被**添加到**`blockpath.json`中。

一个典型的用法是运行完程序后更改文件名。`/bin/dataset.sh`提供了一个示范。

```shell
# Test Dataset
meshName=Cube_4x4x4_E1
pathToMesh=../volume/tbl_2_cubes_compare/$meshName.vol
pieceNumber=4
difficulty=4
seed=50

../bin/High-LevelPuzzle_perf $pathToMesh $pieceNumber $difficulty $seed

# rename the output file
# rename seedpath.json to seedpath_$meshName.json
mv seedpath.json seedpath_$meshName.json
# rename blockpath.json to blockpath_$meshName.json
mv blockpath.json blockpath_$meshName.json
```

## 原理

以SeedPath为例，在`/src/Puzzle/PieceCreator.cpp`，当`SeedPath`结构体被创建时，对应的`seedPathCreationSequence`被创建，每当`SeedPath`的状态发生改变时，更改后的`SeedPath`会被添加到`seedPathCreationSequence`中。最终，`seedPathCreationSequence`会被保存到json文件中。

`PieceCreator::CreateSeedPath`提供了一个示范。除了`SeedPath`以外的其他中间变量均不会被记录。