### 2. 链表

链表随机读写能力差，但增删和重排能力较强。C语言没有链表结构，所以Redis自制了一个。

当一个**列表键包含了数量比较多的元素**， 又或者**列表中包含的元素都是比较长的字符串**时， Redis 就会使用链表作为列表键的底层实现。

除了链表键之外， 发布与订阅、慢查询、监视器等功能也用到了链表， Redis 服务器本身还使用链表来保存多个客户端的状态信息， 以及使用链表来构建客户端输出缓冲区（output buffer）



**链表节点**定义在`adlist.h/listNode`，如下：

```c
typedef struct listNode {
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;
} listNode;
```

这是一个**双端链表**。

[![img](https://camo.githubusercontent.com/fd97e30b9e717eff8d328e1651126ddd9b1f094ca526748c10e3a751bc5dbc7e/68747470733a2f2f6275636b65742d313235393535353837302e636f732e61702d6368656e6764752e6d7971636c6f75642e636f6d2f32303230303130323132343830342e706e67)](https://camo.githubusercontent.com/fd97e30b9e717eff8d328e1651126ddd9b1f094ca526748c10e3a751bc5dbc7e/68747470733a2f2f6275636b65742d313235393535353837302e636f732e61702d6368656e6764752e6d7971636c6f75642e636f6d2f32303230303130323132343830342e706e67)

虽然可以多个Node组成链表，但是为了方便，Redis设计了`adlist.h/list` 来持有链表。

```c
typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 链表所包含的节点数量
    unsigned long len;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
} list;
```

Redis 的链表实现的特性可以总结如下：

- 双端
- 无环，表头和结尾都指向`NULL`
- 带有表头表位指针，访问$ O (1)$
- 自带链表长度计数器
- **多态**：使用`void*`来保存节点值，有泛型编程内味了。