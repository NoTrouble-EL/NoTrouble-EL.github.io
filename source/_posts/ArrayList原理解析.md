---
title: ArrayList原理解析
date: 2021-04-13 22:08:06
tags:
- JAVASE
- Collection
categories: Collection
mathjax: true
---

## ArrayList集合底层数据结构

**ArrayList集合介绍**

List接口的可调整大小的数组实现。一般数组中：一旦初始化长度就不可以发生改变

**数组结构介绍**

* 增删慢：每次删除元素，都需要更改数组的长度，拷贝以及移动元素位置
* 查询快：由于数组在内存中是一块连续空间，因此可以根据地址+索引的方式快速获取对应位置上的元素。

 <!-- more --> 

## ArrayList继承关系

### Serializable标记性接口

**Serializable介绍**

​		类的序列化由实现java.io.Serializable接口的类启用。不实现此接口的类将不会使用任何状态序列化或反序列化。可序列化类的所有子类型都是可序列化的。序列化接口没有字段或方法，仅用于标识可序列化的语义。

​		序列化：将对象的数据写入到文件(写对象)；反序列化：将文件中对象的数据读取出来(读对象)。

**Serializable源码介绍**

```java
public interface Serializable {
}
```

**Serializable基本使用**

```java
//POJO
@Data
public class Student{
    private String name;
    private Integer age;
}
```

```java
//测试类
public class TestDemo{
    public static void main(String[] args) throws Exception{
        writeObject();
        readObject();
    }
    
    private static void writeObject() throws IOException{
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempfile"));
        Student stu1 = new Student("wk", 22);
        oos.WriteObject(stu1);
        oos.close();
    }
    
    private static void readObject() throws IOException{
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("tempfile"));
        Student stu = (Student) ois.readObject();
        ois.close();
        System.out.println(stu);
    }
}
```

**ArrayList序列化反序列化的基本使用**

### Cloneable标记性接口

**Cloneable介绍**

​		一个类实现Cloneable接口来指示Object.clone()方法，该方法对于该类的实例进行字段的复制是合法的。在不实现Cloneable接口的实例上调用对象的克隆方法会导致CloneNotSupportedException被抛出。简而言之：克隆就是依据已有的数据，创建一份新的完全的数据拷贝。

**Cloneable源码介绍**

```java
public interface Cloneable {
}
```

**ArrayList中clone()的实现**

```java
// java.util.ArrayList
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```

```java
// java.lang.Object
protected native Object clone() throws CloneNotSupportedException;
```

**clone方法的使用**

浅拷贝

```java
//浅拷贝
public class CloneDemo{
    public static void main(String[] args) throws CloneNotSupportedException{
        Student stu1 = new Student("wk", 22);
        Object stu2 = stu1.clone();
        System.out.println(stu1 == stu2);
        stu1.setAge(23);
        System.out.println(stu1);
        System.out.println(stu2);
    }
}
```

浅拷贝的局限性：基本数据类型可以完全拷贝；引用数据类型则不可以。原因是当引用数据类型仅仅拷贝了一份引用，当引用数据类型发生改变时，被克隆对象中的属性也随之改变。

深拷贝

​		让属性中的引用数据类型也实现Cloneable接口，并且重写clone()方法。在类中先克隆一个对象。调用这个对象中的属性克隆一个对象。最后让属性克隆出来的对象复制到这个类的属性中。

### RandomAccess标记接口

**RandomAccess介绍**

​		标记接口由List实现使用，以表明它们支持快速随机访问。此接口的主要目的是允许通用算法更改其行为，以便在应用于随机访问列表或顺序访问列表时提供良好的性能。
​		用于操纵随机访问列表的最佳算法(例如ArrayList)可以在应用于顺序访问列表时产生二次行为(如LinkedList)。鼓励通用列表算法在应用如果将其应用于顺序访问列表之前提供较差性能的算法时，结合给定列表是否为instanceof，并在必要时更改其行为以保证可接受的性能。
​		随机访问和顺序访问之间的区别通常是模糊的。例如，一些List实现提供渐进的线性访问时间，如果它们在实践中获得巨大但是恒定的访问时间。这样的一个List实现应该通常实现这个接口。根据经验，List实现应实现此接口，如果对于类的典型实例，此循环：

```java
for(int i = 0, n=list.size(); i < n; i++)
    list.get(i);
```

比这个循环运行得更快：

```java
for(Iterator i=list.iterator(); i.hasNext();)
    i.next();
```

**RandomAccess源码介绍**

```java
public interface RandomAccess {
}
```

### AbstractList抽象类

## ArrayList源码分析

### 构造方法

|              Constructor              |                             描述                             |
| :-----------------------------------: | :----------------------------------------------------------: |
|              ArrayList()              |                 构造一个初始容量为10的空列表                 |
|    ArrayList(int initialCapacity)     |                 构造具有指定初始容量的空列表                 |
| ArrayList(Conllection<? extends E> c) | 构一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序 |

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
private static final Object[] EMPTY_ELEMENTDATA = {};
//集合真正存储数据的容器
transient Object[] elementData;

//空参构造
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

```java
//指定初始容量的构造方法
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

```java
public ArrayList(Collection<? extends E> c) {
    //将参数列表转成数组
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

public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
//Arrays类
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

### add方法

|                           方法名                            |                             描述                             |
| :---------------------------------------------------------: | :----------------------------------------------------------: |
|                   public boolean add(E e)                   |                将指定的元素追加到此列表的末尾                |
|            public void add(int index, E element)            |              在此列表中的指定位置插入指定的元素              |
|      public boolean addAll(Collection<? extends E> c)       | 按指定集合的Iterator返回顺序将指定集合中的所有元素追加到此列表的末尾 |
| public boolean addAll(int index, Collection<? extends E> c) |    将指定集合中的所有元素插入到此列表中，从指定的位置开始    |



 ```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData,
                                             minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
 ```

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

```java
public boolean addAll(Collection<? extends E> c) {
    //把有数据的集合转成数组
    Object[] a = c.toArray();
    //有数据集合的长度
    int numNew = a.length;
    //校验及扩容 
    ensureCapacityInternal(size + numNew);  // Increments modCount
    //正真拷贝的代码
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
	
    //要移动元素的个数
    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

### 修改方法

```java
public E set(int index, E element) {
    rangeCheck(index);
    
	//取出被替换的元素
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

### 获取方法

```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```

### 转换方法

```java
//AbstractCollection

public String toString() {
    Iterator<E> it = iterator();
    if (! it.hasNext())
        return "[]";

    StringBuilder sb = new StringBuilder();
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (! it.hasNext())
            return sb.append(']').toString();
        sb.append(',').append(' ');
    }
}
```

### 迭代器

```java
public Iterator<E> iterator() {
    //创建了一个对象
    return new Itr();
}
//集合内部类
private class Itr implements Iterator<E> {
    //光标
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    //将集合实际修改次数赋值给预期修改次数
    int expectedModCount = modCount;

    Itr() {}
	//判断集合是否有元素
    public boolean hasNext() {
        //光标是否不等于集合的size
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        //将光标赋值给i
        int i = cursor;
        //判断，如果大于集合的size就说明没有元素了
        if (i >= size)
            throw new NoSuchElementException();
        //把集合存储数据的地址赋值给该方法的局部变量
        Object[] elementData = ArrayList.this.elementData;
        //进行判断，如果条件满足就会产生并发修改异常
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        //光标自增
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }
    
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

**ConcurrentModificationException**

​		集合每次调用add方法，实际修改次数变量的值都会自增一次；在获取迭代器时，集合只会执行一次实际修改集合的次数赋值给预期修改集合的次数；集合在删除元素的时候也会针对实际修改次数的变量进行自增的操作。在调用next方法时，若实际修改次数不等于期望修改次数，则会发生并发修改异常。当要删除的元素在集合的倒数第二个位置的时候，不会产生并发修改异常。

#### 迭代器中的remove方法

```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet);
        cursor = lastRet;
        lastRet = -1;
        //把实际修改集合次数赋值给预期修改次数
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}

public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

迭代器调用remove方法删除元素，其实底层真正还是调用集合自己的删除方法来删除元素。在调用remove方法中会每次给预期修改次数的变量赋值。

### 清空方法

```java
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```

### 包含方法

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

### 判断集合是否为空

```java
public boolean isEmpty() {
    return size == 0;
}
```

## 问题小结：

### ArrayList是如何扩容的？

第一次扩容为10，以后每次都是原容量的1.5倍。

### ArrayList频繁扩容导致添加性能急剧下降，如何处理？

使用ArrayList的构造方法，在初始化的时候指定容量，防止频繁扩容问题。

### ArrayList插入或删除元素一定必LinkedList慢么？

不一定慢。

### ArrayList是线程安全的么？

不是线程安全的。 

### ArrayList什么情况下需要加同步？

在集合是局部变量的情况下，不用加同步；而集合是全局变量的时候需要加同步。

### 如何复制某个ArrayList到另外一个ArrayList中取？

使用clone()方法，使用ArrayList构造方法，使用addAll方法

### 已知成员变量集合存储了N多名用户名称，在多线程的环境下，使用迭代器在读取集合数据的同时如何保证还可以正常的写入数据到集合？

读写分离集合CopyOnWriteArrayList

### ArrayList和LinkedList区别？

**ArrayList**：

* 基于动态数组的数据结构
* 对于随机访问的get和set，ArrayList优于LinkedList
* 对于随即操作的add和remove，ArrayList不一定比LinkedList慢(ArrayList底层是由于是动态数组，因此不是每次add和remove的时候都需要创建新数组)

**LinkedList**：

* 基于链表的数组结构
* 对于顺序操作，Linked不一定比ArrayList慢
* 对于随即操作，LinkedList效率明显较低

## 自定义ArrayList

```java
public class MyArrayList<E> extends ArrayList<E>{
    private Object[] elementData;
    private int size;
    private Object[] emptyArray = {};
    private final int DEFAULT_CAPACITY = 10;
    
    public MyArrayList(){
        this.elemntData = emptyArray;
    }
    
    public boolean add(E e){
        if(this.size + 1 >= elementData.length){
            grow();
        }
        elementData[size++] = e;
        return true;
    }
    
    private void grow(){
        if(this.elementData == this.emptyArray){
            this.elementData = new Object[DEFAULT_CAPACITY];
        }
        if(size == elementData.length){
            int oldCapacity = elementData.length;
            int newCapacity = oldCapacity + (oldCapacity >> 1);
            Object[] obj = new Object[newCapacity];
            System.arraycopy(this.elementData, 0, obj, 0, this.elementData.length);
            this.elementData = obj;
        }
    }
    
    public E set(int index, E element){
        checkIndex(index);
        E oldElement = (E)this.elementData[index];
        this.elementData[index] = element;
        return oldElement;
    }
    
    private void checkIndex(int index){
        if(index < 0 || index > size - 1){
            throw new IndexOutOfBoundsException();
        }
    }
    
    public E remove(int index){
        checkIndex(index);
        E oldElement = (E)this.elementData[index];
        int numMoved = size - index - 1;
        if(numMoved > 0)
            System.arraycopy(this.elementData, index+1, this.elementData, index, numMoved);
        this.elementData[--size] = null;
        return oldElement;
    }
    
    public E get(int index){
        checkIndex(index);
        return (E)this.elementData[index];
    }
    
    public int getSize(){
        return this.size;
    }
    
    public String toString(){
        if(this.size == 0){
            return "[]";
        }
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        for(int i = 0; i < size; i++){
            if(i == size - 1){
                sb.append(this.elementData[i]).append("]");
            }else{
                sb.append(this.elementData[i]).append(",").append(" ");
            }
        }
        return sb.toString();
    }
}
```

