# List



## List

```
int size();

boolean isEmpty();

boolean contains(Object o);//其中是否包括对象o

Iteratior<E> iterator();

Object[] toArray(); 每次返回的数组都是不同的数组对象但是内容相同

T[] toArray(T[] a);返回List中T类型的元素按顺序输出为一个数组

boolean add(T e);

boolean remove(Object o);

boolean containsAll(Collection<?> c);//是否全部包含c中的所有元素

boolean addAll(Collection<? extends E> c);将集合c中的全部元素都添加到List后面去

boolean addAll(int index,Collection<? extends E>c);将集合c中的所有元素添加到List的index下标后面去

boolean removeAll(Collection<?> c);将集合c中的所有元素从List中移除

boolean retainAll(Collection<?> c);List中只保留集合c中包含的元素

default void replaceAll(UnaryOperator<E> operator){将List中所有的元素替换为目标元素,如果该列表元素迭代器不支持则会报出异常
 Objects.requireNonNull(operator);//判断其是否为null,如果为null则报异常
 final ListIterator<E>li=this.listIterator();
 while(li.hasNext()){
   li.set(operator.apply(li.next()));//循环将原List中所有元素替换为目标元素
 }
}

@SuppressWarnings({"unchecked","rawtypes"})//表示对未检查转换警告和类型检查警告保持静默
default void sort(Comparator<? super E>c){
   Object[] a=this.toArray();
   Arrays.sort(a,(Comparator)c);
   ListIterator<E> i=this.listIterator();
   for(Object e:a){
   i.next();
   i.set((E) e);
   }

}

void clear();

boolean equals(Object o);//比较对象与该列表是否相同

int hashCode();//返回该list的hash值

E get(int index);//获取列表中某个下标的值

E set(int index,E element);//设定该位置的值，并返回原来的值

void add(int index,E element);想指定位置插入一个元素如果后面还有元素全部往后移

E remove(int index);//移除某个位置的元素并返回该元素的值

int indexOf(Object o);返回该元素的第一个匹配的下标如果不包含就返回-1

int lastIndexOf(Object o);//返回该元素最后一个匹配的下标如果不包含就返回-1

ListIterator<E> listItetator();//返回该列表的迭代器

ListIterator<E> listIterator(int index);//返回从该下标开始迭代器

List<E> subList(int fromIndex,int toIndex);//返回从from开始到to的List部分,包含from端点但是不包含toIndex端点

```



## Arrays

```
private static final int MIN_ARRAY_SORT_GRAN=1<<13;//即少于2的13次方时不会执行并行排序，由于长度太小会导致内存争用反而不会加速

static final class NaturalOrder implements Comparator<Object>{
 public int compare(Object first,Object second){ return ((Comparable<Object>first).compareTo(second);}
 
 static final NaturalOrder INSTANCE=new NaturalOrder();
}//一个比较器可以实现同类型可比较的的自然排序



private static void rangeCheck(int arrayLength,int fromIndex,int toIndex){
   if(fromIndex>toIndex){
   throw new IllegalArgumentException("fromIndex>toIndex";)
   }
   if(fromIndex<0){
   throw new ArrayIndexOutOfBoundsException(fromIndex);
   }
   if(toIndex>arrayLength){
   throw new ArrayIndexOutOfBoundsException(toIndex);
   }
}//检查formIndex和toIndex是否在范围内如果没有就抛出异常


public static void sort(int[] a){DualPivotQuicksort.sort(a,0,a.length-1,null,0,0);}//按升序排序,即默认排序


public static void sort(int[] a,int fromIndex,int toIndex){
 rangeCheck(a.length,fromIndex,toIndex);//检验fromIndex到toIndex是否在a的范围内
 DualPivotQucicksort.sort(a,formIndex,toIndex-1,null,0);//默认升序指定范围排序
}//首先检验是否越界，没有越界之后再按照默认升序排序

有关sort()如果参数为Object那么不会采用DualPivotQuicksort，而是采用归并排序
DualPivotQuicksort.sort()这个排序在1.7之前是普通的快排，但是在1.7之后如果对于长度小于27的数组会使用插入排序超过则会使用双轴快速排序

    public static void parallelSort(byte[] a) {//排序,如果数组小于最小并行粒子数即采用双轴快排或者插入排序，如果大于最小并行粒子数即采用归并排序
        int n = a.length, p, g;
        if (n <= MIN_ARRAY_SORT_GRAN ||
            (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
            DualPivotQuicksort.sort(a, 0, n - 1);
        else
            new ArraysParallelSortHelpers.FJByte.Sorter
                (null, a, new byte[n], 0, n, 0,
                 ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
                 MIN_ARRAY_SORT_GRAN : g).invoke();
    }
    
    public static void paralleSort(byte[] a,int fromIndex,int toIndex){
    rangeCheck(a.length,fromIndex,toIndex);//检验是否越界以及是否满足from<to
    int n=toIndex-fromIndex,p,g;
     if (n <= MIN_ARRAY_SORT_GRAN ||
            (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
            DualPivotQuicksort.sort(a, fromIndex, toIndex - 1);
        else
            new ArraysParallelSortHelpers.FJByte.Sorter
                (null, a, new byte[n], fromIndex, n, 0,
                 ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
                 MIN_ARRAY_SORT_GRAN : g).invoke();
    }//当数组长度较小时使用插入排序或者双轴快排，当数组长度大于最小并排粒子数时使用归并排序
    
    
    public static <T extends Comparable<? super T>> void paralleSort(T[] a){
    int n=a.length,p,g;
    if(n<=MIN_ARRAY_SORT_GRAN||(p=ForkJoinPool.getCommonPoolParallelism==1))
    TimSort.sort(a,0,n,NaturalOrder.INSTANCE,null,0,0);
    else{
      new ArraysParallelSortHelpers.FJObject.Sorter<T>
      (null,a,(T[])Array.newInstance(a.getClass().getComponentType(),n), 0, n, 0, ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
                 MIN_ARRAY_SORT_GRAN : g, NaturalOrder.INSTANCE).invoke())
    }
    }//在泛型数组长度较小的时候使用Tim排序，在数组长度较长得时候使用归并排序
    
    
    public static <T extends Comparable<? super T>>
    void parallelSort(T[] a, int fromIndex, int toIndex) {
        rangeCheck(a.length, fromIndex, toIndex);//检验是否越界和from<to
        int n = toIndex - fromIndex, p, g;
        if (n <= MIN_ARRAY_SORT_GRAN ||
            (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
            TimSort.sort(a, fromIndex, toIndex,         NaturalOrder.INSTANCE, null, 0, 0);
        else
            new ArraysParallelSortHelpers.FJObject.Sorter<T>
                (null, a,
                 (T[])Array.newInstance(a.getClass().getComponentType(), n),
                 fromIndex, n, 0, ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
                 MIN_ARRAY_SORT_GRAN : g, NaturalOrder.INSTANCE).invoke();
    }//检验完是否越界和from<to得情况如果数组长度较小那么使用Tim排序如果超过最小并排粒子数那么使用归并排序
    
    
       public static <T> void parallelSort(T[] a, Comparator<? super T> cmp) {
        if (cmp == null)
            cmp = NaturalOrder.INSTANCE;
        int n = a.length, p, g;
        if (n <= MIN_ARRAY_SORT_GRAN ||
            (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
            TimSort.sort(a, 0, n, cmp, null, 0, 0);
        else
            new ArraysParallelSortHelpers.FJObject.Sorter<T>
                (null, a,
                 (T[])Array.newInstance(a.getClass().getComponentType(), n),
                 0, n, 0, ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
                 MIN_ARRAY_SORT_GRAN : g, cmp).invoke();
    }//当数组长度较小时使用Tim排序按照自定义得cmp来排序，如果cmp为空那么将NaturalOrder.INSTANCE赋值给他,当数组长度较长时使用归并排序
    
    
    
    private static void mergeSort(Object[] src,Object dest,int low,int high,int off){
    int length=high-low;
    if(length<INSERTIONSORT_THRESHOLD){
    插入排序
    }
    归并排序
    }//当排序得长度小于7时采用插入排序，否则采用归并排序
    
    
    public static<T> void parallelPrefix(T[] array,BinaryOperator<T> op){
    检验op是否为null
    当array数组长度大于0时并行相加
    }//例如{2,1,0,3}最后结果为{2,3,3,6}
    
    public static int binarySearch(Object [],int fromIndex,int toIndex,Object key){
      判断是否越界
      查找数组中是否有该元素
    }//数组必须已经排序
    
    public static int binarySearch0(Object [],int fromIndex,int toIndex,Object key){
    查找数组 中是否有该元素
    }//数组必须已经排序，和上面得区别在于不检查是否越界
    
    
    public static void fill(int [] a,int fromIndex,int toIndex,int val){
     检查是否越界
     将val赋值给数组a中的[fromIndex,toIndex)中
    }
    
    public static void fill(short[] a,short val){
    将val赋值给数组a中的所有值
    }
    
    public static<T> T[] copyOf(T[] original,int newLength){
       return (T[]) copyOf(original,newLength,original.getClass());
    }
```



## ArrayList

```
public class ArrayList<E> extends AbstractList<E> implements List<E>,RandomAccess,Cloneable,java.io.Serializable{
   private static final long serialVersionUID=8683452581122892189L;//代表序列化数值避免redis冲突
    
   private static final int DEFAULT_CAPACITY=10; //默认容量
   
   private static final Object[] EMPTY_ELEMENTDATA={};//设置空对象
   
   private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA={};//填充数组为空
   transient Object[] elementData;// 数组缓冲区，ArrayList的容量是此数组缓冲区的长度，在添加第一个元素时任何具有elementData==DEAULTCAPACITY_EMPTY_ELEMENTDATA的空Arraylist都将扩展为DEFAULT_CAPACTY即恢复默认容量
   
   private int size;
   
   private volatile int modCount;//int类型的内存可见性的属性，可以被多线程进行修改
   
   public ArrayList(int initialCapacity){
     //如果initialCapacity大于0就初始化为该长度的Object数组,如果initialCapacity为0就初始化空，如果小于0就报错
   }
   
   public ArrayList(){
   this.elementData=DEFAULTCAPACITY_EMPTY_ELEMENTDATA;//初始化一个默认长度的数组
   }
   
   public ArrayList(Collection<? extend E>c){
    elementData=c.toArray();
    if((size=elementData.length)!=0){
       if(elementData.getClass()!=Object[].class){
         elementData=Arrays.copyOf(elementData,size,Object[].class);
       }
    }else{
        this.elementData=EMPTY_ELEMENTDATA;//初始化为空数组
    }
   }//将参数的集合实际元素(并非容量)全部赋值给新ArrayList的elementData如果参数集合为空那么就赋值一个空数组
   
   
   //ArrayList扩展均为超出容量上限之后才会将容量增加原来容量的一半，如果容量为奇数那么一半向下取整比如9扩展之后为13
   
   
   public void trimToSize(){
   modCount++;//修改次数加一
    //将该列表的容量长度变为实际元素长度
   }
   
   public void ensureCapacity(int minCapacity){
     将List扩容到minCapacity
     如果原List不为空则将minCapacity与0比较如果为空则将minCapacity与10比较，如果minCapacity比要比较得值大那么将原数组长度变为1.5倍之后于其比较，如果minCapacity相比原来的1.5倍还要大并且不超过List最大长度Integer_MAX_VALUE-8那么将List长度变为minCapacity否则变成原数组得1.5倍
   }
   
   public int indexOf(Object o){
   返回数组中第一个匹配对象o得下标
   }
   
   public int lastIndexOf(Object o){
   返回数组中最后一个匹配得对象得下标
   }
   
   public E set(int index,E element){
   rangCheck(index);
   
   更新之后返回旧值
   }
   
   public boolean add(E e){
     ensureCapacityInternal(size+1);
     elementData[size++]=e;
     return true;
   }
   
   public void add(int index,E element){
   rangeCheckForAdd(index);
   ensureCapacityInternal(size+1);
   System.arraycopy(elementData,index,elementData,index+1,size-index);
   elementData[index]=element;
   size++;
   //将指定元素插入指定下标中然后该位置得元素和任何后续元素向右移动一格
   
   }
   
   public E remove(int index){
   rangeCheck(index);
   E oldValue=elementData(index);
   
   int numMoved=size-index-1;
   //删除该List种指定位置得元素，将所有后续元素向左移
   }
   
 
   public boolean retainAll(Collection<?> c){
   Objects.requireNonNull(c);
   return batchRemove(c,true);
   //删除原List里面所有不在c中得数据
   }
   
   private boolean natchRemove(Collection<?> c.boolean complement){
     如果complement为true那么该方法将原List删除c中不存在得元素，即只留在集合c中得元素
     如果complement为false那么该方法江源List删除c中存在得元素，即删除在c集合中存在得元素
   }
   
   
   private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{//相比于默认序列化，该方式序列化不会吧null也序列化过去
     int expectedModCount=modCount;//修改次数
     s.defaultWriteObject();//将List里面除了tranist以外得数据序列化
     
     s.writeInt(size);//序列化长度
     //将不为null得部分全部序列化
     for(int i=0;i<size;i++){
     s.writeObjecta(elementData[i]);
     }
   }
   
   
   private class Itr implements Iterator<E>{
      int cursor;//下一个返回元素得下标
      int lastRet=-1;//上一个返回元素得下标，如果不存在那么值为-1
      int expectedModCount=modCount;//修改次数,为了和modCount保持相等，因为ArrayList是线程不安全得所以expectedModCount可以用来判断该list是否已经倍修改过了，如果不相等那么代表其他线程修改了list，判断是否相等根据方法checkForComodification()来进行判断两个值是否相等,基本每个方法执行操作钱都会运行该方法进行检验,如果不相等回抛出ConcurrentModificationException
      
      public boolean hasNext(){return cursor!=size;}//是否有下一个,判定条件为下一个元素得下标是否和size相同
      
      
      public E next(){
      checkForComodification();
      int i=cursor;
      if(i>=size) throw NoSuchElementException();
      Object[] elementData=ArrayList.this.elementData;
      if(i>=elementData.length){
      throw new ConcurrentModificationException();
      }
      cursor=i+1;//当前修改次数+1，指向下一个得下一个得下标
      return (E) elementData[lastRet=i];返回当前得下一个，并将当前得下标赋值给最后一个操作下标得lastRet,用于remove
      }
      
      public void remove(){
        if(lastRet<0){//当最后一次操作得下标小于0得时候
          抛出异常
        }
        checkForComodification();
        try{
          ArrayList.this.remove(lastRet);
          cursor=lastRet;//将cursor得下标变为删除得下标即删除操作处理
          lastRet=-1;//将其初始化为-1,以免能继续删除造成逻辑错误
          expectedModCount=modCount;//为了保证expectedModCount和modCount相同
          
        }
      }
      
      private class ListItr extends Itr implement ListIterator<E>{
       ListItr(int index){
        super();
        cursor=index;//将代表下一元素下标的cursor赋值为构造函数的值即代表从该元素开始迭代
       }
       
       public boolean hasPrevious(){ return cursor!=0;}
       
       public int nextIndex(){return cursor;}
       
       public int previousIndex(){return cursor-1;}
       
       public E previous(){
         返回cursor的上一个元素
       
       }
      }
      //迭代器 listIterator如果没有参数那么返回的对象为cursor=0,lastRet=-1,expectedModCount为当前modCount的值
      
   }
   
   
   
}
```

  

### ArrayList总结

sort()排序方法：如果长度大于2^13那么会采用并排排序，如果排序数据类型为Object那么不会采用DualPivotQuicksort，而是采用归并排序,如果不为Object数据类型那么如果长度小于27使用的是插入排序,否则使用双轴插入排序,DualPivotQuicksort()排序在1.7之前是普通的快排1.7之后采用了这种方式来加快排序速度

paralleSort(byte[] )排序:如果长度较小那么会使用DualPivotQuicksort()即会采用插入排序或者双轴快排，如果长度大于最小并排长度即2^13即使用归并排序

parallelSort(T[] )排序,如果长度较小会使用Tim排序，如果长度大于最小并排长度2^13会使用归并排序

ArrayList中 size代表当前List中元素的个数,elementDate.length()代表容量长度，elementDate代表容量,modCount是一个多线程可见性变量代表该List的修改次数每次进行操作之后该值都会+1并且是内存可见性的



ArrayList中只有当size=elementDate.length()即**把当前容量装满之后才会触发扩容，容量增加原来的一半并且如果一半的值为小数那么向下取整比如9扩容之后为13=9+4**

**ArrayList初始化如果没有指定参数那么会创建一个默认空容积的对象，如果原来为默认空容积对象那么第一次扩容直接扩容为容量为10得对象,如果构造方法指定了参数那么会生成对应数量容量的对象,如果指定为0那么会创建一个容量为0得对象,如果该对象增加了一个需要进行扩容那么原来是0现在变为1，然后正常操作,如果原来得容量为1那么扩容之后变为2，即最少扩容+1；**

**ArrayList使用数组存取**,他拥有迭代器Iterator, **有关Iterator如果使用无参构造方法构造出来的对象会使得lastRet=-1,cursor=0,expectedModCount=modCount    其中lastRet代表迭代器最后一次所取得元素得下标,cursor代表迭代器下一次所取得元素得下标,expectedModCount是由于ArrayList不是线程安全得所以该List可能在迭代得时候被多个线程进行了修改,为了保证数据得安全性每次迭代器操作都会先调用方法checkForComodification()检验expectedModCount是否和modCount相等,如果不相等会直接爆出异常**

**如果迭代器得初始化使用了带参构造方法那么表示迭代器将从该下标开始迭代,即cursor=参数，lastRet=-1,expectedModCount=modCount;**



## LinkedList

```
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>,Deque<E>,Cloneable,java.io.Serializable{
   transient int size=0;
   /**
   *头指针
   **/
   transient Node<E> first;
   /**
   *尾指针
   **/
   transient Node<E> last;
   /**
   *无参构造函数，即为空
   **/
   public LinkedList(){
   }
   
   public LinkedList(Collection<? extend E>c){
   this();//调用无参构造方法
   addAll(c);//将集合c全部加入,如果指定集合为null那么会爆出NullPointException
   }
  
   private static class Node<E>{
    E item;
    Node<E> next;
    Node<E> prev;
    
    Node(Node<E> prev,E element,Node<E> next){//构造函数,将E变为该节点得值并指定其父节点和子节点
     this.item=element;
     this.next=next;
     this.prev=prev;
    }
   }
   
   private void linkFirst(E e){
    final Node<E> f=first;
    final Node<E> newNode=new Node<>(null,e,f);
    first=newNode;
    if(f==null)
      last=newNode;
    else
      f.prev=newNode;
    size++;
    modCount++;
   }
  
   void linkLast(E e){
     final Node<E> l=last;
     final Node<E> newNode=new Node(l,e,null);
     last=newNode;
     if(l==null)
       first=newNode;
     else
       l.next=newNode;
   }
   
   void linkBefore(E e,Node<E> succ){
     final Node<E> pred=succ.prev;
     final Node<E> newNode=new Node<>(pred,e,succ);//将e插入再succ前面原succ父节点得后面，即插入succ与其父节点之间
     succ.pre=newNode;
     if(pred==null){
     first=newNode;
     }
     else{
     pred.next=newNode;
     }
     size++;
     modCount++;
   }


   private E unlinkFirst(Node<E> f){//用于删除头结点并返回头结点得值
     final E element=fi.item;
     final Node<E> next=f.next;
     f.item=null;
     f.next=null;//帮助GC垃圾回收
     first=next;
     if(next==null){
       last=null;
     }
     else{
     next.prev=null;
     }
     size--;
     modCount++;
     return element;
   }
   
   private E unlinkLast(Node<E> f){//用于删除头结点并返回头结点得值
     final E element=f.item;
     final Node<E> prev=f.prev;
     f.item=null;
     f.next=null;//帮助GC垃圾回收
     last=prev;
     if(prev==null){
       first=null;
     }
     else{
     prev.next=null;
     }
     size--;
     modCount++;
     return element;
   }
   
   E unlink(Node<E> x){//在原List中删除该节点
     final E element=x.item;
     final Node<E> next=x.next;
     final Node<E> prev=x.prev;
     if(prev==null){//说明当前x为first
       first=next;
     }else{
     prev.next=next;//跨过x进行连接
     x.prev=null;
     }
     
     if(next==null){//说明当前x为last
       last=prev;
     }else{//跨过x进行连接
     next.prev=prev;
     x.next=null;
     }
     
     x.item=null;
     size--;
     modCount++;
     return element;
   }
   
   public boolean remove(Object o){
    如果o为null,那么就遍历整个List删除第一个值为null得结点然后返回true;
    如果o不为null那么就遍历整个List删除第一个值为o得结点然后返回true;
    如果遍历完整个List之后都没有找到那么就返回false
   }
   
   Node<E> node(int index){
    //返回相当于对应下标得List中得结点
   }
   
   E get(int index){
    检查index是否会越界
    return node(index).item;
   }
   
   public void add(int index,E element){
      检查index是否会越界
      if(index==size){//末尾之后插入
      linkLast(element);
      }
      else{
       linkBefore(element,node(index));//将插入index位置得结点于其父节点之间，即插入到index得位置
      }
   }
   
   public E peek(){
     返回头结点得值,如果头结点为null那么就返回null
   }
   
   public E poll(){
    删除头结点并返回头结点得值,当头结点为空时返回null
   }
   
   public E pop(){
    删除头结点并返回头结点得值,当头结点为空时是抛出异常
   }
   
   public E remove(){
    删除头文件并返回头结点得值
   }
   
   public boolean offer(E e){
     return add(e);//加入到末尾
   }
   
   
   private class ListItr implements ListIterator<E>{
      //LinkedList中得迭代器与ArrayList得迭代器得区别在于,ArrayList中得迭代器是lastRet和cursor分别代表最后一次取到得下标cursor代表下一次取到得下标,这里使用了lastRetuened和next代表最后一次取到得结点和下一次取到得结点,nextIndex代表下一次取得结点对应List得下标,expectedModCount意义和原来相同
      private Node<E> lastReturned;
      private Node<E> next;
      private int nextIndex;
      private int expectedModCount=modCount;
   }
}

```

### LinkedList总结

```
LinkedList使用链表进行连接，它依然有modCount这个继承而来的属性，它的add,remove分别是在链表尾和链表头进行增加和删除

它拥有first和last都为内置类Node类型的，分别代表链表的头指针和尾指针，有关Node的属性包含prev,next,element分表代表该结点的上一个结点和下一个结点，如果是头结点那么它的prev为null,last也同样如此它的next也为null

LinkedList拥有linkFirst,linkLast,linkBefore,unlinkFirst,unlinkLast,unlink,这些方法来支持LinkedList中的add和remove方法,其中add(E e)调用的为linkLast,remove(E e)调用的为unlinkFirst

LinkedList的迭代器和ArrayList的迭代器不同,它的迭代器拥有lastReturn,next这两个数据类型为Node的属性，分别代表最后一次取到的值和下一次取到的值,也有nextIndex代表下一次取得元素得下标，expectModCount和ArrayList中得expectModCount相同都是因为LinkedList不是线程安全得所谓为了保证迭代器得数据安全每次迭代操作都要验证其expectModCount和modCount是否相同即如果不相同说明这个时间段之间有其他线程对该LinkedList对象进行了操作无法保证线程安全即停止迭代器得操作
```

## LinkedList与ArrayList之前得相同点与不同点

```
相同点:都是顺序储存，都拥有迭代器，都是List得实现类,都拥有modCount这个线程可见性属性
最大容量为INT_MAX-8不能超过这个容量

不同点:ArrayList使用数组存储，LinkedList使用链表存储;LinkedList使用内置类Node来进行存储

ArrayList有容量和扩容操作，当容量满了之后会扩容到原来得1.5被向下取整比如9扩容完为13，LinkedList没有容量和扩容操作


```





   