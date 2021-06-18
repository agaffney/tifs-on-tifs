# tifs-on-tifs

Run TiFS (on TiKV) on TiFS

## Why would you do this?

As with most of my random projects, this started as a combination of boredom and "because I could".
This serves almost no practical purpose, but it does show off TiFS with a real world use case.

## What is TiFS?

TiFS is a POSIX filesystem driver implemented on top of TiKV.

https://pingcap.com/blog/tifs-a-tikv-based-partition-tolerant-strictly-consistent-file-system

## How do I use it?

Unmount and clean up previous mount dirs.

```bash
$ sudo umount -f /mnt/tifs0 /mnt/tifs1
$ sudo rm -rf /mnt/tifs0 /mnt/tifs1
```

Start the first tier PD and TiKV.

```bash
$ docker-compose up -d pd0 tikv0
```

Mount the first tier TiFS.

```bash
$ docker-compose up -d tifs0
```

Make sure the mount succeeded.

```bash
$ mount | grep mnt/tifs0
```

Start the second tier PD and TiKV.

```bash
$ docker-compose up -d pd1 tikv1
```

Mount the second tier TiFS.

```bash
$ docker-compose up -d tifs1
```

Make sure the mount succeeded.

```bash
$ mount | grep mnt/tifs1
```

### Cleaning up

Stop the containers.

```bash
$ docker-compose down
```

Unmount the filesystems.

```bash
$ sudo umount -f /mnt/tifs0 /mnt/tifs1
```

Remove the mount dirs.

```bash
$ sudo rm -rf /mnt/tifs0 /mnt/tifs1
```

## How well does it perform?

This setup has terrible performance, but that's not really the point.

First tier TiFS:

```bash
$ time dd if=/dev/zero of=/mnt/tifs0/big.file bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 39.9818 s, 2.6 MB/s

real    0m40.077s
user    0m0.000s
sys     0m0.082s
```

Second tier TiFS:

```bash
$ time dd if=/dev/zero of=/mnt/tifs1/big.file bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 296.319 s, 354 kB/s

real    4m56.658s
user    0m0.005s
sys     0m0.072s
```
