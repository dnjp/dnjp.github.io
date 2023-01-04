---
title: Plan 9
tags:
categories:
date: 2023-01-03
lastMod: 2023-01-03
---
The goal of Plan 9 was to build a system that was centrally administered and cost-effective using cheap modern microcomputers as its computing elements. The idea was to build a time-sharing system using different computers that handle different tasks. Small machines in your office would serve as terminals to large, central, shared resources such as computing servers and file servers.

Plan 9 is highly influenced by the [Cambridge Distributed System](./cambridge-distributed-system.pdf). The early catch phrase was to build a UNIX out of a lot of little systems, not a system out of a lot of little UNIXes.

Plan 9 adopted the idea of using the file system to manage resources but instead uses a network-level protocol (9P) to enable machines to access files on remote systems.

Uses naming system that allows for customized views of the resources in the network

Allows the user to build a private computing environment and recreates it wherever desired instead of on their private machine

Per-process name spaces and file-system like resources are used extensively throughout the system

There is no 'tty driver' in the kernel - that job is given to the window system

Design

  + The view of the system is built on three principles

    + Resources are named and accessed like files in a hierarchical file system

    + There is a standard protocol (9P) for accessing these resources

    + The disjoint hierarchies provided by different services are joined together into a single private hierarchical file name space

  + This provides the following benefits:

    + The terminal is temporarily personalized by that user

    + Instead of customizing hardware, you can customize one's view of the system provided by the software

    + Personalization is accomplished by giving local, personal names for the publicly visible resources in the network

  + For example `/dev/cons` always refers to the user's terminal and `/bin/date` the correct version of the date command to run, but which files those names represent depends on circumstances such as the architecture of the machine executing `date`.

  + 9P is structures as a set of transactions that send a request from a client to a server and return the result. You can control file systems as well as files with 9P. File access is at the level of bytes, not blocks, which distinguishes 9P from NFS or RFS. [A Comparison of Three Distributed File System Architectures: Vnode, Sprite, and Plan 9](spr_welch.pdf) provides a deeper look into these differences.

Command-level View

  + Each window created by a user is run a separate name space. Changes in one name space do not affect other windows or programs. Each window has a private bitmap and mulpiplexed access to the keyboard, mouse, and other resources through `/dev/mouse`, `/dev/bitblt`, and `/dev/cons` (analogous to `/dev/tty` in unix). Unlike in X, a remote rio application sees the mouse, bitblt, and cons files for windows as usual in `/dev`. It does not know whether the files are local - it just reads/writes to control

The File Server

  + A central file server stores permanent fies and presents them to the network as a file hierarchy exported using 9P. The server is a stand-alone system, accessible over the network, and designed to do its one job well. No user processes are run, only a fixed set of routines compiled into the boot image. The main hierarchy exported by the server is a single tree, representing files on many disks. That hieararchy is shared by users over the network.

  + WORM - write-once-read-many

  + The filesystem disk is a cache for the WORM and memory is a cache for the disk. Every morning at 5am, a dump of the file system occurs automatically. The file system is frozen and all blocks modified since the last dump are queued to be written to the WORM. Once the blocks are queued, service is restored and the read-only root of the dumped file system appears in a hierarchy of all dumps ever taken, named by its date. i.e. the directory `/n/dump/1995/0315` is the root directory of an image of the file system as it appeared on 3/15/1995. This means that restoring a file is as simple as copying over the version from the dump you're interested in. Backup problems can then be solved with standard tools like cp, ls, grep, and diff. The dump file system is read-only the permissions cannot be changed. Once a file is written to WORM, it cannot be removed.

Unusual File Servers

  + Rio provides two interfaces. To the user, they are presented a familiar style of interaction with the keyboard and mouse. To client programs, they are presented a set of files in /`dev` to read/write to.

  + Write to `/dev/cons` to print text to the window

  + To read the mouse, they read `/dev/mouse`

  + Bitmap graphics are implemented by reading/writing encoded messages to `/dev/bitblt`

  + Rio is a file server, serving the files in `/dev` to clients. Each client is given a private nam space with a different set of files in `/dev` to work on. Local name spaces make this possible.

  + Because Rio is implemented as a file server, it can postpone answering read requests for a particular window which is toggled using a reserved key on the keyboard (escape). In 9term, press Esc to turn off reads. This enables you to write as many lines as desired without them being evaluated. When finished, press Esc again and the commands will be evaluated line by line.

  + There is no `ftp` command in Plan 9 - instead a file server called `ftpfs` dials the ftp site, logs in, and uses the ftp protocol to examine files in the remote directory. It essentially translates the FTP protocol into 9P to offer Plan 9 access to ftp sites.

  + `exportfs` is a user process that takes a portion of its own name space and makes it available to other processes by translating 9P requests into system calls to the Plan 9 kernel. It is usually run as a remote server started by a local program, either `import` or `cpu`. `Import` calls the remote machine, starts `exportfs`, and attaches its 9P connection to the local name space.

Configurability and administration

  + Central file servers centralize not just the files, but also their administration and maintenance. One server is the main server, holding all system files, other servers provide extra storage or are available for debugging and other special uses, but the system software resides on one machine. This means that each program has a single copy of the binary for each architecture, so it is trivial to install updates and bug fixes. There is also a ringle user database; there is no need to syncrhonize distinct `/etc/passwd` files.

  + On the central server there is a directory `/lib/ndb` that contains all the information necessary to administer networks. All the machines use the same database to talk to the network. To install a new machine on the local Ethernet, choose a name and IP addres and add these to a single file in `/lib/ndb`. All the machines in the installation will be able to talk to it immediately. To start running, plug the machine into the network, turn it on, and use BOOTP and TFTP to load the kernel.

C Programming
  + Programs in Plan 9 are generally written in rc, alef, or a dialect of ANSI [C]({{< ref "/pages/C" >}}). The Plan 9 [C]({{< ref "/pages/C" >}}) dialect has some minor extensions and a few major restrictions. The most important restriction is that the compiler demands that all function definitions have ANSI prototypes and all function calls appear in the scope of a prototyped declaration of the function. Each system library has an associated header file, declaring all functions in that library. The standard Plan 9 library is called `libc`, so all [C]({{< ref "/pages/C" >}}) source files include `<libc.h>`. These rules guarantee that all functions are called with arguments having the expected types.

  + Another restriction is that the C compilers accept only a subset of the preprocessor directives. The main omission is `#if`. Conditional compiliation, even with `#ifdef`, is used sparingly in Plan 9 - only present in low-level routines in the graphics library.

  + Instead of UNIX's `creat`, Plan 9 has a [create](http://man2.aiju.de/2/open) function that takes three arguments:

    + 
```c
int create(char *file, int omode, ulong perm)
```

    + `perm` defines whether the returned file descriptor is to be opened for reading, writing, or both.

  + Plan 9 uses a 16-bit character called Unicode. To simplify the exchange of text between programs, the characters are packed into a byte stream by the UTF-8 encoding.

  + APE (ANSI C/POSIX Environment) comprises separate include files, libraries, and commands, conforming as much as possible to the strict ANSI C and base-level POSIX specifications.
