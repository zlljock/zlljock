## ArrayList简介和特点
- ArrayList实现了List接口，可扩展的数组
- 底层数据结构为数组
- 线程不安全
- 优点查询快
## 成员变量

**默认初始值为10**

```java
private static final int DEFAULT_CAPACITY = 10;//数组默认初始容量
 
private static final Object[] EMPTY_ELEMENTDATA = {};//定义一个空的数组实例以供其他需要用到空数组的地方调用 
 
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};//定义一个空数组，跟前面的区别就是这个空数组是用来判断ArrayList第一添加数据的时候要扩容多少。默认的构造器情况下返回这个空数组 
 
transient Object[] elementData;//数据存的地方它的容量就是这个数组的长度，同时只要是使用默认构造器（DEFAULTCAPACITY_EMPTY_ELEMENTDATA ）第一次添加数据的时候容量扩容为DEFAULT_CAPACITY = 10 
 
private int size;//当前数组的长度
```
## 构造函数三种方法

- 第一个构造方法用来返回一个初始容量为10的数组（具体过程后面会提到），

- 第二个用来生成一个带数据的ArrayList这边不再赘述

- 第三个构造方法就是自定义初始容量。下面我将根据默认的构造方法来展开下文。
```java
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
 
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
 
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }   
```

## 扩容机制

#### 初始化

ArrayList的底层是一个动态数组，ArrayList首先会对传进来的初始化参数initalCapacity进行判断

- 如果参数等于0，则将数组初始化为一个空数组，
- 如果不等于0，将数组初始化为一个容量为10的数组。

#### **扩容时机**

当数组的大小大于初始容量的时候(比如初始为10，当添加第11个元素的时候)，就会进行扩容，新的容量为旧的容量的1.5倍。

#### **扩容方式（add方法源码分析）**

 扩容的时候，会以新的容量建一个原数组的拷贝，修改原数组，指向这个新数组，原数组被抛弃，会被GC回收

#### **add方法**

1. 

```java
/** 在元素序列尾部插入 */
public boolean add(E e) {
    // 1. 检测是否需要扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 2. 将新元素插入序列尾部
    elementData[size++] = e;
    return true;
}

/** 在元素序列 index 位置处插入 */
public void add(int index, E element) {
    rangeCheckForAdd(index);

    // 1. 检测是否需要扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 2. 将 index 及其之后的所有元素都向后移一位
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 3. 将新元素插入至 index 处
    elementData[index] = element;
    size++;
}
```

2. 

```java
/** 计算最小容量 */
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

/** 扩容的入口方法 */
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

3. 

```java
/** 扩容的核心方法 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // newCapacity = oldCapacity + oldCapacity / 2 = oldCapacity * 1.5
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 扩容
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    // 如果最小容量超过 MAX_ARRAY_SIZE，则将数组容量扩容至 Integer.MAX_VALUE
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}

```

## 删除

```java
/** 删除指定位置的元素 */
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    // 返回被删除的元素值
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将 index + 1 及之后的元素向前移动一位，覆盖被删除值
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 将最后一个元素置空，并将 size 值减1                
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

E elementData(int index) {
    return (E) elementData[index];
}

/** 删除指定元素，若元素重复，则只删除下标最小的元素 */
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        // 遍历数组，查找要删除元素的位置
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

/** 快速删除，不做边界检查，也不返回删除的元素值 */
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

## 注意

### 关于遍历时删除

遍历时删除是一个不正确的操作，即使有时候代码不出现异常，但执行逻辑也会出现问题。

- 【强制】不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。



```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("2");
a.add("3");
Iterator<String> it = a.iterator();
while (it.hasNext()) {
    String temp = it.next();//这边会报错
    System.out.println("temp: " + temp);
    if("1".equals(temp)){
        a.remove(temp);
    }
}

//更改

while (it.hasNext()) {
    String temp = it.next();//这边会报错
    if("1".equals(temp)){
        it.remove();
    }
}

```

```
以上是关于遍历时删除的分析，在日常开发中，我们要避免上面的做法。正确的做法使用迭代器提供的删除方法，而不是直接删除。
```




