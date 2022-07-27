---
title: Skip List 跳跃表
date: 2022-07-27 21:30:59
tags:
    - data structures
    - leetcode
    - algorithm
categories:
    - 伸展运动
    - 数据结构
---
## [Skip List](https://en.wikipedia.org/wiki/Skip_list)

论文地址：[Skip Lists: A Probabilistic Alternative to Balanced Trees](https://15721.courses.cs.cmu.edu/spring2018/papers/08-oltpindexes1/pugh-skiplists-cacm1990.pdf)

### 查询
从最顶层开始，沿着链表指针向右查找到下一个节点值不小于查询值或者不存在下一个节点时，向下一层链表移动继续查找，直到进行至第一层，此时如果下一个节点值等于查找值则查找成功，否则查找失败：

这里分为两个子查询逻辑：

1. 在某一层的查询逻辑（向右查询）
```lua
-- 假设当前遍历节点为x
while x节点的次节点key < searchKey do
        x = x次节点
```
2. 在某个节点的查询逻辑（向下查询）
```lua
-- 假设当前层序为i
for i to 1 do
    同层查询
```

所以整体的查询算法实现：
```lua
Search(list, searchKey)
    x := list→header
    -- loop invariant: x→key < searchKey
    -- 向下查询
    for i := list→level downto 1 do
        -- 同层查询
        while x→forward[i]→key < searchKey do
            x := x→forward[i]
    -- x→key < searchKey ≤ x→forward[1]→key
    x := x→forward[1]
    if x→key = searchKey then return x→value
        else return failure
```

### 插入和删除
通过查询查找到符合条件的节点x（x的次节点不存在或者次节点不小于查找值），有两点需要注意：
1. 为了在插入或者删除的时候能够更新每一层的元素，需要在`某一层查找`结束向下层移动前记录当时的节点。
2. 在插入或者删除后要更新最高层，插入时可能层数+1，删除时则可能最大层数-1。

```lua
Insert(list, searchKey, newValue)
    -- update数组/list用来存放每一层在插入后需要更新的节点
    local update[1..MaxLevel]
    x := list→header
    for i := list→level downto 1 do
        while x→forward[i]→key < searchKey do
            x := x→forward[i]
        -- x→key < searchKey ≤ x→forward[i]→key
        update[i] := x
    x := x→forward[1]
    -- 如果查找成功则不会进行插入，当然也存在允许重复值插入的“变种”跳跃表
    if x→key = searchKey then x→value := newValue
    else
        -- randomLevel给出当前插入的节点最高需要插入到第几层
        lvl := randomLevel()
        -- 如果高于当前最高层，需要更新update数组/list，并更新最高层数
        if lvl > list→level then
            for i := list→level + 1 to lvl do
                update[i] := list→header
            list→level := lvl
        x := makeNode(lvl, searchKey, value)
        -- 遍历每层，更新update中的节点的次节点指针（链表插入操作）
        for i := 1 to level do
            x→forward[i] := update[i]→forward[i]
            update[i]→forward[i] := x
```

```lua
Delete(list, searchKey)
    local update[1..MaxLevel]
    x := list→header
    for i := list→level downto 1 do
        while x→forward[i]→key < searchKey do
            x := x→forward[i]
        update[i] := x
    x := x→forward[1]
    if x→key = searchKey then
        -- 删掉每一层的节点
        for i := 1 to list→level do
            if update[i]→forward[i] ≠ x then break
            update[i]→forward[i] := x→forward[i]
        free(x)
        -- 删除因节点被删而空掉（指向NIL）的层
        while list→level > 1 and 
          list→header→forward[list→level] = NIL do
            list→level := list→level – 1
```
关于`randomLevel()`，论文里也给出了探讨建议。

记预期的i+1层节点数与i层节点数的比值为`p`，则`p`为1/2表示i层有一半的节点会在i+1层出现（1/2拥有i层指针的节点也拥有i+1层的指针）,那么新插入的节点应该最高到第几层呢？从第一层开始每一层这个新节点有`p`的概率能拥有+1层的指针：
```lua
randomLevel()
    lvl := 1
    -- random() that returns a random value in [0...1)
    while random() < p and lvl < MaxLevel do
        lvl := lvl + 1
    return lvl
```

关于从第几层开始搜索，论文的`At what level do we start a search? Defining L(n)`部分也给出了几种方案，不过综合来看从最高层开始搜索也并不会有什么大问题，实现也较为简单。

其中提出的可以用来计算层数`L(n)`的公式还是比较重要的：
```text
    L(n) = log1/p n
```
`p`同上，指某一层的节点能够拥有更高一层的指针的概率，`n`是指skip list中的元素数量（等同于第一层的节点数）

例：

*已知n=16，p=1/2，合适的maxLevel=4*

*已知maxLevel=16，p=1/3，存放n=3^16元素会比较合理*

### Java实现
数据结构部分：
```java
/**
 * 跳跃表
 */
class SkipList{
    Node head;
    // 当前层数
    int level;
    // 最大层数
    int maxLevel;
}

/**
 * 链表节点
 */
class Node{
    Object key;

    // 存放层指针
    Node[] forward;
}
```