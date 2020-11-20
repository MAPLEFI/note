# Mybatis疑难杂症

## 在Mybatis里面使用like

```
Mybatis使用like        select * from message where post like CONCAT('%',#{post_name},'%')
```

## 配置文件

```
mybatis:
  #resources下的mapper的mybatis.xml是全局配置文件
  config-location: classpath:mapper/mybatis.xml
  #resources下的mapper下的DaoMapper文件夹下的所有xml为映射文件
  mapper-locations: classpath:mapper/DaoMapper/*.xml
  #Mybatis返回值的类再java里的位置
  type-aliases-package: com.biye.mall.bean
#  configuration:
#    map-underscore-to-camel-case: true
```

## resources层

![image-20201119223711222](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201119223711222.png)

### mybatis.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

</configuration>
```

### DaoMapper层

里面存取各个Dao层接口的实现SQL

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.biye.mall.dao.CartDao">
   <!-- public int  insert(Cart cart);
    public int delete(Integer id);
    public List<Cart> getall(@Param("user_id") Integer user_id,@Param("page") Integer page,@Param("limit")Integer limit);
    public List<Cart>getAll(Integer user_id);
    public Cart check(@Param("user_id")Integer user_id,@Param("commodity_id") Integer commodity_id);
    public int delete_userId(@Param("user_id") Integer user_id,@Param("commodity_id") Integer commodity_id);
    public int delete_commodity_id(@Param("commodity_id")Integer commodity_id);-->
    <select id="getall" resultType="com.biye.mall.bean.Cart">
        select * from cart where user_id=#{user_id} limit #{page},#{limit}
    </select>
    <select id="check" resultType="com.biye.mall.bean.Cart">
        select * from cart where user_id=#{user_id} and commodity_id=#{commodity_id}
    </select>
    <insert id="insert" parameterType="com.biye.mall.bean.Cart">
        insert into cart(user_id,commodity_id) values (#{user_id},#{commodity_id})
    </insert>
    <delete id="delete" parameterType="java.lang.Integer">
        delete from cart where id=#{id}
    </delete>
    <select id="getAll" resultType="com.biye.mall.bean.Cart">
        select * from cart where user_id=#{user_id}
    </select>
    <delete id="delete_userId" parameterType="java.lang.Integer">
        delete from cart where user_id=#{user_id} and commodity_id=#{commodity_id}
    </delete>
    <delete id="delete_commodity_id" parameterType="java.lang.Integer">
        delete from cart where commodity_id=#{commodity_id}
    </delete>
</mapper>
```

