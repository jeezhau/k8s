**xtrabackup**


# 参数说明
--apply-log-only：prepare备份的时候只执行redo阶段，用于增量备份。

--backup：创建备份并且放入--target-dir目录中

--close-files：不保持文件打开状态，xtrabackup打开表空间的时候通常不会关闭文件句柄，目的是为了正确处理DDL操作。如果表空间数量非常巨大并且不适合任何限制，一旦文件不在被访问的时候这个选项可以关闭文件句柄.打开这个选项会产生不一致的备份。

--compact：创建一份没有辅助索引的紧凑备份

--compress：压缩所有输出数据，包括事务日志文件和元数据文件，通过指定的压缩算法，目前唯一支持的算法是quicklz.结果文件是qpress归档格式，每个xtrabackup创建的*.qp文件都可以通过qpress程序提取或者解压缩

--compress-chunk-size=#：压缩线程工作buffer的字节大小，默认是64K

--compress-threads=#：xtrabackup进行并行数据压缩时的worker线程的数量，该选项默认值是1，并行压缩（'compress-threads'）可以和并行文件拷贝('parallel')一起使用。例如:'--parallel=4 --compress --compress-threads=2'会创建4个IO线程读取数据并通过管道传送给2个压缩线程。

--create-ib-logfile：这个选项目前还没有实现，目前创建Innodb事务日志，你还是需要prepare两次。

--datadir=DIRECTORY：backup的源目录，mysql实例的数据目录。从my.cnf中读取，或者命令行指定。

--defaults-extra-file=[MY.CNF]：在global files文件之后读取，必须在命令行的第一选项位置指定。

--defaults-file=[MY.CNF]：唯一从给定文件读取默认选项，必须是个真实文件，必须在命令行第一个选项位置指定。

--defaults-group=GROUP-NAME：从配置文件读取的组，innobakcupex多个实例部署时使用。

--export：为导出的表创建必要的文件

--extra-lsndir=DIRECTORY：(for --bakcup):在指定目录创建一份xtrabakcup_checkpoints文件的额外的备份。

--incremental-basedir=DIRECTORY：创建一份增量备份时，这个目录是增量别分的一份包含了full bakcup的Base数据集。

--incremental-dir=DIRECTORY：prepare增量备份的时候，增量备份在DIRECTORY结合full backup创建出一份新的full backup。

--incremental-force-scan：创建一份增量备份时，强制扫描所有增在备份中的数据页即使完全改变的page bitmap数据可用。

--incremetal-lsn=LSN：创建增量备份的时候指定lsn。

--innodb-log-arch-dir：指定包含归档日志的目录。只能和xtrabackup --prepare选项一起使用。

--innodb-miscellaneous：从My.cnf文件读取的一组Innodb选项。以便xtrabackup以同样的配置启动内置的Innodb。通常不需要显示指定。

--log-copy-interval=#：这个选项指定了log拷贝线程check的时间间隔（默认1秒）。

--log-stream：xtrabakcup不拷贝数据文件，将事务日志内容重定向到标准输出直到--suspend-at-end文件被删除。这个选项自动开启--suspend-at-end。

--no-defaults：不从任何选项文件中读取任何默认选项,必须在命令行第一个选项。

--databases=#：指定了需要备份的数据库和表。

--database-file=#：指定包含数据库和表的文件格式为databasename1.tablename1为一个元素，一个元素一行。

--parallel=#：指定备份时拷贝多个数据文件并发的进程数，默认值为1。

--prepare：xtrabackup在一份通过--backup生成的备份执行还原操作，以便准备使用。

--print-default：打印程序参数列表并退出，必须放在命令行首位。

--print-param：使xtrabackup打印参数用来将数据文件拷贝到datadir并还原它们。

--rebuild_indexes：在apply事务日志之后重建innodb辅助索引，只有和--prepare一起才生效。

--rebuild_threads=#：在紧凑备份重建辅助索引的线程数，只有和--prepare和rebuild-index一起才生效。

--stats：xtrabakcup扫描指定数据文件并打印出索引统计。

--stream=name：将所有备份文件以指定格式流向标准输出，目前支持的格式有xbstream和tar。

--suspend-at-end：使xtrabackup在--target-dir目录中生成xtrabakcup_suspended文件。在拷贝数据文件之后xtrabackup不是退出而是继续拷贝日志文件并且等待知道xtrabakcup_suspended文件被删除。这项可以使xtrabackup和其他程序协同工作。

--tables=name：正则表达式匹配database.tablename。备份匹配的表。

--tables-file=name：指定文件，一个表名一行。

--target-dir=DIRECTORY：指定backup的目的地，如果目录不存在，xtrabakcup会创建。如果目录存在且为空则成功。不会覆盖已存在的文件。

--throttle=#：指定每秒操作读写对的数量。

--tmpdir=name：当使用--print-param指定的时候打印出正确的tmpdir参数。

--to-archived-lsn=LSN：指定prepare备份时apply事务日志的LSN，只能和xtarbackup --prepare选项一起用。

--user-memory = #：通过--prepare prepare备份时候分配多大内存，目的像innodb_buffer_pool_size。默认值100M如果你有足够大的内存。1-2G是推荐值，支持各种单位(1MB,1M,1GB,1G)。

--version：打印xtrabackup版本并退出。

--xbstream：支持同时压缩和流式化。需要客服传统归档tar,cpio和其他不允许动态streaming生成的文件的限制，例如动态压缩文件，xbstream超越其他传统流式/归档格式的的优点是，并发stream多个文件并且更紧凑的数据存储（所以可以和--parallel选项选项一起使用xbstream格式进行streaming）。


# 备份
## 1 全备

```sh
innobackupex  --user=USER --password=PASSWORD  --use-memory=32M --no-timestamp /backup/xfull/
```

## 2 增量备份

```sh
innobackupex --user=USER --password=PASSWORD  --incremental --no-timestamp --incremental-basedir=/backup/xfull/ /backup/xinc1/
```

# 恢复

# 1、应用全备日志（--apply-log），暂时不需要做回滚操作（--redo-only）
```
innobackupex  --apply-log --redo-only /backup/xfull/
```

# 2、增量合并到全备中（一致性的合并）
```
innobackupex  --apply-log --incremental-dir=/backup/xinc1/  /backup/xfull/
```

# 3、合并完成恢复
+ 第一种
```
cp -a  /backup/xfull/* /var/mysql/data/
chown -R mysql.mysql /var/mysql/data/
```
* 第二种
```
innobackupex --copy-back /backup/xfull/
chown -R mysql.mysql /var/mysql/data/
```

# 备份策略
- 周日: 进行全备
- 周一 ~ 六:  每天做上一天的增备
