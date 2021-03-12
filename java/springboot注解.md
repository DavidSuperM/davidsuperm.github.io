## 变量注入
我一般用3
网上貌似推荐用1
参考
<https://xie.infoq.cn/article/574e5059d971d00246602a340>
这三种注入的区别
1. @Autowired 注解是 Spring 中的，默认按类型注入；@Resource 注解是 Java EE 中自带的，默认按名称注入的，不建议使用。
```
public class UserInfoController {
    @Resource
    private UserInfoService userInfoService;
}
```
2. @autowired写在变量上的注入要等到类完全加载完，才会将相应的bean注入。
```
public class UserInfoController {
    @Autowired
    private UserInfoService userInfoService;
}
```
这种情况下会报错<https://blog.csdn.net/kane0409/article/details/78865964>
3. @autowired写在构造方法上
```
public class UserInfoController {
    private UserInfoService userInfoService;
    @Autowired
    public UserInfoController(UserInfoService userInfoService) {
        this.userInfoService = userInfoService;
    }
}
```

### 循环依赖
循环依赖是违反设计原则的
3 会在springboot启动时检查到循环依赖并报错
1 不会报错
解决：
参考<https://juejin.cn/post/6844904039231012878>
1. 重新设计结构，消除循环依赖。
2. 3的方案的话加 @Lazy
```
@Autowired
    public CircularDependencyA(@Lazy CircularDependencyB circB) {
        this.circB = circB;
    }
```
3. 使用Setter/Field注入

## 时间字段与前端交互，设置特定格式
```
@JsonFormat(shape= JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")
private LocalDateTime createTime;
```

# 问题
问题：springboot后端变量大写，返回给前端会变成小写
原因：spring在序列化和反序列化时做了处理
解决：变量上加注解

```
@JsonProperty("PEOPLEName")
private String peopleName;
```