# RAIDframe Project Developments in NetBSD

The Redundant Array of Independent Disks (RAID) framework is a disk management
framework developed by Carnegie-Mellon University. NetBSD uses RAIDframe as one
of its disks management modules.
It involves setting up multiple disks and creating a disk unit from them.
The current NetBSD RAIDframe framework supports several levels of disks arrangement
in a single array, see [raid(4)](https://man.NetBSD.org/raid.4).

## Abstract

In the current NetBSD's RAIDframe module, RAID levels 0, 1, 5 and 6 are included in source. There are some limitations that this project aims to improve. Firstly, RAID level 1, which is also called mirroring, allows for only two disks in a single mirror pair. Secondly, RAIDframe scrubbing, which involves reading your disks to check for read failures, is not yet supported. Thirdly, RAID level 6, even though included in source, is not well tested and not encouraged to be used. This project aims to implemment an extension of RAID level 1 called `N-way RAID1` to support multiple disks in a RAIDframe mirror, implement RAID scrubbing, and then test RAID level 6 and fix bugs found in them.

## N-way RAID 1

RAID level 1 involves mirroring two disks containing the same data.
They are structured as one primary and one parity (secondary).
Every write to the raid device writes to all disks in the setup that are alive.
Every read from the raid device reads from the disk with the shortest I/O queue.
If there's an encountered failure with any of the disks,
it reads in degraded mode and hence gets the data from any of the available disks.
If all disks fail, I/O (writes/read to and from the disks) aborts.

There is an introduction of a new extension to the RAID 1 setup called n-way RAID1.
This involves setting up more than two disk in a RAID 1 array setup where you have
one primary disk and multiple secondary disks.
This increases redundancy and improves the security of data critical to disk failure that could lead to data loss.

For example, in a five way RAID1 setup, it will involve one primary and 4 parity/secondary disks.
So every disk write will attempt to write to all five disks.
Every disk read will attempt to read from the primary disk or the secondary disk with the shortest I/O queue.

### Usage

Five disks can be configured in a 5 way RAID 1 setup for redundancy
using [raidctl(8)](https://man.NetBSD.org/raidctl.8) with the command below:

```sh
raidctl /dev/raid1 create N /dev/dk1 /dev/dk2 /dev/dk3 /dev/dk4 /dev/dk5.
```

where `/dev/raid1` is the device file for the raid device, and `N` is the level.
In the order of the disks, the first listed is considered the primary
and the rest are considered secondary.

The `/dev/dk*` are the NetBSD disk partition (wedge) driver used for the independent disks, see [dk(4)](https://man.NetBSD.org/dk.4) and [dkctl(8)](https://man.NetBSD.org/dkctl.8).

This, by default, sets up a 128 sectors per stripe unit and a first in first out queuing algorithm and a max queue length of 100.

This can be similarly translated into the raid.conf structure in the setup below.

```
 numrow numcol numspare
 1 5 0

 Identify physical disks
 START disks
/dev/dk1
/dev/dk2
/dev/dk3
/dev/dk4
/dev/dk5

 Layout is simple - 64 sectors per stripe
 START layout
 Sect/StripeUnit StripeUnit/ParityUnit StripeUnit/ReconUnit RaidLevel
 128 1 1 N

 No spares
 START spare

 START queue
 fifo 100

```

### Project deliverables

#### RAIDframe Layout

A new layout structure is introduced for RAIDframe level `N`. number of primary disk remains 1.
Number of parity/secondary becomes number of disks - 1. The rest of the layout component
for RAID 1 (stripe related properties) remains same hence adopted into RAID `N`.

#### Sector/stripe mapping

The current design for RAID 1 involves ASM (Address Stripe Mapping) structures that contain PDAs (Physical Disk Addresses) that are used in mapping the RAID level software addresses to the Physical Disk Addresses.
The PDA structure contain column number, start sector, number of sectors/blocks, type of disk in setup (data/parity disk), data buffer pointer, and then the virtual RAID address corresponding to the Physical Disk Address.
For a simple RAID 1 mirror involving two disks, the writes or reads are striped across the two disks
according to the value set in `SectorsPerStripeUnit` in `raid.conf`, or 128 by default when using [raidctl(8)](https://man.NetBSD.org/raidctl.8).
So 128 sector blocks are written to each stripe are defined by the PDAs.

For two disk in a RAID 1 setup, a single stripe write are defined by two PDAs for each column.
For an introduction of n-way RAID 1, the number of PDAs cannot be known at compile time.
The number of PDAs are dynamically defined by the number parity columns at runtime.

#### DAG execution

RAIDframe uses DAGs (Directed Acyclic Graph) to fire I/O nodes for reads and writes. These DAG nodes are also PDA dependent.
The DAG node creation structure also needed to be updated to accommodate more than two
PDAs when using the level `N`.

#### Reconstruction

RAIDframe reconstruction has been updated to make room for RAID level `N`. When a disk fails,
the current algorithm identifies a non-dead disk and reads the content of that disk
and writes to the spare disk. New checks for RAID N has been added to the code to read from
only one non-dead disk and write to the spare disk. This avoids trying to randomly read and write across
the disk array during a reconstruction.

#### Project benefit

This project adds more redundancy to your disk
data management and reducing the risk of data loss in any case of disk failure.

[Link to work](https://github.com/Emmankoko/altq_refactoring_gsoc/commit/4550afba69fe38ca9407f76d5a7289e3c43d69c2) 



## RAIDframe scrubbing

The scrubbing implementation is a disk sector health check of all components in a disk array.
Disks sectors are read across every stripe in the components and the I/O returns number of
read failures encountered on each component. Disk scrubbing is supported for all
RAID levels in NetBSD.

Starting a scrub on a raid device is done by using `raidctl`. Scrubbing can be done across
certain portion of the disks or the entire disks in the array.

### Usage

RAID scrubbing is achieved by the syntax below:

```sh
raidctl $device scrub percentage $start_percentage $end_percentage
```

Consider a hundred-striped three disks raid 5 array:

```sh
raidctl raid5 scrub percentage 0 10
```

start_stripe = 100 * 0 / 100 = 0
end_stripe = 100 * 10 / 100 = 10 - 1 = 9

This reads the disks from stripe index 0 to stripe index 9 (first ten stripes).

### Results/kernel output after a successful scrub

```
raid5: Total number of read failures on Component /dev/dk1: 10
raid5: Total number of read failures on Component /dev/dk2: 4
raid5: Total number of read failures on Component /dev/dk3: 0
```

### Interpretation

This indicates 10 read failures across `dk1`, 4 read failures across `dk2` and 0 read failures
across `dk3`.

Omitting the percentage parameters defaults to 100 percent scrub action:

```sh
raidctl raid5 scrub
```

**Note**: `end_stripe` is reduced by 1 because indexing of stripes begins from 0.

[Link to work](https://github.com/Emmankoko/altq_refactoring_gsoc/commit/36a34a5b416bb10cd5ea84dd24057b7c8a02681f)


## Future works

RAID 6 is currently being tested and improved too.
