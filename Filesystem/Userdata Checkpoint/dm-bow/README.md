# DM-BOW

## Overview

```
dm-bow(Backup On Write) 会利用block设备的剩余空间来备份被覆盖(更新)的数据，在
Android Q版本后，dm-bow被用于AB升级失败后userdata数据的回退
dm-bow 与dm-snapshot中COW(Copy On Write) 较为类似
```

## DM-BOW Device Status

```
There is Three Status For DM-BOW
- TRIM (0)
- CHECKPOINT (1)
- COMMIT (2)
可以通过向/sys/block/dm-x/dm/bow/state节点写数据以更改DM-BOW设备状态
```

![dm-bow status](dm-bow.png)

### TRIM Status

```
此时dm-bow会收集被trim的block, 进行合并后添加到bow_range tree中, 类型记录为TRIMMED, 用于后续的备份覆盖写数据。

dm-bow设备创建完成后，状态为trim(0)
dm-bow设备进行挂载后，应该立即执行FITRIM操作，以确保dm-bow拥有空间进行备份。并且设置dm-bow状态为CHECKPOINT.
```

#### TRIM
  ```
  当dm-bow设备处于TRIM状态时，会收集trim操作回收的block, 将这些block合并后插入到dm-range set中, 并标记为TRIMMED状态。
  ```

#### WRITE
  ```
  对于写操作，dm-bow会判断block是否为TRIMMED状态。将block从TRIMMED状态中移除，改为UNCHANGED状态
  ```

#### OTHER
  ```
  对于其他操作，dm-bow是一个passthrough设备
  ```

### CHECKPOINT
```
当dm-bow处于CHECKPOINT状态时，dm-bow会对覆盖写的数据进行备份操作，并且将更新记录添加到日志. CHECKPOINT具体流程将在后续章节详细介绍
```

#### WRITE

  - 对于覆盖更新，会为其创建备份
  - 对于请求设备第一个block, 会映射到sector0_current
  - 对于新增的block, 将bow-range set中该block类型修改为CHANGED即可

#### READ
  - 对于请求设备第一个block, 会映射到sector0_current

#### OTHER
  - passthrough 设备

### COMMIT
```
  当切换到COMMIT状态时，恢复sector0, 删除dm-bow状态信息. 此时dm-bow仅作为一个passthrough设备
```

## Bow-Range
- bow-range用于维护一段具有相同类型并且地址连续的block信息，bow-range具有如下类型
  - INVALID
  - TOP
  - SECTOR0
  - SECTOR0_CURRENT
  - CHANGED
  - UNCHANGED
  - TRIMMED
  - BACKUP
- bow-range均会被存放在bow-range set中进行统一管理
- bow range的数据结构如下

  ![Alt text](bow-range.png)
### TOP
- 描述整个设备
- 此时sector不在描述起始地址，而是整个设备大小
- dm-bow设备刚完成创建时，会在bow-range set中添加此种类型的bow-range

### SECTOR0
- 设备的第一个扇区，作为dm-bow的日志块
- 如下图sector0 会指向原设备的正在第一个sector数据的地址

![Sector 0](sector0.png)
### SECTOR0_CURRENT
- 存放原设备真正sector0的数据

### UNCHANGED
- 描述未改变的block
- dm-bow设备刚完成创建时，认为整个设备均为UNCHANGED状态，因此会在bow-range set中添加此种类型的bow-range描述整个设备

### CHANGED
- 当UNCHANGED区域数据发生改动时，会将该区域改为CHANGED状态
- 再次修改CHANGED BOW-RANGE，无需进行任何改动

### BACKUP
- 描述备份数据

### TRIMMED
- 已经被trim的block
- 对于修改trimmed状态的block，认为是新增数据

## 流程解析
### TRIM流程
### CHECKPOINT流程
### COMMIT流程
