# Use ZFS to store Minecraft world saves

## Background

- love playing minecraft
- would like to have periodic world backups (hourly, daily, weekly, monthly, yearly)
- have a world (roughly a month old), got 62 backups (full copy of the world save), already at 3.6G
- as you can see, plain copies takes a lot space
- Guess/Assumption: A lot files are the same among different backups.
- would like to save as much space as I can
- ZFS got a few features that can help save space

I want to find out which option gives me more free space

## Options

0. Copy the game save directory each time when backing up with `cp`. Use ZFS's compression and **dedup** features
0. Don't use copy `cp`. Use ZFS's compression and **snapshots** features

## Intro to ZFS's compression, **dedup** and **snapshots**

## Advantages / Disadvantages

- Dedup needs to store a dedup/checksum table in memory. Large the data, larger the memory it needs. (320 bytes per block)
- Dedup: No extra commands needed other than plain old `cp`
- Snapshot: No memory requirement

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
```

(Note: Some `zpool` and `zfs` commands need root previllage, e.g. `create`, `set`, etc.)

### Setting properties

```sh
zfs set dedup=on test/dedup
zfs set compression=on test/dedup
```

### After copying to `/test/dedup`

After copying 3.7 GB data to `/test/dedup` (which has `dedup` turned on), we can see `/test/dedup` only used 2.7GB of data. At this point, I realized I also need to copy all the data to a dataset with `compression` turned on as a control. Thus we can see how much space can be saved by turning on `compression` and `dedup` individually.

```
$ zfs list
NAME             USED  AVAIL  REFER  MOUNTPOINT
test            2.70G  18.4G    23K  /test
test/dedup      2.70G  18.4G  2.70G  /test/dedup
```

But looking at the output of `zpool list`, `dedup` is 3.58x while 3.7/2.7 is definitely not `3.58`... Hmmmm...

```
$ zpool list
NAME   SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
test  19.9G   780M  19.1G         -     3%     3%  3.58x  ONLINE  -
```

Compression rate is at 1.4x.

```
$ zfs list -o name,compressratio
NAME             RATIO
test             1.40x
test/dedup       1.40x
```

The following command "Simulate the effects of deduplication".

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

If you look at the `dedup * compress / copies = 5.04` at the bottom right corner, it means that, in the theory, the pool right now only takes about `3.7/5.04 = ~750MB`. I am so confused... Why would `zfs list` says `test/dedup` is using 2.7G while `zpool list` tells us the pool is only using 780M?

**AH HA!**

If I looked closer, I would have found out that the header of the amount of data listed in the output of `zfs list` says *"Refer"* while the header in `zpool list` says *"Alloc"*...

> ALLOC

> The amount of physical space allocated to all datasets and internal metadata. Note that this amount differs from the amount of disk space as reported at the file system level.

[Querying ZFS Storage Pool Status - Oracle](http://docs.oracle.com/cd/E19253-01/819-5461/6n7ht6r01/index.html)

Obviously `zpool` and `zfs` have different understanding of how much space they use. `zpool` reports physical space allocated while `zfs`

### Destorying and recreating zpool

Since we only get actual space allocated from `zpool`, to make it easy to compare, we need to destroy and recreate the zpool.

```sh
zpool destroy test
zpool create test /dev/ada1
```

```sh
$ zpool list
NAME   SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
test  19.9G    86K  19.9G         -     0%     0%  1.00x  ONLINE  -
```

### Creating and setting up `snapshot` dataset

```sh
zfs create test/snapshot
zfs set compression=on test/snapshots
```

### Copying and snapshotting

I created a little Python snippet to help me with this. Basically, it works like this

1. Get a list of absolute paths of all the world saves (a save is a directory containing some files) in my backup directory
2. Get the modified time from a file within the saves (since these directories themselves were moved around)
3. Combining outputs of step 1 and 2 and form a list of tuple like this `(path, time)`
4. Sort this list so the first tuple contains the path to the earliest save
5. Iterate through the list of tuples and do a `cp` and `zfs snapshot` each time

See the full snippet here: [script](https://github.com/NicholasTD07/demos/blob/master/vagrant/freebsd-12.0/scripts/t.py)

### Stats when using snapshots to store all the minecraft saves

As the following output shows, `test/snapshots` dataset only refers to 52M data in the file system. Because when the data is copied, it overwrites the previous one.

```
$ zfs list
NAME             USED  AVAIL  REFER  MOUNTPOINT
test            2.70G  16.6G    23K  /test
test/snapshots  2.70G  16.6G  52.0M  /test/snapshots
```

Double check with `du`. `du` also reports only 52M is used.

```
$ du -sh /test/snapshots/
 52M	/test/snapshots/
```

However, as shown in the following output, the dataset (the only dataset on this pool) is using 2.7G physical space to store all the data. It's quite a lot, compared to when all the data was saved on the `dedup` dataset when both `dedup` and `compression` was turned on (780M).

 ```
$ zpool list
NAME   SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
test  19.9G  2.70G  17.2G         -     0%    13%  1.00x  ONLINE  -
 ```

### Establishing the control dataset

After destorying and recreating zpool,

1. Create another dataset `zfs create test/compressed`

2. Turn on its `compression` property `zfs set compression=on test/compressed`

3. Copy the data into it

```sh
$ zpool list
NAME   SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
test  19.9G  2.70G  17.2G         -     0%    13%  1.00x  ONLINE  -
```

##### Something I don't understand

I noticed when I was copying all the data to the control dataset (with only `compression` turned on), there was high CPU usage but this did not happen when I was copying all the data to the dataset with both `compression` and `dedup` turned on.
