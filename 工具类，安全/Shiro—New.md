# Shiro

可以完成认证授权加密会话管理，Web继承，缓存等

## Shiro架构(外部)

![image-20200411161648907](C:\Users\Administrator\Desktop\image-20200411161648907.png)

1.获取当前用户对象Subject

2.通过当前用户拿到Sessionn

3.判断当前用户是否被认证，如果没认证就不能使用，如果已经认证就会返回一个token令牌

4.login(token)执行登录操作，异常判断：用户不存在，密码错误，用户被锁定

## Subject

```
Subject currentUser=SecurityUtils.getSubject();//获取当前的用户对象
Session session=currentUser.getSession();//通过当前用户拿到Session
currentUser.isAuthenticated();//是否被认证
currentUser.login(token);
currentUser.getPrincipal();//可以获得当前用户的一个认证
currentUser.hasRole();//该用户是否拥有角色
currentUser.isPermitted();//该用户是否拥有权限
currentUser.logout();//注销
```



## 编写配置类(注意开启cglib代理，否则可能会导致代理冲突使用注释时Controller无法使用)

```
@Configuration
public class ShiroConfig {

    @Bean
    public ShiroFilterFactoryBean shiroFilter(@Qualifier("securityManager")DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, Filter> filterMap = shiroFilterFactoryBean.getFilters();
        filterMap.put("authc", new MyAuthenticationFilter());
        //拦截器.
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<String,String>();
        filterChainDefinitionMap.put("/login/**","anon");
        filterChainDefinitionMap.put("/test/**","anon");
        filterChainDefinitionMap.put("/shir/**","anon");
        filterChainDefinitionMap.put("/**","authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        /**
         * 登录失败
         */
        shiroFilterFactoryBean.setLoginUrl("/test/login");
        /**
         * 没有授权
         */
        shiroFilterFactoryBean.setUnauthorizedUrl("/shir/msg");
        return shiroFilterFactoryBean;
    }


    @Bean
    public SessionManager sessionManager() {
        MySessionManage mySessionManager = new MySessionManage();
        return mySessionManager;
    }

    @Bean
    public RealmConfig realmConfig() {
        RealmConfig realmConfig = new RealmConfig();
        return realmConfig;
    }
    @Bean
    public DefaultWebSecurityManager securityManager(@Qualifier("realmConfig") RealmConfig realmConfig) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(realmConfig);
        securityManager.setSessionManager(sessionManager());
        return securityManager;
    }

    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(@Qualifier("securityManager") DefaultWebSecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }

    /**
     * Shiro生命周期处理器,管理Shrio的bean
     *
     * @return
     */
    @Bean
    public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    /**
     * 扫描上下文，寻找所有的Advistor(通知器）
     * 必须在LifecycleBeanPostProcessor之后创建
     *
     * @return
     */
    @Bean
    public static DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        // 强制使用cglib，防止重复代理和可能引起代理出错的问题
        // https://zhuanlan.zhihu.com/p/29161098
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }

}
```

## 配置RealmConfig

```
public class RealmConfig extends AuthorizingRealm {
    @Autowired
    UserService userService;
    @Autowired
    RoleService roleService;

//    @Override
//    public void setCredentialsMatcher(CredentialsMatcher credentialsMatcher) {
//        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
//        matcher.setHashAlgorithmName("SHA-256");
//        matcher.setHashIterations(1024);
//        super.setCredentialsMatcher(matcher);
//    }

    /**
     * 认证
     *
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String username = (String) authenticationToken.getPrincipal();
        User user = userService.getUser(username);
        if (user == null) {
            return null;
        }
        return new SimpleAuthenticationInfo(user.getUsername(), user.getPassword(), getName());
    }


    /**
     * 授权
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String username = (String) principalCollection.getPrimaryPrincipal();
        List<String> roleList = roleService.getRoleName(username);
        Set<String> roles = new HashSet(roleList);
        return new SimpleAuthorizationInfo(roles);
    }
}

```

## 前后端分离配置跨域

```
public class MyAuthenticationFilter extends FormAuthenticationFilter {
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        if (request instanceof HttpServletRequest) {
            if (((HttpServletRequest) request).getMethod().toUpperCase().equals("OPTIONS")) {
                return true;
            }
        }
        return super.isAccessAllowed(request, response, mappedValue);
    }
    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response)
            throws Exception {
        WebUtils.toHttp(response).sendError(HttpServletResponse.SC_UNAUTHORIZED);
        return false;
    }
}
配置再shiroFactoryBean里的FilterMap里面
 Map<String, Filter> filterMap = shiroFilterFactoryBean.getFilters();
        filterMap.put("authc", new MyAuthenticationFilter());
```



```
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    /**
     * 跨域配置
     * @param registry
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

## 配置每次请求验证token的类

```
@Configuration
public class MySessionManage extends DefaultWebSessionManager {

    private static final String AUTHORIZATION = "Authorization";

    private static final String REFERENCED_SESSION_ID_SOURCE = "Stateless request";

    public MySessionManage() {
        super();
    }

    @Override
    protected Serializable getSessionId(ServletRequest request, ServletResponse response) {
        String id = WebUtils.toHttp(request).getHeader(AUTHORIZATION);
        //如果请求头中有 Authorization 则其值为sessionId
        System.out.println(id);
        if (!StringUtils.isEmpty(id)) {
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE, REFERENCED_SESSION_ID_SOURCE);
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID, id);
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_IS_VALID, Boolean.TRUE);
            return id;
        } else {
            //否则按默认规则从cookie取sessionId
            return super.getSessionId(request, response);
        }
    }
}

```

## 配置异常处理（ControllerAdvice）

```
@ControllerAdvice
public class GlobalDefaultExceptionHandler {

    @ExceptionHandler(UnauthorizedException.class)
    @ResponseBody
    public ResultVo defaultExceptionHandler(HttpServletRequest req, Exception e){
        return ResultUtil.error("权限不足");
    }
}
```

