# xv6 Kernel Extensions
> A collection of projects that each extend a different subsystem of the xv6 teaching kernel: syscall boundary, CPU scheduler, physical memory, and filesystem.

## Overview
This repository collects xv6 kernel-extension projects from an Operating Systems
course into one place. Each project is a full xv6 tree with its own extension and
its own README.

Every project follows the same shape: pick one kernel subsystem, extend it, wire
the new behavior through the user/kernel syscall boundary, and keep the state it
adds coherent with the kernel's existing source of truth. 

## Projects

### [System call boundary](https://github.com/jaeunda/os-p1-syscall.git)
`syscall dispatch table` `argfd`/`argptr`/`copyout` `struct file` `ptable.lock`

Adds new syscalls through the full user/kernel path. 
- `lseek(fd, offset, whence)` repositions a file's read/write offset by reusing xv6's existing `struct file.off` plumbing. 
- `get_procinfo(pid, *)` returns a point-in-time snapshot of one process-table entry, copied into a kernel buffer under `ptable.lock` and released before `copyout`,
  so no spinlock is held across a page-faulting copy and there is no TOCTOU between snapshot and copy.

### [Stride scheduler](https://github.com/jaeunda/os-p2-scheduler.git)
`stride scheduling` `pass/stride accounting` `overflow rebase` `ptable.lock`

Replaces round-robin with proportional-share scheduling: 

Each process's CPU share is set by a ticket count requested from user space, where more tickets mean a smaller stride and more frequent selection.
The central problem is bounding an ever-growing `pass` counter without corrupting the relative ordering selection
depends on, handled by a periodic rebase with a lossy distance-cutting clamp.


### [Physical page-frame tracking](https://github.com/jaeunda/os-p3-pageframe.git)
`inverted page table` `IPT` `per-frame table` `software TLB` `kalloc`/`kfree` `two-phase boot gate`

Makes physical memory observable and reverse-mappable:

Adds a per-frame usage table updated inside `kalloc`/`kfree`, an inverted page table mapping a physical
frame to its `(pid, va, flags)` chains, and a 64-entry software TLB. The derived
state is kept consistent with the hardware page tables at every mapping boundary,
with a two-phase enable gate to survive boot ordering.


### [Filesystem snapshots](https://github.com/jaeunda/os-p4-snapshot.git)
`copy-on-write` `block-level CoW` `on-disk refcount` `bmap` `xv6 log` `atomic commit` `read-only inode`

Adds point-and-restore filesystem snapshots via block-level copy-on-write:

A snapshot shares data and indirect blocks with the live filesystem and a block
diverges only on the first write after it is shared, tracked by an on-disk
reference count. Read-only enforcement rejects writes to snapshot inodes at open
time, before a file descriptor is built.


## Getting Started
Each project builds and boots as stock xv6 from its own directory:

```sh
cd <project>/xv6-public
make qemu            # or: make qemu-nox
```

See each project's README for its demo programs and per-project setup.

