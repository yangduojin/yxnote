# Reflect

- [Reflect](#reflect)
  - [反射的作用](#反射的作用)
  - [Class实例可以是那些结构的说明](#class实例可以是那些结构的说明)
  - [New 对象的方式](#new-对象的方式)
  - [获取属性/方法/构造器/父类/接口/父类的泛型/包/注解/异常等](#获取属性方法构造器父类接口父类的泛型包注解异常等)
  - [调用指定的属性/方法/构造器](#调用指定的属性方法构造器)

## 反射的作用

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时获取泛型信息
- 在运行时调用任意一个对象的成员变量和方法
- 在运行时处理注解
- 生成动态代理
- 加载到内存中的类，为运行时类，作为Class的一个实例,换句话说，Class的实例就对应着一个运行时类。
除了第一个引导加载器，其余的类加载器都是平级，getparent()只是拿到名字而已

## Class实例可以是那些结构的说明

1. class:外部类，成员(成员内部类，静态内部类)，局部内部类，匿名内部类
1. interface
1. []
1. enum
1. annotation:@interface
1. primitive type:基本数据类型
1. void

## New 对象的方式

```java
//方式一：调用运行时类的属性：.class
Class clazz1 = Person.class;
Person obj = clazz.newInstance();// 必须有空参构造器，权限得是public
System.out.println(clazz1);

//方式二：通过运行时类的对象,调用getClass()
Person p1 = new Person();
Class clazz2 = p1.getClass();
System.out.println(clazz2);

//方式三：调用Class的静态方法：forName(String classPath)常用！
Class clazz3 = Class.forName("com.atguigu.java.Person");
//clazz3 = Class.forName("java.lang.String");
System.out.println(clazz3);
System.out.println(clazz1 == clazz2);
System.out.println(clazz1 == clazz3);

//方式四：使用类的加载器：ClassLoader  (了解)
ClassLoader classLoader = ReflectionTest.class.getClassLoader();
Class clazz4 = classLoader.loadClass("com.atguigu.java.Person");
System.out.println(clazz4);
System.out.println(clazz1 == clazz4);
```

## 获取属性/方法/构造器/父类/接口/父类的泛型/包/注解/异常等

```java
我们可以通过反射，获取对应的运行时类中所有的属性、方法、构造器、父类、接口、父类的泛型、包、注解、异常等。。。。
public void test1(){
    Class clazz = Person.class;
// 获取属性结构
        //getFields():获取当前运行时类及其父类中声明为public访问权限的属性
        Field[] fields = clazz.getFields();
        for(Field f : fields){
            System.out.println(f);
        }
        System.out.println();
        //getDeclaredFields():获取当前运行时类中声明的所属性。（不包含父类中声明的属性
        Field[] declaredFields = clazz.getDeclaredFields();
        for(Field f : declaredFields){
            System.out.println(f);}}

// 获取方法
    @Test
    public void test1(){
        Class clazz = Person.class;
        //getMethods():获取当前运行时类及其所父类中声明为public权限的方法
        Method[] methods = clazz.getMethods();
        for(Method m : methods){
            System.out.println(m);}
        System.out.println();
        //getDeclaredMethods():获取当前运行时类中声明的所方法。（不包含父类中声明的方法
        Method[] declaredMethods = clazz.getDeclaredMethods();
        for(Method m : declaredMethods){
            System.out.println(m);}}

// 获取构造器结构
    @Test
    public void test1(){
        Class clazz = Person.class;
        //getConstructors():获取当前运行时类中声明为public的构造器
        Constructor[] constructors = clazz.getConstructors();
        for(Constructor c : constructors){
            System.out.println(c);
        }
        System.out.println();
        //getDeclaredConstructors():获取当前运行时类中声明的所的构造器
        Constructor[] declaredConstructors = clazz.getDeclaredConstructors();
        for(Constructor c : declaredConstructors){
            System.out.println(c);}}

// 获取运行时类的父类
    @Test
    public void test2(){
        Class clazz = Person.class;
        Class superclass = clazz.getSuperclass();
        System.out.println(superclass);}

// 获取运行时类的带泛型的父类
    @Test
    public void test3(){
        Class clazz = Person.class;
        Type genericSuperclass = clazz.getGenericSuperclass();
        System.out.println(genericSuperclass);}

// 获取运行时类的带泛型的父类的泛型
代码：逻辑性代码  vs 功能性代码
    @Test
    public void test4(){
        Class clazz = Person.class;
        Type genericSuperclass = clazz.getGenericSuperclass();
        ParameterizedType paramType = (ParameterizedType) genericSuperclass;
获取泛型类型
        Type[] actualTypeArguments = paramType.getActualTypeArguments();
        //System.out.println(actualTypeArguments[0].getTypeName());
        System.out.println(((Class)actualTypeArguments[0]).getName());}

获取运行时类实现的接口
    @Test
    public void test5(){
        Class clazz = Person.class;
        Class[] interfaces = clazz.getInterfaces();
        for(Class c : interfaces){
            System.out.println(c);
        }
        System.out.println();
获取运行时类的父类实现的接口
    Class[] interfaces1 = clazz.getSuperclass().getInterfaces();
    for(Class c : interfaces1){
        System.out.println(c);}}

获取运行时类所在的包
    @Test
    public void test6(){
        Class clazz = Person.class;
        Package pack = clazz.getPackage();
        System.out.println(pack);}

获取运行时类声明的注解
    @Test
    public void test7(){
        Class clazz = Person.class;
        Annotation[] annotations = clazz.getAnnotations();
        for(Annotation annos : annotations){
            System.out.println(annos);}}
```

## 调用指定的属性/方法/构造器

```java
调用指定的属性：
    @Test
    public void testField1() throws Exception {
        Class clazz = Person.class;
        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();
        //1. getDeclaredField(String fieldName):获取运行时类中指定变量名的属性
        Field name = clazz.getDeclaredField("name");
        //2.保证当前属性是可访问的
        name.setAccessible(true);
        //3.获取、设置指定对象的此属性值
        name.set(p,"Tom");
        System.out.println(name.get(p));}

调用指定的方法：
        @Test
        public void testMethod() throws Exception {
            Class clazz = Person.class;
            //创建运行时类的对象
            Person p = (Person) clazz.newInstance();
            /*
            1.获取指定的某个方法
            getDeclaredMethod():参数1 ：指明获取的方法的名称  参数2：指明获取的方法的形参列表
                */
            Method show = clazz.getDeclaredMethod("show", String.class);
            //2.保证当前方法是可访问的
            show.setAccessible(true);
            /*
            1. 调用方法的invoke():参数1：方法的调用者  参数2：给方法形参赋值的实参
            invoke()的返回值即为对应类中调用的方法的返回值。
                */
            Object returnValue = show.invoke(p,"CHN"); //String nation = p.show("CHN");
            System.out.println(returnValue);
            System.out.println("*************如何调用静态方法*****************");
            // private static void showDesc()
            Method showDesc = clazz.getDeclaredMethod("showDesc");
            showDesc.setAccessible(true);
            //如果调用的运行时类中的方法没返回值，则此invoke()返回null
            // Object returnVal = showDesc.invoke(null);
            Object returnVal = showDesc.invoke(Person.class);
            System.out.println(returnVal);//null}

调用指定的构造器：
    @Test
    public void testConstructor() throws Exception {
        Class clazz = Person.class;
        //private Person(String name)
        /*
        1.获取指定的构造器
        getDeclaredConstructor():参数：指明构造器的参数列表
            */
        Constructor constructor = clazz.getDeclaredConstructor(String.class);
        //2.保证此构造器是可访问的
        constructor.setAccessible(true);
        //3.调用此构造器创建运行时类的对象
        Person per = (Person) constructor.newInstance("Tom");
        System.out.println(per);}
```
