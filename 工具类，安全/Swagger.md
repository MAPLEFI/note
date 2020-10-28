# Swagger

API文档与API定义同步更新，自动生成

直接运行可以在线测试API接口

再项目使用Swagger需要导入jar包

·swagger2

·ui



## SpringBoot集成Swagger



导入相关依赖：

```
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2<artifactId>
  <version>2.9.2</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui<artifactId>
  <version>2.9.2</version>
</dependency>
```

## 配置Swagger

```
需要放到容器中去，需要写一个配置类：
@Configuration
@EnableSwagger2  //开启Swagger2
public class SwaggerConfig{
//配置了Swagger的Docket的bean实例
    @Bean
    public Docket docket(){
    return new 
      //这里是为了覆盖掉原来默认的信息所做出的处理，使用自定义的apiInfo来覆盖原来的信息Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).
      select()
      //RequestHandlerSelectors,配置要扫描接口的方式
      //basePackage：工作上常用方法指定要扫描的包
      //withClassAnnotation:扫描类上的注解，参数是一个类上注解类比如RequestMapping.class，表示只扫描有该注解的类
      //withMethodAnnotationn:扫描方法上的注解，参数是方法上的注解类，表示只扫描有该注解的方法         .apis(RequestHandlerSelectors.basePackage("com.xihua.controller"))
      //过滤器,ant后面写过滤的路径
      .path(PathSelector.ant(""))
      .build();
      ;
  
    }
    
    
    private ApiInfo apiInfo(){
        //作者信息
        Contact contact =new Contact("青江","www.baidu.com","1309520478@qq.com")分别是作者名，作者团队地址和作者邮件
        return new AipInfo(
           "API文档标题信息",
           "API文档描述信息",
           "v1.0版本号",
           "团队网址链接",
           contact,
           "Apache2.0"，（默认不改动）
           “http://www.apache.org/licenses/LICENSE-2.0”,
           new ArrayList()
        );
    }
  

}
```

访问http://localhost:8080/swagger-ui.html#/即可查看到描述信息和接口信息

## 配置是否启动Swagger



默认启动，可以再select() build()这一套之外添加enable()里面写true和flase,表示是否开启Swagger

```
  @Bean
    public Docket docket(){
    return new 
     Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo())
      .enable(false)//判断是否启动Swagger默认为true即为启动，false即不启动
      .select()
      //RequestHandlerSelectors,配置要扫描接口的方式
      //basePackage：工作上常用方法指定要扫描的包
      //withClassAnnotation:扫描类上的注解，参数是一个类上注解类比如RequestMapping.class，表示只扫描有该注解的类
      //withMethodAnnotationn:扫描方法上的注解，参数是方法上的注解类，表示只扫描有该注解的方法         .apis(RequestHandlerSelectors.basePackage("com.xihua.controller"))
      //过滤器,ant后面写过滤的路径
      .path(PathSelector.ant(""))
      .build();
      ;
  
    }
```





## 最终体案例：

```
package com.xihua.config;

import com.sun.org.apache.xpath.internal.functions.FuncConcat;
import io.swagger.models.Contact;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@Configuration
public class Swagger2Config {
   // 需要放到容器中去，需要写一个配置类：
    @Configuration
    @EnableSwagger2  //开启Swagger2
    public class SwaggerConfig{
        //配置了Swagger的Docket的bean实例
        @Bean
        public Docket docket(){
            return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo())
                    //这里是为了覆盖掉原来默认的信息所做出的处理，使用自定义的apiInfo来覆盖原来的信息
                    .select()
                    //RequestHandlerSelectors,配置要扫描接口的方式
                    //basePackage：工作上常用方法指定要扫描的包
                    //withClassAnnotation:扫描类上的注解，参数是一个类上注解类比如RequestMapping.class，表示只扫描有该注解的类
                    //withMethodAnnotationn:扫描方法上的注解，参数是方法上的注解类，表示只扫描有该注解的方法         .apis(RequestHandlerSelectors.basePackage("com.xihua.controller"))
                    //过滤器,ant后面写过滤的路径
                    .apis(RequestHandlerSelectors.basePackage("com.xihua.controller"))
                    .build();


        }


        private ApiInfo apiInfo(){
            //作者信息
           // Contact contact =new Contact("啊啊啊啊","blog.csdn.net","aaa@gmail.com")
            return new ApiInfo(
                    "API文档标题信息",
                    "API文档描述信息",
                    "v1.0版本号",
                    "团队网址链接",
                    ApiInfo.DEFAULT_CONTACT,
                    "Apache2.0",
                    "http://www.apache.org/licenses/LICENSE-2.0",
            new ArrayList()
        );
        }
         //访问ttp://localhost:8080/swagger-ui.html#/

   }
}


```

## 如果想在shiro框架中放行Swagger2

```
        filterChainDefinitionMap.put("/swagger-ui.html/**","anon");
        filterChainDefinitionMap.put("/swagger-ui.html","anon");
        filterChainDefinitionMap.put("/swagger/**","anon");
        filterChainDefinitionMap.put("/webjars/**", "anon");
        filterChainDefinitionMap.put("/swagger-resources/**","anon");
        filterChainDefinitionMap.put("/v2/**","anon");
        filterChainDefinitionMap.put("/static/**", "anon");
```

## 项目中使用注解来丰富Swagger信息

```
在Controller上加上@Api(tags="")即可在Swagger里协商该Controller得说明
在Controller内部得Mapper方法上协商@ApiOperation("")即可在Swagger里看到对应得方法得说明
在Bean内部得属性上加上@ApiModelProperty（value=""）如果在Controller返回该类时会将写在各个属性上得说明显示出来
```

