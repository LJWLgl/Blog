---
title: "Powermockito和Mockito使用总结"
date: 2020-07-11 14:32:55
tags: PowerMock
categories: 实践
summary:  "最近公司在推进Java应用的单元测试，要求将单元测试的覆盖率提高到50%以上" 
---

最近公司在推进Java应用的单元测试，要求将单元测试的覆盖率提高到50%以上，保证上线代码充分自测<!-- more -->。公司单元测试框架选用了`Junit 4.12`，Mock框架选用了`Mockito`和`PowerMock`，同时选用`JaCoCo`来做覆盖率检测，下面详细介绍一下我在使用这几个框架的使用总结。

## 依赖引入
```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.8.9</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>1.7.4</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>1.7.4</version>
    <scope>test</scope>
</dependency>
```
## PowerMockito的使用

**Mockito、EasyMock、JMock等比较流行Mock框架有个共同的缺点，都不能mock静态、final、私有方法等，而PowerMock可以完美解决以上框架的不足**，接下来让我们看看无所不能的PowerMock是如何解决上述问题，我们先从一个实例来看，下面的代码是需要单测的类，被测类中既有对Spring Bean的调用，同时又包含对Redis的静态读取，另外还有`getInstance()`单例类的使用，现在，我们一一来mock上述的外部类。

**被单测类（主要测试queryStudentScoreByKeyword方法）**

```java
@Component
public class StudentService {

    @Resource
    private StudentDao studentDao;

    public List<StudentBo> queryStudentScoreByKeyword(String name) {
        System.out.println("invoke StudentService.queryStudentScoreByKeyword ...");

        List<StudentBo> cacheList = RedisUtils.getArray(name, StudentBo.class);
        if (CollectionUtils.isNotEmpty(cacheList)) {
            return cacheList;
        }
        String keyword = processKeyword(name);
        List<Student> students = studentDao.queryStudentByKeyWord(keyword);
        List<Integer> ids = students.stream().map(Student::getId).collect(Collectors.toList());
        List<Person> personList = SchoolManageProxy.getInstance().queryPerson(ids);

        List<StudentBo> studentBos = CommonUtils.toBo(personList, students);
        // 高亮结果
        highlightResult(studentBos, name);
        // 缓存到Redis
        RedisUtils.setArray(name, studentBos, 10 * 60);
        return studentBos;
    }

    private String processKeyword(String name) {
        System.out.println("invoke StudentService.processKeyword ...");
        String newName = name;
        // do somethings
        return newName;
    }

    private void highlightResult(List<StudentBo> result, String name) {
        System.out.println("invoke StudentService.highlightResult ...");
        // do keyword highlight
    }

}
```

**单测类**

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({SchoolManageProxy.class, RedisUtils.class, StudentService.class})
// @PowerMockIgnore({"javax.management.*", "javax.net.ssl.*"})
@SuppressStaticInitializationFor({"cn.ganzhiqiang.ares.unittest.SchoolManageProxy"})
public class StudentServiceTest {

    @Mock
    private StudentDao mockStudentDao;

    @InjectMocks
    private StudentService studentServiceUnderTest;

    @Before
    public void setUp() {
        initMocks(this);
    }

    @Test
    public void testQueryStudentScoreByKeyword() throws Exception {
        studentServiceUnderTest = PowerMockito.spy(studentServiceUnderTest);
        PowerMockito.mockStatic(RedisUtils.class);
        PowerMockito.mockStatic(SchoolManageProxy.class);

        // mock单例调用
        SchoolManageProxy mockSchoolManageProxy = PowerMockito.mock(SchoolManageProxy.class);
        PowerMockito.when(SchoolManageProxy.getInstance()).thenReturn(mockSchoolManageProxy);
        when(mockSchoolManageProxy.queryPerson(anyList())).thenReturn(Collections.emptyList());

        // mock掉对Redis的静态调用
        PowerMockito.when(RedisUtils.getArray(eq("tom"), eq(StudentBo.class))).thenReturn(Collections.emptyList());
        // 显示的mock掉静态的void的方法（可以不mock）
        PowerMockito.doNothing().when(RedisUtils.class, "setArray", anyString(), anyList(), anyInt());

        // mock私有方法processKeyword
        PowerMockito.doReturn("tom").when(studentServiceUnderTest, "processKeyword", anyString());

        // 跳过私有方法highlightResult的执行
        PowerMockito.suppress(PowerMockito.method(StudentService.class, "highlightResult"));

        // 使用Mockito来mock服务的调用
        when(mockStudentDao.queryStudentByKeyWord(anyString())).thenReturn(Collections.emptyList());

        // Run the test
        final List<StudentBo> result = studentServiceUnderTest.queryStudentScoreByKeyword("tom");

    }

}
```

### 使用mockito来mock实例

首选我们先用Mockito来mock对Spring Bean的调用，`Mockito.mock`可以mock一个实例，我们这里选用`@Mock`注解，效果是一样的。

```java
// 使用Mockito来mock服务的调用 
when(mockStudentDao.queryStudentByKeyWord(anyString())).thenReturn(Collections.emptyList());
```

### mock对Redis的静态调用

接下来我们使用PowerMock来mock对静态方法的调用，注意需要将`RedisUtils`类，加入`@PrepareForTest`注解中，我们既mock了`getArray`方法，也mock了`setArray`方法，其实`setArray`不需要mock这里显式的mock了

```java
PowerMockito.mockStatic(RedisUtils.class);
// mock掉对Redis的静态调用
PowerMockito.when(RedisUtils.getArray(eq("tom"), eq(StudentBo.class))).thenReturn(Collections.emptyList());
// 显式的mock掉静态的void的方法（可以不mock）
PowerMockito.doNothing().when(RedisUtils.class, "setArray", anyString(), anyList(), anyInt());
```

### mock单例类

mock单例类相对来说复杂一点，逻辑上先用Powermock mock出单例类，然后再给单例类的`getInstance`方法打桩，返回之前mock，再对mock类实际调用的方法打桩即可，代码如下

```java
PowerMockito.mockStatic(SchoolManageProxy.class);
// Powermock mock出单例类
SchoolManageProxy mockSchoolManageProxy = PowerMockito.mock(SchoolManageProxy.class);
// 给单例类的getInstance方法打桩

PowerMockito.when(SchoolManageProxy.getInstance()).thenReturn(mockSchoolManageProxy);
// 对mock类queryPerson的方法打桩
when(mockSchoolManageProxy.queryPerson(anyList())).thenReturn(Collections.emptyList());
```

### mock私有方法

可以看到`queryStudentScoreByKeyword`方法调用了该类的私有方法`processKeyword`，如果该方法耗时过长，使用powermock也可以mock该私有方法，**需要注意的studentServiceUnderTest需要用spy()来mock**

```java
// mock 实例
// spy的标准是：如果不打桩，默认执行真实的方法，如果打桩则返回桩实现。
studentServiceUnderTest = PowerMockito.spy(studentServiceUnderTest);
// mock私有方法processKeyword
// doReturn(...) when(...)不做真实调用，但是when(...) thenReturn(...)还是会真实调用原方法，只是返回了指定的结果
PowerMockito.doReturn("tom").when(studentServiceUnderTest, "processKeyword", anyString());
```

### PowerMock跳过方法执行

使用PowerMock也可以跳过私有方法的执行

```java
// 跳过私有方法highlightResult的执行
PowerMockito.suppress(PowerMockito.method(StudentService.class, "highlightResult"));
```



## 总结

笔者之前写代码很少会写单测，自从公司强制要求提高单测覆盖率之后，虽然开发效率变慢了，但是确实引起我对单测的重视，进而研究了一下PowerMockito、Mokcito等Mock框架，Powermock之所以无所不能，是因为它使用了自定义的加载器和字节码操作技术，与此同时，它还十分简单易用，确实是个很优秀的框架。

Demo地址：[https://github.com/LJWLgl/mock-data](https://github.com/LJWLgl/mock-data)

参考文档

- [PowerMock](http://powermock.github.io/)
- [powermockito单元测试之深入实践](https://www.cnblogs.com/qingshanli/p/9763130.html)
- [浅谈测试之PowerMock](https://juejin.im/post/5b1d4708f265da6e0e3c0a48)
- [无所不能的PowerMock，mock私有方法，静态方法，测试私有方法，final类](https://www.cnblogs.com/ljw-bim/p/9391770.html)
- [Mock和Spy的区别](https://breeze924.github.io/2018/08/27/Mock%E5%92%8CSpy%E7%9A%84%E5%8C%BA%E5%88%AB/)

