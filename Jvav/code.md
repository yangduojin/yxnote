# 代码展示

## Iterable 和 Iterator

```java
import java.util.Arrays;
import java.util.Iterator;//    迭代器
public class YangTtr<T> implements Iterable  {
    private static final Integer default_capacity = 10;
    private int endindex = 0;
    private Object[] elems;
    public YangTtr(){ this.elems = new Object[default_capacity]; }

    public T[] add(T t){
         if (elems.length - 1 < endindex){
             this.elems = new Object[default_capacity * 2]; }
         elems[endindex++] = t;
         return (T[])elems;}

    class Itr implements Iterator<T>{
        private int point;
        private int len;
        public Itr(){
            this.point = 0;
            this.len = endindex;}
        @Override
        public boolean hasNext() {return point < endindex?true:false ;}
        @Override
        public T next() {return (T)elems[point++];}}
    
    @Override
    public Iterator<T> iterator() {return new Itr();}
    public T get(int i){
         if(i < endindex + 1 && i > 0){return (T) elems[i - 1];}
         throw new RuntimeException("查询索引越界");}
    
    public static void main(String[] args) {
        YangTtr<Object> yang = new YangTtr<>();
        yang.add(111);
        yang.add(222);
        yang.add(333);
        yang.add(444);
//        System.out.println(yang.get(1));
//        System.out.println(yang.get(2));
//        System.out.println(yang.get(3));
//        System.out.println(yang.get(4));
//        System.out.println(yang.get(0));
        Iterator iterator = yang.iterator(); // 内部类实现Iterator后
        while (iterator.hasNext()){
            System.out.println(iterator.next());}

        yang.forEach(v ->{      // 外部类实现Iterable后
            System.out.println(v);});}
//    接口是用于给实现类提供某种能力。
//    从这个例子中可以很清晰的理解这结论的准确性：
//    Iterable:给实现类提供一个获取迭代器的能力。
//    Iterator：给实现类提供迭代的能力。
}
```

## 各种判断

```java
// 判断 无视大小写 ,以及BigDecimal的判断
if(bigDecimal.compareTo(new BigDecimal("0")) == 1){
    wrapper.le("price", max);}

// 起始页判断
boolean addPageFlag = searchResponseVo.getTotal() % searchParam.getPageSize()==0;
long totalPage=0;
if(addPageFlag){
    totalPage = searchResponseVo.getTotal() / searchParam.getPageSize();
    }else{
    totalPage = searchResponseVo.getTotal() / searchParam.getPageSize()+1;}
```
