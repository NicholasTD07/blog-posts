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
