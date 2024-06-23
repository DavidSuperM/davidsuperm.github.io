# spring知识点

### 属性注入
实体类上加
```
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
```
相当于用构造函数的方式属性注入
不加 (onConstructor = @__(@Autowired)) 这个也行，则构造函数上没有@Autowired，spring也能注入
加了这个是指定构造函数上有 @Autowired 

