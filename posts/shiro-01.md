---
title: 'Shiro 初体验'
date: 2019-11-08 17:39:44
tags: [Shiro]
published: true
hideInList: false
feature: /post-images/shiro-01.png
---
Shiro 是一个Java安全框架，主要是进行身份认证，权限认证，密码加密，会话管理的一套框架，下面让我们一起来认识一下Shiro吧!
<!-- more -->
## 1、认识Shiro的功能
看图说话，下图中是Shiro官网给出的描述，很明显主要包括四点：
- Authentication(身份认证)
- Authorization(权限认证)
- Session Management(会话管理)
- Cryptography(密码学)
![Shiro功能](https://bofengsun.github.io//post-images/1573123546063.png)

## 2、关键概念认识
- Subject (org.apache.shiro.subject.Subject) 主体
  指正在访问的主体，通常指当前用户，亦可指爬虫、定时任务等。
- SecurityManager (org.apache.shiro.mgt.SecurityManager) 安全管理器
  安全管理器，是Shiro的核心，管理着所有的Subject，负责与Shiro的其他组件进行交互，类似与SpringMvc 中的DispatcherServlet。
- Realms 安全数据
  主要是储存Shiro的安全数据（用户、角色、权限等），SecurityManager验证用户是否登陆、是否拥有权限等需要从其中获取数据。
![](https://bofengsun.github.io//post-images/1573126718187.png)
## 3、初体验
```Java
//git clone https://github.com/apache/shiro.git 下载Shiro源码包括该代码。

public class Quickstart {

    private static final transient Logger log = LoggerFactory.getLogger(Quickstart.class);


    public static void main(String[] args) {

        //用来从配置文件初始化Shiro环境，一般不会用
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        //获取当前用户,从这以后的代码都比较重要
        Subject currentUser = SecurityUtils.getSubject();
        //获取Session
        Session session = currentUser.getSession();
        session.setAttribute("someKey", "aValue");
        String value = (String) session.getAttribute("someKey");
        if (value.equals("aValue")) {
            log.info("Retrieved the correct value! [" + value + "]");
        }

        // 判断用户是否经过身份认证，如果没用则进行身份认证
        if (!currentUser.isAuthenticated()) {
            //通过用户名密码进行登陆
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");   //设置记住我
            token.setRememberMe(true);
            try {
                //进行登陆
                currentUser.login(token);
            } catch (UnknownAccountException uae) {//用户不存在异常
                log.info("There is no user with username of " + token.getPrincipal());
            } catch (IncorrectCredentialsException ice) {//密码不正确异常
                log.info("Password for account " + token.getPrincipal() + " was incorrect!");
            } catch (LockedAccountException lae) {//用户被锁定异常
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            }
            // ... catch more exceptions here (maybe custom ones specific to your application?
            catch (AuthenticationException ae) {//认证异常，查看其子类发现Shiro提供的认证异常
                //unexpected condition?  error?
            }
        }

        log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");

        // 判断用户是否拥有角色
        if (currentUser.hasRole("schwartz")) {
            log.info("May the Schwartz be with you!");
        } else {
            log.info("Hello, mere mortal.");
        }

        //判断用户是否拥有权限
        //判断用户是否拥有winnebago:drive:eagle5权限
        //winnebago 类型，driver 行为，eagle5对象
        //如 当前用户拥有对张三（对象）这个用户（类型）的删除行为
        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                    "Here are the keys - have fun!");
        } else {
            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
        }

        //用户退出登陆
        currentUser.logout();

        System.exit(0);
    }
}
```