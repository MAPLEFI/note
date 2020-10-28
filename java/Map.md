# Map



## Map

```
int size();

boolean isEmpty();

boolean containsKey(Object key);//查询Map中是覅偶含有该Key

boolean containsValue(Object value);//查询在MaP中是否拥有该value值

V get(Object K);//获得对应key的value值

V put(K key,V value);//将这一对Key Value值插入Map中

V remove(Object key);//移除该key对应的键值对

void putAll(Map<? extends K,? extends V> m);//将参数中的所有键值对加入到该Map对象中

void clear();//清空Map

Set<K> keySet();//将该Map中的Key转到Set中

Collection<V> values();//将该Map中的values值放入一个集合中返回

Set<Map.Entry<K,V>> entrySet();//将Map中的键值对全部加入到Set中

interface Entry<K,V>{//表示Map中的一对键值对
   K getKet();
   
   V getValue();

   V setValue(V value);//设置新的值替换原有Map中的值
   
   boolean equals(Object o);//比较两个Entry是否相同,通过比较该键值对的Key和Value的hash值 e.getKey()==null ?0:e.getKey().hashcode();e.getValue()==null?0:e.getValue().hashcode()
   会比较两个键值对的Key和Value的hash值是否相等来判断两个键值对是否相等
   
   int hashCode();返回该Entry的键值对，一半来说为
   (key==null ? 0 : key.hashcode())^(value==null ? 0 : key.hashcode())
   
   public static <K extends Comparable<? super K>,V> Comparator<Map.Entry<K,V>>comparingByKey(){
   return (Comparator<Map.Entry<K,V>>&Serializable)
       (c1,c2)->c1.getKey().compareTo(c2.getKey());
   }//返回一个比较器可以按照顺序排序比较Entry的value
}



default V getOrDefault(Object key,V defaultValue){
     V v;
     return (((v=get(key)!=null)||containsKey(key)))? v :defaultValue   
}//如果当前key在Map中有映射那么返回它的对应值value,如果该key不存在映射或者key所对应的值为null那么返回参数的defaultValue
   
default void forEach(BiConsumer<? super K,> super V>action){
Object.requireNonNull(action);
for(map.Entry<K,V>:entrySet()){
  K k;
  V v;
  try{
   k=entry.getKey();
   v.entry.getValue();
  }catch(IllegalStateException ise){
    throw new ConcurrentModificationException(ise);
  }
}
   action.accept(k,v);
}//遍历该Map中的每一个Entry键值对执行action操作，知道全部按顺序执行完毕或者报错为止
   
   
 default void replaceAll(BiFunction<? super K,? super V,? extend V>function){
 Objects.requireNonNull(function);//检验该处理函数是否为null
 for(Map.Entry<K,V> entry :entrySet()){
  K k;
  V v;
  try{
    k=entry.getKey();
    v=entry.getValue();
  }catch(IllegalStateException){
    throw new ConcurrentModificationException(ise);
  }
  v=function.apply(k,v);
  try{
     entry.setValue(v);
  }catch(IllegalStateException ise){
    throw new ConcurrentModificationException(ise);
  }
 } 
} //将Entry中每个v进行function函数操作之后替换原来的entry中的值 
   
   default V putIFAbsent(K key,V value){
     V v=get(key);
     if(v==null){
      v.put(key,value);
     }
   }//如果该Map中不存在key这个键，或者key所对应的值为null的时候就会将参数的更新入Map中 
}
```

## HashMap

```
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>,Cloneable,Serializable{
   private static final long serialVersionUID=362498820763181265L;
   
   static final int DEFAULT_INITIAL_CAPACITY=1<<4; //初始默认容量为16，容量必须为2的幂
   
   static final int MAXIMUM_CAPACITY=1<<30;//最大容量为2的30次方，如果有带参的构造函数想要创造一个容量比这个大的容量的对象，那么会默认变为最大容量即2得30次方
   
   static final float DEFAULT_LOAD_FACTOR=0.75f;//负载因数0.75，指当使用量达到容量得百分之75得时候就进行扩充，将其容量乘2
   
    static final int TREEIFY_THRESHOLD=8;//桶得树化阈值为8，表示如果链表使用量超过8就会变为红黑树
    
    static final int UNTREEIFY_THRESHOLD=6;//桶得链表还原阈值为6，表示红黑树结点如果小于6就会从红黑树塌陷为数组(List)    
    
    static final int MIN_TREEIFY_CAPACITY=64;//最小树化容量阈值，当哈希表中得容量大于该阈值得时候才允许树化链表，即容量大于64时才允许使用量大于8得时候从链表转换为红黑树，否则桶内元素太多时选择得是直接扩容而不是树化，为了避免进行扩容，树化选择冲突这个值不能小于4*TREEIFY_THRESHOLD=32,即如果从桶转化为树那么最少需要容量为64，如果已经为红黑树那么在容量大于32得情况下还是可以继续树化
    
    static class Node<K,V> implements Map.Entry<K,V>{//Map.Entry为Map中得键值对，Node为HashMap中得结点可以看作桶
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        
        Node(int hash,K key,V value,Node<K,V> next){
          this.hash=hash;
          this.key=key;
          this.value=value;
          this.next=next;
        }
        
        public final K getKey() { return key;}
        public final V getValue() {return value;}
        public final String toString() { return key+"="+value}
        
        public final int hashCode() {return Object.hashCode(key)^Object.hashCode(value);}
        
        public final V setValue(V newValue){
          V oldValue=value;
          value=newValue;
          return oldValue;
        }
        
        public final boolean equals(Object o){
           if(o==this) return ture;
           if(o instannceof Map.Entry){//如果比较对象是一个Map得键值对那么就比较值是否相等
                Map.Entry<?,?> e=(Map,Entry<?,?>)o;
                if(Object.equals(key,e.getKey())&&Object.equals(value,e.getValue()))
                return true;
           }
           return false;
        }
    }
    
    static final int hash(Object key){
     int h;
     return (key==null)? 0 : (h=key.hashCode())^(h>>>16);//采用了扰动函数使得原来不常使用得高位和低位平衡
    }
    
     
      static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }//返回一个能cap装下得2得次幂得最小容量,比如7返回值为8,5得返回值也为8,不能超过最大容量1<<30,超过即返回最大容量
    
    transient Node<K,V>[] table;//该表在首次使用时初始化，并根据需要调整大小，分配时长度始终是2得幂（在某些操作中我们也允许长度为0）,table为HashMap得桶容量
    
    transient Set<Map.Entry<K,V>> entrySet;//键值对集合
    
    transient int size;//HashMap中存放KV得数量
    
    transient int modCount;//修改次数，用于迭代器，因为线程不安全所以用法上和ArrayList中得相同
    
    int threshold;//表示当HashMap得size大于threshold时会执行resize操作,即为扩容因子,未初始化时为0，一半情况下为容量*loadFactor
    
    final float loadFactor;//哈希表中得负载因子,计算为size/capacity而不是占用桶数量出去capacity
    
    public HashMap(int initialCapacity,float loadFactor){
         if(initialCapacity<0) throw new IllegalArgumentException();//设置初始长度小于0就报错
         if(initialCapacity> MAXIMUM_CAPACITY) initialCapacity=MAXIMIM_CAPACITY;//设置初始长度如果大于最大长度就将其变为最大长度1<<30
         if(loadFactor<=0||Float.isNaN(loadFactor)) throw new IllegalArgumentException();//如果负载因子小于等于0或者不是一个Float那么就报异常
         
         this.loadFactor=loadFactor;//更新负载因子
         this.threshold=tableSizeFor(initialCapacity);//设定扩容因子得值为比initialCapacity大得最近得一个2得幂次方得数
    }//为了避免多次得resize()浪费性能，比如需要开一个1000容量得HashMap,但是1000*0.75<1000,为了让size*0.75>1000就让其开2048个空间最佳
    
    
    public HashMap(int initialCapacity){ this(initialCapacity,DEFAULT_LOAD_FACTOR)
    }//即初始化一个容量能包含initialCapacity得2得幂得值，给负载因子赋值为默认值0.75
    
    public HashMap(Map<? extends K,? extends V>m){
      this.loadFactor=DEFAULT_LOAD_FACTOR;//负载因子为默认值
      putMapEntries(m,false);//将m得键值对加入到里面
    }
    
    final void putMapEntries(Map<? extends K,? extends V>m,boolean evict){
      int s=m.size();
      if(s>0){
          if(table==null){//pre-size  当前桶容量为空		
           float ft=((float)s/loadFactor)+1.0F;
           int t=((ft<(float)MAXIMUM_CAPACITY)? (int)ft:MAXIMUM_CAPACITY);
           if(t>threshold) threshold=tableSizeFor(t);//设定扩容因子为比m得长度得4/3+1大得最近得一个2得幂次方       
           }
          else if(s>threshold) resize();//如果当前容器里有元素，并且m得长度大于了扩容因子那么就进行扩容
          
          for(MapEntry<? extends K,? extends V>e:m.entrySet()){
            K key=e.getKey();
            V value=e.getValue();
            putVal(hash(key),key,value,false,evict);
          }
      }
    }
    
    final Node<K,V>[] resize(){//扩容方法,非常消耗性能所以尽量初始化得时候让其尽量避免resize
    
      Node<K,V>[] oldTab=table;//table为该对象当前得桶数组
      int oldCap=(oldTab==null)? 0 :oldTab.length;//当前对象桶容量
      int oldThr=threshold;//threshold为该对象扩容因子
      int newCap,newThr=0;
      if(oldCap>0){//原桶容量大于0
        if(oldCap>=MAXIMUM_CAPACITY){//已经到达最大容量不能继续扩容了直接返回
           threshold=Integer.MAX_VALUE;
           return oldTab;
        }
        //如果原桶容量大于0并且没有达到最大容量先将新容量变为原容量得两倍
        else if ((newCap=oldCap<<1)<MAXIMUM_CAPACITY&&oldCap>=DEFAULT_INITIAL_CAPACITY)//新容量变为原来元素个数得两倍并且小于最大容量，并且原来元素个数大于默认容量16，即超过默认容量并且扩容之后没有超出最大容量
        newThr=oldThr<<1;//扩容因子变为原来得两倍
      }
      
      else if(oldThr>0)  newCap=oldThr;//原来得容量为0，但是扩容因子不为0，将原扩容因子当成新的容量
      
      else{//如果原来得元素个数为0，并且原来得扩容因子也为0，即完全初始化得状态
         newCap=DEFAULT_INITIAL_CAPACITY;//新的容量为默认容量16
         newThr=(int)(DEFAULT_LOAD_FACTOR*DEFAULT_INITIAL_CAPCITY);//16*0.75f=12
      }
      
      if(newThr==0){//如果新的扩容因子为0，只有当原来得元素个数已经超过最大容量或者原来得元素个数为0但是扩容因子不为0得情况下满足newThr==0
        float ft=(float)newCap*loadFactor;
        newThr=(newCap<MAXIMUM_CAPACITY&&ft<(float)MAXIMUM_CAPACITY?(int)ft:Integer,MAX_VALUE);//将其更新为新容量*负载因子
      }
      threshold=newThr;//更新扩容因子，新的扩容因子只可能为，容量最大值(原值已经为容量最大值)，默认容量*默认负载因子(初始化得情况)，新容量*当前负载因子(原扩容因子不为0但原容量为0)
      Node<K,V>[] newTab=(Node<K,V>[])new Node[newCap];
      table=newTab;//更新容量,新容量只有可能为最大容量(原值已经为最大容量,原值两倍之后超过最大容量),默认容量16(初始化情况)，原容量得两倍(原值不为0)，原扩容因子(原值为0但是原扩容因子不为0)
      
      if(oldTab!=null){//原容量不为0
          for(int j=0;j<oldCap;++j){
           Node<K,V> e;
           if((e=oldTab[j])!=null){
             oldTab[j]=null;
             if(e.next==null) 
              newTab[e.hash&(newCap-1)]=e;
             else if(e instanceof TreeNode)//如果已经树化调用数得split方法
               ((TreeNode<K,V>)e).split(this,newTab,j,oldCap);
             else{
               Node<K,V> loHead=null,loTail=null;
               Node<K,V> hiHead=null,hiTail=null;
               Node<K,V> next;
               do{
                 next=e.next;
                 if((e.hash&oldCap)==0){
                   if(loTail==null)
                     loHead=e;
                   else
                     loTail.next=e;
                 }
                 else{
                    if(hiTail==null)
                      hiHead=e;
                    else
                      hiTail.next=e;
                    hiTail=e;  
                 }
               }while((e.next)!=null);
               if(loTail!=null){
                loTail.next=null;
                newTab[j]=loHead;
               }
               if(hiTail!=null){
                  hiTail.next=null;
                  newTab[j+oldCap]=hiHead;
               }
             }  
           }
          }
      }
      return newTab;
    }
    
    public V put(K key,V value){return putVal(hash(key),key,value,false,true); }//put方法,调用putVal方法并且设定如果存在Key那么将对应得Value进行更新，然后将其加入链表尾部
    
    
    public V putVal(int hash,K key,V value,boolean onlyIfAbsent,boolean evict){
      Node<K,V>[] tab;Node<K,V>p;int n,i;
      if((tab=table)==null||(n=tab.length)==0) //容量为0或者在初始化状态
        n=(tab=resize()).length;//初始化扩容,n为其桶容量
      if((p=tab[i=(n-1)&hash])==null)//如果容量-1和hash得与运算结果得下标为null那么就将其放到这个下标下面
        tab[i]=newNode(hash,key,value,null);
      else{//如果该下标不为null
         Node<K,V> e; K k;
         if(p.hash==hash&&((k=p.key)==key||(key!=null&&key.equals(k))))
           e=p;//如果该桶得起始点和参数得值相同并且hash值也相同那么就将桶得起始点赋值给e,此处得p由于上一个p得作用该p是桶首结点
         else if (p instanceof TreeNode)//如果和起始点不相同并且是一个树结点那么直接插入到树结点中
           e=((TreeNode<K,V>)p).putTreeVal(this,tab,hash,key,value);
           
         else{//如果参数和起始点不同并且不为树结点
             for(int binCount=0; ;++binCount){
                 if((e=p.next)==null){//如果下一个结点为null
                     p,next=newNode(hash,key,value,null);//新建结点添加到其后面
                     if(binCount>=TREEIFY_THRESHOLD-1)//如果已经大于了树化阈值那么就进行树化
                         treeifyBin(tab,hash);//将该hash值所对应得桶进行树化
                     break;    
                 }
                 
                 if(e.hash==hash&&((k=e.key)==key||(key!=null&&key.equails(k))))//如果该节点和参数得值相等那么就直接break
                 break;
                 
                 p=e;//每次for循环之后将p置换为它原来得下一个结点
             }
         }
         
         if(e!=null){//成功插入结点或者找到了和他相同得结点
           V oldValue=e.value;
           if(!onlyIfAbsent||oldValue==null)//如果原来得值为空或者不是只有一个存在就不能修改得情况下进行值得更新
             e.value=value;
           afterNodeAccess(e);  
           return oldValue;
         }
      }  
      ++modCount;
      if(++size>threshold)
         resize();
      afterNodeInsertion(evict);   //模板方法设计模式，在父类中可以有默认实现
    }
    
    public V remove(Object key){//通过调用removeNode进行删除,删除节点之后拼接断点
      Node<K,V>e;
      return (e=removeNode(hash(key),key,null,false,true))==null? null:e.value;
    }
    
    
    final Node<K,V> removeNode(int hash,Object key,Object value,boolean matchValue,boolean movable){
       Node<K,V>[] tab;Node<K,V> p;int n,index;
       if((tab=table)!=null&&(n=tab.length)>0&&(p=tab[index=(n-1)&hash])!=null){//赋值并且该hash值对应得桶结点不为空,将p得值赋值为该桶节点的首结点
         Node<K,V>node=null,e; K k;V v;
         if(p.hash==hash&&((k=p.key)==key||(key!=null&&key.equals(k))))//如果p结点就是要找的结点
           node=p;//将p赋值给node结点
         else if((e=p.next)!=null){//如果首结点不是要找得结点那么就往下找，并将首结点得下一个结点赋值给e
             if(p instanceof TreeNode)//如果是树状结点那么就使用树方法找到该结点
               node=((TreeNode<K,V>)p).getTreeNode(hash,key);
             else{//一直往下找，直到找到key相等得或者将这个桶找完都没有找到
                do{
                   if(e.hash==hash&&((k=e.key)==key||(key!=null&&key.equals(k))){
                     node=e;
                     break;
                   }
                   p=e;
                }while((e=e.next)!=null)
             }  
         }  
         if(node!=null&&(!matchValue||(v=node.value)==value||(value!=null&&value.equals(v)))){//如果node不为null并且(值相等或者matchValue=false)
            
            if(node instanceof TreeNode)
              ((TreeNode<K,V>)node).removeTreeNode(this,tab,movable);
            else if(node==p)
               tab[index]=node.next;
            else
               p.next=node.next;
               
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
         }
       }
       return null;
    }
    
    public void clear(){
       Node<K,V>[] tab;
       modCount++;
       if((tab=table)!=null&&size>0){
          size=0;
          for(int i=0;i<tab.length;++i){
             tab[i]=null;
          }
       }
    }
    
    abstract class HashIterator{
      Node<K,V> next;//下一个结点
      Node<K,V> current;//当前结点
      int expectedModCount;//期忘修改次数，由于HashMap是线程不安全得所以加上这个属性用来验证每次操作时有无别的线程对该对象造成了操作，如果别的线程修改了该对象那么该迭代器得功能就不会使用以确保该对象数据安全
      int index;//当前桶位于桶数组得下标
      
      HashIterator(){
         expectedModCount=modCount;
         Node<K,V> t=table;
         current=next=null;
         index=0;
      if(t!=null&&size>0){//如果下一个结点为null但是桶数组不为null,那么就往后面得桶挪直到不为null或者全部挪完也为null位置
          do{} while(index<t.length&&(next=t[index++])==null);
      }
     }
     
     public final boolean hasNext(){ return next!=null; }
     
     final Node<K,V> nextNode(){
        Node<K,V>[] t;
        Node<K,V> e=next;
        if(modCount!=expectedModCount) throw new ConcurrentModificationException();
        
        if(e==null) throw new NoSuchElementException();
        
        if((next=(current=e).next)==null&&(t=table)!=null){//如果下一个结点为null但是桶数组不为null,那么就往后面得桶挪直到不为null或者全部挪完也为null位置
           do{}while(index<t.length&&(next=t[index++])==null);
        }
        return e;
     }
     
     public final void remove(){
        Node<K,V>p=current;
        if(p==null) throw new IllegalStateException();
        if(modCount!=expectedModification) throw new ConcurrentModifivatioException();
        
        current=null;
        K key=p.key;
        removeNode(hash(key),key,null,false,false);
        expectedModCount=modCount;//因为已经通过了上面的modCount得检验那么为了预防在下面得环节有线程进入进行了修改但是又不能停止该操作，所以在操作完成之后更新expectedModCount
     }
    
    }
    
    
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V>{
        TreeNode<K,V> parent;
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;
        boolean red;
        TreeNode(int hash,K key,V val,Node<K,V> next) {super(hash,key,val,next);}
    }
}
```

### HashMap总结

```
HashMap通常使用Node桶结点来存取，在满足树化阈值和树化容量阈值得条件下会将原来得桶从数组得形式变成红黑树TreeNode得形式，树化阈值为8,树化容量阈值为64，如果put所调用得putVal方法插入完成之后判断该桶是否已经使用了大于等于8个结点，如果是那么调用treeifyBin方法进行桶树化，treeifyBin方法树化得是putVal中得hash值所对应得桶，treeifyBin进去之后首先判断目前HashMap得容量是否达到树化容量阈值，
如果没有达到就进行扩容,扩容操作为resize(),作用将容量扩大两倍或者变为默认初始容量但是不能超出最大容量，并且随着容量改变而改变对应得thresold值即扩容因子，一般为table.length*loadFactor,并且将原来得元素重新赋值到新的容量里面
如果达到了树化容量阈值和树化阈值那么就将该桶从单链表改编成为一个红黑树

数组化:通过resize()方法里面调用split方法来进行将原来的元素赋值给新容量，在此时检测当原来已经为树的情况下如果当前桶的使用量已经<=6，时会红黑树变为数组


HashMap默认容量为16,ArrayList得默认容量为10


属性说明：table桶数组,size该对象所使用得所有结点数量(并不一定等于使用得桶得数量，大部分情况下会超过)，loadFactor负载因子用于扩容，threshold扩容因子一般情况下等于loadFactor*table.length,当size>threshold得情况下会触发扩容方法，一般情况下如果为默认值使用得size>8得情况下会将原对象得容量从16扩充到64，由于resize()扩容操作非常消耗内存和耗时，所以需要尽量避免resize()操作，比如需要一个可以容纳1000得HashMap对象如果直接初始化1000会变为一个容量为1024得对象但是loadFactor*table.length=768,也就是说当size超过768得时候就需要扩容了，为了避免这个情况所以需要初始化一个table.length*loadFactor=1000即初始化一个容量为1334即初始化容量为2048的对象



HashMap中的put和remove都是分别调用了putVal和removeVal方法，put方法会在key的hash值下面找寻空结点或者和他相同的结点进行赋值操作，removeVal会通过key的hash值来寻找对应的桶来寻找到需要删除的点进行和三处并且链接该桶断裂的两点


get方法实际调用的是getNode方法，同样式通过key和key的hash值进行查询,getNode会遍历对应桶找到对应节点进行返回，如果返回的不为null那么get方法就取该节点的value值

Hash值内的迭代器HashIterator 其中有属性current,next,index,expectedModCount分别代表当前节点，下一个节点，当前桶的下标，和期忘修改次数，和以往的迭代器相同由于线程不安全所以使用expectedModCount来保证数据安全性，其中getnext()和初始化方法均会在末尾进行下一节点的判断如果下一节点next为null那么会转移到下一个桶继续判断是否为空，直到匹配到一个不为空的节点或者整个对象的桶数组都匹配完成之后都没有那就为null
```

## LinkedHashMap

```
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>{
     static class Entry<K,V> extends HashMap.Node<K,V>{
       Entry<K,V> before,after;
       Entry(int hash,K key,V value,Node<K,V> next){ super(hash,key,value,next);}//继承了HashMap中的Node节点但是构建成了一个双向链表,原Node中有 K V hash next
     }
     /**
     头结点
     */
     transient LinkedHashMap.Entry<K,V> head;
     /**
     尾节点
     */
     transient LinkedHashp.Entry<K,V> tail;
     
     final boolean accessOrder;
     
     private void linkNodeLast(LinkedHashMap.Entry<K,V> p){
        LinkedHashMap.Entry<K,V> last=tail;
        tail=p;
        if(last==null)
          head=p;
         else{
          p.before=last;
          last.after=p;
         }  
     }
     
     private void transferLinks(LinkedHashMap.Entry<K,V> src,LinkedHashMap.Entry<K,V> dst){//将src的链接应用于dst,相当于用dst替换了src
      LinkedHashMap.Entry<K,V> b=dst.before=src.before;
      LinkedHashMap.Entry<K,V> a=dst.after=src.after;
      if(b==null) head=dst;
      else b.after=dst;
      if(a==null) tail=dst;
      else a.before=dst;
     }
     
     Node<K,V> replacementNode(Node<K,v> p,Node<K,V>next){//主要用于树化
        LinkedHashMap.Entry<K,V> q=(LinkedHashMaq.Entry<K,V>)p;
        LinkedHashMap.Entry<K,V> t=new LinkedHashMapa.Entry<K,V>(q.hash,q.key,q.value,next);
        transferLinks(q,t);
        return t;
     }
     
     TreeNode<K,V> replacementTreeNode(Node<K,V>p,Node<K,V> next){//主要用于去树化
        LinkedHashMap.Entry<K,V>q=(LinkedHashMap.Entry<K,V>)p;
        TreeNode<K,V> t=new TreeNode<K,V>(q.hash,q.key,q.value,next);
        transferLinks(q,t);
        return t;
     }
     
     void afterNodeRemoval(Node<K,V> e){//用于删除节点后处理其对应链表前后关系
       LinkedHashMap.Entry<K,V>p=(LinkedHashMap.Entry<K,V>)e,b=p.before,a=p.after;
       p.before=p.after=null;
       if(b==null) head=a;
       else b.after=a;
       
       if(a==null) tail=b;
       else a.before=b;
     }
     
     public boolean removeEldestEntry(Map.Entry<K,V> eldest){
        return size()>capacity;
     }
     
     void afterNodeInsertion(boolean evict){
        LinkedHashMap.Entry<K,V> first;
        if(evict&&(first=head)!=null&&removeEldestEntry(first)){//必须参数为true并且超出容器时才起作用，这时候移除最老的首结点
          K key=first.key;
          removeNode(hash(key),key,null,false,true);
        }
     }

}
```

### LinkedHashMap总结

```
双向链表，放弃了HashMap的散列表
```

