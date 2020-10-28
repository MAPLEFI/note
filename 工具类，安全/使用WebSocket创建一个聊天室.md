# 使用WebSocket创建一个聊天室

![image-20200504133535552](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200504133535552.png)

## 依赖

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
```

![image-20200504133712610](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200504133712610.png)

## WebSocketServer核心

### websocket配置

```
@Configuration
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    /**
     *  配置一个sockjs连接的端口，并配置跨域
     * @param registry
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").setAllowedOrigins("*").withSockJS();//中间的*是用来websocket跨域的，注册端点
    }


    @Bean
    public ServerEndpointExporter  serverEndpointExporter(){
        return new ServerEndpointExporter();
    }//配置websocket
}
```

### websocket控制连接时的处理

```
@Controller
public class MyWebSocketController {

    @GetMapping("/socket/{id}")
    public ModelAndView socket(@PathVariable String id)
    {
        ModelAndView modelAndView=new ModelAndView("/socket");
        modelAndView.addObject("id",id);
        return modelAndView;
    }

     @ResponseBody
     @RequestMapping("/socket/push")
    public String pushToWeb(String id,String message){
        try{
            WebSocketServer.sendInfo(message,id);
        }
        catch (Exception e)
        {
            e.printStackTrace();
            return "推送失败";
        }
        return "发送成功";
    }
}

```

### websocket具体的监听各个事件和发送消息处理

```
package com.spring.websocket.serevice;


import com.alibaba.fastjson.JSONObject;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.CopyOnWriteArraySet;

@ServerEndpoint("/websocket/{sid}")
@Service
@Slf4j
public class WebSocketServer {
  private static Integer onlineCount=0;
  private static CopyOnWriteArraySet<WebSocketServer> websocketSet=new CopyOnWriteArraySet<>();
  private Session session;
  private String sid="";//当前用户名
 @OnOpen
  public void Onopen(Session session,@PathParam("sid") String sid){
     boolean status=true;
     for(WebSocketServer webSocketServer:websocketSet)
     {
         if(webSocketServer.sid.equals(sid))
         {
             status=false;
             break;
         }
     }
     if(status==false)
     {
         String res="连接失败"+sid+"已经被注册了";
         try {
             sendMessage(res);
         }
         catch (Exception e)
         {
             e.printStackTrace();
         }
         return ;
     }
      this.session=session;
      websocketSet.add(this);
      addOnlineCount();
      log.info("有新窗口"+sid+"进入监视"+"当前人数为："+getOnlineCount());
      this.sid=sid;
      try{
          String res="连接成功,"+sid+"进入当前人数为:"+String.valueOf(getOnlineCount());
          sendMessage(res);
        }
      catch (Exception e)
      {
          e.printStackTrace();
      }
  }


  @OnClose
  public void Onclose(){
      subOnlineCount();
      String res=sid + "离场,当前人数为:" + getOnlineCount();
      try {
          sendMessage(res);
      }
      catch (Exception e)
      {
          e.printStackTrace();;
      }
     websocketSet.remove(this);
     log.info(sid+"离场"+"当前人数为"+getOnlineCount());
  }

  @OnMessage
  public void Onmessage(String message,Session session)
  {
      log.info("收到窗口"+sid+"的消息："+message);
      String [] arr=message.split("@#");
      String id="";
      Integer status=0;
      if(arr[0].startsWith("publish"))
      {
          message=arr[1];
          log.info("发送给全体的消息为："+message);
      }
      else if(arr[0].startsWith("sign"))
      {
          id=arr[0].substring(4,arr[0].length());
          log.info("发送消息至"+id);
          message=arr[1];
          status=1;
      }
      log.info("消息无误");
      String res=sid+"发送消息:"+arr[1];
      if(status==0) {
          for (WebSocketServer web : websocketSet) {
              try {
                  web.sendMessage(res);

              } catch (Exception e) {
                  e.printStackTrace();
              }
          }
      }
      else {
          try {
              WebSocketServer.sendInfo(res, id);
          }
          catch (Exception e)
          {
              e.printStackTrace();
          }
      }
  }
  @OnError
  public void Onerror(Session session,Throwable error)
  {
      log.error("发生错误");
      error.printStackTrace();
  }

  public void sendMessage(String message) throws IOException, EncodeException {
     this.session.getBasicRemote().sendText(message);
  }

    /**
     * 实现群发消息，当sid为null时全部推送
     * @param message
     * @param sid
     * @throws IOException
     */
  public static void sendInfo(String message,@PathParam("sid")String sid) throws IOException
  {
      log.info(sid+"推送到消息窗口"+",推送消息为:"+message);
      for(WebSocketServer web:websocketSet)
      {
          try{
              if(sid==null)
              {
                  web.sendMessage(message);
              }
              else if(web.sid.equals(sid))
              {
                  web.sendMessage(message);
              }
          }
          catch (Exception e)
          {
             continue;
          }
      }
  }

  public void addOnlineCount(){
     WebSocketServer.onlineCount++;
  }
  public void subOnlineCount(){
     WebSocketServer.onlineCount--;
  }
  public Integer getOnlineCount(){
     return onlineCount;
  }
}

```

### 在启动类中加入支持websocket启动

```
@EnableWebSocketMessageBroker//开启websocket支持
@SpringBootApplication
public class WebsocketApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebsocketApplication.class, args);
    }

}
```

## 开启websocket wss端口（微信中必须使用wss，就像https一样）

端点改成wss之后需要在启动类中加上：

```
    @Bean
    public TomcatContextCustomizer tomcatContextCustomizer() {
        System.out.println("TOMCATCONTEXTCUSTOMIZER INITILIZED");
        return new TomcatContextCustomizer() {
            @Override
            public void customize(Context context) {
                context.addServletContainerInitializer(new WsSci(), null);
            }

        };
    }
```

### 和https相同需要配置证书，他和https的证书相同可以共用

![image-20200608165003251](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200608165003251.png)

#### 配置证书

```
server:
  port: 8083
  ssl:
    key-store: classpath:3741759_wx.tourci.cn.pfx
    key-store-type: PKCS12
    key-store-password: q07AEEAI

```

## Websocket中消息传输使用对象传输

