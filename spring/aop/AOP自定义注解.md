# AOP自定义注解

## 依赖

```
        <!--aop依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <!--声明式注解依赖-->
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>21.0</version>
        </dependency>
```

## 自定义注解类

```
package com.dabatase_class.aop;

import java.lang.annotation.*;

/**
 * @author Mapleplane
 * @Data 2020/8/31 12:54
 * @description 定义声明式注解
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
//@Inherited
//@Mapping
public @interface CheckRole {
    String[] role();
}

```

## 实现声明式注解

```
package com.dabatase_class.aop;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.dabatase_class.entity.User;
import com.dabatase_class.service.Impl.UserServiceImpl;
import com.dabatase_class.util.JwtTokenUtil;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.util.ArrayList;
import java.util.List;

/**
 * @author Mapleplane
 * @Data 2020/8/31 13:49
 * @description
 */
@Aspect
@Component
public class JWTAspect {


    @Autowired
    JwtTokenUtil jwtTokenUtil;

    @Autowired
    UserServiceImpl userService;

    @Autowired
    User_RoleServiceImpl user_roleService;

    @Autowired
    MerchantServiceImpl merchantService;

    private String[] Role={"SHARED","LITTLE","BIG","CONSUME","ADMIN"};


    @Pointcut("@annotation(com.example.fenxiao2.aop.CheckRole)")
    public void addAdvice(){}


    @Around("@annotation(checkRole)")
    public Object Interceptor(ProceedingJoinPoint joinPoint, CheckRole checkRole) throws WrongRoleException,Throwable {
        Signature signature=joinPoint.getSignature();
        System.out.println("检验开始"+signature.getName());
           String []roles=checkRole.role();
           Object param=joinPoint.getArgs()[0];//获取第一个参数
           HttpServletRequest request=(HttpServletRequest)param;
           String headers=request.getHeader("Authorization");
           String token=headers.substring("Bearer".length());
           String username=jwtTokenUtil.getUserNameFromToken(token);
           User user=userService.getone(username);
           if(user!=null) {
               QueryWrapper<User_Role> wrapper = new QueryWrapper<>();
               wrapper.eq("user_id", user.getId());
               User_Role user_role = user_roleService.getOne(wrapper);
               List<Integer> role_id = new ArrayList<>();
               for (int i = 0; i < Role.length; i++) {
                   for (int j = 0; j < roles.length; j++) {
                       if (roles[j].equals(Role[i])) {
                           role_id.add(i + 1);
                           break;
                       }
                   }
               }
               boolean have = false;
               for (int i = 0; i < role_id.size(); i++) {
                   if (role_id.get(i) == user_role.getRoleId()) {
                       have = true;
                       break;
                   }
               }
               if (have) {
                   System.out.println("角色校验成功");
                   return joinPoint.proceed();
               } else {
                   System.out.println("角色校验失败");
                   throw new WrongRoleException("角色校验失败", 403);
               }
           }
           else{
               QueryWrapper<Merchant> wrapper=new QueryWrapper<>();
               wrapper.eq("account",username);
               Merchant merchant=merchantService.getOne(wrapper);
               boolean have=false;
               if(merchant!=null){
                   for(int i=0;i<roles.length;i++)
                   {
                       if(roles[i].equals("CONSUME")){
                           have=true;
                           break;
                       }
                   }
                   if(have){
                       System.out.println("角色校验成功");
                       return joinPoint.proceed();
                   }else {
                       System.out.println("角色校验失败");
                       throw new WrongRoleException("角色校验失败", 403);
                   }
               }
               else{
                   System.out.println("角色校验失败");
                   throw new WrongRoleException("角色校验失败", 403);
               }
           }

    }
}

```

