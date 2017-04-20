# Use ZFS to store Minecraft world saves

## Background

- love playing minecraft
- would like to have periodic world backups (hourly, daily, weekly, monthly, yearly)
- have a world (roughly a month old), got 62 backups (full copy of the save), already at 3.6G
- as you can see, plain copies takes a lot space
- would like to save as much space as I can
- ZFS got quite a few features that can help save time

I want to find out which option gives me more free space

## Options

1. No copy. Use ZFS Snapshots and compression
2. Use copies. Use ZFS compression and dedup

## How to test which one is the best?

1. Create a ZFS pool
2. Create two datasets
  2.1 Dataset A: Turn on compression (No copy)
  2.2 Dataset B: Turn on compression and dedup (copies)
3. Copy the existing copies into dataset A and dataset B
  3.1 For dataset A, we can do a plain old `cp`
  3.2 For dataset B, we need some magic:
    - Copy the first backup
    - Take a snapshot
    - Copy the next backup to overwrite the previous backup
    - Repeat the last two steps
4. See how much space dataset A and B use


## Experiment

### Creating a pool and datasets

```sh
zpool create test /dev/ada1
zfs create test/dedup
zfs create test/snapshots
```

(Note: Some `zpool` and `zfs` commands need root previllage, e.g. `create`, `set`, etc.)

### Setting properties

```sh
zfs set dedup=on test/dedup
zfs set compression=on test/dedup

zfs set compression=on test/snapshots
```

### After copying to `/test/dedup`

After copying 3.7 GB data to `/test/dedup` (which has `dedup` turned on), we can see `/test/dedup` only used 2.7GB of data. At this point, I realized I also need to copy all the data to a dataset with `compression` turned on as a control. Thus we can see how much space can be saved by turning on `compression` and `dedup` individually.

```
$ zfs list
NAME             USED  AVAIL  REFER  MOUNTPOINT
test            2.70G  18.4G    23K  /test
test/dedup      2.70G  18.4G  2.70G  /test/dedup
test/snapshots    23K  18.4G    23K  /test/snapshots
```

```
$ zdb -S test
Simulated DDT histogram:

bucket              allocated                       referenced
______   ______________________________   ______________________________
refcnt   blocks   LSIZE   PSIZE   DSIZE   blocks   LSIZE   PSIZE   DSIZE
------   ------   -----   -----   -----   ------   -----   -----   -----
     1    6.52K    787M    625M    625M    6.52K    787M    625M    625M
     2      622   70.0M   49.5M   49.5M    1.38K    161M    113M    113M
     4      493   60.9M   43.2M   43.2M    2.74K    347M    246M    246M
     8      163   20.3M   14.2M   14.2M    1.46K    186M    130M    130M
    16       48   5.56M   3.77M   3.77M    1.12K    134M   90.2M   90.2M
    32      405   49.1M   33.7M   33.7M    18.2K   2.21G   1.51G   1.51G
 Total    8.21K    993M    770M    770M    31.4K   3.79G   2.69G   2.69G

dedup = 3.58, compress = 1.41, copies = 1.00, dedup * compress / copies = 5.04
```

Confused... In the theory, the pool right now only takes about `3.7/5.04 = ~750MB`

### Establishing the control dataset

1. Create another dataset `zfs create test/compressed`

2. Turn on its `compression` property `zfs set compression=on test/compressed`

3. Copy the data into it


#### Something I don't understand

I noticed when I was copying all the data to the control dataset (with only `compression` turned on), there was high CPU usage but this did not happen when I was copying all the data to the dataset with both `compression` and `dedup` turned on.
