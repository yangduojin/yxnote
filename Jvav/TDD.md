# TDD

## 注解

| @Test              |                                                             |
| ------------------ | ----------------------------------------------------------- |
| @TestFactory       | 方法是基于数据驱动的动态测试数据源                          |
| @ParameterizedTest | 方法是测试方法，且该方法可以用不同的入参运行多次            |
| @RepeatedTest      | 可自定义重复运行次数                                        |
| @BeforeEach        | 运行前运行,junit5中可运行@Test@ParamerizedTest@RepeatedTest |
| @AfterEach         | 与上一个类似，不过是运行后运行                              |
| @BeforeAll         | 与4的@BeforeClass类似，运行前运行                           |
| @AfterAll          | 与4的@BeforeClass类似，运行后运行                           |
| @Disabled          | 4类似@Igonre，类或方法不再运行                              |
| @Nested            | 为测试添加嵌套层级，以便组织用例结构                        |
| @Tag               | 为测试类或方法加标签，以便有选择的执行                      |

断言在org.junit.jupiter.api.Assertions类中，提供的方法如下

| Fail                                    | 断言测试失败                                                     |
| --------------------------------------- | ---------------------------------------------------------------- |
| assertTrue/assertFalse                  | 断言条件为真或为假                                               |
| assertNull/assertNotNull                | 断言指定值为null或非null                                         |
| assertEquals/assertNotEquals            | 指定两个值相等或不等，基本类型，值比较，引用类型，equals方法比较 |
| assertArrayEquals                       | 断言数组元素全部相等                                             |
| assertSame/assertNotSame                | 断言指定的两个对象是否为同一个对象                               |
| assertThrows/assertDoesNotThrow         | 断言是否抛出了一个特定类型的异常                                 |
| assertTimeout/assertTimeoutPreemptively | 断言是否执行超时，区别在于测试程序是否在同一个线程内执行         |
| assertIterableEquals                    | 断言迭代器中的元素全部相等                                       |
| assertLinesMatch                        | 断言字符串列表元素全部正则匹配                                   |
| assertAll                               | 断言多个条件同时满足                                             |
| assumeTrue/assumeFalse                  | 先判断给定的条件真假，再决定是否要测试下去                       |

AssertJ 流式断言

契约式编程 防御式编程(推荐)，允许集合return null

所有的压缩都是有条件的，例：替换方法名用数字编号代替

Springboot 测试类
@RunWith
@SpringApplicationConfiguration(classes = 入口类.class)加载环境

## 书籍

1. 书籍包括《测试驱动开发》、《重构》、《BDD In Action》以及《系统思考》等，从而充分理解TDD优点和局限。

## 收集的文章

1. 单测的意义 [不好写单测的代码都是烂代码](https://zhenbianshu.github.io/2019/10/talk_about_unit_test.html)
2. 有些公司限制一个方法不超过 10 行
3. 有注释的地方都可以抽取方法，用方法名来代替注释：
