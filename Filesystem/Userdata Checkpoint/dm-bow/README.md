# DM-BOW

## Overview

```
DM-BOW(Backup On Write) is a device-mapper target driver for backup the overwrite data. 
In Android Q, it's used for impliment User Data Checkpoint(UDC) 
```

## DM-BOW Device Status

```
There is Three Status For DM-BOW
- TRIM (0)
- CHECKPOINT (1)
- COMMIT (1)
status can be changed by write value to /sys/block/dm-x/dm/bow/state
```

![dm-bow status](dm-bow.png)

### TRIM Status

```
When dm-bow device in trim status, dm-bow will collect trimmed block.
The device will fall into trim status once dm-bow device is created
```

#### trim operation
  ```
  For trim operation, dm-bow will collect all trimmed blocks, and add to bow-range set(the bow-range type will be TRIMMED).
  ```

#### write operation
  ```
  For write operation, dm-bow will check whether the block in trimmed status, 
  ```

#### other operation
  ```
  ```