# springcloud权限配置

### 配置url是否有权限访问
WebSecurityConfigurerAdapter

#####
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>


@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 除了“/”,”/home”(首页), "/hello/*", "/user-info/*", "/test/*",
     * ”/login”(登录),”/logout”(注销),之外，其他路径都需要认证。
     * 指定“/login”该路径为登录页面，当未认证的用户尝试访问任何受保护的资源时，都会跳转到“/login”。
     * 默认指定“/logout”为注销页面
     *
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/", "/home", "/hello/*", "/user-info/*", "/test/*").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
                .logout()
                .permitAll();
    }

}

        
```


参考<https://juejin.cn/post/6844903872096387080>
<https://blog.springlearn.cn/posts/36693/>