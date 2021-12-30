# Java特性

- [Java特性](#java特性)
  - [函数式接口](#函数式接口)
  - [方法引用](#方法引用)
    - [构造器引用](#构造器引用)
    - [数组引用](#数组引用)
  - [Stream](#stream)
    - [递归获取分类的写法](#递归获取分类的写法)
    - [Stream流获取分类的写法](#stream流获取分类的写法)
    - [stream 实例化 中间操作 终止操作](#stream-实例化-中间操作-终止操作)
  - [Optional](#optional)

## 函数式接口

一个接口中，只声明了一个抽象方法，则此接口就称为函数式接口。我们可以在一个接口上使用 @FunctionalInterface 注解，这样做可以检查它是否是一个函数式接口  
Lambda表达式的本质：作为函数式接口的实例

| 接口类型       | 参数类型 | 返回值  | 方法        |
| -------------- | -------- | ------- | ----------- |
| Consumer\<T>   | T        | void    | accept(T t) |
| Supplier\<T>   |          | T       | get()       |
| Function\<T,R> | T        | R       | apply(T t)  |
| Predicate\<T>  | T        | Boolean | test(T t)   |

函数式接口lambda：

1. 一个接口只有一个待实现的public方法 () -> { }，
2. 一般有@FunctionalInterface,没有也会自动加上,
3. 可以有多个用default修饰已实现的方法,不会算在待实现里面，依然可以lambda
4. 用static修饰已实现的方法，也不算在待实现里面

## 方法引用

1. 格式
   - 情况1 对象 :: 非静态方法
   - 情况2 类 :: 静态方法
   - 情况3 类 :: 非静态方法
2. 要求
   - 要求接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同！(针对于情况1和情况2)
   - 当函数式接口方法的第一个参数是需要引用方法的调用者，并且第二个参数是需要引用方法的参数(或无参数)时：ClassName::methodName(针对于情况3)

    ```java
    使用举例：
    // 情况一：对象 :: 实例方法
    // Consumer中的void accept(T t)
    // PrintStream中的void println(T t)
        @Test
        public void test1() {
            Consumer<String> con1 = str -> System.out.println(str);
            con1.accept("北京");
            System.out.println("*******************");
            PrintStream ps = System.out;
            Consumer<String> con2 = ps::println;
            con2.accept("beijing");
        }
    // Supplier中的T get()
    // Employee中的String getName()
        @Test
        public void test2() {
            Employee emp = new Employee(1001,"Tom",23,5600);
            Supplier<String> sup1 = () -> emp.getName();
            System.out.println(sup1.get());
            System.out.println("*******************");
            Supplier<String> sup2 = emp::getName;
            System.out.println(sup2.get());
        }
    // 情况二：类 :: 静态方法
    // Comparator中的int compare(T t1,T t2)
    // Integer中的int compare(T t1,T t2)
        @Test
        public void test3() {
            Comparator<Integer> com1 = (t1,t2) -> Integer.compare(t1,t2);
            System.out.println(com1.compare(12,21));
            System.out.println("*******************");
            Comparator<Integer> com2 = Integer::compare;
            System.out.println(com2.compare(12,3));
        }
    // Function中的R apply(T t)
    // Math中的Long round(Double d)
        @Test
        public void test4() {
            Function<Double,Long> func = new Function<Double, Long>() {
                @Override
                public Long apply(Double d) {
                    return Math.round(d);
                }
            };
            System.out.println("*******************");
            Function<Double,Long> func1 = d -> Math.round(d);
            System.out.println(func1.apply(12.3));
            System.out.println("*******************");
            Function<Double,Long> func2 = Math::round;
            System.out.println(func2.apply(12.6));
        }
    // 情况：类 :: 实例方法  (难度)
    // Comparator中的int comapre(T t1,T t2)
    // String中的int t1.compareTo(t2)
        @Test
        public void test5() {
            Comparator<String> com1 = (s1,s2) -> s1.compareTo(s2);
            System.out.println(com1.compare("abc","abd"));
            System.out.println("*******************");
            Comparator<String> com2 = String :: compareTo;
            System.out.println(com2.compare("abd","abm"));
        }
    // BiPredicate中的boolean test(T t1, T t2);
    // String中的boolean t1.equals(t2)
        @Test
        public void test6() {
            BiPredicate<String,String> pre1 = (s1,s2) -> s1.equals(s2);
            System.out.println(pre1.test("abc","abc"));
            System.out.println("*******************");
            BiPredicate<String,String> pre2 = String :: equals;
            System.out.println(pre2.test("abc","abd"));
        }
    // Function中的R apply(T t)
    // Employee中的String getName();
        @Test
        public void test7() {
            Employee employee = new Employee(1001, "Jerry", 23, 6000);
            Function<Employee,String> func1 = e -> e.getName();
            System.out.println(func1.apply(employee));
            System.out.println("*******************");
            Function<Employee,String> func2 = Employee::getName;
            System.out.println(func2.apply(employee));
    }
    ```

### 构造器引用

1. 格式 类名::new
2. 要求：和方法引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一致。抽象方法的返回值类型即为构造器所属的类的类型

    ```java
    //Supplier中的T get()
    //Employee的空参构造器：Employee()
    @Test
    public void test1(){
        Supplier<Employee> sup = new Supplier<Employee>() {
            @Override
            public Employee get() {
                return new Employee();}};

        System.out.println("*******************");
        Supplier<Employee>  sup1 = () -> new Employee();
        System.out.println(sup1.get());
        System.out.println("*******************");
        Supplier<Employee>  sup2 = Employee :: new;
        System.out.println(sup2.get());}

    //Function中的R apply(T t)
    @Test
    public void test2(){
        Function<Integer,Employee> func1 = id -> new Employee(id);
        Employee employee = func1.apply(1001);
        System.out.println(employee);
        System.out.println("*******************");
        Function<Integer,Employee> func2 = Employee :: new;
        Employee employee1 = func2.apply(1002);
        System.out.println(employee1);}

    //BiFunction中的R apply(T t,U u)
    @Test
    public void test3(){
        BiFunction<Integer,String,Employee> func1 = (id,name) -> new Employee(id,name);
        System.out.println(func1.apply(1001,"Tom"));
        System.out.println("*******************");
        BiFunction<Integer,String,Employee> func2 = Employee :: new;
        System.out.println(func2.apply(1002,"Tom"));}
    ```

### 数组引用

1. 格式 数组类型[] :: new

    ```java
    //Function中的R apply(T t)
    @Test
    public void test4(){
        Function<Integer,String[]> func1 = length -> new String[length];
        String[] arr1 = func1.apply(5);
        System.out.println(Arrays.toString(arr1));
        System.out.println("*******************");
        Function<Integer,String[]> func2 = String[] :: new;
        String[] arr2 = func2.apply(10);
        System.out.println(Arrays.toString(arr2));
    }
    ```

## Stream

stream trace 用来调试流bug

```java
List<User> list = Arrays.asList(u1,u2,u3,u4,u5);
list.stream()
    .filter(p -> {return p.getId() % 2 == 0;}).filter(p -> {return p.getAge() > 24;})
    .map(f -> {return f.getUserName().toUpperCase();})
    .sorted((o1, o2) -> {return o2.compareTo(o1);})
    .limit(1).forEach(System.out::println);

BigDecimal totalPrincipal = lendItemReturnAllList.stream()
    .filter(item -> item.getLendReturnId().longValue() ==lendReturn.getId().longValue())
    .map(LendItemReturn::getPrincipal).reduce(BigDecimal.ZERO,BigDecimal::add);

BigDecimal sumPrincipal = lendItemReturnList.stream()
    .map(LendItemReturn::getPrincipal).reduce(BigDecimal.ZERO,BigDecimal::add);
BigDecimal lastPrincipal = lendItem.getInvestAmount().subtract(sumPrincipal,MathContext.UNLIMITED);

public List<CategoryEntity> listWithTree() {
List<CategoryEntity> categoryEntities = baseMapper.selectList(null);
List<CategoryEntity> rootCatatoryEntities = categoryEntities.stream()
    .filter((item) -> { return item.getParentCid() == 0; })
    .map((menu) -> { menu.setChildList(getChildrens(menu,categoryEntities));return menu; })
    .sorted((menu1,menu2) -> (menu1.getSort()==null?0:menu1.getSort()) - (menu2.getSort()==null?0:menu2.getSort()))
    .collect(Collectors.toList());
return rootCatatoryEntities;}

private List<CategoryEntity> getChildrens(CategoryEntity root,List<CategoryEntity> all){
List<CategoryEntity> collect = all.stream()
    .filter((item) -> { return root.getCatId().equals(item.getParentCid()); })
    .map((menu) -> { menu.setChildList(getChildrens(menu, all));return menu; })
    .sorted((menu1, menu2) -> (menu1.getSort() == null ? 0 : menu1.getSort()) - (menu2.getSort() == null ? 0 : menu2.getSort()))
    .collect(Collectors.toList());
return collect;}


public void mapReduceDengGao() {
    List<String> list = Lists.newArrayList("无边落木萧萧下","不尽长江滚滚来","万里悲秋常作客","百年多病独登台");
    String result = list.stream()
            .map(word->word+"\n")
            .reduce((a,b)->a+""+b)
            .get();
    System.out.println(result);}
```

### 递归获取分类的写法

```java
public List<CategoryEntity> listWithTree() {
    List<CategoryEntity> categoryEntities = baseMapper.selectList(null);
    List<CategoryEntity> rootCatatoryEntities = categoryEntities.stream()
            .filter((item) -> { return item.getParentCid() == 0; })
            .map((menu) -> { menu.setChildren(getChildrens(menu,categoryEntities));return menu; })
            .sorted((menu1,menu2) -> (menu1.getSort()==null?0:menu1.getSort()) - (menu2.getSort()==null?0:menu2.getSort()))
            .collect(Collectors.toList());
    return rootCatatoryEntities;}

    private List<CategoryEntity> getChildrens(CategoryEntity root,List<CategoryEntity> all){
    List<CategoryEntity> collect = all.stream()
            .filter((item) -> { return root.getCatId().equals(item.getParentCid()); })
            .map((menu) -> { menu.setChildren(getChildrens(menu, all));return menu; })
            .sorted((menu1, menu2) -> (menu1.getSort() == null ? 0 : menu1.getSort()) - (menu2.getSort() == null ? 0 : menu2.getSort()))
            .collect(Collectors.toList());
    return collect;}
```

### Stream流获取分类的写法

```java
public Map<String, List<Catelog2Vo>> getCatelogJsonFromDB() {
    // 性能优化 一次性查出所有的
    List<CategoryEntity> selectList = baseMapper.selectList(null);


    List<CategoryEntity> level1Categorys =getParent_cid(selectList,0L);
    Map<String, List<Catelog2Vo>> parent_cId = level1Categorys.stream().collect(Collectors.toMap(k -> k.getCatId().toString(), v -> {
        List<CategoryEntity> categoryEntities = getParent_cid(selectList,v.getCatId());
        List<Catelog2Vo> catelog2Vos = null;
        if (!CollectionUtils.isEmpty(categoryEntities)) {
            catelog2Vos = categoryEntities.stream().map(l2 -> {
                Catelog2Vo catelog2Vo = new Catelog2Vo(l2.getCatId().toString(), l2.getName(), v.getCatId().toString(), null);

                List<CategoryEntity> Catalog3Vos = getParent_cid(selectList,l2.getCatId());
                if (!CollectionUtils.isEmpty(Catalog3Vos)) {
                    List<Catalog3Vo> collect = Catalog3Vos.stream().map(l3 -> {
                        Catalog3Vo catalog3Vo = new Catalog3Vo(l3.getCatId().toString(), l3.getName(), l2.getCatId().toString());
                        return catalog3Vo;
                    }).collect(Collectors.toList());
                    catelog2Vo.setCatalog3List(collect);
                }
                return catelog2Vo;
            }).collect(Collectors.toList());
        }
        return catelog2Vos;
    }));

    return parent_cId;}

private List<CategoryEntity> getParent_cid(List<CategoryEntity> selectList,Long parent_cid) {
    List<CategoryEntity> collect = selectList.stream().filter(item -> item.getParentCid() == parent_cid).collect(Collectors.toList());
    return collect;
// return baseMapper.selectList(new QueryWrapper<CategoryEntity>().eq("parent_cid", v.getCatId()));}
// 流式:从一张大表上面查询的数据在内存中操作,一般是一级一级的套,并且上一级的结果作为下一级的开始.简单版就是自己写循环,复杂版就是递归
```

1. 去重及flatmap

```java
List<Book> collect = authors.stream().distinct()
                .filter(author -> author.getAge() < 18)
                .map(author -> author.getBooks())
                .flatMap(Collection::stream)
                .filter(book -> book.getScore() > 70 )
                .distinct()
                .collect(Collectors.toList());
        System.out.println(collect);

baseCategoryViews.stream().collect(Collectors.groupingBy(BaseCategoryView::getCategory1Id));
```

### stream 实例化 中间操作 终止操作

1. 步骤一：Stream实例化

    ```java
    //创建 Stream方式一：通过集合
    @Test
    public void test1(){
        List<Employee> employees = EmployeeData.getEmployees();
    //        default Stream<E> stream() : 返回一个顺序流
        Stream<Employee> stream = employees.stream();
    //        default Stream<E> parallelStream() : 返回一个并行流
        Stream<Employee> parallelStream = employees.parallelStream();
    }
    //创建 Stream方式二：通过数组
    @Test
    public void test2(){
        int[] arr = new int[]{1,2,3,4,5,6};
        //调用Arrays类的static <T> Stream<T> stream(T[] array): 返回一个流
        IntStream stream = Arrays.stream(arr);
        Employee e1 = new Employee(1001,"Tom");
        Employee e2 = new Employee(1002,"Jerry");
        Employee[] arr1 = new Employee[]{e1,e2};
        Stream<Employee> stream1 = Arrays.stream(arr1);
    }
    //创建 Stream方式三：通过Stream的of()
    @Test
    public void test3(){
        Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6);
    }
    //创建 Stream方式四：创建无限流
    @Test
    public void test4(){
    //      迭代
    //      public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)
        //遍历前10个偶数
        Stream.iterate(0, t -> t + 2).limit(10).forEach(System.out::println);
    //      生成
    //      public static<T> Stream<T> generate(Supplier<T> s)
        Stream.generate(Math::random).limit(10).forEach(System.out::println);
    }
    ```

2. 步骤二：中间操作

    ```java
    a. 筛选与切片
        Distinct()              筛选，通过流所生成的元素的hashcode()和equals()去除重复元素
        Limit(long maxSize)     截断流，使其元素不超过给定数量
        Skip(long n)            跳过元素，返回一个扔掉了前n个元素的流，若流中元素不足n，则返回一个空流，与limit互
    b. 映射
        map(Function f)     函数做参数，该函数作用到每个元素上，并将其映射成一个新的元素
        mapToDouble(ToDoubleFunction f) 每个元素作用一个函数，生成一个新的DoubleSteam
        mapToInt(ToIntFunction f) 每个元素作用一个函数，生成一个新的IntSteam
        mapToLong(ToLongFunction f) 每个元素作用一个函数，生成一个新的LongSteam
        FlatMap(Function f) 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
    c. 排序
        sorted() 产生一个新流，其中按自然顺序排序
        sorted(Comparator com) 产生一个新流，其中按比较器顺序排序
    ```

3. 步骤三：终止操作

    ```java
    a. 匹配与查找
        allMatch(Predicate p) 检查是否匹配所有元素
        anyMatch(Predicate p) 检查是否至少匹配一个元素
        noneMatch(Predicate p) 检查是否没有匹配所有元素
        findFirst()
        findAny()
        count()     返回总元素数
        max(Comparator c)   返回流中最大值
        min(Comparator c)   返回流中最小值
        forEach(Consumer c) 内部迭代
    b. 归约
        reduce(T iden,BinaryOperator b) 可以将流中元素反复结合起来，得到一个值，返回T
        reduce(BinaryOperator b)        将流中元素反复结合起来，得到一个值，返回Optional<T>
    c. 收集
    collect(Collector c)    将流转换为其他形式，接受一个Collector接口的实现，用于给Stream中元素做汇总的方法
        collect(Collectors.toList())
        collect(Collectors.toSet())
    collect(Collectors.toCollection(ArrayList::new))
    ```

## Optional

java.util.Optional类 为了解决java中的空指针问题而生！  
Optional\<T> 类(java.util.Optional) 是一个容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null,表示这个值不存在。原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避免空指针异常。

```java
@Test
    public void test1(){
        //empty():创建的Optional对象内部的value = null
        Optional\<Object> op1 = Optional.empty();
        if(!op1.isPresent()){//Optional封装的数据是否包含数据
            System.out.println("数据为空");
        }
        System.out.println(op1);
        System.out.println(op1.isPresent());
// 如果Optional封装的数据value为空，则get()报错。否则，value不为空时，返回value.
// System.out.println(op1.get());
    }
    @Test
    public void test2(){
        String str = "hello";
//        str = null;
        //of(T t):封装数据t生成Optional对象。要求t非空，否则报错。
        Optional<String> op1 = Optional.of(str);
        //get()通常与of()方法搭配使用。用于获取内部的封装的数据value
        String str1 = op1.get();
        System.out.println(str1);
    }
    @Test
    public void test3(){
        String str = "beijing";
        str = null;
        //ofNullable(T t) ：封装数据t赋给Optional内部的value。不要求t非空
        Optional<String> op1 = Optional.ofNullable(str);
        //orElse(T t1):如果Optional内部的value非空，则返回此value值。如果
        //value为空，则返回t1.
        String str2 = op1.orElse("shanghai");
        System.out.println(str2);//}
```

典型练习：能保证如下的方法执行中不会出现空指针的异常。

```java
//使用Optional类的getGirlName():
public String getGirlName2(Boy boy){
    Optional<Boy> boyOptional = Optional.ofNullable(boy);
    //此时的boy1一定非空
    Boy boy1 = boyOptional.orElse(new Boy(new Girl("迪丽热巴")));
    Girl girl = boy1.getGirl();
    Optional<Girl> girlOptional = Optional.ofNullable(girl);
    //girl1一定非空
    Girl girl1 = girlOptional.orElse(new Girl("古力娜扎"));
    return girl1.getName();}

@Test
public void test5(){
    Boy boy = null;
    boy = new Boy();
    boy = new Boy(new Girl("苍老师"));
    String girlName = getGirlName2(boy);
    System.out.println(girlName);}
```
