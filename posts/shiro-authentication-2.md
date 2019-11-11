---
title: 'Shiro 身份认证（续）'
date: 2019-11-11 13:25:45
tags: [Shiro]
published: true
hideInList: false
feature: /post-images/shiro-authentication-2.png
---
上一节我们简单介绍了一下Shiro的身份认证，但其中有些细节之处我们没用讨论到，那么这一节中，我们就一些问题进行一下思考讨论。
<!-- more -->
## 一、如何和支持WEB、APP、以及其他方式进行身份认证？
先回顾一下上一节我们的登陆部分代码：
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
可以看到我们在登陆成功后，把用户的SessionId返回给客户端，但在传统WEB其实是不需要的，因为在认证成功后会将SessionId保存在cookie里JSESSIONID，那么如果我们需要兼容APP或者其他方式这样就行不通，一种比较简单的方式就是我们把SessionId返回给客户端，客户端保存一份，每次请求时，将其作为请求头请求服务端，同时服务端也要改变获取SessionId的方式，不是从cookie里获取，而是从header里获取，这样就可以解决该问题，那么下面我们就看一下在Shiro中该如何做吧。
1、自定义SessionManager继承DefaultWebSessionManager
```Java
/**
 * 自定义SessionManager
 */
@Slf4j
public class CustomWebSessionManager extends DefaultWebSessionManager{

    private static final String JSESSIONID="JSESSIONID";

    @Override
    public Serializable getSessionId(SessionKey key) {
        Serializable id = super.getSessionId(key);
        //如果从Cookie中没有获取到SessionId,则从请求头中获取
        if (id == null) {
            HttpServletRequest request = (HttpServletRequest)WebUtils.getRequest(key);
            id = request.getHeader(JSESSIONID);
        }
        return id;
    }
}
```
2、将自定义的SessionManager放入容器中
3、设置SecurityManager的SessionManager为自定义的SessionManager
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
        filterChainDefinitionMap.put("/openapi/**", "anon");
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
    public SecurityManager securityManager(CustomRealm customRealm, SessionManager sessionManager) {
        DefaultWebSecurityManager defaultSecurityManager = new DefaultWebSecurityManager();
        defaultSecurityManager.setSessionManager(sessionManager);
        defaultSecurityManager.setRealm(customRealm);
        return defaultSecurityManager;
    }

    /**
     * 配置自定义SessionManager
     * @return
     */
    @Bean("sessionManager")
    public CustomWebSessionManager customWebSessionManager(){
        return new CustomWebSessionManager();
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
4、客户端登陆成功后将sessionId保存起来，每次请求的时候带上该请求头
## 二、分布式环境下，如何来维护Session的呢？
那么这里就是如何统一维护Session的问题了，我们一般会将Session保存在缓存中间件里，例如Redis，这样即保证性能，又实现功能，下面我们就体验一下吧？
1、连接Redis以及编写RedisUtil（该部分仍是采用MAP进行处理，待后面和Redis继承时修改这里）
```Java
/**
 * Redis工具类
 */
public class RedisUtil {
    static Map<String,Object> SESSION_ID_CACHE=new HashMap<>();
    public static void set(String key,Object value){
        SESSION_ID_CACHE.put(key,value);
    }
    public static Object get(String key){
        return SESSION_ID_CACHE.get(key);
    }
    public static void del(String key){
        SESSION_ID_CACHE.remove(key);
    }
    public static Collection values(){
        return SESSION_ID_CACHE.values();
    }
}
```
2、编写自定义SessionDao 继承EnterpriseCacheSessionDAO 重写读取Session的方法
```Java
/**
 * 自定义SessionDAO从Redis中读取Session
 */
public class CustomSessionDAO extends EnterpriseCacheSessionDAO {
    @Override
    protected Session doReadSession(Serializable serializable) {
        return (Session) RedisUtil.get(serializable.toString());
    }
}
```
3、编写自定义SerssionListener 维护Session中的值
```Java
/**
 * 自定义SerssionListener 维护Session中的值
 */
public class CustomSessionListener implements SessionListener {
    @Override
    public void onStart(Session session) {
        RedisUtil.set(session.getId().toString(),session);
    }

    @Override
    public void onStop(Session session) {
        RedisUtil.del(session.getId().toString());
    }

    @Override
    public void onExpiration(Session session) {
        RedisUtil.del(session.getId().toString());
    }
}

```
3、将CustomSessionDAO、CustomSerssionListener加入Shiro配置中
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
        filterChainDefinitionMap.put("/openapi/**", "anon");
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
    public SecurityManager securityManager(CustomRealm customRealm, SessionManager sessionManager) {
        DefaultWebSecurityManager defaultSecurityManager = new DefaultWebSecurityManager();
        defaultSecurityManager.setSessionManager(sessionManager);
        defaultSecurityManager.setRealm(customRealm);
        return defaultSecurityManager;
    }

    /**
     * 配置自定义SessionManager
     * @return
     */
    @Bean("sessionManager")
    public CustomWebSessionManager customWebSessionManager(SessionDAO sessionDAO){
        CustomWebSessionManager customWebSessionManager=new CustomWebSessionManager();
        customWebSessionManager.setSessionDAO(sessionDAO);
        //配置自定义
        List<SessionListener> listeners=new ArrayList<>();
        listeners.add(new CustomSessionListener());
        customWebSessionManager.setSessionListeners(listeners);
        return customWebSessionManager;
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

    /**
     * 配置自定义SessionDAO
     * @return CustomSessionDAO
     */
    @Bean("sessionDAO")
    public CustomSessionDAO customSessionDao(){
        CustomSessionDAO customSessionDAO=new CustomSessionDAO();
        customSessionDAO.setSessionIdGenerator(new JavaUuidSessionIdGenerator());
        return customSessionDAO;
    }
}
```
## 三、基于Token的验证方式
现如今，很多Web应用的验证方式都会采用基于token的验证方式，实际上这种方式很像我们上面所说的将SessionId放入请求头中，只是基于token的这种方式，token的保存的信息可能更具有意义。那在Shiro中如何来实现呢？
1、登陆成功时，生成对应的token,并保存在redis中
2、编写认证服务器,继承FormAuthenticationFilter,并重写onAccessDenied，从请求头中获取获取TOKEN，并从redis中获取token，对比token即可。
## 四、统一认证中心
统一认证中心主要是进行统一的身份认证，主要是以下几个步骤：
1、请求认证中心时，需要将验证后需要跳转的URL作为请求参数；
2、服务端修改为不仅可以从Cookie和Header中获取对应的token或者sessionId，还需要可以从请求参数里获取到对应的值；
3、服务端认证成功后将token或者sessionId作为参数重定向到登陆认证的回调地址。