# IDEA 常用模板

## 获取线程名

- mythread Thread.currentThread().getName()
  
## 测试 test

```java
@Test
public void test$var1$(){
    $var2$
}
```

## 线程睡眠 tsleep

```java
try {TimeUnit.SECONDS.sleep($var1$);} catch (Exception e) {e.printStackTrace();}
    $var2$
```
