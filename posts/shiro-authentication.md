---
title: 'Shiro 身份认证'
date: 2019-11-10 18:57:42
tags: [Shiro]
published: true
hideInList: false
feature: /post-images/shiro-authentication.png
---

上一节 【Shiro 初体验】中我们认识了几个Shiro中重要的概念，以及一个QuickStart代码示例演示了Shiro的基本使用，也知道Shiro的几个主要功能，首先就是身份认证，那么下面我们就来看一下Shiro是如何进行身份认证的吧。
<!-- more -->
在上一节中我们看到Shiro的登陆，当时的用户信息是在配置文件中，但在实际项目中并不会这样，而是存储在数据库中那么，这种情况下Shiro又是怎样进行身份认证的呢？我们对上一章的代码做一个简单的修改，以了解认证的过程：
依赖包：
```XML
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.8</version>
        </dependency>
        <!--整合Shiro-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.4.0</version>
        </dependency>
    </dependencies>
```
前置代码：
```Java
/**
 * 统一维护返回的数据
 */
public enum Result{
    //默认返回数据
    SUCCESS(0,"请求成功！"),
    FAILURE(500,"请求失败！"),
    //认证授权部分
    LOGIN_FAILURE(501,"用户名或者密码错误！"),
    LOGIN_FAILURE_LOCK(501,"用户被锁定,请联系管理员！"),
    LOGIN_FAILURE_RELOGIN(502,"用户已经登陆,无须再次登陆！"),
    AUTH_FAILURE(511,"无权限！");

    private Integer code;
    private String msg;
    private Result(Integer code,String msg){
        this.code=code;
        this.msg=msg;
    }
    public Integer getCode() {
        return code;
    }
    public String getMsg() {
        return msg;
    }
}
/**
 * 统一返回的数据类型
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ResultVo<T> {
    private Integer code;
    private String msg;
    private T data;
    public static <T> ResultVo<T> success(T data){
        return new ResultVo<>(Result.SUCCESS.getCode(),Result.SUCCESS.getMsg(),data);
    }
    public static<T> ResultVo<T> failure(Result result){
        return new ResultVo<>(result.getCode(),result.getMsg(),null);
    }
}
```
Shiro的配置：Realm、SecurityManager、shiroFilter
- Relam:安全数据
- SecurityManager： Shiro 的大管家负责组件协调
- shiroFilter： Filter对哪些请求进行拦截
```Java
/**
 * Shiro 配置
 */
@Configuration
public class ShiroConfig {

    /**
     * 配置ShiroFilter
     * @param securityManager 注入SecurityManager
     * @return ShiroFilterFactoryBean
     */
    @Bean(name = "shiroFilter")
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        filterChainDefinitionMap.put("/auth/login", "anon");
        filterChainDefinitionMap.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    /**
     * 配置SecurityManager
     * @param customRealm 注入自定义Realm
     * @return SecurityManager
     */
    @Bean
    public SecurityManager securityManager(CustomRealm customRealm) {
        DefaultWebSecurityManager defaultSecurityManager = new DefaultWebSecurityManager();
        defaultSecurityManager.setRealm(customRealm);
        return defaultSecurityManager;
    }

    /**
     * 配置自定义的Realm
     * @return CustomRealm
     */
    @Bean
    public CustomRealm customRealm() {
        CustomRealm customRealm = new CustomRealm();
        return customRealm;
    }
}
```

登陆认证部分的代码：
```Java
/**
 * 认证控制器
 */
@RestController
@RequestMapping("/auth")
@Slf4j
public class AuthController {

    /**
     * 登陆
     * @param username 用户名
     * @param password 密码
     * @return 登陆结果
     */
    @PostMapping("/login")
    public ResultVo<String> login(@RequestParam("username") String username,@RequestParam("password") String password){
        Subject currentUser = SecurityUtils.getSubject();
        if (!currentUser.isAuthenticated()) {
            UsernamePasswordToken token = new UsernamePasswordToken(username, password);
            try {
                currentUser.login(token);
                return ResultVo.success(currentUser.getSession().getId().toString());
            }catch (LockedAccountException lae) {
                return ResultVo.failure(Result.LOGIN_FAILURE_LOCK);
            }
            catch (AuthenticationException ae) {
                return ResultVo.failure(Result.LOGIN_FAILURE);
            }
        }else {
            return ResultVo.failure(Result.LOGIN_FAILURE_RELOGIN);
        }
    }
}
```
流程分析：
1、先通过SecurityUtils#getSubject()获取当前用户
2、Subject#isAuthenticated()调用判断当前用户是否登陆
3、若没用进行登陆认证，则将用户的username和password为封装UsernamePasswordToken
4、调用Subject#login(token)进行登陆
5、通过自定义Realm进行身份认证，自定义Realm如果只是用来进行身份认证则只需继承org.apache.shiro.realm.AuthenticatingRealm即可，如果还需要进行鉴权，则需要继承org.apache.shiro.realm.AuthorizingRealm,这里我们就先继承AuthenticatingRealm。
查看Realm的前置代码：
```Java
/**
 * 用户实体信息
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class User {

    /**
     * 逻辑ID
     */
    private String id;
    /**
     * 用户名(唯一)
     */
    private String username;
    /**
     * 密码
     */
    private String password;
    /**
     * 状态是否锁定:1 是；0否
     */
    private Integer status;
}
/**
 * 用户服务
 */
public interface UserService {
    /**
     * 根据用户名获取用户
     * @param username 用户名
     * @return User
     */
    User getByUsername(String username);
}
@Service
public class UserServiceImpl implements UserService {
    /**
     * 初始化用户数据源
     */
    static final Map<String,User> USERS = new HashMap<>();
    static {
        USERS.put("admin",User.builder()
                .id("1")
                .username("admin")
                .password("123456")
                .status(0)
                .build());
        USERS.put("guest",User.builder()
                .id("2")
                .username("guest")
                .password("123456")
                .status(0)
                .build());
        USERS.put("lockeduser",User.builder()
                .id("2")
                .username("lockeduser")
                .password("123456")
                .status(1)
                .build());
    }

    @Override
    public User getByUsername(String username) {
        return USERS.get(username);
    }
}
/**
 * 常量类
 */
public interface Constant {
    /**
     * 用户被锁定状态
     */
    Integer USER_LOCKED=1;
}
/**
 * 自定义Realm
 */
@Slf4j
public class CustomRealm extends AuthenticatingRealm {

    @Autowired
    private UserService userService;

    /**
     * 实现该方法进行认证，在调用Subject#login的时候会进入该方法
     * @param authenticationToken 令牌
     * @return 认证的信息
     * @throws AuthenticationException 认证异常
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        log.info("开始进行身份认证！");
        UsernamePasswordToken usernamePasswordToken=(UsernamePasswordToken)authenticationToken;
        String username = usernamePasswordToken.getUsername();
        User user = userService.getByUsername(username);
        if(user==null){
            log.info("用户未找到！");
            throw new UnknownAccountException(Result.LOGIN_FAILURE.getMsg());
        }else if (Constant.USER_LOCKED.equals(user.getStatus())){
            log.info("用户被锁定！");
            throw new LockedAccountException(Result.LOGIN_FAILURE_LOCK.getMsg());
        }else {
            String principal=username;
            String credentials=user.getPassword();
            String realmName=getName();
            SimpleAuthenticationInfo simpleAuthenticationInfo=new SimpleAuthenticationInfo(principal,credentials,realmName);
            log.info("认证成功！");
            return simpleAuthenticationInfo;
        }
    }
}
```
