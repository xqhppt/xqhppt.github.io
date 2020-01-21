---
title: Java集合框架-迭代器
date: 2020-01-21 21:48:00
author: xqh
tags:
- java
- collection
categories: java
---

# 一、概述
Java集合框架的集合类因为内部结构不同，为了对容器内元素操作更为简单引入了迭代器模式。

迭代器模式就是提供一种方法对一个容器对象中的各个元素进行访问，而又不暴露该对象容器的内部细节。

Collection 继承了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。

## Iterator接口
Iterator为一个接口，它只提供了迭代的基本规则。

```
public interface Iterator<E> {
    boolean hasNext();

    E next();

    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

## Iterable接口
Iterable接口包含一个能产生Iterator对象的方法，并且Iterable被foreach用来在序列中移动。因此如果创建了实现Iterable接口的类，都可以将它用于foreach中。


```
public interface Iterable<T> {
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

# 二、遍历

## List遍历

### 1.使用for遍历

```
List<String> list = new ArrayList<>();
        list.add("新");
        list.add("年");
        list.add("快");
        list.add("乐");

        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
```

### 2.使用迭代器遍历

```
Iterator<String> listTerator = list.iterator();

        while (listTerator.hasNext()) {
            System.out.println(listTerator.next());
        }
```

### 3.使用foreach遍历


```
for (String str : list) {
    System.out.println(str);
}
```

> 注：Set除了不能使用for遍历2、3两种都可以

## Map遍历

### 1.通过keySet遍历

```
Map<String, String> map = new HashMap<>();
map.put("a", "新");
map.put("b", "年");
map.put("c", "快");
map.put("d", "乐");

for (String key : map.keySet()) {
    System.out.println(String.format("key=%s value=%s", key, map.get(key)));
}
```

### 2.通过entrySet使用迭代器

```
Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();

while (iterator.hasNext()) {
    Map.Entry<String, String> entry = iterator.next();
    System.out.println(String.format("key=%s value=%s", entry.getKey(), entry.getValue()));
}
```

### 3.通过enrtySet遍历

```
for (Map.Entry<String, String> entry : map.entrySet()) {
    System.out.println(String.format("key=%s value=%s", entry.getKey(), entry.getValue()));
}
```

### 4.通过values遍历value

```
for (String value : map.values()) {
    System.out.println(String.format("value=%s", value));
}
```

> Queue后面单独来看

# 三、Iterator遍历注意点

### ConcurrentModificationException异常

在使用Iterator时候对所遍历的容器进行改变其大小的操作。例如对集合进行了add、remove操作就会出现ConcurrentModificationException异常。

示例代码：

```
List<String> list = new ArrayList<>();
list.add("新");
list.add("年");
list.add("快");
list.add("乐");

Iterator<String> iterator = list.iterator();

while (iterator.hasNext()) {
    String next = iterator.next();

    if (iterator.next().equals("年")) {
        list.remove(next);
    }
}
```
### 源码分析

看ArrayList的iterator方法实现，源码如下：

```
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}

```

1.首先看`checkForComodification()`方法，会比较modCount和expectedModCount是否相等
- `modCount`是当前集合的版本号，每次修改集合都会加1
- `expectedModCount`是当前迭代器的版本号，在迭代器实例化时初始化为`modCount`

2.ArrayList调用`add()`或`remove()`方法都只更新`modCount`，`expectedModCount`则没有更新，所以调用迭代器的`next()`方法时会抛`ConcurrentModificationException`异常

3.如果使用`Iterator.remove()`方法则不会抛出异常，因为源码中调用了`expectedModCount = modCount;`所以当调用`next()`方法的时候就正常。

> 该机制主要目的是为了实现ArrayList中的快速失败机制（fail-fast），在Java集合中较大一部分集合是存在快速失败机制的。







