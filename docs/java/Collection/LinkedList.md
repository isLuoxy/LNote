# LinkedList 解析

## 1、何为 LinkedList

![](https://i.loli.net/2019/01/17/5c40152605611.png)

采用 Idea 查看其 UML 图如上，可以看出 LinkedList 实现了 List 接口，继承 AbstractCollection

## 2、LinkedList 组成

这里是基于 JDK 1.8 下的分析（以下代码均来自于 LinkedList 的源码）。下面文章将通过源码进行分析，会比较多代码。如果感到危险直接跳到总结即可。

在 JDK 1.8之前，LinkedList 采用的结构是**双向循环链表**，而 JDK 1.8 之后，LinkedList 改为采用**双向链表**，存储单元是其一个内部类，如下

~~~java
// LinkedList 源码 -- 内部存储单元
private static class Node<E>{
    E item; // 存储的数据
    Node<E> next; // 下一个结点
    Node<E> prev; // 上一个结点
  
    Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
     }
}
~~~



LinkedList 通过定义 first 和 last 这两个变量指向链表的头和尾，因为是双向链表，所以也有双向循环链表的优点。（例如查找尾结点）。从注释中我们可以看出对链表头结点和尾结点的定义。LinkedList 还定义一个 size 变量，是用来统计链表大小的，初始为0。

![1547714746479](https://i.loli.net/2019/01/17/5c40851f6e16f.png)



接下来我们看下 LinkedList 的操作。因为 LinkedList 继承和实现（之前图片可以看出）一些抽象类和接口，所以大部分方法是定义好的了，例如 size()， add() , remove() 这些基本方法几乎可以在每个集合中看到，这是一种面向接口编程的思想，把共同属性不同表现的行为抽象出来，以后扩展的时候就方便多了。

## 3、LinkedList 方法解析

对 LinkedList 的操作方法解析前，先了解 LinkedList 自己定义的一些方法，因为这些方法才是重点，最终调用的都是这些方法。

~~~java
/**
 * Links e as first element.
 * 头插入一个结点
 */
private void linkFirst(E e) {
     	// 获取头结点
        final Node<E> f = first;
        // 新结点，上一个结点为空，值为e，下一个结点为原头结点
        final Node<E> newNode = new Node<>(null, e, f);
    	// 设置新节点为头结点
        first = newNode;
    	// 如果原头结点为空，说明链表为空，此时插入新元素后只有一个元素存在，那么头结点和尾结点都指向新节点
        if (f == null){
            last = newNode;
        }
        else{
            // 把新节点的下一个结点指向原头结点（此时新节点已成为头结点）
            f.prev = newNode;
        }
    	// 元素+1
        size++;
        // 这个 modCount 是 AbstractList 定义的一个变量，记录操作的次数，下同。
        // 链表操作数 +1
        modCount++;
    }
~~~

~~~java
/**
 * Links e as last element.
 * 尾插入一个结点,代码含义同头插入的原理一样。
 */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null){
            first = newNode;
        }
        else{
            l.next = newNode;
        }
        size++;
        modCount++;
    }
~~~

~~~java
/**
 * Inserts element e before non-null Node succ.
 * 在某个元素前面插入新元素
 */
void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev; 
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            // 说明 succ 是原头结点，那插入新元素后头结点就是新元素了
            first = newNode;
        else
            // 把 pred 的下一个结点指向新结点
            pred.next = newNode;
        size++;
        modCount++;
    }
~~~

~~~java
/**
 * Unlinks non-null first node f.
 * 删除头结点
 */
private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            // 链表为空
            last = null;
        else
            // 头结点的前一个结点为 null
            next.prev = null;
        // 元素 -1
        size--;
        // 操作数 +1
        modCount++;
    	// 返回被删除的头结点元素值
        return element;
    }
~~~

~~~java
/**
 * Unlinks non-null last node l.
 * 删除尾结点
 */
 private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            // 链表为空
            first = null;
        else
            // 尾结点的后一个结点为 null
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
~~~

~~~java
 /**
  * Unlinks non-null node x.
  * 删除某个结点，
  */
 E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
~~~

~~~java
/**
 * Returns the (non-null) Node at the specified element index.
 * 通过序号查找返回结点
 */
    Node<E> node(int index) {
        // assert isElementIndex(index);
        // 如果 index 小于链表长度的一半，那么从头结点开始查询
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else ｛
            // 如果 index 大于链表的一半，那么从尾结点开始查询
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
~~~

~~~java
/**
 * 内部声明一个 ListItr 实现 ListIterator 接口，而 ListIterator 继承了 Iterator 接口，所以可以采用迭代器
 */
private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;
    
    // 这个构造方法被调用，然后传入index，默认传入的是 0 
    ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
      }
    
    // 我们就看经常在迭代器用到的 next() 和 hasNext()
    public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

        	// 把当前值赋值给 lastReturned，然后继续找下一个
            lastReturned = next;
            next = next.next;
        	// 记录读取的个数
            nextIndex++;
            return lastReturned.item;
        }
    
    public boolean hasNext() {
        // 通过 nextIndex 判断是否遍历完成
        return nextIndex < size;
        }
}
~~~



上面就是 LinkedList 自定义的方法，这些方法理解了，LinkedList 可以说就搞定了，因为后面这些方法最后都是调用自定义的方法。

------



###  add(E e)

~~~java
/**
* 添加一个元素
*/
public boolean add(E e) {
        linkLast(e);
        return true;
 } 

void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null){
            // 如果原尾结点为空，说明链表为空链表，所以插入新结点后头结点也指向新节点。（此时链表只有一个元素，头尾结点都是本身）
            first = newNode;
        }
        else{
            // 说明原链表不为空，那么让原尾结点的下一个结点指向新结点
            l.next = newNode;
        }
    	
        size++;
    	// 这个 modCount 是 AbstractList 定义的一个变量，记录操作的次数
        modCount++;
 }
~~~

### add(int index, E element)

~~~java
public void add(int index, E element) {
    	// 检查 index 的正确性
        checkPositionIndex(index);
		
        if (index == size)
            // 如果添加的位置等于长度，直接尾插入
            linkLast(element);
        else
            // 调用自定义的 linkBefore 方法添加元素，
            linkBefore(element, node(index));
 }
~~~

### addFirst(E e)、addLast(E e)

~~~java
 /**
  * Inserts the specified element at the beginning of this list.
  *
  * @param e the element to add
  */
    public void addFirst(E e) {
        // 调用自定义方法
        linkFirst(e);
    }

  /**
   * Appends the specified element to the end of this list.
   *
   * <p>This method is equivalent to {@link #add}.
   *
   * @param e the element to add
   */
    public void addLast(E e) {
        // 调用自定义方法
        linkLast(e);
    }
~~~

### contain(Object o)

~~~java
public boolean contains(Object o) {
       // 查找某个元素是否存在，通过序号判断
        return indexOf(o) != -1;
 }

public int indexOf(Object o) {
   		// 如果元素存在，则返回位置；如果不存在，返回-1
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
}
~~~

### get(int index)

~~~java
/**
 * 查询某个位置的值
 */
public E get(int index) {
    	// 检查合法性
        checkElementIndex(index);
    	// node 返回该值的元素引用，最后取出该元素的值
        return node(index).item;
 }
~~~

### remove(int index)、remove(Object o)

~~~java
// 通过序号删除元素
public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
 }

// 通过元素值删除元素,删除成功返回true,删除失败或者不存在元素返回false
public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
~~~

### set(int index, E e)

~~~java
// 设置某个位置的值为某元素，返回旧元素值
public E set(int index, E element) {
       // 检查 index 合法性
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
 }
~~~
## 4、 总结

LinkedList 的一些主要方法已经解析，总的工作原理就是通过一个双向链表，设置头结点和尾结点。然后默认添加元素是尾添加。因为 LinkedList 是通过链表思想实现的，所以在添加和删除元素方面较快，而在查找元素上，如果是查找某位置的元素，那么会跟链表长度的一半进行比较，如果小于一半则从头结点开始向后查找；反之从尾结点开始向前查找。

对于遍历上，从 LinkedList 的实现接口中发现并没有实现 RandomAccess 接口，所以当你用普通的 for 循环去遍历时速度是非常非常慢的，一般不使用这种方式遍历，而采用迭代器去遍历。

~~~java
// 普通 for 循环遍历，每次都要get，而 get（） 的话从前面可以得知都是从头结点或者尾结点开始一个个慢慢查，效率极低
List<Integer> list = new LinkedList<>();
for(int i = 0; i<list.size(); i++){
    list.get(i);
}
~~~

~~~java
// 由源码可知，iterator 是先找到第一个元素的引用，然后不断 next 并输出。所以很快。
List<Integer> list = new LinkedList<>();
Iterator<Integer> iterator = list.iterator();
while(iterator.hasNext()){
    iterator.next();
}

~~~



那什么时候使用 LinkedList 呢？当你频繁增删的时候，可以使用 LinkedList 。









