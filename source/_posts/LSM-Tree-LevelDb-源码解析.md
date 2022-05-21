---
title: LSM-Tree - LevelDb 源码解析
subtitle: LevelDb 源码解析
author: 阿东
url_suffix: leveldb-source
tags:
  - 数据库
categories:
  - 数据库-LevelDB
keywords: 源码解析，levelDB
description: 源码解析
copyright: true
date: 2022-05-21 19:13:33
---

# LSM-Tree - LevelDb 源码解析
# 引言
在上一篇文章[[LSM-Tree - LevelDb了解和实现]]中介绍了LevelDb相关的数据结构和核心组件，LevelDB的核心读写部分，以及为什么在这个数据库中写入的速度要比读取的速度快上好几倍。 

LevelDB的源代码还是比较好懂的，好懂到我只学过学JAVA只有定点基础C语言入门知识的人也能看懂，另一方面作者在关键的地方都给了注释，甚至告诉你为什么要这么设计<s>（写的很好很棒让人落泪为什么自己没这样的同事）</s>。

如果还是看不懂，作者也写了很多数据结构介绍的md文档（在doc目录中）告诉你核心组件的作用。

总之，不要惧怕这个数据库，无论是作为优秀代码和设计模式还是各种主流数据结构算法应用都非常值得学习和参考。

> Tip：这一节代码内容非常多，所以不建议在手机或者移动设备阅读，更适合在PC上观看。

<!-- more -->


## 源码运行
LevelDB的编译是比较简单的，可以从官网直接克隆代码。

地址：[https://github.com/google/leveldb](https://link.zhihu.com/?target=https%3A//github.com/google/leveldb)

具体操作步骤如下(也可以参考仓库中的`README`)：

```shell
git clone --recurse-submodules https://github.com/google/leveldb.git
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release .. && cmake --build .
```

完成整个编译动作之后，我们可以新增一个动态库，一个静态库和test目录，接着就可以编写单元测试了，同时官方的源代码中有很多的单元测试可以提供自己编写的测试程序进行调试使用，当然这里跳过这些内容，直接从源码开始。

## 底层存储存储结构
关联：[[SSTable]]

在LevelDB中**SSTable**是整个数据库最重要的结构，所有的SSTable文件本身的内容是**不可修改**的，虽然通常数据在内存中操作，但是数据不可能无限存储，当数据到达一定量之后就需要持久化到磁盘中，而压缩合并的处理就十分考验系统性能了，为此LevelDb使用分层的结构进行存储，下面我们从外部的使用结构开始来了解内部的设计。

整个外部的黑盒就是数据库本身了，以事务性数据库为例，通常的操作无非就是ACID四种，但是放到LSM-Tree的数据结构有点不一样，因为更新和删除其实都会通过“新增”与“合并”的方式完成新数据对旧数据的覆盖。

扯远了，我们从简单的概念开始，首先是整个DB的源代码，DB源代码可以通过以下路径访问：

https://github.com/google/leveldb/blob/main/include/leveldb/db.h

首先我们需要了解DB存储结构，可以看到存储引擎的对外提供的接口十分简单：

```c++
class LEVELDB_EXPORT DB {

public:

	// 设置数据库的key-value结构，如果没有返回OK则视为操作失败，
	// 备注：考虑默认打开sync=true操作，`Put` 方法在内部最终会调用 `Write` 方法，只是在上层为调用者提供了两个不同的选择。
	virtual Status Put(const WriteOptions& options, const Slice& key,
	
	const Slice& value) = 0;
	
	// 成功返回OK，如果异常则不返回OK，如果什么都返回，说明被删除的Key不存在，
	virtual Status Delete(const WriteOptions& options, const Slice& key) = 0;
	
	// 写入操作
	virtual Status Write(const WriteOptions& options, WriteBatch* updates) = 0;
	
	// 根据key获取数据
	virtual Status Get(const ReadOptions& options, const Slice& key,
	
	std::string* value) = 0;

} 
```

`Get` 和 `Put` 是 LevelDB 为上层提供的用于读写的接口，注意这个接口的`Update`和`Delele`操作 实际上是通过`Put`完成的，实现方式是内部做了类型判断，十分有意思，这里可以先留意一下。


## write部分
下面先从写入操作开始，看看数据是如何进入到LevelDb，以及内部是如何管理的。

Write的内部逻辑算是比较复杂的，所以这里画了下基本流程图：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204171601989.png)

我们从DB的`Write()`接口方法切入，简化代码之后大致的流程如下：

```c++
	//  为写入构建足够的空间，此时可以不需要加锁。
	Status status = MakeRoomForWrite(updates == nullptr);
	//  通过 `AddRecord` 方法向日志中追加一条写操作的记录；
	status = log_->AddRecord(WriteBatchInternal::Contents(write_batch));
	//  如果日志记录成功，则将数据进行写入
	if (status.ok()) {
		status = WriteBatchInternal::InsertInto(write_batch, mem_);
	}
```

整个执行流程如下：

- 首先调用 `MakeRoomForWrite` 方法为即将进行的写入提供足够的空间。
	- 如果当前空间不足需要冻结当前的`memtable`，此时发生` Minor Compaction `并创建一个新的 `MemTable` 对象。
	- 如果满足触发`Major Compaction`需要对数据进行压缩并且对于SSTable进行合并。
-  通过`AddRecord`方法向日志中追加一条写操作记录。
-  最终调用**memtable**往内存结构中添加**key/value**，完成最终写入操作。

将写入操作的源代码逻辑简化之后最终如下：

```c++

Status DBImpl::Write(const WriteOptions& options, WriteBatch* my_batch) {
  Writer w(&mutex_);
  w.batch = my_batch;

  MakeRoomForWrite(my_batch == NULL);
	
  uint64_t last_sequence = versions_->LastSequence();
  Writer* last_writer = &w;
  WriteBatch* updates = BuildBatchGroup(&last_writer);
  WriteBatchInternal::SetSequence(updates, last_sequence + 1);
	// 记录最终的操作记录点
  last_sequence += WriteBatchInternal::Count(updates);
	// 日志编写
  log_->AddRecord(WriteBatchInternal::Contents(updates));
	// 将数据写入memtable
  WriteBatchInternal::InsertInto(updates, mem_);

  versions_->SetLastSequence(last_sequence);
  return Status::OK();
}
```

上面代码有较多的方法封装，这里我们一个个来看。

`MaybeScheduleCompaction()`压缩合并（如果觉得这里突兀可以请参阅上文的流程图）在源码中系统会定时检查是否可以进行压缩合并，if/else用于多线程并发写入的时候进行合并写入的操作，当发现有不同线程在操作就会等待结果或者等到拿到锁之后接管合并写入的操作。

> 如果对于下面的代码有疑问可以阅读[[LSM-Tree - LevelDb了解和实现]]中关于“合并写入”的部分，为了节省时间，可以在网页中直接输入关键字“**合并写入**”快速定位，这里假设读者已经了解基本的工作流程，就不再赘述了。

#LevelDb合并写入操作

```c++
void DBImpl::MaybeScheduleCompaction() {

mutex_.AssertHeld();

if (background_compaction_scheduled_) {

// Already scheduled
	// 正在压缩

} else if (shutting_down_.load(std::memory_order_acquire)) {

// DB is being deleted; no more background compactions
	
	// DB正在被删除；不再有后台压缩

} else if (!bg_error_.ok()) {

// Already got an error; no more changes
	// 已经发生异常，不能做更多改动。

} else if (imm_ == nullptr && manual_compaction_ == nullptr &&

	!versions_->NeedsCompaction()) {

	// 不需要合并则不工作

	} else {
	// 设置当前正常进行压缩合并

	background_compaction_scheduled_ = true;
	// 开始压缩合并

	env_->Schedule(&DBImpl::BGWork, this);

	}

}
```

**不可变memtable**：

在write的函数内部有这样一串代码，此时会暂停解锁等待写入，这个写入又是干嘛的？

```cpp
Status status = MakeRoomForWrite(updates == nullptr);
```

进入方法内部会发现通过一个`while`循环判断当前的 `memtable`状态，一旦发现memtable写入已经写满整个`mem`，则需要停止写入并且将当前的`memtable`转为**imutiablememtable**，并且创建新的`mem`切换写入，此时还会同时根据一些条件判断是否可以进行压缩 `mem`。

这里额外解释源码中**GUARDED_BY**含义：

GUARDED_BY是数据成员的属性，该属性声明数据成员受给定功能保护。对数据的读操作需要**共享**访问，而写操作则需要**互斥**访问。

该 GUARDED_BY属性声明线程必须先锁定**listener_list_mutex**才能对其进行读写listener_list，从而确保增量和减量操作是原子的。

**总结：其实就是一个典型的互斥共享锁，至于实现不是本文的重点。**

mem可以看作是当前的系统备忘录或者说临时的记账板，和大多数的日志或者关系型数据库类似，都是先写入日志在进行后续的所有“事务”操作，也就是**日志优先于记录操作** 原则，根据日志写入操作加锁来完成并发操作的正常运行。

`MakeRoomForWrite` 方法中比较关键的部分都加了注释，很多操作作者都有介绍意图，代码逻辑都比较简单，多看几遍基本了解大致思路即可。（C++语法看不懂不必过多纠结，明白他要做什么就行，主要是我也看不懂，哈哈）

```cpp
while (true) {

	if (!bg_error_.ok()) {
	
		// Yield previous error
		
		s = bg_error_;
		
		break;
	
	} else if (allow_delay && versions_->NumLevelFiles(0) >=
	
	config::kL0_SlowdownWritesTrigger) {
		// 我们正接近于达到对L0文件数量的硬性限制。L0文件的数量。当我们遇到硬性限制时，与其将单个写操作延迟数而是在我们达到硬限制时，开始将每个mem单独写1ms以减少延迟变化。另外。这个延迟将一些CPU移交给压缩线程，因为 如果它与写入者共享同一个核心的话。
		
		mutex_.Unlock();
		
		env_->SleepForMicroseconds(1000);
		// 不要将一个单一的写入延迟超过一次
		allow_delay = false; 
		mutex_.Lock();
	
	} else if (!force &&
	
	(mem_->ApproximateMemoryUsage() <= options_.write_buffer_size)) {
		
		// 在当前的mem中还有空间
		break;
	
	} else if (imm_ != nullptr) {
		
		// 我们已经填满了当前的memtable，但之前的的mem还在写入，所以需要等待
		
		background_work_finished_signal_.Wait();
	
	} else if (versions_->NumLevelFiles(0) >= 
				config::kL0_StopWritesTrigger) {
	
		
		background_work_finished_signal_.Wait();
	
	} else {
	
		// A试图切换到一个新的memtable并触发对旧memtable的压缩
		assert(versions_->PrevLogNumber() == 0);
		// 新建文件号
		uint64_t new_log_number = versions_->NewFileNumber(); //return next_file_number_++;
		
		WritableFile* lfile = nullptr;
		// 新建可写入文件, 内部通过一个map构建一个文件：文件状态的简易文件系统
		// typedef std::map<std::string, FileState*> FileSystem;
		s = env_->NewWritableFile(LogFileName(dbname_, new_log_number), &lfile);
		
		if (!s.ok()) {
			// 避免死循环重复新增文件号
			versions_->ReuseFileNumber(new_log_number);
			break;
		
		}
	
		delete log_;
		
		delete logfile_;
		
		logfile_ = lfile;
		
		logfile_number_ = new_log_number;
		// 写入日志
		log_ = new log::Writer(lfile);
		// **重点：imm_ 就是immutable 他将引用指向当前已经写满的mem，其实和mem对象没什么区别，就是加了一个互斥共享锁而已（写互斥，读共享）**
		imm_ = mem_;
		
		has_imm_.store(true, std::memory_order_release);
		// 新建新的memtable
		mem_ = new MemTable(internal_comparator_);
		// 引用至新块
		mem_->Ref();
		
		force = false; // Do not force another compaction if have room
		// 尝试对于已满mem压缩合并 ，此处承接上文
		MaybeScheduleCompaction();
	
	}

}
```

下面用一个简单的示意图了解上面的大致流程：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204181000328.png)

注意这对于[[SSTable]]的原始理论的实现结构显然是有一定出入，当然这是很正常的理论和实践的差别。

在通常情况下`memtable`可以通过短暂的延迟读写请求等待压缩完成，但是一旦发现mem占用的内存过大，此时就需要给**当前的mem加锁变为_imu状态**，然后创建一个新的 MemTable 实例并且把**新进来的请求转到新的mem中**，这样就可以继续接受外界的写操作，不再需要等待 `Minor Compaction` 的结束了。

> 再次注意此处会通过函数 **MaybeScheduleCompaction** 是否进行压缩合并的操作判断。

这种无等待的设计思路来自于：[[Dynamic-sized NonBlocking Hash table]]，可以自己下下论文来看看，当然也可以等我后面的文章。

## log部分
写入的大致操作流程了解之后，下面来看看LevelDb的日志管理也就是`AddRecord()`函数的操作：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204172014056.png)



注意日志的核心部分并不在`AddRecord()`内部，因为内部只有一些简单的字符串拼接操作，这里将核心放到了`RecordType`的部分，可以看到这里通过当前日志字符长度判断不同的类型，`RecordType`标识当前记录在块里面的位置：

```c++
	//....
	enum RecordType {

		// Zero is reserved for preallocated files
		
		kZeroType = 0,
		
		  
		
		kFullType = 1,
		
		  
		
		// For fragments
		
		kFirstType = 2,
		
		kMiddleType = 3,
		
		kLastType = 4
	
	};
	//....
	
	
	RecordType type;
	
	const bool end = (left == fragment_length);
	
	if (begin && end) {
	
		type = kFullType;
	
	} else if (begin) {
	
		type = kFirstType;
	
	} else if (end) {
	
		type = kLastType;
	
	} else {
	
		type = kMiddleType;
	
	}
```

> First：是用户记录第一个片段的类型，
> Last：是用户记录的最后一个片段的类型。
   Middle：是一个用户记录的所有内部片段的类型。

如果看不懂源代码，可以根据作者的md文档介绍也可以大致了解日志文件结构：

```cpp
 record :=
      checksum: uint32     // crc32c of type and data[] ; little-endian
      length: uint16       // little-endian
      type: uint8          // One of FULL, FIRST, MIDDLE, LAST
      data: uint8[length]
```

我们可以根据描述简单画一个图：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204171749969.png)

从`RecordType`内部的定义可以看到日志固定为**32KB**大小，在日志文件中将分为多部分，但是一个日志只包含在一个单一的文件块。


RecordType 存储的内容如下：
- 前面4个字节用于CRC校验
- 接着两个字节是块数据长度
- 接着是一个字节的类型标识（标识当前日志记录在块中位置）
- 最后是数据payload部分

**32kb**大小选择是考虑到日志记录行的磁盘对齐和日志读写，针对日志写的速度也非常快，写入的日志先写入内存的文件表，然后通过`fdatasync(...)`方法将缓冲区`fflush`到磁盘中并且持久化，最后通过日志完成故障恢复的操作。

需要注意如果日志记录较大可能存在于多个block块中。

一个记录永远不会在一个块的最后六个字节内开始，理由是一个记录前面需要一些其他部分占用空间（也就是记录行的校验和数据长度标识信息等）。

为了防止单个日志块被拆分到多个文件以及压缩考虑，这种“浪费”是可以被接受。

如果读者非要清楚最后几个字节存储的是什么，想满足自己的好奇心，可以看下面的代码：

`dest_->Append(Slice("\x00\x00\x00\x00\x00\x00", leftover));`

**日志写流程图**：

日志写的流程比较简单，主要分歧点是当前块剩余空间是否够写入一个header，并且最后6个字节将会填充空格进行补齐。

在日志写入的过程中通过一个`while(ture)`不断判断`buffer`大小，如果大小超过**32KB**-最后6个字节，则需要停止写入并且把开始写入到现在位置为一个数据块。

下面是日志写流程图：

![日志写流程图](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204171802290.png)

下面是日志读流程图：

![日志读流程图](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204171802000.png)

既然日志大小为32kb，那么日志的读写单位也应该是32kb，接着便是扫描数据块，在扫描chunk的时候如果发现CRC校验不通过则返回错误信息，如果数据破损则丢弃当前chunk。

翻了一下代码，简单来说就是读取通过`while(true)`循环`read`，直到读取到类型为`Last`的`chunk`，日志记录读取完成。

`memtable`比较有意思的特点是无论插入还是删除都是通过“新增”的方式实现的（你没有看错），内部通过`Mainfest`维护状态，同时根据版本号和序列号维护一条记录是新增还是删除并且保证读取到的内容是最新值，具体介绍同样在上一节[[LSM-Tree - LevelDb了解和实现]]中。

注意**写入日志之后记录是不能查询**的（因为中间有可能存在断电故障导致真实记录没有写入），日志仅作为故障恢复，只有**数据写入到mem之后才被访问到**。

关于mem新增和删除的代码如下：

```cpp
namespace {

class MemTableInserter : public WriteBatch::Handler {

public:
	
	SequenceNumber sequence_;
	
	MemTable* mem_;
	
	void Put(const Slice& key, const Slice& value) override {
	
		mem_->Add(sequence_, kTypeValue, key, value);
		
		sequence_++;
	
	}
	
	void Delete(const Slice& key) override {
	
		mem_->Add(sequence_, kTypeDeletion, key, Slice());
		
		sequence_++;
	
	}
	
	};

} // namespace
```

在`Add()`函数的内部通过一个[[LSM-Tree - LevelDb Skiplist跳表]]完成数据的插入，在数据的node中包含了记录键值，为了保证读取的数据永远是最新的，记录需要在`skiplist`内部进行排序，节点排序使用的是比较常见的比较器`Compare`，如果用户想要自定义排序（例如处理不同的字符编码等）可以编写自己的比较器实现。

对于一条记录的结构我们也可以从 `Add()` 函数中看到作者的注释：

```cpp
// Format of an entry is concatenation of:
  //  key_size     : varint32 of internal_key.size()
  //  key bytes    : char[internal_key.size()]
  //  tag          : uint64((sequence << 8) | type)
  //  value_size   : varint32 of value.size()
  //  value bytes  : char[value.size()]
```

> [[VarInt32编码]]：在这里虽然是变长整型类型但是实际使用4个字节表示。
> `uint64((sequence << 8) | type`：位运算之后实际为7个字节的sequence长度
> 注意在tag和value_size中间有一个ValueType标记来标记记录是新增还是删除。

VarInt32 (vary int 32)，即：长度可变的 32 为整型类型。一般来说，int 类型的长度固定为 32 字节。但 VarInt32 类型的数据长度是不固定的，VarInt32 中每个字节的最高位有特殊的含义。如果最高位为 1 代表下一个字节也是该数字的一部分。

因此，表示一个整型数字最少用 1 个字节，最多用 5 个字节表示。如果某个系统中大部分数字需要 >= 4 字节才能表示，那其实并不适合用 VarInt32 来编码。

根据`get()`代码内部通过`valueType`进行区分，`valueType`占用一个字节的空间进行判断新增还是删除记录，默认比较器判断新增或者删除记录逻辑如下：

```cpp
if (comparator_.comparator.user_comparator()->Compare(

	Slice(key_ptr, key_length - 8), key.user_key()) == 0) {
	
	// Correct user key
	
	const uint64_t tag = DecodeFixed64(key_ptr + key_length - 8);
	
	switch (static_cast<ValueType>(tag & 0xff)) {
	
	case kTypeValue: {
	
		Slice v = GetLengthPrefixedSlice(key_ptr + key_length);
		
		value->assign(v.data(), v.size());
		
		return true;
	
	}
	
	case kTypeDeletion:
	
		*s = Status::NotFound(Slice());
		
		return true;
	
	}

}
```

根据代码定义和上面的描述画出下面的结构图：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204171916505.png)

**Compare键排序**

LevelDb的memtable通过跳表维护了键，内部默认情况下通过`InternalKeyComparator`对于键进行比较，下面是比较内部逻辑：

比较器通过 `user_key` 和 `sequence_number` 进行排序，同时按照user_key进行升序排序，**序列号通过插入的时间递增**，以此来保证无论是增加还是删除都是获取到最新的信息。

```cpp
/*
 一个用于内部键的比较器，它使用一个指定的比较器用于用户键部分比较，并通过递减序列号来打破平衡。
*/
int InternalKeyComparator::Compare(const Slice& akey, const Slice& bkey) const {

	// Order by:
	
	// 增加用户密钥（根据用户提供的比较器）。
	
	// 递减序列号
	
	// 递减类型（尽管序列号应该足以消除歧义）。
	
	int r = user_comparator_->Compare(ExtractUserKey(akey), ExtractUserKey(bkey));
	
	if (r == 0) {
	
		const uint64_t anum = DecodeFixed64(akey.data() + akey.size() - 8);
		
		const uint64_t bnum = DecodeFixed64(bkey.data() + bkey.size() - 8);
		
		if (anum > bnum) {
		
			r = -1;
		
		} else if (anum < bnum) {
		
			r = +1;
		
		}
	
	}
	
	return r;

}
```

需要注意被比较key**可能包含完全不同的内容**，这里读者肯定会有疑问对于key获取值进行提取信息是否会有影响，然而从get的逻辑来看它可以通过键长度，和序列号等信息进行获取Key，并且获取是**header的头部信息**，所以key是任何类型都是没有影响的。

**记录查询**

现在我们再回过头来看一下`memtable`是如何读取的，从`memtable`和`imumemble`的关系可以看出有点类似**缓存**，当`memtable`写满之后转为`imumem`并且等待同步至磁盘。

key读取和查找的顺序如下：
- 在memtable中获取指定Key，如果数据符合条件则结束查找。
- 在Imumemtable中查找指定Key，如果数据符合条件则结束查找。
- 按低层至高层的顺序在level i层的sstable文件中查找指定的key，若搜索到符合条件的数据项就会结束查找，否则返回Not Found错误，表示数据库中不存在指定的数据。

记录按照层级关系进行搜索，首先是从当前内存中正在写入`memtable`搜索，接着是`imumemtable`，再接着是存在于磁盘不同层级的`SSTable`，SSTable通过`*.ldb`的形式进行标记，可以快速找到。

最终我们可以把LevelDb的查询看作下面的形式：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204172111973.png)

**小结**：

这一部分我们了解了LevelDB源代码部分等基础结构DB，介绍了LevelDB的基础对外接口，LevelDB和map的接口看起来十分类似，这一部分重点讲述了读写操作等源代码，以及内部合并压缩的一些细节。

另外记录查询等动作和之前介绍LevelDB等读写流程大致类似，当然代码简化了很多的内容，读者可以根据自己感兴趣的内容研究。

## SSTable操作
前面我们提到了记录的增删改查底层查询，和日志的读写细节，下面则针对谷歌发明的特殊数据结构`SSTable`进行介绍。

**SSTable如何工作？**

`SSTable`在初始的论文中可以总结出下面的特点：
-   写入的时候不写入磁盘而是先写入内存表的数据结构。
-   当数据结构内存占用超过一定的阈值就可以直接写入到磁盘文件由于已经是排好序的状态，所以可以直接对旧结构覆盖，写入效率比较高。并且写入和数据结构改动可以同时进行。
-   读写顺序按照 内存 - 磁盘 - 上一次写入文件 - 未找到。
-   后台定时线程定时合并和压缩排序分段，将废弃值给覆盖或者丢弃。

[[SSTable]] 最早出现在谷歌2006年的论文当中，LevelDB的SSTable设计也有部分特性体现这个数据结构，当然并不是完全一致的，LevelDB利用SSTable在磁盘中维护多层级的数据节点。

可以认为了解SSTable结构就相当于了解了LevelDb的核心数据结构设计。

**多层级SSTable**

我们重点看看多层级的SSTable部分，levelDB在磁盘中扫描SSTable的时候LevelDB并不会跳过层级，这里肯定会有疑问每个层级都扫一遍的效率问题，针对这个问题作者在db中设计了下面的数据结构：

```cpp
struct FileMetaData {

	FileMetaData() : refs(0), allowed_seeks(1 << 30), file_size(0) {}
	int refs;
	
	int allowed_seeks; // 允许压缩搜索
	
	uint64_t number;
	
	uint64_t file_size; // 文件大小
	
	InternalKey smallest; // 表提供的最小内部密钥
	
	InternalKey largest; // 表提供最大内部密钥

};
```

在上面的结构体声明中定义了压缩SSTable文件的全部信息，包括最大值和最小值，运行查找次数，文件引用次数和文件号，SSTable会按照固定的形式存储到同一个目录下面，所以可以通过文件号进行快速搜索。

查找和记录key顺序类似，都是按照**从小到大**的顺序进行读取的，以Level0为例，里面通常包含**4个固定的SSTable**，并且内部通常存在key交叉，所以会按照从SSTable1-4的顺序进行读取，而更高层次的层级则通过查找上面结构体的最大值和最小值的信息（smallest和largest）。

具体的文件搜索细节可以通过`TableCache::FindTable`查找 ，由于篇幅有限这里就不贴代码了，简要逻辑是配合缓存和`RandomAccessFile`对于文件进行读写，然后把读到的文件信息写入到内存中方便下次获取。

> 如果了解Mysql Btree设计会发现文件搜索有些类似页目录的查找。不同的是Btree页目录通过页目录等稀疏搜索。

**SSTable合并**

我们再来看看SSTable是如何合并的，之前提到过SSTable通过**MaybeScheduleCompaction**尝试合并，需要注意这个合并压缩和Bigtable的形式类似，都是根据不同的条件判断是否进行合并，一旦可以合并便执行`BackgroundCompaction`操作。

合并分为两种情况，一种是**Minor Compaction**，另一种是将**Memtable**数据写满转为不可变对象（实际就是加锁），执行`CompactMemtable`进行压缩。

合并操作简化版源代码如下：

```cpp
void DBImpl::CompactMemTable() {

	VersionEdit edit;
	Version* base = versions_->current();
	WriteLevel0Table(imm_, &edit, base);
	versions_->LogAndApply(&edit, &mutex_);
	RemoveObsoleteFiles();

}
```

CompactMemTable方法会先构建当前的修改版本号，然后调用`WriteLevel0Table()`方法尝试把当前的Imumtable写入到Level0的层级。
如果发现Level0的层级SSTable过多，则进一步进行**Major Compaction**，同时根据`BackgroudCompcation()`选择合适的压缩层级和压缩方式。

下面是`writeLevel0`的简化代码：

简化代码的最后几行代码会获取文件信息的最大值和最小值以此判断是否在当前SSTable搜索还是跳转到下一个。

数据如果是写入Level0我们可以看作是**Major Compaction**。

```cpp
Status DBImpl::WriteLevel0Table(MemTable* mem, VersionEdit* edit,

Version* base) {
	
	// SSTable文件信息
	FileMetaData meta;
	
	meta.number = versions_->NewFileNumber();
	
	pending_outputs_.insert(meta.number);

	Iterator* iter = mem->NewIterator();
	// 构建SSTable文件
	BuildTable(dbname_, env_, options_, table_cache_, iter, &meta);

	pending_outputs_.erase(meta.number);

	// 注意，如果 file_size 为零，则该文件已被删除，并且不应被添加到清单中。
	// 获取文件信息的最大值和最小值
	const Slice min_user_key = meta.smallest.user_key();
	
	const Slice max_user_key = meta.largest.user_key();
	// level层级扫描
	base->PickLevelForMemTableOutput(min_user_key, max_user_key);
	// 写入文件
	edit->AddFile(level, meta.number, meta.file_size, meta.smallest,
	
	return Status::ok();
	
```

结合上下两段源码可以发现文件管理最终是通过`VersionEdit`来完成的，如果写入成功了则返回当前的SSTable的`FileMetaData`，在`VersionEdit`内部通过`logAndApply`的方式记录文件内部的变化，也就是前文介绍的日志管理功能了，完成之后通过`RemoveObsoleteFiles()`方法进行数据的清理操作。

如果`Level0`写满了此时就需要进行**Major Compaction**，这个压缩会比前面的要复杂一些因为涉及低层级到高层级的压缩。

这里需要再回看`BackgroundCompaction`的代码，具体代码如下：

```cpp
void DBImpl::BackgroundCompaction() {  
	// 如果存在不可变imumem,进行压缩合并
	CompactMemTable();
  
	versions_->PickCompaction();

	VersionSet::LevelSummaryStorage tmp;

	CompactionState* compact = new CompactionState(c);
	
	DoCompactionWork(compact);

	CleanupCompaction(compact);

	c->ReleaseInputs();

	RemoveObsoleteFiles();

}
```

首先根据`VersionSet` 查找需要压缩的信息，并且打包加入到 **Compaction** 对象，这个对象根据查询次数和大小限制来选择需要压缩的**两个层级**，因为level0中包含很多重叠键，则会在更高层级找到有重叠的键的SSTable，再通过`FileMetaData`找到需要压缩的文件，另外查询频繁的SSTable将会“升级”到更高层级进行压缩存储，并且更新文件信息方便下一次查找。

**合并的触发条件**

每个 SSTable 在创建之后的 `allowed_seeks` 都为 100 次，当 `allowed_seeks < 0` 时就会触发该文件的与更高层级和合并，因为频繁查询的数据通常会降低系统性能。

这样的设计理由是**在高层级搜索键说明在上一层肯定是相同的键查找**，同时也是为了减少每次都覆盖扫描多层级扫描寻找数据。最终这种设计方式核心是以更新**FileMetaData** 来减少下一次查询的性能开销。

另外这种处理可以简单理解为我们在操作系统中进行深层次文件夹搜索的时候，如果频繁查询某个深层次的数据很麻烦，解决此问题的第一种方式是建立一个“快捷方式”的文件夹，另一种是直接做标签直接指向这个目录，其实两者都是差不多的，所以压缩设计也是同理。

LevelDB 中的 `DoCompactionWork` 方法会对所有传入的 SSTable 中的键值使用**归并排序**进行合并，最后会在更高层级中生成一个新的 SSTable。

> 归并排序主要是对于key进行归并，使得迭代的时候key就是有序的可以直接合并到指定的高高层级。关键代码存在于下面的代码
> `Iterator* input = versions_->MakeInputIterator(compact->compaction);`

**归并排序**

**DoCompactionWork** 归并排序 的源代码如下：

```cpp
Status DBImpl::DoCompactionWork(CompactionState* compact) {
	int64_t imm_micros = 0; // Micros spent doing imm_ compactions
	

	if (snapshots_.empty()) {
		// 快照为空，找到直接采用记录信息的最后序列号
	
		compact->smallest_snapshot = versions_->LastSequence();
	
	} else {
		// 快照存在，则抛弃之前所有的序列
		compact->smallest_snapshot = snapshots_.oldest()->sequence_number();
	
	}
	
	  
	// 对于待压缩数据进行，内部生成一个MergingIterator，当构建迭代器之后键内部就是有序的状态了，也就是前面说的归并排序的部分
	Iterator* input = versions_->MakeInputIterator(compact->compaction);

	input->SeekToFirst();
	
	Status status;
	
	ParsedInternalKey ikey;
	
	std::string current_user_key;
	//当前记录user key
	bool has_current_user_key = false;
	
	SequenceNumber last_sequence_for_key = kMaxSequenceNumber;
	
	while (input->Valid() && !shutting_down_.load(std::memory_order_acquire)) {
	
	// 优先考虑imumemtable的压缩工作
	
	if (has_imm_.load(std::memory_order_relaxed)) {
	
		const uint64_t imm_start = env_->NowMicros();
		
		imm_micros += (env_->NowMicros() - imm_start);
	
	}
	
	
	Slice key = input->key();
	
	if (compact->compaction->ShouldStopBefore(key) &&
		
		compact->builder != nullptr) {
		
		status = FinishCompactionOutputFile(compact, input);
	
	}
	
	  
	
	// 处理键/值，添加到状态等。
	
	bool drop = false;
	
	if (!ParseInternalKey(key, &ikey)) {
	
		// 删除和隐藏呗删除key
		
		current_user_key.clear();
		
		has_current_user_key = false;
		// 更新序列号
		last_sequence_for_key = kMaxSequenceNumber;
	
	} else {
	
		if (!has_current_user_key ||
		
			user_comparator()->Compare(ikey.user_key, Slice(current_user_key)) !=
		
		0) {
		
		// 用户key第一次出现
		
		current_user_key.assign(ikey.user_key.data(), ikey.user_key.size());
		
		has_current_user_key = true;
		
		last_sequence_for_key = kMaxSequenceNumber;
		
	}
		
	  
	
	if (last_sequence_for_key <= compact->smallest_snapshot) {
	
		// 压缩以后旧key边界的被新的覆盖
		
		drop = true; // (A)
	
	} else if (ikey.type == kTypeDeletion &&
	
		ikey.sequence <= compact->smallest_snapshot &&
	
		compact->compaction->IsBaseLevelForKey(ikey.user_key)) {
	
		// 对于这个用户密钥：
	
		// (1) 高层没有数据
		
		// (2) 较低层的数据会有较大的序列号
		
		// (3) 层中的数据在此处被压缩并具有
		
		// 较小的序列号将在下一个被丢弃
		
		// 这个循环的几次迭代（根据上面的规则（A））。
		
		// 因此，此删除标记已过时，可以删除。
	
		drop = true;
	
	}
	
	  
	
	last_sequence_for_key = ikey.sequence;

}

}
```

归并排序并且处理完键值信息完成跨层级压缩，之后便是是一些收尾工作，收尾工作需要对于压缩之后的信息统计。

```cpp
	CompactionStats stats;

	stats.micros = env_->NowMicros() - start_micros - imm_micros;
	//选择两个层级的SSTable
	for (int which = 0; which < 2; which++) {
	
		for (int i = 0; i < compact->compaction->num_input_files(which); i++) {
		
			stats.bytes_read += compact->compaction->input(which, i)->file_size;
		
		}
	
	}
	for (size_t i = 0; i < compact->outputs.size(); i++) {
	
		stats.bytes_written += compact->outputs[i].file_size;
	
	}
	// 压缩到更高的层级
	stats_[compact->compaction->level() + 1].Add(stats);
	// 注册压缩结果
	InstallCompactionResults(compact);
	// 压缩信息存储
	VersionSet::LevelSummaryStorage tmp;
	Log(options_.info_log, "compacted to: %s", versions_->LevelSummary(&tmp));
	
	return status;
```

最后层级压缩的默认层级为**7个层级**，在源代码中有如下定义：

`static const int kNumLevels = 7;`

**小结**

这里我们小结一下合并压缩的两个操作：`Minor Compaction`和`Major Compaction`：

`Minor Compaction`：这个GC主要是Level0层级的一些压缩操作，由于Level0层级被较为频繁使用，类似一级缓存，键值不会强制要求进行排序，所以重叠的键会比较多，整个压缩的过程比较好理解，关键部分是skiplist（跳表）中构建一个新的SSTable并且插入到指定层级。

注：`Minor Compaction`进行的时候会暂停`Major Compaction`操作。

**Minor Compaction**：这个比Minor Compaction复杂不少，不仅包含跨层级压缩，还包括键范围确定和迭代器归并排序和最终的统计信息操作，其中最最关键的部分是归并排序压缩列表，之后将旧文件和新文件合并生产新的`VersionSet`信息，另外这里除开全局的压缩进度和管理操作之外。

另外Minor Compaction完成之后还会再尝试一次Minor Compaction，因为Minor Compaction可能带来更多的重复键，所以再进行一次压缩可以进一步提高查找效率。

**Major Compaction**：这个操作需要暂停整个LevelDB的读写，因为此时需要对于整个LevelDb的多层级进行跨层级合并，跨层级压缩要复杂很多，具体的细节会在后面介绍。

> 这里可以认为是作者在测试的过程发现一种情况并且做的优化。

**存储状态 - VersionSet**

从这个对象名称来看直接理解为“版本集合”，在内部通过一个Version的结构体对于键值信息进行“版本控制”，毫无疑问这是由于多线程压缩所带来的特性，所以最终是一个双向链表+历史版本的形式串联，但是永远只有一个版本是当前版本。
VersionSet最为频繁也是比较关键的一个操作函数`LogAndApply`，下面是简化之后的`VersionSet::LogAndApply`代码：

> 这里可以对照关系型数据库Mysql的Mvcc中的undo log类比进行理解

```cpp
Status VersionSet::LogAndApply(VersionEdit* edit, port::Mutex* mu) {
	// 更新版本链表信息
	if (!edit->has_prev_log_number_) {
	
		edit->SetPrevLogNumber(prev_log_number_);
	
	}

	edit->SetNextFile(next_file_number_);
	
	edit->SetLastSequence(last_sequence_);

	Version* v = new Version(this);
	// 构建当前的版本version，委托给建造器进行构建
	Builder builder(this, current_);
	
	builder.Apply(edit);
	
	builder.SaveTo(v);

	// 关键方法：内部通过打分机制确定文件所在的层级，值得注意的是level0的层级确定在源代码中有较多描述
	Finalize(v);

// 如有必要，通过创建包含当前版本快照的临时文件来初始化新的描述符日志文件。

	std::string new_manifest_file;
	//  没有理由在这里解锁*mu，因为我们只在第一次调用LogAndApply时（打开数据库时）碰到这个路径。
	new_manifest_file = DescriptorFileName(dbname_, manifest_file_number_);
	// 写入mainfest文件
	env_->NewWritableFile(new_manifest_file, &descriptor_file_);

	// 写入版本信息快照
	WriteSnapshot(descriptor_log_);

	// 把记录写到 MANIFEST中

	descriptor_log_->AddRecord(record);

	//如果创建了新的文件，则将当前版本指向这个文件

	SetCurrentFile(env_, dbname_, manifest_file_number_);

	// 注册新版本
	
	AppendVersion(v);
	
	return Status::OK();

}
```

关键部分注释已给出，这里的**Mainfest**细节在之前没有提到过，在作者提供的`impl.md`是这样介绍mainfest的：

> MANIFEST 文件列出了组成每个级别的排序表集、相应的键范围和其他重要的元数据。 每当重新打开数据库时，都会创建一个新的 MANIFEST 文件（文件名中嵌入了一个新编号）。 MANIFEST 文件被格式化为日志，并且对服务状态所做的更改（随着文件的添加或删除）被附加到此日志中。

从个人的角度来看，这个文件有点类似BigTable中的元数据`Meta`。

**SSTable文件格式**

理解这部分不需要急着看源代码，在仓库中的`table_format.md`的文件中同样有相关描述，这里就直接照搬官方文档翻译了：

> leveldb 文件格式
<beginning_of_file>
[数据块 1]
[数据块 2]
...
[数据块N]
[元块 1]
...
[元块K]
[元索引块]
[索引块]
[页脚]（固定大小；从 file_size - sizeof(Footer) 开始）
<end_of_file>

我们可以根据描述画一个对应的结构图：

![](https://adong-picture.oss-cn-shenzhen.aliyuncs.com/adong/202204172242656.png)


上面的结构图从上至下的介绍如下：

- 数据块：按照LSM-Tree的数据存储规范，按照key/value的顺序形式进行排序，数据块根据`block.builder.cc`的内部逻辑进行格式化，并且可以选择是否压缩存储。
- 元数据块：元数据块和数据块类似也使用`block.builder.cc`进行格式化，同时可选是否压缩，元数据块后续扩展更多的类型（主要用作数据类型记录）
- “元索引”块：为每个其他元数据块索引，键为元块的名称，值为指向该元块的 BlockHandle。
- “索引块”：包含数据块的索引，键是对应**字符串>=数据块的最后一个键**，并且在连续的数据块的第一个键之前，值是 数据块的 BlockHandle。
- 文件的最后是一个固定长度的页脚，其中包含元索引和索引块的 BlockHandle 以及一个**幻数**。

> 幻数又被称为魔数，比如JAVA的字节码第一个字节8位是`CAFEBABE`，数值和字节大小没什么意义，更多是作者的兴趣。

注意Footer页脚固定48个字节的大小，我们能在其中拿到 **元索引块** 和 **索引块**的位置，然后通过这两个索引寻找其他值对应的位置。

更详细的内容可以继续参考`table_format.md`介绍，这里就不再赘述了。

**TableBuilder**：

SSTable接口定义于一个`TableBuilder`构建器当中，**TableBuilder** 提供了用于构建 Table 的接口，关于此接口的定义如下：

TableBuilder提供了用于建立表的接口  (一个从键到值的不可变和排序的映射)。

多个线程可以在一个TableBuilder上调用const方法而不需要外部同步。但如果任何一个线程可能调用一个非常量方法，所有访问同一个TableBuilder的线程必须使用外部同步。

```cpp
// TableBuilder 提供了用于构建 Table 的接口
//（从键到值的不可变且排序的映射）。
//
// 多个线程可以在 TableBuilder 上调用 const 方法，而无需
// 外部同步，但如果任何线程可能调用
// 非常量方法，所有访问同一个 TableBuilder 的线程都必须使用
// 外部同步。
class LEVELDB_EXPORT TableBuilder {

public:

	TableBuilder(const Options& options, WritableFile* file);
	
	TableBuilder(const TableBuilder&) = delete;
	
	TableBuilder& operator=(const TableBuilder&) = delete;
	/*
	改变该构建器所使用的选项。注意：只有部分的
	选项字段可以在构建后改变。如果一个字段是
	不允许动态变化，并且其在结构中的值
	中的值与传递给本方法的结构中的值不同。
	结构中的值不同，该方法将返回一个错误
	而不改变任何字段。
	*/
	Status ChangeOptions(const Options& options);
	
	void Add(const Slice& key, const Slice& value);
	
	void Flush();
	
	Status status() const;
	
	Status Finish();
	/*
	表示应该放弃这个建设者的内容。停止
	在此函数返回后停止使用传递给构造函数的文件。
	如果调用者不打算调用Finish()，它必须在销毁此构建器之前调用Abandon()
	之前调用Abandon()。
	需要。Finish()、Abandon()未被调用。
	*/
	void Abandon();
	
	uint64_t NumEntries() const;
	
	uint64_t FileSize() const;
	
	private:
	
		bool ok() const { return status().ok(); }
		
		void WriteBlock(BlockBuilder* block, BlockHandle* handle);
		
		void WriteRawBlock(const Slice& data, CompressionType, BlockHandle* handle);
		
		struct Rep;
		
		Rep* rep_;
	
	};
} 
```

**小结**：

SSTable 相关的设计在整个LevelDB中有着重要的地位和作用，我们介绍了SSTable的多层级合并和压缩的细节，以及两种不同的压缩形式，第一种是针对Level0的简单压缩，简单压缩只需要把存在于内存中的SSTable也就是将Imumemtable压缩到磁盘中存储，特别注意的是这个动作在第一次完成之后通常还会再执行一次，目的是为了防止合并之后产生的。

另一种是针对频繁Key查询进行的多层级压缩，多层级压缩要比简单压缩复杂许多，但是多层级压缩是提高整个LevelDB写入性能和查询性能到关键。

最后，从LevelDB中也可以看到很多经典数据结构和算法的实现，比键管理利用了跳表+归并排序的方式提高管理效率，排序的内容不仅利于查询，在存储的时候也有利于数据的顺序扫描。

## Skiplist跳表
跳表不仅在LevelDb中使用，还在许多其他的中间件中存在实现，这一部分内容将会放到下一篇文章单独介绍。

压缩文件使用了归并排序的方式进行键合并，而内部的数据库除了归并排序之外还使用了比较关键的[[LSM-Tree - LevelDb Skiplist跳表]]来进行有序键值管理，在了解LevelDB跳表的细节之前，需要先了解跳表这个数据结构的基本概念。

[[LevelDb跳表实现]]

## 布隆过滤器
Bloom Filter是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合。Bloom Filter的这种高效是有一定代价的：在判断一个元素是否属于某个集合时，有可能会把不属于这个集合的元素误认为属于这个集合（false positive）。因此，Bloom Filter不适合那些“零错误”的应用场合。而在能容忍低错误率的应用场合下，Bloom Filter通过极少的错误换取了存储空间的极大节省。

leveldb中利用布隆过滤器判断指定的key值是否存在于sstable中，若过滤器表示不存在，则该key一定不存在，由此加快了查找的效率。

布隆过滤器在不同的开源组件中用的也比较多，所以这里同样放到了一篇单独文章讲解。

[[LSM-Tree - LevelDb布隆过滤器]]

# 写在最后
LevelDB的设计还是很有意思的，关键是大部分的代码都有解释和介绍。

源代码内容很多，但是仔细分析的话不难分析，感谢看到最后。

# 参考资料

- [leveldb-handbook](https://leveldb-handbook.readthedocs.io/zh/latest/index.html)
- [bigtable-leveldb](https://draveness.me/bigtable-leveldb/)
- [Skip List--跳表（全网最详细的跳表文章没有之一） - 简书 (jianshu.com)](https://www.jianshu.com/p/9d8296562806)
-  [# Bloom Filter概念和原理](https://blog.csdn.net/jiaomeng/article/details/1495500)

# 相关阅读

- [LSM-Tree - LevelDb了解和实现](https://segmentfault.com/a/1190000041720205)
- [《数据密集型型系统设计》LSM-Tree VS BTree](https://segmentfault.com/a/1190000041652003)