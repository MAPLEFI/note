# 腾讯云COS存储

## 依赖

```
 <dependency>
            <groupId>com.qcloud</groupId>
            <artifactId>cos_api</artifactId>
            <version>5.6.8</version>
        </dependency>
```

## 主要代码

```
package com.example.fenxiao2.utils;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.util.Calendar;
import java.util.UUID;

import com.qcloud.cos.COSClient;
import com.qcloud.cos.ClientConfig;
import com.qcloud.cos.auth.BasicCOSCredentials;
import com.qcloud.cos.auth.COSCredentials;
import com.qcloud.cos.exception.CosClientException;
import com.qcloud.cos.exception.CosServiceException;
import com.qcloud.cos.model.ObjectMetadata;
import com.qcloud.cos.model.PutObjectRequest;
import com.qcloud.cos.model.PutObjectResult;
import com.qcloud.cos.model.StorageClass;
import com.qcloud.cos.region.Region;
import com.qcloud.cos.transfer.TransferManager;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpSession;

/**
 * @author: py
 * @version:2019年6月10日 下午9:52:06
 * @Desc 腾讯云  cos对象存储
 */
public class TencentCosUtil_py {

    static String bucketName = "youxiangtu-1302963009"; //桶的名称
    static String region = "ap-nanjing";//区域北京则  beijing
    static String appId = "1302963009"; //APPID
    static COSCredentials cred = null;
    static TransferManager transferManager = null;
    static ClientConfig clientConfig = null;
    static String key="youxiangtu";
    static String SecretKey="c5IEtQ3TdV7VnsgNrRjwpR8Y12qfDwB6";
    static String SecretId="AKIDz0lGUEuuqV25rMeIjJdkbroZKfrqtmkS";
    static String qianzui="youxiangtu";
    static String path="https://youxiangtu-1302963009.cos.ap-nanjing.myqcloud.com";
    static {
        // 1 初始化用户身份信息(secretId, secretKey)
        //SecretId 是用于标识 API 调用者的身份
        String SecretId = "AKIDz0lGUEuuqV25rMeIjJdkbroZKfrqtmkS";
        //SecretKey是用于加密签名字符串和服务器端验证签名字符串的密钥
        String SecretKey = "c5IEtQ3TdV7VnsgNrRjwpR8Y12qfDwB6";
        cred = new BasicCOSCredentials(SecretId, SecretKey);
        // 2 设置bucket的区域,
        clientConfig = new ClientConfig(new Region(region));
    }


    public static String Upload(MultipartFile file){

        if(file == null){

            return null;

        }

        String oldFileName = file.getOriginalFilename();

        String eName = oldFileName.substring(oldFileName.lastIndexOf("."));

        String newFileName = UUID.randomUUID()+eName;

        Calendar cal = Calendar.getInstance();

        int year = cal.get(Calendar.YEAR);

        int month=cal.get(Calendar.MONTH);

        int day=cal.get(Calendar.DATE);

        // 1 初始化用户身份信息(secretId, secretKey)

        COSCredentials cred = new BasicCOSCredentials(SecretId, SecretKey);

        // 2 设置bucket的区域, COS地域的简称请参照 https://cloud.tencent.com/document/product/436/6224

        ClientConfig clientConfig = new ClientConfig(new Region(region));

        // 3 生成cos客户端

        COSClient cosclient = new COSClient(cred, clientConfig);

        // bucket的命名规则为{name}-{appid} ，此处填写的存储桶名称必须为此格式


        // 简单文件上传, 最大支持 5 GB, 适用于小文件上传, 建议 20 M 以下的文件使用该接口

        // 大文件上传请参照 API 文档高级 API 上传

        File localFile = null;

        try {

            localFile = File.createTempFile("temp",null);

            file.transferTo(localFile);

            // 指定要上传到 COS 上的路径

            String key = "/"+qianzui+"/"+year+"/"+month+"/"+day+"/"+newFileName;

            PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, key, localFile);

            PutObjectResult putObjectResult = cosclient.putObject(putObjectRequest);

            return path + putObjectRequest.getKey();

        } catch (IOException e) {

            return null;

        }finally {

            // 关闭客户端(关闭后台线程)

            cosclient.shutdown();

        }

    }

}
```

