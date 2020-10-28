# MybatisPlus

## 依赖

```
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.3.1.tmp</version>
        </dependency>
        
        引入MybatisPlus依赖之后不要再次引入Mybatis依赖
```

## 快速开始

建立表，建立实体类

在启动类上 写上@MapperScan("")注解

然后编写Mapper类

```
public interface UserMapper extends BaseMapper<User> {

}
```

Mapper实体类中可以什么都不写，MybatisPlus会内置封装很多CRUD方法可以直接使用，比如我可以直接使用userMapper.selectList(null)即会查询出所有的User,XML都可不用写

## 注解

![image-20200519190114455](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519190114455.png)

常用的属性为value![image-20200519190454829](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519190454829.png)

type根据情况添加

![image-20200519191058971](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519191058971.png)

通常不使用该注解，由于在使用@TableName时已经和数据库对接好了，该注解一般来说只用于实体类中有不在数据库 中的属性时使用该注解的exist=false来解除该属性和数据库的绑定

![image-20200519191732453](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519191732453.png)

## CRUD接口

![image-20200519193331907](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519193331907.png)

如果想要自己在Service层里面写逻辑也可，相当于继承了原来的Service层，Plus打包好的接口还是可以调用

![image-20200519193639564](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519193639564.png)

![image-20200519193800340](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519193800340.png)

​    ![image-20200519193915073](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519193915073.png)

   ![image-20200519194151255](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519194151255.png)

![image-20200519195541085](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519195541085.png)

![image-20200519195816365](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519195816365.png)

![image-20200519195943131](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519195943131.png)

例如：

```
QueryWrapper<Clean> cleanQueryWrapper = new QueryWrapper<>();
cleanQueryWrapper.eq("room_id",roomId);
return ResultUtil.success(cleanService.page(new Page<>(pageNo,pageSize),cleanQueryWrapper));
```

![image-20200519200218221](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519200218221.png)

## Mapper CRUD接口

![image-20200519200410967](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519200410967.png)

![image-20200519200515067](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519200515067.png)

![image-20200519200534135](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519200534135.png)

![image-20200519200545189](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519200545189.png)