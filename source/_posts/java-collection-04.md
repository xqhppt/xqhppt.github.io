---
title: Java集合框架-Vector源码分析
date: 2020-01-28 14:57:00
author: xqh
tags:
- java
- collection
- Vector
- 源码
categories: java
---

# 一、类关系图

![Vector关系图](Vector.jpg)

Vector实现同ArrayList差不多，方法里多了synchronized进行同步。


# 二、源码分析

## 初始化


```
protected Object[] elementData;

protected int capacityIncrement;

public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}

public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}

public Vector() {
    this(10);
}
```

1.Vector是基于数组（`Object[] elementData`）实现的

2.capacityIncrement是扩容指定增量

3.默认容量是10

## 添加元素

```
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

private void ensureCapacityHelper(int minCapacity) {
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

1.首先判断当`if (minCapacity - elementData.length > 0)`容量大于初始容量时候会进行扩容

2.如果没有指定`capacityIncrement`，每次扩展为原来的两倍`int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);`

3.通过`Arrays.copyOf(elementData, newCapacity);`把原数组复制给新数组，因为这个操作代价高所以建议初始化时候就给对象大概容量大小，减少扩容次数

## 获取元素


```
E elementData(int index) {
    return (E) elementData[index];
}

public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```

1.看源码很简单，获取指定下标数组元素

## 移除元素

```
public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    E oldValue = elementData(index);

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--elementCount] = null; // Let gc do its work

    return oldValue;
}
```

1.原理同ArrayList移除元素，详见[ArrayList源码分析](https://xqhppt.github.io/2020/01/22/java-collection-03/)

## 获取数量

```
public synchronized int size() {
    return elementCount;
}
```

## 获取元素下标


```
public synchronized int indexOf(Object o, int index) {
    if (o == null) {
        for (int i = index ; i < elementCount ; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = index ; i < elementCount ; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```



## fail-fast机制

1.详见[ArrayList源码分析](https://xqhppt.github.io/2020/01/22/java-collection-03/)fail-fast部分




