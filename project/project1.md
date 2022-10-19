# project1
一开始使用了set，但是set的相等判断和顺序判断使用了同一个符号重载，而且最重要的是一开始使用了自定义对象，基于时间戳判断时间，但是对象生成的速度过快，导致时间戳完全一致，因此时间戳并不是一个可靠的衡量对象生成顺序的工具，加入集合的自然顺序才最精确。因为要频繁调整集合中元素的顺序，因此选用链表比较合适。

LRU的实现最好的是基于 **Hash + 双向链表** 的方式，因为LRU涉及头尾的增删，和随机查询，时间复杂度为O(1)。如果基于单个链表实现，则时间复杂度为O(n)。因为bufferpool频繁调用LRU，不使用hash加速查询会导致一些测试time out。

## Task2
Task2的关键在于理解frame（或者说frame中存放的page）的状态在 **free、pin 和 unpin 状态之间的转移**。

bufferpool 主要维护了3个数据结构，一个 freelist 、一个 replacer 和一个 table ：
- freelist 存放空闲的 frame
- replacer 存放可释放的 frame
- table 记录 frame 和 page 之间的对应关系

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291321116.png)

bufferpool 的实现思路如下：
- 当 new 或者 fetch 一个 page 时，就取出一个空闲的 frame ，将 page 装入 frame ，此时page pin count ++，该 frame 状态变为 pinned
- 当该page在 bufferpool 之外被修改完毕，执行unpin，如果pin count变为0，则该frame进入空闲状态，放入 replacer ，在需要时释放，用于存放新的page。
- 如果该 page 被删除，则该 frame 被释放，重新进入 free list

bufferpool的几个方法对应不同类型的sql：
- new 就相当于执行 insert sql，需要调用 diskmanager 分配磁盘空间
- fetch则用于执行 update、select sql，其中 update 会使 is_dirty 变为 true，而select的 is_dirty 则为 false
- delete用于执行 delete sql

project 中 page 的 allocate 和 deallocate 方法比较简单，直接基于 id 的递增来分配page，也没有实现 deallocate 函数。

**实现的过程中有不少坑点**：

建议使用 **递归锁**。如果使用不可重用锁，则不能进行函数的嵌套调用，会引发死锁。

注意重用page的时候重置 page 的状态，包括初始化 id、page data、is_dirty_ 和pincount。

注意flush的函数声明，当 page 不存在时，也返回 true

```cpp
bool BufferPoolManagerInstance::DeletePgImp(page_id_t page_id) {  
  // 0.   Make sure you call DeallocatePage!  
  // 1.   Search the page table for the requested page (P).  // 1.   If P does not exist, return true.  // 2.   If P exists, but has a non-zero pin-count, return false. Someone is using the page.  // 3.   Otherwise, P can be deleted. Remove P from the page table, reset its metadata and return it to the free list.  std::scoped_lock lock{latch_};  
  DeallocatePage(page_id);  
  if(page_table_.count(page_id) == 0){  
    return true;  
  }  
  frame_id_t frame_id = page_table_.at(page_id);  
  Page * page = pages_ + frame_id;  
  if(page->pin_count_ != 0){  
    return false;  
  }  
  page->page_id_ = INVALID_PAGE_ID;  
  page->ResetMemory();  
  page->is_dirty_ = false;  
  page->pin_count_ = 0;  
  page_table_.erase(page->page_id_);  
  free_list_.push_back(frame_id);  
  replacer_->Pin(frame_id); // 从replacer中移除  
  return true;  
}
```

协调好replacer的状态：
- 删除page的时候，也要将page从replacer中删除。
- new、fetch调用replace->pin

处理好page的pincount：
- new 和 fetch 和 delete 时要 pin count++，fetch多次则pin多次
- unpin 时 pincount--

Task3 

直接调用 instance 的函数即可。

不需要并发安全