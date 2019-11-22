---
title: 'Shiro 权限认证'
date: 2019-11-13 12:39:28
tags: [Shiro]
published: true
hideInList: false
feature: /post-images/shiro-quan-xian-ren-zheng.png
---
前面几篇文章中我们提到Shiro的几个主要功能中，其中还包括了权限认证，那么这一节 我们就简单体验一下Shiro的权限认证吧。
<!-- more -->
## 1、实现权限认证逻辑
基于前面的文章，自定义Realm继承AuthorizingRealm 需要实现doGetAuthenticationInfo 和 doGetAuthorizationInfo,其中doGetAuthenticationInfo主要用来身份认证，doGetAuthorizationInfo主要用来进行权限认证，具体代码如下：
前置代码：
```Java
    private static final Map<String,Set<String>> USER_ROLE= new HashMap<>();
    static {
        USER_ROLE.put("1",new HashSet<>(Arrays.asList("item:add","item:search","item:update","item:delete")));
        USER_ROLE.put("2",new HashSet<>(Arrays.asList("item:add","item:search")));
    }

    @Override
    public Set<String> getUserRoles(String userId) {
        Set<String> result = USER_ROLE.get(userId);
        return result!=null?result:Collections.EMPTY_SET;
    }
```
```Java
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        log.info("开始进行权限认证！");
        SimpleAuthorizationInfo simpleAuthorizationInfo=new SimpleAuthorizationInfo();
        String userId = (String)principalCollection.getPrimaryPrincipal();
        Set<String> userRoles = roleService.getUserRoles(userId);
        for (String userRole:userRoles){
            simpleAuthorizationInfo.addStringPermission(userRole);
        }
        return simpleAuthorizationInfo;
    }
```
## 2、配置LifecycleBeanPostProcessor、DefaultAdvisorAutoProxyCreator、AuthorizationAttributeSourceAdvisor
因为Shiro需要对Controller进行代理，所以需要开启Controller的代理，需要配置LifecycleBeanPostProcessor、DefaultAdvisorAutoProxyCreator、AuthorizationAttributeSourceAdvisor
```Java
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor(){
        return new LifecycleBeanPostProcessor();
    }
    @Bean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator(){
        DefaultAdvisorAutoProxyCreator creator=new DefaultAdvisorAutoProxyCreator();
        creator.setProxyTargetClass(true);
        return creator;
    }
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor=new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }
```
## 3、配置请求权限码
通过注解配置权限信息，@RequiresRoles、@RequiresPermissions 以@RequiresPermissions为例：
```Java
    @RequiresPermissions("item:search")
    @GetMapping("/search")
    public ResultVo<String> search(){
        return ResultVo.success("search");
    }
    @RequiresPermissions("item:add")
    @GetMapping("/add")
    public ResultVo<String> add(){
        return ResultVo.success("add");
    }
    @RequiresPermissions("item:update")
    @GetMapping("/update")
    public ResultVo<String> update(){
        return ResultVo.success("update");
    }
    @RequiresPermissions("item:delete")
    @GetMapping("/delete")
    public ResultVo<String> delete(){
        return ResultVo.success("delete");
    }
```
## 4、演示效果
用户登陆：
![登陆用户](https://bofengsun.github.io//post-images/1573745823357.png)
有权限：
![有权限](https://bofengsun.github.io//post-images/1573745976892.png)
无权限：
![无权限](https://bofengsun.github.io//post-images/1573746081151.png)