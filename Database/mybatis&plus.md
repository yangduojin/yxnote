# mybatis

- [mybatis](#mybatis)
  - [基础](#基础)
    - [一级缓存与二级缓存的不同之处](#一级缓存与二级缓存的不同之处)
    - [返回自增主键](#返回自增主键)
    - [各种关键字使用](#各种关键字使用)
    - [四大对象 和 四大步骤](#四大对象-和-四大步骤)
  - [Mybatis-plus](#mybatis-plus)
    - [配置](#配置)
    - [注解](#注解)
    - [条件构造器 (queryWrapper | updateWrapper)](#条件构造器-querywrapper--updatewrapper)
    - [分页，乐观锁插件](#分页乐观锁插件)
    - [动态查询](#动态查询)
    - [AutoGenerator 文件生成器](#autogenerator-文件生成器)

## 基础

### 一级缓存与二级缓存的不同之处

- Mybatis的一级缓存默认开启，而二级缓存默认关闭。
- Mybatis的一级缓存指的是Mybaits中SqlSession对象的缓存，而二级缓存指的是SqlSessionFactory对象的缓存。一个SqlSessionFactory对象包括多个SqlSession对象。
- SqlSession对象中存放的是返回数据的对象，而SqlSessionFactory对象中存放的是数据，不是对象。
- Mybatis和Spring整合的时候，一级缓存与事务有关，而二级缓存与事务无关。

### 返回自增主键

```xml
// 方式一 必须支持自增并设置了自增
    <insert id="insertEmp4" useGeneratedKeys="true" keyProperty="empId">
        insert into t_emp values(null,#{empName},#{empSalary})</insert>
// 方式二 不支持自增
    <insert id="insertEmployee" parameterType="Employee" databaseId="oracle">
        <selectKey order="BEFORE" keyProperty="id" resultType="integer">
        select employee_seq.nextval from dual </selectKey>
        insert into…</insert>

    <insert id="insertEmployee" parameterType="Employee" databaseId="oracle">
        <selectKey order="AFTER" keyProperty="id" resultType="integer">
        select employee_seq.currval from dual</selectKey> insert into…</insert>

    <insert id="insertEmp5">
        <selectKey order="AFTER" keyProperty="empId" resultType="int">
            SELECT @@identity </selectKey> insert into t_emp values(null,#{empName},#{empSalary})</insert>

一对一 association 属性 property、javaType
    多对多 同步查询

    <resultMap id="customerMap" type="Customer">
        <id property="customerId" column="customer_id"></id>
        <result property="customerName" column="customer_name"></result>
        <collection property="orderList" ofType="com.atguigu.entity.Order">
            <id property="orderId" column="order_id"></id>
            <result property="orderName" column="order_name"></result>
        </collection>
    </resultMap>

// 分步多表查询
    <resultMap id="customerMap2" type="Customer">
        <id property="customerId" column="customer_id"></id>
        <result property="customerName" column="customer_name"></result>
        <collection property="orderList"
            column="customer_id"
            select="com.atguigu.mapper.OrderMapper.findOrderByCustomerId"></collection>
    </resultMap>
// 延迟开关
    <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
    fetchType="eager" // 分开关
```

### 各种关键字使用

```xml
// sql语句提取
<sql id="empSelectColumns">
        emp_id empId,emp_name  empName ,emp_salary empSalary
</sql>
<select id="findEmp" resultType="employee">
    select <include refid="empSelectColumns"></include>  from t_emp   
</select>

// where if 
<select id="findEmp2" resultType="employee">
    select emp_id empId,emp_name  empName ,emp_salary empSalary  from t_emp
    <where>
        <if test="empName!=null and empName !=''">
            and emp_name like concat("%",#{empName},"%")
        </if>
        <if test="minSalary>0">
            and emp_salary>=#{minSalary}
        </if>
    </where>
</select>

// foreach 单简单参数
<select id="findEmp5" resultType="employee">
    select emp_id empId,emp_name  empName ,emp_salary empSalary  from t_emp where emp_id in
    <foreach collection="idArr" item="empId" open="(" close=")" separator=",">
        #{empId}
    </foreach>
</select>   

// foreach 集合参数
<insert id="insertMoreEmp">
    INSERT INTO t_emp
    <foreach collection="employeeList" item="emp" open="VALUES" separator=",">
        (null,#{emp.empName},#{emp.empSalary})
    </foreach>

// set
    <update id="updateEmp">
    UPDATE t_emp
    <set>
        <if test="empName!=null and empName !=''">
            emp_name =#{empName},
        </if>
        <if test="empSalary >0">
            emp_salary = #{empSalary},
        </if>
    </set>
    WHERE emp_id = 40
</update>


// trim
<select id="findEmp3" resultType="employee">
    select emp_id empId,emp_name  empName ,emp_salary empSalary  from t_emp
    <trim prefix="where"  prefixOverrides="and">
        <if test="empName!=null and empName !=''">
            and emp_name like concat("%",#{empName},"%")
        </if>
        <if test="minSalary>0">
            and emp_salary>=#{minSalary}
        </if>
    </trim>
    </select>

// choose/then/otherwise
    <select id="findEmp4" resultType="employee">
    select emp_id empId,emp_name  empName ,emp_salary empSalary  from t_emp where
    <choose>
        <when test="empName!=null and empName !=''"> emp_name like concat("%",#{empName},"%")</when>
        <when test="minSalary>0"> emp_salary>=#{minSalary}</when>
            <otherwise>1=1</otherwise>
    </choose>
</select>
like "%"#{customerName}"%"  或者  like concat('%',#{customerName},'%')
```

### 四大对象 和 四大步骤

- Executor: Sqlsession 调用执行器，由执行器来调度StatementHandler、并由StatementHandler来调度ParameterHandler、ResultSetHandler等来执行对应的SQL。
- StatementHandler: 使用数据库的Statemet（PreparedStatement）执行操作
- ParameterHandler: 处理SQL参数；比如查询的where条件、和insert、update的字段值的占位符绑定。
- ResultSetHandler: 处理结果集ResultSet的封装返回。将数据库字段的值赋给实体类对象的成员变量。
- 第一步：获取数据库连接  Connection connection = this.getConnection(statementLog);
- 第二步：获取Statement（PreparedStatement） Statement stmt = handler.prepare(connection, this.transaction.getTimeout());
- 第三步：给select、insert、update的SQL语句的占位符？赋值 handler.parameterize(stmt);
- 第四步：执行CRUD操作 var9 = handler.query(stmt, resultHandler); //select var6 = handler.update(stmt); //insert update delete

## Mybatis-plus

### 配置

```xml
application.properties
server.port=8080
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&characterEncoding=utf-8
spring.datasource.username=root
spring.datasource.password=root
mybatis-plus.mapper-locations=classpath:mapper/*Mapper.xml
mybatis-plus.configuration.map-underscore-to-camel-case=true
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

- mapper接口继承 BaseMapper\<pojo>,里面封装单表的crud方法.需要注解@Mapper 或 入口类注解@MapperScan("mapper文件夹路径")
- service接口需要继承 IService\<pojo>,里面也封装了基本方法,而实现类也要继承ServiceImpl<mapper, pojo>,@Service

### 注解

1. @TableName指定表名
1. @TableId(type = IdType.ASSIGN_ID | IdType.AUTO ) id自增，需要数据库该字段同时有自增策略，不然会报错
   - 自动填充字段需要实现 MetaObjectHandler 这个类
1. @TableField(fill = FieldFill.INSERT)
   - private LocalDateTime createTime;
   - this.strictInsertFill(metaObject, "createTime | updateTime", LocalDateTime.class, LocalDateTime.now());
   - 或 this.strictInsertFill(metaObject, "createTime", Date.class, new Date());
1. @TableField(fill = FieldFill.INSERT_UPDATE)
   - private LocalDateTime updateTime;
1. @TableLogic 逻辑删除（非物理删除）数据库对应类型为tinyint，1。同时需要@TableField指明映射字段，可指定删除的标志编号为delval
1. @Version 乐观锁(版本控制，用一个字段来控制版本)

### 条件构造器 (queryWrapper | updateWrapper)

1. 比较大小 eq | ne | gt | ge | lt | le
2. 范围 between | notBetween | in | notIn | inSql | notInSql
3. 模糊 like | notLike | likeLeft | likeRight
4. 空值比较 isNull | isNotNull
5. 分组排序 groupBy | orderByAsc | orderByDesc | having
6. 拼接、嵌套 sql：（or、and、nested、apply）
   - or();
   - or(Consumer\<Param> consumer);
   - and(Consumer\<Param> consumer);
   - nested(Consumer\<Param> consumer);
   - apply(String applySql, Object... params); // 拼接sql（若不使用 params 参数，可能存在 sql 注入）
   - last(String lastSql); // 无视优化规则直接拼接到 sql 的最  后，可能存若在 sql 注入。
   - exists(String existsSql); // 拼接 exists 语句。
7. QueryWrapper 条件：
   - select(String... sqlSelect);
   - select(Predicate\<TableFieldInfo> predicate); //Lambda，过滤需要的字段。
   - lambda(); // 返回一个 LambdaQueryWrapper
8. UpdateWrapper 条件：
   - set(String column, Object val); // 用于设置set字段值。
   - etSql(String sql); // 用于设置 set 字段值。
   - lambda(); // 返回一个 LambdaUpdateWrapper

wrapper.and(w -> w.eq("id", key).or().like("spu_name",key)); 在需要跟别的拼接才需要and,单字段字节or

### 分页，乐观锁插件

```java
// 新建一个配置类 @configuration  @Bean里面retur MybatisPlusInterceptor()
分页：
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor(){
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));//分页，之前的有bug过时了
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor()); // 乐观锁用于@Version 
    return interceptor;
}
    // test
    IPage<User> iPage = new Page<>(3,3);
    IPage<User> userIPage = userMapper.selectPageByPage(iPage, 10);// age > 10
System.out.println(userIPage);
```

### 动态查询

```java
queryWrapper.like(name!=null&&name!="","name",name)
        .ge(ageBegin!=null,"age",ageBegin)
        .le(ageEnd!=null,"age",ageEnd);
updateWrapper.set("email","yx@gmail.com")
        .like("name","yoohooo")
        .gt("age",5);
queryWrapper.like(StringUtils.isNotBlank(name), User::getName, "n")
            .ge(ageBegin != null, User::getAge, ageBegin)
            .le(ageEnd != null, User::getAge, ageEnd);
updateWrapper.set(User::getAge, 18)
            .set(User::getEmail, "user@atguigu.com")
            .like(User::getName, "n")
        .and(i -> i.lt(User::getAge, 18).or().isNull(User::getEmail)); //lambda表达式内的逻辑优先运算
```

### AutoGenerator 文件生成器

```java
public void autoGenerate() {
    AutoGenerator mpg = new AutoGenerator();
    GlobalConfig gc = new GlobalConfig();
    String projectPath = "E:\\myProject\\test\\test_mybatis_plus";
    gc.setOutputDir(projectPath + "/src/main/java");
    gc.setAuthor("lyh");//doc
    gc.setOpen(false);// 配置是否打开目录
    gc.setSwagger2(true);//开启Swagger2模式
    gc.setFileOverride(false);//新生成是否覆盖
    gc.setIdType(IdType.ASSIGN_ID);//可选
    gc.setDateType(DateType.ONLY_DATE);//可选
    gc.setServiceName("%sService");//删除自动前缀I
    mpg.setGlobalConfig(gc);

    DataSourceConfig dsc = new DataSourceConfig();
    dsc.setUrl("jdbc:mysql://localhost:3306/testMyBatisPlus?useUnicode=true&characterEncoding=utf8");
    // dsc.setSchemaName("testMyBatisPlus"); // 可以直接在 url 中指定数据库名
    dsc.setDriverName("com.mysql.cj.jdbc.Driver");
    dsc.setUsername("root");
    dsc.setPassword("123456");
    mpg.setDataSource(dsc);

    PackageConfig pc = new PackageConfig();
    pc.setParent("com.lyh.test");
    pc.setModuleName("test_mybatis_plus");
    pc.setEntity("entity");
    pc.setMapper("mapper");
    pc.setService("service");
    pc.setController("controller");
    mpg.setPackageInfo(pc);

    StrategyConfig strategy = new StrategyConfig();
    strategy.setInclude("test_mybatis_plus_user");
    strategy.setNaming(NamingStrategy.underline_to_camel);
    strategy.setColumnNaming(NamingStrategy.underline_to_camel);
    strategy.setEntityLombokModel(true);
    strategy.setRestControllerStyle(true);
    strategy.setControllerMappingHyphenStyle(true);
    // 配置表前缀，生成实体时去除表前缀
    // 此处的表名为 test_mybatis_plus_user，模块名为 test_mybatis_plus，去除前缀后剩下为 user。
    strategy.setTablePrefix(pc.getModuleName() + "_");
    mpg.setStrategy(strategy);
    // Step6：执行代码生成操作
    mpg.execute();}
AutoGenerator mpg = new AutoGenerator();
    // 2、全局配置
    GlobalConfig gc = new GlobalConfig();
    String projectPath = System.getProperty("user.dir");
    gc.setOutputDir(projectPath + "/src/main/java");
    gc.setAuthor("Helen");
    gc.setOpen(false); //生成后是否打开资源管理器
    gc.setServiceName("%sService"); //去掉Service接口的首字母I
    gc.setIdType(IdType.AUTO); //主键策略
    gc.setSwagger2(true);//开启Swagger2模式
    mpg.setGlobalConfig(gc);
    // 3、数据源配置
    DataSourceConfig dsc = new DataSourceConfig();
    dsc.setUrl("jdbc:mysql://localhost:3306/srb_core?serverTimezone=GMT%2B8&characterEncoding=utf-8");
    dsc.setDriverName("com.mysql.cj.jdbc.Driver");
    dsc.setUsername("root");
    dsc.setPassword("123456");
    dsc.setDbType(DbType.MYSQL);
    mpg.setDataSource(dsc);
    // 4、包配置
    PackageConfig pc = new PackageConfig();
    pc.setParent("com.atguigu.srb.core");
    pc.setEntity("pojo.entity"); //此对象与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
    mpg.setPackageInfo(pc);
    // 5、策略配置
    StrategyConfig strategy = new StrategyConfig();
    strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略
    strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略
    strategy.setEntityLombokModel(true); // lombok
    strategy.setLogicDeleteFieldName("is_deleted");//逻辑删除字段名
    strategy.setEntityBooleanColumnRemoveIsPrefix(true);//去掉布尔值的is_前缀（确保tinyint(1)）
    strategy.setRestControllerStyle(true); //restful api风格控制器
    mpg.setStrategy(strategy);
    // 6、执行
    mpg.execute();
```
