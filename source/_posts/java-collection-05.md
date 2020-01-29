---
title: Java集合框架-LinkedList源码分析
date: 2020-01-29 16:17:00
author: xqh
tags:
- java
- collection
- LinkedList
- 源码
categories: java
---

# 一、类关系图

![LinkedList关系图](LinkedList.jpg)

LinkedList是基于双向链表实现的，下面简单回顾相应下数据结构基本概念。

## 什么是线性表？

线性表是具有相同特性的数据元素的一个有限序列

## 线性表分类

1.顺序存储，如数组

2.链式存储，如链表

## 什么是链表？

1.链表是一种物理存储结构上非连续、非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。

2.每个元素以结点的形式存储，而每个结点都由数据域和指针域构成
 

## 链表类别

1.单向链表：一个节点指向下一个节点

![单向链表](01.jpg)

2.双向链表：一个节点有两个指针域，分别指向前一个节点和后一个节点

![双向链表](03.jpg)

3.环形链表：能通过任何一个节点找到其他所有的节点，将两种(双向/单向)链表的最后一个结点指向第一个结点从而实现循环

![单向循环链表](02.jpg)

![双向循环链表](04.jpg)


# 二、源码分析

## 结构定义


```
/**
 * Pointer to first node.
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * Pointer to last node.
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;

private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

1.`first`和`last`分别是头节点和尾节点

2.`Node`由数据`item`、上个节点`prev`和下个节点`next`组成

## 链表操作

### 初始化

```
//一个空构造函数
public LinkedList() {}
```

### 添加

#### 尾部添加节点
```
public void addLast(E e) {
    linkLast(e);
}

public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    //把尾节点赋给临时变量
    final Node<E> l = last;
    //构造插入节点，前一个节点指向l，后一个节点为null
    final Node<E> newNode = new Node<>(l, e, null);
    //新节点做尾节点
    last = newNode;
    //如果空链表，头节点就是新节点
    if (l == null)
        first = newNode;
    else
        //l下个节点指向新插入节点
        l.next = newNode;
    size++;
    modCount++;
}
```

#### 头部添加节点

```
public void addFirst(E e) {
    linkFirst(e);
}

private void linkFirst(E e) {
    //把头节点赋给临时变量
    final Node<E> f = first;
    //构造插入节点，前一个节点指向null，后一个节点指向f
    final Node<E> newNode = new Node<>(null, e, f);
    //新节点做头节点
    first = newNode;
    //如果是空链表，新节点也就是尾节点
    if (f == null)
        last = newNode;
    else
        //f的前一个节点指向新插入节点
        f.prev = newNode;
    size++;
    modCount++;
}
```

#### 指定位置添加节点

```
public void add(int index, E element) {
    checkPositionIndex(index);
    //如果插入位置等于链表长度，则插到最后
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

//返回指定位置元素
Node<E> node(int index) {
    //如果位置在链表前半部则从前向后寻找
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

void linkBefore(E e, Node<E> succ) {
    //succ可以理解为插入到这个节点之前
    //pred为succ的前一个节点
    final Node<E> pred = succ.prev;
    //构造插入节点,前一个节点为pred,后一个节点为succ
    final Node<E> newNode = new Node<>(pred, e, succ);
    //把succ的前一个节点指向新插入节点
    succ.prev = newNode;
    //如果pred为null，新插入节点就是第一个节点
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

### 删除

#### 删除头部

```
public E removeFirst() {
    //把头节点赋给f
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

rivate E unlinkFirst(Node<E> f) {
    final E element = f.item;
    final Node<E> next = f.next;
    //把头节点数据和下一个节点指向设置为null
    f.item = null;
    f.next = null;
    //下一个节点指向头节点
    first = next;
    //如果下一个节点为null，last节点也为null
    if (next == null)
        last = null;
    else
        //下一个节点前一个节点为null
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

#### 删除尾部

```
public E removeLast() {
    //把尾节点赋给l
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

private E unlinkLast(Node<E> l) {
    final E element = l.item;
    final Node<E> prev = l.prev;
    //清空尾节点数据和指向
    l.item = null;
    l.prev = null;
    //prev就是新的尾节点
    last = prev;
    if (prev == null)
        first = null;
    else
        //prev下一个节点为null
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```

#### 删除指定位置

```
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}

E unlink(Node<E> x) {
    //x为待删除节点
    final E element = x.item;
    //x的后一个节点
    final Node<E> next = x.next;
    //x的前一个节点
    final Node<E> prev = x.prev;

    //如果前一个节点为null，下一个节点就是头节点
    if (prev == null) {
        first = next;
    } else {
        //前一个节点的下一个节点就是next
        prev.next = next;
        x.prev = null;
    }

    //下一个节点为null，prev就是尾节点
    if (next == null) {
        last = prev;
    } else {
        //next节点的前一个节点就是prev
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

1.比如原先有1、2、3三个节点，删除2这个节点，原先1的下一个节点指向3，原先3的上一个节点指向1，2这个节点清空

### 查找

#### 查找头节点或尾节点
```
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```

#### 查找指定位置节点

```
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

Node<E> node(int index) {
    //如果位置在链表前半部则从前向后寻找
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

### 更新节点

```
public E set(int index, E element) {
    checkElementIndex(index);
    //找到指定节点
    Node<E> x = node(index);
    E oldVal = x.item;
    //更新新数据
    x.item = element;
    return oldVal;
}
```

### 清空链表

```
public void clear() {
    //从头节点开始遍历直到为null
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```

### fail-fast机制

1.详见[ArrayList源码分析](https://xqhppt.github.io/2020/01/22/java-collection-03/)fail-fast部分

### 遍历

1.详见[Java集合框架-迭代器](https://xqhppt.github.io/2020/01/21/java-collection-02/)


# 三、参考资料
[数据结构之链表-动图演示](https://www.cnblogs.com/newobjectcc/p/11798410.html)