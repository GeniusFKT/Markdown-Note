# nfs mkdir

## vfs -> nfs

nfs_mkdir函数位于dir.c的1929行，为vfs创建文件夹对应操作。

```c
/*
 * See comments for nfs_proc_create regarding failed operations.
 */
int nfs_mkdir(struct inode *dir, struct dentry *dentry, umode_t mode)
{
	struct iattr attr;
	int error;

	dfprintk(VFS, "NFS: mkdir(%s/%lu), %pd\n",
			dir->i_sb->s_id, dir->i_ino, dentry);

	attr.ia_valid = ATTR_MODE;			// 要使用ia_mode以通知mode的变化，将其值置为ATTR_MODE
	attr.ia_mode = mode | S_IFDIR;	// 由于我们要添加一个目录，将S_IFDIR标志置为1

	trace_nfs_mkdir_enter(dir, dentry);
	error = NFS_PROTO(dir)->mkdir(dir, dentry, &attr);
	trace_nfs_mkdir_exit(dir, dentry, error);
	if (error != 0)
		goto out_err;
	return 0;
out_err:
	d_drop(dentry);
	return error;
}
```

这个函数有三个参数，这三个参数与vfs_mkdir参数一致：

- dir：父目录的索引节点
- dentry：所创建文件夹的dentry
- mode：所创建文件夹的权限等标志

iattr结构体：Inode Attributes Structure. It uses the above definitions as flags, to know which values have changed.

```c
trace_nfs_mkdir_enter(dir, dentry);
trace_nfs_mkdir_exit(dir, dentry, error);
```

这两行对mkdir操作进行系统层面上的追踪，追踪点用于系统追踪以及性能统计。

若error返回了一个错误信息，则释放想要创建的dentry并将错误返回。

NFS_PROTO: include/linux/nfs_fs.h: 279

```c
// 返回客户端所支持的rpc操作
// rpc_ops为nfs_rpc_ops*类型，其内部存放着很多函数指针对应不同的文件操作（如mkdir）
static inline const struct nfs_rpc_ops *NFS_PROTO(const struct inode *inode)
{
	return NFS_SERVER(inode)->nfs_client->rpc_ops;
}
```

NFS_SERVER: 269

```c
// 这个函数由索引节点得到对应的nfs_server结构体
static inline struct nfs_server *NFS_SERVER(const struct inode *inode)
{
	return NFS_SB(inode->i_sb);
}
```

NFS_SB: 259

``` c
// s->fs_info指向超级块所属文件系统的具体信息，其为void*类型，
// 将其转换为nfs_server*类型即可得到服务器的信息
static inline struct nfs_server *NFS_SB(const struct super_block *s)
{
	return (struct nfs_server *)(s->s_fs_info);
}
```

nfs_prc_ops类型定义如下：(include/linux/nfs_xdr.h: 1647)

```c
struct nfs_rpc_ops {
	u32	version;		/* Protocol version */
	const struct dentry_operations *dentry_ops;
	const struct inode_operations *dir_inode_ops;
	const struct inode_operations *file_inode_ops;
	const struct file_operations *file_ops;
	const struct nlmclnt_operations *nlmclnt_ops;

	int	(*getroot) (struct nfs_server *, struct nfs_fh *,
			    struct nfs_fsinfo *);
	int	(*submount) (struct fs_context *, struct nfs_server *);
	int	(*try_get_tree) (struct fs_context *);
	int	(*getattr) (struct nfs_server *, struct nfs_fh *,
			    struct nfs_fattr *, struct nfs4_label *,
			    struct inode *);
	int	(*setattr) (struct dentry *, struct nfs_fattr *,
			    struct iattr *);
	...
	int	(*mkdir)   (struct inode *, struct dentry *, struct iattr *);	// 我们想要的mkdir
	int	(*rmdir)   (struct inode *, const struct qstr *);
	int	(*readdir) (struct dentry *, const struct cred *,
			    u64, struct page **, unsigned int, bool);
	...
};
```

那么这个函数是在哪里被注册到这个结构体的呢？

以v2版本为例，在fs/nfs/proc.c: 715中：

```c
const struct nfs_rpc_ops nfs_v2_clientops = {
	.version	= 2,		       /* protocol version */
	.dentry_ops	= &nfs_dentry_operations,
	.dir_inode_ops	= &nfs_dir_inode_operations,
	.file_inode_ops	= &nfs_file_inode_operations,
	.file_ops	= &nfs_file_operations,
	.getroot	= nfs_proc_get_root,
	.submount	= nfs_submount,
	.try_get_tree	= nfs_try_get_tree,
	.getattr	= nfs_proc_getattr,
	.setattr	= nfs_proc_setattr,
	...
	.mkdir		= nfs_proc_mkdir,	// 我们想要的mkdir
	.rmdir		= nfs_proc_rmdir,
	.readdir	= nfs_proc_readdir,
};
```

这个clientops在声明后被nfs客户结构体所引用，这里便不赘述。

## nfs -> nfs_proc

因此我们查看以上所引用的nfs_proc_mkdir函数(proc.c: 449)

```c
static int
nfs_proc_mkdir(struct inode *dir, struct dentry *dentry, struct iattr *sattr)
{
  // 声明数据
	struct nfs_createdata *data;
  // 声明rpc过程的信息，nfs_procedures数组保存着各过程的信息，我们通过NFSPROC_MKDIR获得创建文件夹的过程信息
  // 如encode、decode等函数
	struct rpc_message msg = {
		.rpc_proc	= &nfs_procedures[NFSPROC_MKDIR],
	};
  // 将初始状态置为超出内存
	int status = -ENOMEM;

	dprintk("NFS call  mkdir %pd\n", dentry);
  // 分配数据并根据父文件夹、创造的文件夹等信息初始化
	data = nfs_alloc_createdata(dir, dentry, sattr);
  // 分配失败则报错
	if (data == NULL)
		goto out;
  // 将rpc的参数及结果与data绑定
	msg.rpc_argp = &data->arg;
	msg.rpc_resp = &data->res;

  // 调用rpc函数进行一次同步的rpc调用，返回调用的状态
  // 并非文件系统内的函数，这里不细看
	status = rpc_call_sync(NFS_CLIENT(dir), &msg, 0);
  // 重新检查父文件夹的合理性
	nfs_mark_for_revalidate(dir);
  // 若一切正常，进行实例化
	if (status == 0)
		status = nfs_instantiate(dentry, data->res.fh, data->res.fattr, NULL);
  // 释放data并返回status
	nfs_free_createdata(data);
out:
	dprintk("NFS reply mkdir: %d\n", status);
	return status;
}
```



相关的信息

```c
struct rpc_message {
	const struct rpc_procinfo *rpc_proc;	/* Procedure information */
	void *			rpc_argp;	/* Arguments */
	void *			rpc_resp;	/* Result */
	const struct cred *	rpc_cred;	/* Credentials */
};
```

```c
#define PROC(proc, argtype, restype, timer)				\
[NFSPROC_##proc] = {							\
	.p_proc	    =  NFSPROC_##proc,					\
	.p_encode   =  nfs2_xdr_enc_##argtype,				\
	.p_decode   =  nfs2_xdr_dec_##restype,				\
	.p_arglen   =  NFS_##argtype##_sz,				\
	.p_replen   =  NFS_##restype##_sz,				\
	.p_timer    =  timer,						\
	.p_statidx  =  NFSPROC_##proc,					\
	.p_name     =  #proc,						\
	}
const struct rpc_procinfo nfs_procedures[] = {
	PROC(GETATTR,	fhandle,	attrstat,	1),
	PROC(SETATTR,	sattrargs,	attrstat,	0),
	PROC(LOOKUP,	diropargs,	diropres,	2),
	PROC(READLINK,	readlinkargs,	readlinkres,	3),
	PROC(READ,	readargs,	readres,	3),
	PROC(WRITE,	writeargs,	writeres,	4),
	PROC(CREATE,	createargs,	diropres,	0),
	PROC(REMOVE,	removeargs,	stat,		0),
	PROC(RENAME,	renameargs,	stat,		0),
	PROC(LINK,	linkargs,	stat,		0),
	PROC(SYMLINK,	symlinkargs,	stat,		0),
	PROC(MKDIR,	createargs,	diropres,	0),
	PROC(RMDIR,	diropargs,	stat,		0),
	PROC(READDIR,	readdirargs,	readdirres,	3),
	PROC(STATFS,	fhandle,	statfsres,	0),
};
```

```c
struct nfs_createdata {
	struct nfs_createargs arg;
	struct nfs_diropok res;
	struct nfs_fh fhandle;
	struct nfs_fattr fattr;
};

static struct nfs_createdata *nfs_alloc_createdata(struct inode *dir,
		struct dentry *dentry, struct iattr *sattr)
{
	struct nfs_createdata *data;
	// 动态分配data结构体
	data = kmalloc(sizeof(*data), GFP_KERNEL);
	
  // 设置参数及结果的相关内容：名字、长度、属性、file handle等
	if (data != NULL) {
		data->arg.fh = NFS_FH(dir);
		data->arg.name = dentry->d_name.name;
		data->arg.len = dentry->d_name.len;
		data->arg.sattr = sattr;
		nfs_fattr_init(&data->fattr);
		data->fhandle.size = 0;
		data->res.fh = &data->fhandle;
		data->res.fattr = &data->fattr;
	}
	return data;
};
```

file handle（fh 或者 fhandle）在 NFS 客户端和 NFS 服务器之间相互传递，建立 NFS 客户端的 inode 和 NFS 服务器的 inode 的关联关系。它主要表征的是 NFS 服务器上 inode 和物理设备的信息。file handle 对于 NFS 客户端来说是透明的，NFS 客户端不需要知道它的具体内容。file handle 在 NFS 客户端的定义是 66 个字节，前两个字节组成一个无符号 short 型，表示 file handle 的大小，后 64 个字节组成数据区，存储 file handle 的内容。

> https://www.ibm.com/developerworks/cn/linux/l-cn-nfs/index.html