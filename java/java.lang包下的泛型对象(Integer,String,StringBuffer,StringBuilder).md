# java.lang包下的泛型对象(Integer,String,StringBuffer,StringBuilder)

## Integer和int的区别

Integer是int的包装类，是一个泛型类，而int是java的一种基本数据类型

Integer需要实例化后才能使用而int不需要

Integer的引用实际上是对象的引用而int是直接存储数值

Integer的默认值是null，int的默认值是0



Integer对象之间的比较，就算两个Integer对象的值相等但是其实上是两个不同的变量内存地址不同所以两者是不相等的

Integer 和int之间的比较只要两个变量的值是相等的那么就为true,实际上是Integer在int比较时java会自动拆包成为int,然后进行比较，实际上就是两个int的比较



非new生成的Integer对象和new生成的Integer对象比较时，就算值相同结果也为false,因为Integer对象的值在-128-127之间的情况下非new出来的对象是直接在java常量池中取值，而new出来的对象还是在堆中新建对象，所以两者不是用一个内存地址比较为false,如果超出-128-127这个范围的非new出来的对象会内部变成new的一个对象但是和两个Integer比较相同两个对象内存对象不相同所以比较为false,同样两个非new出来的Integer对象如果都在-128-127之前的区域如果值相同比较为true

## String StringBuffer String Builder三者区别

```
private final char value[];//代表String由一个char数组搞定

private int hash;//默认值为0

public String(){this.value=new char[0];}

public String(String original){
   this.value=original.value;
   this.hash=original.hash;
}

public String(char value[]){
   this.value=Arrays.copyOf(value,value.length);//转换成一个新的对象，内存地址不同
}


public String(int[] codePoints,int offset,int count){//将一个int数组的元素当作ASCII码从offset下标开始一共count个元素转换成字符赋值给String对象
   .........
   进行越界判定
   .........
   
   final int end=offset+count;
   
   int n=count;
   for(int i=offset;i<end;i++){
      int c=codePoints[i];
      if(Character.isBmpCodePoint(c))
         continue;
      else if(Character.isValidCodePoint(c))
         n++;
      else throw new IllegalArgumentException(Integer.toString(c));   
   }
   
   final char[] v=new char[n];
   for(int i=offset,j=0;i<end;i++,j++){
       int c=codePoints[i];
       if(Character.isBmpCodePoint(c))
         v[j]=(char)c;
       else
         Character.toSurrogates(c,v,j++);
   }
   this.value=v;
}

public String(StringBuffer buffer){
    synchronized(buffer){//运用synchronized关键字锁定该StringBuffer参数,保证线程安全
      this.value=Arrays.copyOf(buffer.getValue(),buffer.length());
    }
}

public boolean equals(Object anObject){//判断两个String的值是否相等如果不是String类型直接返回false
      if(this==anObject) return true;//如果直接相等就直接返回
     if(anObject instanceof String){//如果这个参数是String才进行下面的判断
      String anotherString=(String)anObject;
      int n=value.length;
      if(n==anotherString.value.length){//判断长度
        char v1[]=value;
        char v2[]=antherString.value;
        int i=0;
        while(n--!=0){
           if(v1[i]!=v2[i])
             return false;
           i++;  
        }
        return true;
     }
     }
     return false;
}

public int compareTo(String anotherString){//比较两个String的值是否相等，如果不相等返回两者的ASCII码只差，如果相等返回0
    int len1=value.length;
    int len2=anotherString.value.length;
    int lim=Math.min(len1,len2);
    char v1[]=value;
    char v2=anotherString.value;
    
    int k=0;
    while(k<lim){
     char c1=v1[k];
     char c2=v2[k];
     if(c1!=c2){
       return c1-c2;
     }
     k++;
    }
    return len1-len2;
}

public int hashCode(){
   int h=hash;
   if(h==0&&value.length>0){
     char val[]=value;l
     
     for(int i=0;i<value.length;i++){
       h=31*h+val[i];
     }
     hash=h;
   }
   return h;
}
```



