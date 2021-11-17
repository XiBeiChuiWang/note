# Redis底层数据结构

![](https://pic.imgdb.cn/item/605324f7524f85ce2962c35a.jpg)

### Redis这样设计有两个好处：

- 可以偷偷的改进内部编码，而对外的数据结构和命令没有影响，这样一旦开发出更优秀的内部编码，无需改动对外数据结构和命令。
- 多种内部编码实现可以在不同场景下发挥各自的优势。例如ziplist比较节省内存，但是在列表元素比较多的情况下，性能会有所下降。这时候Redis会根据配置选项将列表类型的内部实现转换为linkedlist。

### 数据结构

- #### int

  如果value为整数，且长度未超过20，会将这个value保存成整数形式。

  redis可以保存10000个整数（ \#define OBJSHAREDINTEGERS 10000 ）

- #### raw（SDS）

  我们来看看源代码

  ```c
  typedef char *sds;
  
  struct sdshdr {
          // 记录 buf 数组中已使用字节的数量
          // 等于 SDS 所保存字符串的长度
          int len;
          // 记录 buf 数组中未使用字节的数量
          int free;
          // 字节数组，用于保存字符串
          char buf[];
  };
  ```

   在Redis中，只有在使用到不会被修改的字符串字面量时（比如打印日志），Redis才会采用c语言传统的字符串（以空字符结尾的字符数组），**而在Redis数据库中，所有的字符串在底层都由SDS来实现的**。 

  #####  使用sds结构的优点

  - 1.有利于减少内存碎片，提高存储效率
  - 2.常数复杂度获取字符串长度
    - C语言源生的获取字符串长度的方式是遍历整个char数组，因此复杂度为O(N)，SDS采用len字段记录长度，且header和char数组紧凑排列，获取的复杂度为O(1)。设置和更新SDS长度的工作是由SDS的api在执行时**自动完成**的。
  - 3.杜绝缓冲区溢出
    - C语言字符串不记录自身长度，也容易造成缓冲区溢出。而当SDS对自身字符串进行修改时，API会先检查SDS的剩余空间是否满足需要，如果不满足，则会先拓展空间，再执行API。
  - 4.空间预分配
    - SDS在重新分配空间的时候，会预分配一些空间来作为冗余。当SDS的len属性长度小于1MB时，Redis会分配和len相同长度的free空间。至于为什么这样分配呢，上次用了len长度的空间，那么下次程序可能也会用len长度的空间，所以Redis就为你预分配这么多的空间。
    - 但是当SDS的len属性长度大于1MB时，程序将多分配1M的未使用空间。这个时候我在根据这种惯性预测来分配的话就有点得不偿失了。所以Redis是将1MB设为一个风险值，没过风险值你用多少我就给你多少，过了的话那这个风险值就是我能给你临界值。
  - 5.惰性空间释放
    - Redis的内存回收采用惰性回收，即你把字符串变短了，那么多余的内存空间也不会立刻还给操作系统，先留着，用header的字段将其记录下来，以防接下来又要被使用呢。

- #### embstr

   embstr编码是专门用来保存短字符串的一种优化编码方式，其实他和raw编码一样，底层都会使用SDS，只不过raw编码是调用两次内存分配函数分别创建redisObject和SDS，而embstr只调用一次内存分配函数来分配一块连续的空间，embstr编码的的redisObject和SDS是紧凑在一起的。 

  其优势是：

  - embstr的创建只需分配一次内存，而raw为两次（一次为sds分配对象，另一次为objet分配对象，embstr省去了第一次）。
  - 相对地，释放内存的次数也由两次变为一次。
  - embstr的objet和sds放在一起，更好地利用缓存带来的优势。

- #### linkedlist

  ```c
  typedef struct listNode {
  
          //指向前一个节点
      struct listNode *prev;
  
      //指向后一个节点
      struct listNode *next;
  
      //值
      void *value;
  } listNode;
  ```

  很明显，和我们Java中的差不多

- #### ziplist

  压缩列表是list，hash和zset的底层实现之一，当一个列表只包含少量元素，并且每个元素要么就是小整数值，要么就是长度比较短的字符串，那么Redis使用ziplist作为列表实现。

  压缩表是为了节约内存而开发的，压缩表可以包含任意个节点，每个节点保存一个字节数组（字符串）或一个整数值。

  **压缩表数据结构**

  ![](https://pic.imgdb.cn/item/6053468c524f85ce298307f3.png)

  - Zlbytes 类型：uint32_t 记录整个压缩表占用的内存字节数，对压缩表进行内存重分配和或者计算zlend位置时被使用
  - Zltail_offset 类型：uint32_t 记录压缩列表尾节点entryN距离压缩列表的起始地址的字节数。用来快速确定表尾节点的地址。
  - Zllength 类型：uint16_t 若不超过uint16的极值65535，就是**记录着压缩表节点的数量**。否则，真实的节点数量需要遍历压缩表才能得出
  - Zlend 类型：uint8_t 特殊值0xFF（十进制255），用于标记表的末端。
  - Entry char[]或uint 长度不定，节点的长度随保存的内容而改变。

  **压缩表节点数据结构**

  字段：

  - prevrawlen：前置节点的长度（以字节为单位）
  - prevrawlensize：存储 prevrawlen 的值所需的字节大小
  - len：当前节点的长度
  - lensize：存储 len 的值所需的字节大小
  - headersize：当前节点 header 的大小，等于 prevrawlensize + lensize
  - encoding：当前节点值所使用的编码类型
  - p：指向当前节点的指针

  虽然定义了这个结构体，但是Redis**根本就没有使用**zlentry结构来作为压缩列表中用来存储数据节点中的结构，这个结构总共在32位机占用了28个字节(32位机)，在64位机占用了32个字节。这不符合压缩列表的设计目的：提高内存的利用率。

  ziplist在存储节点信息时，并没有将zlentry数据结构所有属性保存，而是做了简化。

  ![img](https://pic.imgdb.cn/item/605346fe524f85ce29838a91.png)

  **虽然在压缩列表中使用的是”压缩版”的zlentry结构，但是在对节点操作时，还是要将”压缩版” “翻译”到zlentry结构中，因为我们无法对着一串字符直接进行操作。**

  因此，就有了下面的函数：

  ```c
  /* Return a struct with all information about an entry. */
  // 将p指向的列表节点信息全部保存到zlentry中，并返回该结构
  static zlentry zipEntry(unsigned char *p) {
      zlentry e;
  
      // e.prevrawlensize 保存着编码前一个节点的长度所需的字节数
      // prevrawlen 保存着前一个节点的长度
      ZIP_DECODE_PREVLEN(p, e.prevrawlensize, e.prevrawlen);  //恢复前驱节点的信息
  
      // p + e.prevrawlensize将指针移动到当前节点信息的起始地址
      // encoding保存当前节点的编码格式
      // lensize保存编码节点值长度所需的字节数
      // len保存这节点值的长度
      ZIP_DECODE_LENGTH(p + e.prevrawlensize, e.encoding, e.lensize, e.len);  //恢复当前节点的信息
  
      //当前节点header的大小 = lensize + prevrawlensize
      e.headersize = e.prevrawlensize + e.lensize;    
      e.p = p;    //保存指针
      return e;
  }
  
  //ZIP_DECODE_PREVLEN和ZIP_DECODE_LENGTH都是定义的两个宏，在ziplist.c文件中
  ```

  **prev_entry_len**

  prev_entry_len成员实际上就是zlentry结构中prevrawlensize(记录存储prevrawlen值的所需的字节个数)和prevrawlen(记录着上一个节点的长度)这两个成员的压缩版。

  - 如果前一节点的长度小于254（即2^8-1）字节，则pre_entry_len用一个字节记录其长度。
  - 当前驱节点的长度大于等于255（即2^8-1）字节，那么prev_entry_len使用5个字节表示。
    - 并且用5个字节中的最高8位(最高1个字节)用 0xFE来标志prev_entry_len占用了5个字节，后四个字节才是真正保存前驱节点的长度值。
  - pre_entry_len**最大的用处是用来从后向前遍历**，因为**前一个节点的指针c = 当前节点指针p –pre_entry_len**，可以快速往前上溯。

- #### hashtable

  同hashmap

  **注意：**

  **Rehash是渐进式的**

   在redis的实现中，没有集中的将原有的key重新rehash到新的槽中，而是分解到各个命令的执行中，以及周期函数中 

- #### skiplist

  **跳表**

  ![](https://pic.imgdb.cn/item/60534b9f524f85ce29886c24.jpg)

  跳表可以把查找的复杂度从o(n)降到o（logn）

- ####  intset

   整数集合是set的底层实现之一，当一个集合中**只包含整数值**，并且元素数量不多时，redis使用整数集合作为set的底层实现。 

  ```c
typedef struct intset {
      //保存元素所使用类型的长度
      uint32_t encoding;
      //保存元素的个数
      uint32_t length;
      //保存元素的数组
      int8_t contents[];
  } intset;
  ```
  
  当在一个int16类型的整数集合中插入一个int32类型的值，整个集合的所有元素都会转换成32类型。 整个过程有三步：
  
  1. 根据新元素的类型（比如int32），扩展整数集合底层数组的空间大小，并为新元素分配空间。
  2. 将底层数组现有的所有元素都转换成与新元素相同的类型， 并将类型转换后的元素放置到正确的位上， 而且在放置元素的过程中， 需要继续维持底层数组的有序性质不变。
  3. 最后改变encoding的值，length+1。
  
  举个例子， 假设现在有一个`INTSET_ENC_INT16`编码的整数集合， 集合中包含三个 int16_t 类型的元素。
  
  ![img](https://pic.imgdb.cn/item/60540750524f85ce29e38ce3.png)
  
  因为每个元素都占用 16 位空间， 所以整数集合底层数组的大小为 3 * 16 = 48 位， 图 6-4 展示了整数集合的三个元素在这 48 位里的位置。
  
  ![img](https://pic.imgdb.cn/item/6054075e524f85ce29e39265.png)
  
  现在， 假设我们要将类型为 int32_t 的整数值 65535 添加到整数集合里面， 因为 65535 的类型 int32_t 比整数集合当前所有元素的类型都要长， 所以在将 65535 添加到整数集合之前， 程序需要先对整数集合进行升级。
  
  升级首先要做的是， 根据新类型的长度， 以及集合元素的数量（包括要添加的新元素在内）， 对底层数组进行空间重分配。
  
  整数集合目前有三个元素， 再加上新元素 65535 ， 整数集合需要分配四个元素的空间， 因为每个 int32_t 整数值需要占用 32 位空间， 所以在空间重分配之后， 底层数组的大小将是 32 * 4 = 128 位， 如图 6-5 所示。
  
  ![img](https://pic.imgdb.cn/item/6054076d524f85ce29e398f7.png)
  
  虽然程序对底层数组进行了空间重分配， 但数组原有的三个元素 1 、 2 、 3 仍然是 int16_t 类型， 这些元素还保存在数组的前 48 位里面， 所以程序接下来要做的就是将这三个元素转换成 int32_t 类型， 并将转换后的元素放置到正确的位上面， 而且在放置元素的过程中， 需要维持底层数组的有序性质不变。
  
  首先， 因为元素 3 在 1 、 2 、 3 、 65535 四个元素中排名第三， 所以它将被移动到 contents 数组的索引 2 位置上， 也即是数组 64 位至 95位的空间内。因为元素 2 在 1 、 2 、 3 、 65535 四个元素中排名第二， 所以它将被移动到 contents 数组的索引 1 位置上， 也即是数组的 32 位至 63 位的空间内， 如图 6-7 所示。
  
  ![img](https://pic.imgdb.cn/item/6054077e524f85ce29e3a0a9.png)
  
  之后， 因为元素 1 在 1 、 2 、 3 、 65535 四个元素中排名第一， 所以它将被移动到 contents 数组的索引 0 位置上， 也即是数组的 0 位至 31位的空间内， 如图 6-8 所示。
  
  ![img](https://pic.imgdb.cn/item/60540798524f85ce29e3aa8c.png)
  
  然后， 因为元素 65535 在 1 、 2 、 3 、 65535 四个元素中排名第四， 所以它将被添加到 contents 数组的索引 3 位置上， 也即是数组的 96 位至 127 位的空间内， 如图 6-9 所示。
  
  ![img](https://pic.imgdb.cn/item/605407aa524f85ce29e3b220.png)
  
  最后， 程序将整数集合 encoding 属性的值从 INTSET_ENC_INT16 改为 INTSET_ENC_INT32 ， 并将 length 属性的值从 3 改为 4 ， 设置完成之后的整数集合如图 6-10 所示。
  
  ![img](https://pic.imgdb.cn/item/605407bc524f85ce29e3baca.png)
  
   **注意，整数集合只支持升级操作，不支持降级操作** 
  
  ![](https://pic.imgdb.cn/item/60540939524f85ce29e45f27.png)