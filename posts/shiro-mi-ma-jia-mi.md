---
title: 'Shiro 密码加密与会话管理介绍'
date: 2019-11-14 23:54:05
tags: []
published: true
hideInList: false
feature: /post-images/shiro-mi-ma-jia-mi.png
---
密码加密主要是在用户登录与注册的时候进行的，用户在注册时生成密码，通过加密存储密文到数据库中，用户在登录时候，将前台用户填写的密码再次进行加密与数据库中的密码进行比对，一致即可通过；会话管理主要时类似Session、Token的生命周期，下面我们来简单认识一下Shiro的密码加密和会话管理。
<!-- more -->
# Shiro 密码加密

## 0、前置代码:后面用到，以便理解
```Java
    /**
     * 加密次数
     */
    Integer HASH_ITERATIONS=1024;
    /**
     * 加密算法
     */
    String HASH_ALGORITHMNAME="MD5";
```
## 1、注册：注册和认证时密码需要用同样的策略，以保证密码密文的一致性。
```Java
    /**
     * 注册
     * @param registerVo
     * @return
     */
    @PostMapping("/register")
    public ResultVo<Boolean> register(@RequestBody LoginRegisterVo registerVo){
        //加密方式 可另行制定
        String algorithmName= Constant.HASH_ALGORITHMNAME;
        //密码明文
        Object source=registerVo.getPassword();
        //密码盐值 可另行制定
        Object salt=registerVo.getUsername();
        //加密次数 可另行制定
        int hashIterations=Constant.HASH_ITERATIONS;
        SimpleHash simpleHash = new SimpleHash(algorithmName, source, salt, hashIterations);
        //加密后密码
        String password = simpleHash.toHex();
        log.info(password);
        boolean flag=userService.addUser(registerVo.getUsername(),password);
        if(!flag){
            ResultVo.failure(Result.REGISTER_FAILURE);
        }
        return ResultVo.success(true);
    }
```
根据以上注册可生成密码，那么在登陆的时候，该如何进行匹配呢？下面我来看一下登录如何进行
## 2、认证：在进行用户认证(doGetAuthenticationInfo)时需要指定盐值，颜值需要唯一
```Java
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
            Object principal=user.getId();
            String credentials=user.getPassword();
            String realmName=getName();
            //指定盐值
            ByteSource credentialsSalt = ByteSource.Util.bytes(usernamePasswordToken.getUsername());
            SimpleAuthenticationInfo simpleAuthenticationInfo=new SimpleAuthenticationInfo(principal,credentials,credentialsSalt,realmName);
            log.info("用户认证！");
            return simpleAuthenticationInfo;
        }
    }
```
## 3、配置Realm:在Realm密码策略以及加密次数
```Java
    /**
     * 配置自定义的Realm
     * @return CustomRealm
     */
    @Bean
    public CustomRealm customRealm() {
        CustomRealm customRealm = new CustomRealm();
        HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher(Constant.HASH_ALGORITHMNAME);
        credentialsMatcher.setHashIterations(Constant.HASH_ITERATIONS);
        customRealm.setCredentialsMatcher(credentialsMatcher);
        return customRealm;
    }
```
## 4、扩展
实际上Shiro支持多种密码策略，具体可以查看org.apache.shiro.authc.credential.CredentialsMatcher的子类
# Shiro 会话管理
## 1、使用当前会话
```Java
    SecurityUtils.getSubject().getSession();
```
## 2、会话管理器(SessionManager):管理session包括创建、维护、删除、失效、验证等工作。
```Java
/**
 * 自定义SessionManager:从请求头中也可获取JSESSIONID
 */
@Slf4j
public class CustomWebSessionManager extends DefaultWebSessionManager{

    private static final String JSESSIONID="JSESSIONID";

    @Override
    protected Serializable getSessionId(ServletRequest request, ServletResponse response) {
        Serializable id = super.getSessionId(request,response);
        //如果从Cookie中没有获取到SessionId,则从请求头中获取
        if (id == null) {
            id=WebUtils.toHttp(request).getHeader(JSESSIONID);
        }
        return id;
    }
}
```
## 3、会话监听器(SessionListener):监听会话状态，触发对应的方法
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
## 4、会话增删改查接口(SessionDAO)
```Java
public interface SessionDAO {
    Serializable create(Session var1);

    Session readSession(Serializable var1) throws UnknownSessionException;

    void update(Session var1) throws UnknownSessionException;

    void delete(Session var1);

    Collection<Session> getActiveSessions();
}
```
