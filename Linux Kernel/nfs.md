# nfs文件系统



## nfs简介

### 什么是nfs

网络文件系统，英文Network File System(NFS)，是由[SUN](https://baike.baidu.com/item/SUN/69463)公司研制的[UNIX](https://baike.baidu.com/item/UNIX/219943)[表示层](https://baike.baidu.com/item/表示层/4329716)协议(presentation layer protocol)，能使使用者访问网络上别处的文件就像在使用自己的计算机一样。		-百度百科

NFS是基于[UDP](https://baike.baidu.com/item/UDP/571511)/IP协议的应用，其实现主要是采用[远程过程调用](https://baike.baidu.com/item/远程过程调用)[RPC](https://baike.baidu.com/item/RPC/609861)机制，RPC提供了一组与机器、操作系统以及低层传送协议无关的存取远程文件的操作。

NFS 的第一个版本是 SUN Microsystems 在 20 世纪 80 年代开发出来的，至今为止，NFS 经历了 NFS，NFSv2，NFSv3 和 NFSv4 共四个版本。现在，NFS 最新的版本是 4.1，也被称为 pNFS（parallel NFS，并行网络文件系统）。

前四个版本的 NFS，作为一个文件系统，它几乎具备了一个传统桌面文件系统最基本的结构特征和访问特征，不同之处在于它的数据存储于远端服务器上，而不是本地设备上，因此不存在磁盘布局的处理。NFS 需要将本地操作转换为网络操作，并在远端服务器上实现，最后返回操作的结果。因此，NFS 更像是远端服务器文件系统在本地的一个文件系统代理，用户或者应用程序通过访问文件系统代理来访问真实的文件系统。



### nfs的特点

1. 提供透明文件访问以及文件传输；
2. 容易扩充新的资源或软件，不需要改变现有的工作环境；
3. 高性能，可灵活配置。

### 工作原理

NFS的工作原理是使用客户端/服务器架构，由一个客户端程序和服务器程序组成。服务器程序向其他计算机提供对文件系统的访问，其过程称为输出。NFS客户端程序对共享文件系统进行访问时，把它们从NFS服务器中“输送”出来。文件通常以块为单位进行传输。其大小是8KB（虽然它可能会将操作分成更小尺寸的分片）。NFS传输协议用于服务器和客户机之间文件访问和共享的通信，从而使客户机远程地访问保存在存储设备上的数据。



### 网络文件系统架构

NFS 允许计算的客户 — 服务器模型。服务器实施共享文件系统，以及客户端所连接的存储。客户端实施[用户接口](https://baike.baidu.com/item/用户接口)来共享文件系统，并加载到本地文件空间当中。 [3] 

在 Linux中，[虚拟文件系统](https://baike.baidu.com/item/虚拟文件系统/10986803)交换（[VFS](https://baike.baidu.com/item/VFS/7519887)）提供在一个主机上支持多个并发文件系统的方法（比如 [CD-ROM](https://baike.baidu.com/item/CD-ROM/513612) 上的 International Organization for Standardization [ISO] 9660，以及本地硬盘上的 ext3fs）。VFS 确定需求倾向于哪个存储，然后使用哪些文件系统来满足需求。由于这一原因，NFS 是与其他文件系统类似的可插拔文件系统。对于 NFS 来说，唯一的区别是输入/输出（I/O）需求无法在本地满足，而是需要跨越网络来完成。 [3] 

一旦发现了为 NFS 指定的需求，VFS 会将其传递给[内核](https://baike.baidu.com/item/内核)中的 NFS 实例。NFS 解释 I/O 请求并将其翻译为 NFS 程序（OPEN、ACCESS、CREATE、READ、CLOSE、REMOVE 等等）。这些程序，归档在特定 NFS RFC 中，指定了 NFS 协议中的行为。一旦从 I/O 请求中选择了程序，它会在远程程序调用（[RPC](https://baike.baidu.com/item/RPC/609861)）层中执行。正如其名称所暗示的，RPC 提供了在系统间执行程序调用的方法。它将封送 NFS 请求，并伴有参数，管理将它们发送到合适的远程对等级，然后管理并追踪响应，提供给合适的请求者。



### 网络文件系统协议

从客户端的角度来说，NFS 中的第一个操作称为 mount。Mount 代表将远程文件系统加载到本地文件系统空间中。该流程以对 mount(Linux [系统调用](https://baike.baidu.com/item/系统调用/861110)）的调用开始，它通过 VFS 路由到 NFS 组件。确认了加载[端口号](https://baike.baidu.com/item/端口号/10883658)之后（通过 get_port 请求对远程服务器 RPC 调用），客户端执行 RPC mount 请求。这一请求发生在客户端和负责 mount 协议（rpc.mountd）的特定守护进程之间。这一守护进程基于服务器当前导出文件系统来检查客户端请求；如果所请求的文件系统存在，并且客户端已经访问了，一个 RPC mount 响应为文件系统建立了[文件句柄](https://baike.baidu.com/item/文件句柄/3978023)。客户端这边存储具有本地加载点的远程加载信息，并建立执行 I/O 请求的能力。这一协议表示一个潜在的安全问题；因此，NFSv4 用内部 RPC 调用替换这一辅助 mount 协议，来管理加载点。 [3] 

要读取一个文件，文件必须首先被打开。在 RPC 内没有 OPEN 程序；反之，客户端仅检查目录和文件是否存在于所加载的文件系统中。客户端以对目录的 GETATTR RPC 请求开始，其结果是一个具有目录属性或者目录不存在指示的响应。接下来，客户端发出 LOOKUP RPC 请求来查看所请求的文件是否存在。如果是，会为所请求的文件发出 GETATTR RPC 请求，为文件返回属性。基于以上成功的 GETATTRs 和 LOOKUPs，客户端创建[文件句柄](https://baike.baidu.com/item/文件句柄)，为用户的未来需求而提供的。 [3] 

利用在远程文件系统中指定的文件，客户端能够触发 READ RPC 请求。READ 包含文件句柄、状态、偏移、和读取计数。客户端采用状态来确定操作是否可执行（那就是，文件是否被锁定）。偏移指出是否开始读取，而计数指出所读取字节的数量。服务器可能返回或不返回所请求字节的数量，但是会指出在 READ RPC 回复中所返回（随着数据）字节的数量。



## 源码

nfs文件系统的源码位于fs/nfs（客户端）以及fs/nfsd（服务端）

### fs/nfs/super.c

这个文件定义了nfs文件系统超级块的各个处理函数，并且定义了nfs文件系统的注册函数。

```c
int __init register_nfs_fs(void)
```

### fs/nfs/inode.c

定义inode以及superblock处理函数。nfs的初始化与销毁函数在该文件内，作为模块的入口以及出口。

### Fs/nfs/file.c

定义了处理文件的相关操作

以文件打开为例子，首先调用file_operation对读函数nfs_file_open进行注册（842行）

```c
const struct file_operations nfs_file_operations = {
	.llseek		= nfs_file_llseek,
	.read_iter	= nfs_file_read,
	.write_iter	= nfs_file_write,
	.mmap		= nfs_file_mmap,
	.open		= nfs_file_open,
	.flush		= nfs_file_flush,
	.release	= nfs_file_release,
	.fsync		= nfs_file_fsync,
	.lock		= nfs_lock,
	.flock		= nfs_flock,
	.splice_read	= generic_file_splice_read,
	.splice_write	= iter_file_splice_write,
	.check_flags	= nfs_check_flags,
	.setlease	= simple_nosetlease,
};
```

文件打开的函数如下（65行）：

```c
/*
 * Open file
 */
static int
nfs_file_open(struct inode *inode, struct file *filp)
{
	int res;

	dprintk("NFS: open file(%pD2)\n", filp);

	nfs_inc_stats(inode, NFSIOS_VFSOPEN);
	res = nfs_check_flags(filp->f_flags);
	if (res)
		return res;

	res = nfs_open(inode, filp);
	return res;
}
```

其中nfs_inc_states函数用于向系统更新当前文件系统的状态为读，nfs_check_flags用于检查这个文件的标志位，实现如下（52行）：

```c
int nfs_check_flags(int flags)
{
	if ((flags & (O_APPEND | O_DIRECT)) == (O_APPEND | O_DIRECT))
		return -EINVAL;

	return 0;
}
```

这个文件不能同时有O_APPEND以及O_DIRECT标志位（为什么呢？），否则会返回非法参数错误。

[^O_APPEND]: O_APPEND的含义是在每次写之前，都讲标志位移动到文件的末端。
[^O_DIRECT]: O_DIRECT选项后，可以不使用缓存直接写入



最后再调用nfs_open函数，其定义于inode.c的1097行：

```c
/*
 * These allocate and release file read/write context information.
 */
int nfs_open(struct inode *inode, struct file *filp)
{
	struct nfs_open_context *ctx;

	ctx = alloc_nfs_open_context(file_dentry(filp), filp->f_mode, filp);
	if (IS_ERR(ctx))
		return PTR_ERR(ctx);
	nfs_file_set_open_context(filp, ctx);
	put_nfs_open_context(ctx);
	nfs_fscache_open_file(inode, filp);
	return 0;
}
```

alloc_nfs_context用于分配出一块负责存储该文件上下文的空间，并进行简单的初始化（957行）：

```c
struct nfs_open_context *alloc_nfs_open_context(struct dentry *dentry,
						fmode_t f_mode,
						struct file *filp)
{
	struct nfs_open_context *ctx;
	const struct cred *cred = get_current_cred();

	ctx = kmalloc(sizeof(*ctx), GFP_KERNEL);
	if (!ctx) {
		put_cred(cred);
		return ERR_PTR(-ENOMEM);
	}
	nfs_sb_active(dentry->d_sb);
	ctx->dentry = dget(dentry);
	ctx->cred = cred;
	ctx->ll_cred = NULL;
	ctx->state = NULL;
	ctx->mode = f_mode;
	ctx->flags = 0;
	ctx->error = 0;
	ctx->flock_owner = (fl_owner_t)filp;
	nfs_init_lock_context(&ctx->lock_context);
	ctx->lock_context.open_context = ctx;
	INIT_LIST_HEAD(&ctx->list);
	ctx->mdsthreshold = NULL;
	return ctx;
}
```

nfs_file_set_open_context(filp, ctx)将刚刚获得的上下文与文件绑定在一起，这里便不赘述。

put_nfs_open_context(ctx)函数没有看很懂= =



> 以下部分参考博客https://blog.csdn.net/ycnian/article/details/8523231?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase

再以文件读为例，nfs_file_read定义如下（file.c 155行）

```c
ssize_t
nfs_file_read(struct kiocb *iocb, struct iov_iter *to)
{
	struct inode *inode = file_inode(iocb->ki_filp);
	ssize_t result;

	if (iocb->ki_flags & IOCB_DIRECT)
		return nfs_file_direct_read(iocb, to);

	dprintk("NFS: read(%pD2, %zu@%lu)\n",
		iocb->ki_filp,
		iov_iter_count(to), (unsigned long) iocb->ki_pos);

	nfs_start_io_read(inode);
	result = nfs_revalidate_mapping(inode, iocb->ki_filp->f_mapping);
	if (!result) {
		result = generic_file_read_iter(iocb, to);
		if (result > 0)
			nfs_add_stats(inode, NFSIOS_NORMALREADBYTES, result);
	}
	nfs_end_io_read(inode);
	return result;
}
```

如果有DIRECT标志位，调用直接读的函数nfs_file_direct_read，若没有调用普通IO的方法。

- nfs_start_to_read: Declare that a buffered read operation is about to start, and ensure that we block all direct I/O.

- nfs_revalidate_mapping: Revalidate the pagecache

- generic_file_read_iter: generic filesystem read routine

  

### fs/nfs/read.c

读页函数，个人理解可参见我的注释：

```c
int nfs_readpages(struct file *filp, struct address_space *mapping,
		struct list_head *pages, unsigned nr_pages)
{
	struct nfs_pageio_descriptor pgio;	// io描述符，里面的链表结构包含传输的数据，其余成员有传输的总量等
	struct nfs_pgio_mirror *pgm;
	struct nfs_readdesc desc = {				// 包含pgio以及用户信息两个成员
		.pgio = &pgio,
	};
	struct inode *inode = mapping->host;
	unsigned long npages;
	int ret = -ESTALE;

	dprintk("NFS: nfs_readpages (%s/%Lu %d)\n",
			inode->i_sb->s_id,
			(unsigned long long)NFS_FILEID(inode),
			nr_pages);
	nfs_inc_stats(inode, NFSIOS_VFSREADPAGES);	// 设置状态

	if (NFS_STALE(inode))
		goto out;

  // 获取用户信息
  // 若文件为空
	if (filp == NULL) {
    // Given an inode, search for an open context with the desired characteristics
		desc.ctx = nfs_find_open_context(inode, NULL, FMODE_READ);
		if (desc.ctx == NULL)
			return -EBADF;
	} else
		desc.ctx = get_nfs_open_context(nfs_file_open_context(filp));

	/* attempt to read as many of the pages as possible from the cache
	 * - this returns -ENOBUFS immediately if the cookie is negative
	 */
  // 若缓存中有数据则直接读取
  // 实际上这个函数的实现直接返回负值
	ret = nfs_readpages_from_fscache(desc.ctx, inode, mapping,
					 pages, &nr_pages);
	if (ret == 0)
		goto read_complete; /* all pages were read */
	
  // 初始化pgio（链表置空，已传输数据量为0等）
	nfs_pageio_init_read(&pgio, inode, false,
			     &nfs_async_read_completion_ops);

  // 将缓存页添加至pgio之中
  // 该函数为系统提供的函数，提供一个地址空间以及一些页并进行对页的读操作
  // readpage_async_filler将缓存的页与pgio相关联，并为每个页生成读请求
	ret = read_cache_pages(mapping, pages, readpage_async_filler, &desc);
  // 读结束，释放pgio
	nfs_pageio_complete(&pgio);

	/* It doesn't make sense to do mirrored reads! */
	WARN_ON_ONCE(pgio.pg_mirror_count != 1);

	pgm = &pgio.pg_mirrors[0];
	NFS_I(inode)->read_io += pgm->pg_bytes_written;
	npages = (pgm->pg_bytes_written + PAGE_SIZE - 1) >>
		 PAGE_SHIFT;
	nfs_add_stats(inode, NFSIOS_READPAGES, npages);
read_complete:
	put_nfs_open_context(desc.ctx);
out:
	return ret;
}
```

