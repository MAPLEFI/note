# OSS上传文件到达服务器

## 首先需要依赖使得FileUtil工具类能够使用

```
依赖
    <!--FileUtils依赖用处Multipartfile和file的相互转化-->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.4</version>
        </dependency>


使用FiltUtil.copyInputStreamToFile(multipartfile.getInputStream(),file)讲multipartfile转成File类型，我们首先需要先创建一个File用于存储，使用完成后使用file.delete()来进行删除并判定是否成功删除

```

## 然后通过自己写OSSUtile来进行操作

```
@Data
public class OSSUtil {
    //SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
    private String endpoint = "http://oss-cn-beijing.aliyuncs.com";
    private String accessKeyId = "LTAI4GEB5VgbBcQuE7q4ezB3";
    private String accessKeySecret = "vgHKbTkV2tZgQMHzDJKsQr3VRiCMED";
    //private String fileName = "/Users/dalaoyang/Desktop/aliyun.jpeg";
    private String bucketName = "canteen-system";

    public  String upload(MultipartFile file) throws IOException {
        String fileName=file.getOriginalFilename();
        SimpleDateFormat sdf=new SimpleDateFormat("yyyyMMdd");
        System.out.println(fileName);
        //获取文件后缀名
        String suffixName=fileName.substring(fileName.lastIndexOf("."));
        //生成上传的文件名
        String objectName=sdf.format(new Date())+"/"+ UUID.randomUUID().toString()+"/"+suffixName;
        OSSClient ossClient=new OSSClient(endpoint,accessKeyId,accessKeySecret);
       // File file1=new File(fileName);
        File file1=new File("user//img");
        FileUtils.copyInputStreamToFile(file.getInputStream(),file1);
        ossClient.putObject(bucketName,objectName,file1);
        //设置有效期为10年
        Date expiration=new Date(System.currentTimeMillis()+3600*1000*24*3650);
        URL url=ossClient.generatePresignedUrl(bucketName,objectName,expiration);
        if(!file1.delete())
        {
            return "删除失败";
        }
        ossClient.shutdown();
        return url.toString();
    }
}

然后使用对象.方法的方式来完成操作
```

