---
title: Spring security 配置
date: 2020-05-31
banner_img: /img/baisc/spring-binner.jpg
index_img: /img/baisc/spring-index.jpg
tags: 
 - security
categories:
 - java
 - spring
---
**生活加油:摘一句子:**

**“我希望自己能写这样的诗。我希望自己也是一颗星星。如果我会发光，就不必害怕黑暗。如果我自己是那么美好，那么一切恐惧就可以烟消云散。于是我开始存下了一点希望—如果我能做到，那么我就战胜了寂寞的命运。”**

​                                         -----------------------------王小波《我在荒岛上迎接黎明》

# 初步嘗試一下：

**新建项目,导入依赖**

```java
package com.liruilong;
 
import org.springframework.web.bind.annotation.GetMapping;
 
import org.springframework.web.bind.annotation.RestController;
 
/**
 * @Description : security学习
 * @Author: Liruilong
 * @Date: 2019/12/24 12:54
 */
@RestController
public class HollerController {
    @GetMapping("/hello")
    public String hello(){
        return "hello Security";
    }
}
```

**访问接口:请求都被保护起来,用户名默认user,密码为控制台打印的字符串.**

[![img](https://img-blog.csdnimg.cn/2019122413045146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/2019122413045146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)

[![img](https://img-blog.csdnimg.cn/20191224130732495.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20191224130732495.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)

 **手工配置用户名和密码:**

一,配置类方式:

```java
package com.liruilong.securityl.demo;
 
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
 
/**
 * @Description : security配置
 * @Author: Liruilong
 * @Date: 2019/12/24 13:13
 */
@Configuration
public class config extends WebSecurityConfigurerAdapter {
 
    /**
     * @Author Liruilong 
     * @Description 密码处理,告诉系统不加密访问
     * @Date 13:20 2019/12/24
     * @Param [] 
     * @return org.springframework.security.crypto.password.PasswordEncoder 
     **/
    
    @Bean
    PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }
    
    /**
     * @Author Liruilong 
     * @Description 配置用户名密码.密码必须加密
     * @Date 13:16 2019/12/24
     * @Param [auth] 
     * @return void 
     **/
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("liruilong").password("123").roles("admin")
                .and()
                .withUser("liruilongs").password("123").roles("user");
 
    }
 
}
```

**配置文件方式:**

[![img](https://img-blog.csdnimg.cn/2019122413304389.png)](https://img-blog.csdnimg.cn/2019122413304389.png)

 

## HttpScurity的简单配置:

**基于内存的认证，以自定义类继承自 webSecurityConfigurerAdapter ，进行自定义配置。**

```java
package com.liruilong.securityl.demo;
 
 
/**
 * @Description : security配置
 * @Author: Liruilong
 * @Date: 2019/12/24 13:13
 */
@Configuration
public class config extends WebSecurityConfigurerAdapter {
 
    /**
     * @Author Liruilong 
     * @Description 密码处理,告诉系统不加密访问
     * @Date 13:20 2019/12/24
     * @Param [] 
     * @return org.springframework.security.crypto.password.PasswordEncoder 
     **/
    
    @Bean
    PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }
    
    /**
     * @Author Liruilong 
     * @Description 配置用户名密码.密码必须加密
     * @Date 13:16 2019/12/24
     * @Param [auth] 
     * @return void 
     **/
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("liruilong").password("123").roles("admin")
                .and()
                .withUser("liruilongs").password("123").roles("user");
 
    }
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //开启配置，开启 HtψSecurity 的配直
        http.authorizeRequests()
                // 指定admin角色可以访问该路径
                .antMatchers("/admin/**").hasRole("admin")
                // 指定admin和user可以访问该路径
                .antMatchers("/user/**").hasAnyRole("admin", "user")
                // 剩下的请求登录之后就可以访问
                .anyRequest().authenticated()
                .and()
                // 表单登录的url,请求地址
                .formLogin()
                .loginProcessingUrl("/dolog")
                .permitAll()
                .and()
                // 关闭csrf
                .csrf().disable();
 
    }
}
```

### 表单登录的详细配置:

```java
     @Override
    protected void configure(HttpSecurity http) throws Exception {
        //开启配置
        http.authorizeRequests()
                // 指定admin角色可以访问该路径
                .antMatchers("/admin/**").hasRole("admin")
                // 指定admin和user可以访问该路径
                .antMatchers("/user/**").hasAnyRole("admin", "user")
                // 剩下的请求登录之后就可以访问
                .anyRequest().authenticated()
                .and()
                // 表单登录的url,请求地址
                .formLogin()
                // url,请求地址
                .loginProcessingUrl("/dolog")
                // 登录页面
                .loginPage("login")
                // 修改默认的键,默认为username和password
                .usernameParameter("uname")
                .passwordParameter("passwd")
                // 前后端分离,登录成功的处理
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        //authentication里保存了登录成功的用户信息
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 200);
                        // 登录成功的用户信息
                        map.put("mes", authentication.getPrincipal());
                        // 返回一个json
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                 //前后端不分,页面跳转
                .successForwardUrl("成功跳转")
                //登录失败的处理
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 401);
                        if (e instanceof LockedException){
                            map.put("msg","账户被锁定请联系管理员!");
                        }else if(e instanceof CredentialsExpiredException){
                            map.put("msg","密码过期请联系管理员!");
                        }else if (e instanceof AccountExpiredException){
                            map.put("msg","账户过期请联系管理员!");
                        }else if(e instanceof DisabledException){
                            map.put("msg","账户被禁用请联系管理员!");
                        }else if (e instanceof BadCredentialsException){
                            map.put("msg","用户名密码输入错误,请重新输入!");
                        }
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                // 前后端不分跳转
                .failureForwardUrl("失败跳转")
                .permitAll()
                .and()
                // 关闭csrf
                .csrf().disable();
 
 
 
    }
}
```

 **登录接口为 "/login”，即可以直接调用“／login”接口，发起一个 POST 请求进行登录，登录参数中用户 名必须命名为 usemam巳，密码必须命名为 password，配置了 loginProcessingUrl 接口主要方便或者移动端调用登录接口 。最后还配置了 permitAll，表示和登录相关的接口都不需要认 证即可访问。** 

**anonymous() 允许匿名用户访问
permitAll() 无条件允许访问**

### 注销登录的配置:

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        //开启配置
        http.authorizeRequests()
                // 指定admin角色可以访问该路径
                .antMatchers("/admin/**").hasRole("admin")
                // 指定admin和user可以访问该路径
                .antMatchers("/user/**").hasAnyRole("admin", "user")
                // 剩下的请求登录之后就可以访问
                .anyRequest().authenticated()
                .and()
                // 表单登录的url,请求地址
                .formLogin()
                // url,请求地址
                .loginProcessingUrl("/dolog")
                // 登录页面
                .loginPage("login")
                // 修改默认的键,默认为username和password
                .usernameParameter("uname")
                .passwordParameter("passwd")
                // 前后端分离,登录成功的处理
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        //authentication里保存了登录成功的用户信息
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 200);
                        // 登录成功的用户信息
                        map.put("mes", authentication.getPrincipal());
                        // 返回一个json
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                 //前后端不分,页面跳转
                .successForwardUrl("成功跳转")
                //登录失败的处理
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        Map<String, Object> map = new HashMap<>();
                        map.put("status", 401);
                        if (e instanceof LockedException){
                            map.put("msg","账户被锁定请联系管理员!");
                        }else if(e instanceof CredentialsExpiredException){
                            map.put("msg","密码过期请联系管理员!");
                        }else if (e instanceof AccountExpiredException){
                            map.put("msg","账户过期请联系管理员!");
                        }else if(e instanceof DisabledException){
                            map.put("msg","账户被禁用请联系管理员!");
                        }else if (e instanceof BadCredentialsException){
                            map.put("msg","用户名密码输入错误,请重新输入!");
                        }
                        out.write(new ObjectMapper().writeValueAsString(map));
                        out.flush();
                        out.close();
                    }
                })
                .failureForwardUrl("失败跳转")
                // 任何角色可以访问
                .permitAll()
                .and()
                .logout()
                // 注销请求路劲
                .logoutUrl("/logout")
                // 注销成功的处理
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        out.write(new ObjectMapper().writeValueAsString(new ObjectMapper().writeValueAsString("注销成功!")));
                        out.flush();
                        out.close();
                    }
                })
                .logoutSuccessUrl("注销成功的跳转")
                .and()
                // 关闭csrf
                .csrf().disable();
    }
}
```

### 多HttpSecurity配置： 

**config 不需要继承 WebSecurityConfigurerAdapter, 在 MultiHttpSecurityConfig 中创建静态内部类继承 WebSecurityConfigurerAdapter 即可，静态 内部类上添加＠Configuration 注解和＠Order 注解，＠Order 注解表示该配直的优先级，数字 越小优先级越大，未加＠Order 注解的配直优先级最小。** 

```java
/**
 * @Description : 多HttpSecurity配置
 * @Author: Liruilong
 * @Date: 2019/12/24 17:01
 */
 
@Configuration
public class MultiHttpSecurity {
    @Bean
    PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }
    @Autowired
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("liruilong").password("123").roles("admin")
                .and()
                .withUser("liruilongs").password("123").roles("user");
    }
    @Configuration
    @Order(1)
    public static class AdminSecurityConfig extends WebSecurityConfigurerAdapter{
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            //开启配置
            http.antMatcher("/admin/**").authorizeRequests().anyRequest().hasAnyRole("admin")
        }
    }
    @Configuration
    public static class OtherSecurityConfig extends WebSecurityConfigurerAdapter{
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            //开启配置
            http.authorizeRequests().anyRequest().authenticated()
                    .and()
                    .formLogin()
                    .failureForwardUrl("/dolog")
                    .permitAll()
                    .and()
                    .csrf().disable();
        }
    }
}
```

### 密码加盐处理:

**通过BCryptPasswordEncoder生成密码密文,**

```java
void contextLoads() {
        for (int i = 0; i < 0; i++){
            BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
            System.out.println(encoder.encode("123"));
        }
    }
 
 
 @Bean
    PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }
 
替换为
@Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
```

### 方法安全:

**即该方法加一个权限,明确该方法是什么角色可以调用的.**

**开发者也可以通过注解来灵活地配置方法安全，要 使用相关注解，首先要通过＠EnableGloba!MethodSecurity 注解开启基于注解的安全配置：**
 

```java
@Configuration
//用于解锁注解。
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class MultiHttpSecurityConfig {
 
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
 
    @Autowired
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("javaboy").password("$2a$10$G3kVAJHvmRrr6sOj.j4xpO2Dsxl5EG8rHycPHFWyi9UMIhtdSH15u").roles("admin")
                .and()
                .withUser("江南一点雨").password("$2a$10$kWjG2GxWhm/2tN2ZBpi7bexXjUneIKFxIAaMYJzY7WcziZLCD4PZS").roles("user");
    }
 
    @Configuration
    @Order(1)
    public static class AdminSecurityConfig extends WebSecurityConfigurerAdapter{
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.antMatcher("/admin/**").authorizeRequests().anyRequest().hasAnyRole("admin");
        }
    }
 
    @Configuration
    public static class OtherSecurityConfig extends WebSecurityConfigurerAdapter{
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests().anyRequest().authenticated()
                    .and()
                    .formLogin()
                    .loginProcessingUrl("/doLogin")
                    .permitAll()
                    .and()
                    .csrf().disable();
        }
    }
}
```

 

```java
package com.liruilong.securityl.demo.service;
 
import org.springframework.security.access.annotation.Secured;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;
 
/**
 * @Description :
 * @Author: Liruilong
 * @Date: 2019/12/24 17:31
 */
@Service
public class userservlce {
    //
    @PreAuthorize("hasRole('admin')")
    public String admin(){
        return "hello admin";
    }
    @Secured("ROLB_user")
    public String users(){
        return "hello user";
    }
    @PreAuthorize("hasAnyRole('admin','user')")
    public String hello(){
        return "hello hello";
    }
}
```

代码解释： •

- **@Secured(”ROLE_ AD MIN＂）注解表示访问该方法需要 ADMIN 角色，注意这里需要在角色前加一个前缀“ROLE ’**
- **@PreAuthorize（”hasRole（’AD MIN’） and hasRole('DBA＇）”）注解表示访问该方法既需妥 ADMIN 角色又需要 DBA 角色。 •**
- **@PreAuthorize("hasAnyRole（’ADMIN','DBA','USER’）”）表示访问该方法需要 ADMIN、 DBA 或 USER 角色。**
- **•@PreAuthorize 和＠PostAuthorize 中都可以使用基于表达式的语法。**

# 基于数据库的认证

### 动态配置权限

- 配置自定义权限拦截withObjectPostProcessor
  - [![img](https://img-blog.csdnimg.cn/20200216114348376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200216114348376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)
- 定义自定义拦截逻辑,获取有当前访问路径权限的所有角色,返回角色数组.
  - **通过FilterlnvocationSecurityMetadataSource 接口中的 getAttributes 方法来确定一个请求需要哪些角色**
  - [![img](https://img-blog.csdnimg.cn/20200216114755243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200216114755243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)
- 自定义角色比对之后的逻辑.判断当前用户是否有访问角色.的权利,
  - **自定义 AccessDecisionManager 并重写 decide 方法，decide有三个参数,当前登录用户的信息,请求对象,上一个getAttribute返回的角色数组.**
  - [![img](https://img-blog.csdnimg.cn/20200216124415775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200216124415775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)

# [![img](https://img-blog.csdnimg.cn/20200213122344351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200213122344351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70) [![img](https://img-blog.csdnimg.cn/20200213121307187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200213121307187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)

```java
package com.liruilong.hros.model;
 
 
import com.fasterxml.jackson.annotation.JacksonInject;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonIgnoreType;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
 
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
 
public class Hr implements UserDetails {
    private Integer id;
 
    private String name;
 
    private String phone;
 
    private String telephone;
 
    private String address;
 
    private Boolean enabled;
 
    private String username;
 
    private String password;
 
    private String userface;
 
    private String remark;
 
    private List<Role> roles;
 
    public Integer getId() {
        return id;
    }
 
    public void setId(Integer id) {
        this.id = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name == null ? null : name.trim();
    }
 
    public String getPhone() {
        return phone;
    }
 
    public void setPhone(String phone) {
        this.phone = phone == null ? null : phone.trim();
    }
 
    public String getTelephone() {
        return telephone;
    }
 
    public void setTelephone(String telephone) {
        this.telephone = telephone == null ? null : telephone.trim();
    }
 
    public String getAddress() {
        return address;
    }
 
    public void setAddress(String address) {
        this.address = address == null ? null : address.trim();
    }
 
 
 
    public void setEnabled(Boolean enabled) {
        this.enabled = enabled;
    }
 
    @Override
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username == null ? null : username.trim();
    }
 
    @Override
    public String getPassword() {
        return password;
    }
 
    public void setPassword(String password) {
        this.password = password == null ? null : password.trim();
    }
 
    public String getUserface() {
        return userface;
    }
 
    public void setUserface(String userface) {
        this.userface = userface == null ? null : userface.trim();
    }
 
    public String getRemark() {
        return remark;
    }
 
    public void setRemark(String remark) {
        this.remark = remark == null ? null : remark.trim();
    }
 
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
 
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
 
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
 
    @Override
    public boolean isEnabled() {
        return enabled;
    }
 
 
    @Override
    @JsonIgnore
    public Collection<? extends GrantedAuthority> getAuthorities() {
      List<SimpleGrantedAuthority> authorities = new ArrayList<>(roles.size());
      roles.stream().forEach( (role) ->authorities.add(new SimpleGrantedAuthority(role.getName())));
        return authorities;
    }
 
    public List<Role> getRoles() {
        return roles;
    }
 
    public void setRoles(List<Role> roles) {
        this.roles = roles;
    }
}
```

[![img](https://img-blog.csdnimg.cn/20200216112457918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200216112457918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)

**要实现动态配置权限，首先要自定义 FilterlnvocationSecurityMetadataSource,自定义权限拦截,获取当前请求的所有角色**, **Spring Security 中通过 FilterlnvocationSecurityMetadataSource 接口中的 getAttributes 方法来确定一个请求需要哪些 角色， FilterlnvocationSecurityMetadataSource 接口的默认实现类是 DefaultFilterlnvocationSecurityMetadataSource ，参考 DefaultFilterlnvocationSecurityMetadataSource 的实现，开发者可以定义自己的 FilterlnvocationSecurityMetadataSource，**

**`SecurityMetadataSource`是`Spring Security`的一个概念模型接口。用于表示对受权限保护的"安全对象"的权限设置信息。一个该类对象可以被理解成一个映射表，映射表中的每一项包含如下信息 :**

- **安全对象**
- **安全对象所需权限信息**

**围绕该映射表，**

```java
package com.liruilong.hros.config;
 
import com.liruilong.hros.mapper.MenuMapper;
import com.liruilong.hros.model.Menu;
import com.liruilong.hros.model.Role;
import com.liruilong.hros.service.MenuService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.access.SecurityConfig;
import org.springframework.security.web.FilterInvocation;
import org.springframework.security.web.access.intercept.FilterInvocationSecurityMetadataSource;
import org.springframework.stereotype.Component;
import org.springframework.util.AntPathMatcher;
 
import java.util.Collection;
import java.util.List;
 
/**
 * @Description : 权限处理,根据请求,分析需要的角色,该类的主要功能就是通过当前的请求地址，获取该地址需要的用户角色
 * @Author: Liruilong
 * @Date: 2019/12/24 12:17
 */
@Component
public class CustomFilterInvocationSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {
    @Autowired
    MenuService menuService;
    //路径比较工具
    AntPathMatcher antPathMatcher = new AntPathMatcher();
    /**
     * @return java.util.Collection<org.springframework.security.access.ConfigAttribute>
     * @Author Liruilong
     * @Description 当前请求需要的角色
     * @Date 18:13 2019/12/24
     * @Param [object]
     **/
    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {
        //获取当前请求路径
        String requestUrl = ((FilterInvocation) object).getRequestUrl();
        //获取所有的菜单url路径
        List<Menu> menus = menuService.getAllMenusWithRole();
        for (Menu menu : menus) {
            if (antPathMatcher.match(menu.getUrl(), requestUrl)) {
                //拥有当前菜单权限的角色
                List<Role> roles = menu.getRoles();
                String[] strings = new String[roles.size()];
                for (int i = 0; i < roles.size(); i++) {
                    strings[i] = roles.get(i).getName();
                }
                return SecurityConfig.createList(strings);
            }
        }
        // 没匹配上的资源都是登录
        return SecurityConfig.createList("ROLE_LOGIN");
    }
    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }
 
    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }
}
```

###  **开发者自定义 FilterlnvocationSecurityrMetadataSource** :

**主要实现该接口中的 getAttributes 方法， 该方法的参数是一个 FilterInvocation， 开发者可以从 Filterlnvocation 中提取出当前请求的 URL，返回值是 Collection<ConfigAttribute>，表示当前请求 URL 所需的角色**。

- 创建一个 AntPathMatcher，主要用来实现 ant 风格的 URL 匹配。
- 从参数中提取出当前请求的 URL。
- 从数据库中获取所有的资源信息，即本案例中的 menu 表以及 menu 所对应的 role,
- 追历资源信息，边历过程中获取当前请求的 URL 所需要的角色信息并返回。如 果当前请求的 URL 在资源表中不存在相应的模式，就假设该请求登录后即可访问，即直接返代码解释： ROLE LOGJN。
- getAllConfigAttributes 方法用来返回所有定义好的权限资源， Spring Security 在启动时会校验 相关配置是否正确，如果不需要校验，那么该方法直接返回 null 即可。 supports 方法返回类对象是否支持校验。 

###  自定义 AccessDecisionManager

**当一个请求走完 FilterlnvocationSecurityMetadataSource 中的 getAttributes 方法后，接下来就会 来到 AccessDecisionManager 类中进行角色信息的比对，自定义 AccessDecisionManager 如下：**
 

```java
package com.liruilong.hros.config;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.AccessDecisionManager;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.authentication.AnonymousAuthenticationToken;
import org.springframework.security.authentication.InsufficientAuthenticationException;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.stereotype.Component;
 
import java.util.Collection;
 
/**
 * @Description : 判断当前用户是否具备菜单访问，当一个请求走完 FilterlnvocationSecurityMetadataSource 中的 getAttributes 方法后，接下来就会 来到 AccessDecisionManager 类中进行角色信息的比对
 * @Author: Liruilong
 * @Date: 2019/12/24 19:12
 */
@Component
public class CustomUrlDecisionManager implements AccessDecisionManager {
 
    /**
     * @return void
     * @Author Liruilong
     * @Description decide 方法有三个参数， 第一个参数包含当前登录用户的信息；
     * 第二个参数则是一个 Filterlnvocation 对 象 ，可以 获 取当前请求对 象等；
     * 第 三个参 数就是 FilterlnvocationSecurityMetadataSource 中的 getAttributes 方法的返回值， 即当前请求 URL 所 需要的角色。
     * @Date 18:28 2020/2/13
     * @Param [authentication, object, configAttributes]
     **/
 
    @Override
    public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
            throws AccessDeniedException, InsufficientAuthenticationException {
        for (ConfigAttribute configAttribute : configAttributes) {
            String needRole = configAttribute.getAttribute();
            if ("ROLE_LOGIN".equals(needRole)) {
                //判断用户是否登录
                if (authentication instanceof AnonymousAuthenticationToken) {
                    throw new AccessDeniedException("尚未登录，请登录!");
                } else {
                    return;
                }
            }
            Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
            for (GrantedAuthority authority : authorities) {
                if (authority.getAuthority().equals(needRole)) {
                    return;
                }
            }
        }
        throw new AccessDeniedException("权限不足，请联系管理员!");
    }
 
    @Override
    public boolean supports(ConfigAttribute attribute) {
        return true;
    }
 
    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }
}
// authorities.stream().anyMatch((authority) ->authority.getAuthority().equals(attribute));
```

**自定义 AccessDecisionManager 并重写 decide 方法，**

**在该方法中判断当前登录的用户是否具 备当前请求 URL 所需要的角色信息，如果不具备，就抛出 AccessDeniedException 异常，否 则不做任何事即可。**

**decide 方法有三个参数，**

**第一个参数包含当前登录用户的信息；**

**第二个参数则是一个 Filterlnvocation 对 象 ，可以 获 取当前请求对 象等；**

**第 三个参 数就是 FilterlnvocationSecurityMetadataSource 中的 getAttributes 方法的返回值， 即当前请求 URL 所 需要的角色。**

**进行角色信息对比，如果需要的角色是 ROLE_LOG，说明当前请求的 URL 用 户登录后即可访问，如果 auth 是 UsemamePasswordAuthenticationToken 的实例，那么说明当前用户已登录，该方法到此结束，否则进入正常的判断流程，如果当前用户具备当前请求需 要的角色，那么方法结束。**

**springSecurity配置流程分析:**

[![img](https://img-blog.csdnimg.cn/20200216171913215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200216171913215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)

[![img](https://img-blog.csdnimg.cn/20200216171937340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200216171937340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Nhbmhld3V5YW5n,size_16,color_FFFFFF,t_70)

```java
package com.liruilong.hros.config;
 
import com.fasterxml.jackson.databind.ObjectMapper;
import com.liruilong.hros.filter.VerifyCodeFilter;
import com.liruilong.hros.model.Hr;
import com.liruilong.hros.model.RespBean;
import com.liruilong.hros.service.HrService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.*;
import org.springframework.security.config.annotation.ObjectPostProcessor;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.security.web.access.intercept.FilterSecurityInterceptor;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.security.web.authentication.logout.LogoutSuccessHandler;
 
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
 
/**
 * @Description :
 * @Author: Liruilong
 * @Date: 2019/12/18 19:11
 */
 
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    HrService hrService;
    @Autowired
    CustomFilterInvocationSecurityMetadataSource customFilterInvocationSecurityMetadataSource;
    @Autowired
    CustomUrlDecisionManager customUrlDecisionManager;
     @Autowired
    VerifyCodeFilter verifyCodeFilter ;
    @Autowired
    MyAuthenticationFailureHandler myAuthenticationFailureHandler;
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.addFilterBefore(verifyCodeFilter, UsernamePasswordAuthenticationFilter.class)
                .authorizeRequests()
                //.anyRequest().authenticated()
                //所有请求的都会经过这进行鉴权处理。返回当前请求需要的角色。
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O object) {
                        object.setSecurityMetadataSource(customFilterInvocationSecurityMetadataSource);
                        object.setAccessDecisionManager(customUrlDecisionManager);
                        return object;
                    }
                })
                .and().formLogin().usernameParameter("username").passwordParameter("password")
                //设置登录请求的url路径
                .loginProcessingUrl("/doLogin")
                /*需要身份验证时，将浏览器重定向到/ login
                我们负责在请求/ login时呈现登录页面
                当身份验证尝试失败时，将浏览器重定向到/ login？error（因为我们没有另外指定）
                当请求/ login？error时，我们负责呈现失败页面
                成功注销后，将浏览器重定向到/ login？logout（因为我们没有另外指定）
                我们负责在请求/ login？logout时呈现注销确认页面*/
                .loginPage("/login")
                //登录成功回调
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        Hr hr = (Hr) authentication.getPrincipal();
                        //密码不回传
                        hr.setPassword(null);
                        RespBean ok = RespBean.ok("登录成功！", hr);
                        //将hr转化为Sting
                        String s = new ObjectMapper().writeValueAsString(ok);
                        out.write(s);
                        out.flush();
                        out.close();
                    }
                })
                //登失败回调
                .failureHandler(myAuthenticationFailureHandler)
                //相关的接口直接返回
                .permitAll().and().logout()
                //注销登录
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(HttpServletRequest httpServletRequest,
                                                HttpServletResponse httpServletResponse,
                                                Authentication authentication) throws IOException, ServletException {
                        httpServletResponse.setContentType("application/json;charset=utf-8");
                        PrintWriter out = httpServletResponse.getWriter();
                        out.write(new ObjectMapper().writeValueAsString(RespBean.ok("注销成功!")));
                        out.flush();
                        out.close();
                    }
                })
                .permitAll().and().csrf().disable().exceptionHandling()
                //没有认证时，在这里处理结果，不要重定向
                .authenticationEntryPoint(
                        //myAuthenticationEntryPoint;
                        new AuthenticationEntryPoint() {
                    @Override
                    public void commence(HttpServletRequest req, HttpServletResponse resp, AuthenticationException authException) throws IOException, ServletException {
                        resp.setContentType("application/json;charset=utf-8");
                        resp.setStatus(401);
                        PrintWriter out = resp.getWriter();
                        RespBean respBean = RespBean.error("访问失败!");
                        if (authException instanceof InsufficientAuthenticationException) {
                            respBean.setMsg("请求失败，请联系管理员!");
                        }
                        out.write(new ObjectMapper().writeValueAsString(respBean));
                        out.flush();
                        out.close();
                    }
                });
    }
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(hrService);
    }
    /**
     * @Author Liruilong
     * @Description  放行的请求路径
     * @Date 19:25 2020/2/7
     * @Param [web]
     * @return void
     **/
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/auth/code","/login","/css/**","/js/**", "/index.html", "/img/**", "/fonts/**","/favicon.ico");
    }
}

```

SpringSecurity执行流程分析:

[![img](https://img-blog.csdn.net/20180318211512445?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3UwMTM0MzU4OTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)](https://img-blog.csdn.net/20180318211512445?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3UwMTM0MzU4OTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

UsernamePasswordAuthenticationFilter就是拦截我们通过表单提交接口提交的用户名和密码，如果是Basic提交的话，就会被BasicAuthenticationFilter拦截，最后的橙色FilterSecurityInterceptor是首先判断我们当前请求的url是否需要认证，如果需要认证，那么就看当前请求是否已经认证，是的话就放行到我们要访问的接口，否则重定向到认证页面。
 

UsernamePasswordAuthenticationFilter首先会拦截请求，而UsernamePasswordAuthenticationFilter是继承于AbstractAuthenticationProcessingFilter的，在这个抽象类中已经定义好了doFilter的方法，而里面有一个attemptAuthentication方法是由子类实现的。所以当提交表单时spring security会发现这个一个表单提交，然后就调用了UsernamePasswordAuthenticationFilter的doFilter方法

springSecurity其实就是一组过滤器，请求和响应都会经过这些过滤器，在系统启动的时候，spring boot会自动配上

黄色：已经存储的认证信息

绿色：处理用户身份认证

橙色：捕获黄色抛出的异常

蓝色：决定当前请求是否通过之前某个过滤器的身份认证，不能通过就抛出异常，通过了会帮我们直接跳转

[![img](https://img-blog.csdnimg.cn/20191205165135360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW5kZWFpNTIw,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20191205165135360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoYW5kZWFpNTIw,size_16,color_FFFFFF,t_70)

 

它的身份认证其实是始于访问资源开始。如果一个用户已登录，那么访问受保护的资源，则会校验该用户是否有权限访问。如果没有权限，则会调用权限拒绝的处理器进行处理。如果有权限，则能顺利访问该资源；

一个用户未登录情况下，也即匿名用户，访问受保护的资源时，spring security会首先检查该资源是否需要权限，如果需要权限，然后再检查，该资源是否是白名单里面。如果是白名单，也能正常访问。如果是受保护的资源，则会提示该用户需要登录。