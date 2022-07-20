---
title: minIO的使用
description: minIO的安装，使用，java API
published: true
date: 2022-07-20T02:56:27.709Z
tags: 
editor: markdown
dateCreated: 2022-05-07T12:07:31.953Z
---

# 安装minIO

参考：https://www.hxstrive.com/subject/minio.htm

1、拉取docker镜像

```plaintext
>docker pull minio/minio
```

2、设置minio用到的文件路径

```plaintext
>mkdir minio
>cd minio
>mkdir data
>mkdir config
```

3、启动服务：注意，这里要单独设置console的端口，不然会报错，且无法访问

```plaintext
>docker run --name minio \
-p 9000:9000 \
-p 9999:9999 \
-d --restart=always \
-e "MINIO_ROOT_USER=admin" \
-e "MINIO_ROOT_PASSWORD=qq123456" \
-v /home/minio/data:/data \
-v /home/minio/config:/root/.minio \
minio/minio server /data \
--console-address '0.0.0.0:9999'
```

这种安装方式 MinIO 自定义 Access 和 Secret 密钥要覆盖 MinIO 的自动生成的密钥

4、 防火墙设置，需要为minio开放两个端口，一个9000端口，一个静态端口，此处为9999

```plaintext
>firewall-cmd --zone=public --add-port=9000/tcp --permanent
>firewall-cmd --zone=public --add-port=9999/tcp --permanent
>firewall-cmd --reload
```

5.、登录客户端（浏览器）：注意—>此处的端口，是你设置的console的端口：9999

此处的用户名密码为启动服务时，设置的用户名密码：`admin`  `qq123456`

# java API的使用

1、引入依赖

```xml
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>7.0.2</version>
</dependency>
```

2、配置

```java
minio:
    endpoint: http://124.221.239.207:9000
    bucketName: qqcloud
    accessKey: admin
    secretKey: qq123456
```

3、示例

```java
package com.qq.common.system.service.impl;

import com.qq.common.core.utils.DateUtils;
import com.qq.common.core.utils.file.ImageUtils;
import com.qq.common.core.utils.file.MimeTypeUtils;
import com.qq.common.system.service.MinIoService;
import com.qq.common.system.vo.FileVO;
import io.minio.MinioClient;
import io.minio.PutObjectOptions;
import io.minio.Result;
import io.minio.errors.InvalidEndpointException;
import io.minio.errors.InvalidPortException;
import io.minio.messages.DeleteError;
import io.minio.messages.Item;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletResponse;
import java.io.BufferedInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;

/**
 * @Description:
 * @Author Administrator
 * @Date 2022/5/7
 **/
@Service
@Slf4j
public class MinIoServiceImpl implements MinIoService {

    private MinioClient minioClient;


    @Value("${minio.endpoint}")
    private String ENDPOINT;
    @Value("${minio.bucketName}")
    private String BUCKET_NAME;
    @Value("${minio.accessKey}")
    private String ACCESS_KEY;
    @Value("${minio.secretKey}")
    private String SECRET_KEY;

    public MinioClient getMinioClient() throws InvalidPortException, InvalidEndpointException {
        if(minioClient == null){
            minioClient = new MinioClient(ENDPOINT, ACCESS_KEY, SECRET_KEY);
        }
        return minioClient;
    }
    @Override
    public String upload(MultipartFile file) {
        try {
            //创建一个MinIO的Java客户端
            MinioClient minioClient = getMinioClient();
            // 检查存储桶是否已经存在
            boolean isExist = minioClient.bucketExists(BUCKET_NAME);
            if (isExist) {
                log.info("存储桶已经存在！");
            } else {
                //创建存储桶
                minioClient.makeBucket(BUCKET_NAME);
            }
            String filename = file.getOriginalFilename();
            // 设置存储对象名称
            String objectName = DateUtils.getDate() + "/" + filename;
            // 使用putObject上传一个文件到存储桶中
            InputStream inputStream = file.getInputStream();
            minioClient.putObject(BUCKET_NAME,objectName, inputStream,new PutObjectOptions(inputStream.available(),-1));
            log.info("文件上传成功!");
            return ENDPOINT + "/" + BUCKET_NAME + "/" + objectName;
        } catch (Exception e) {
            e.printStackTrace();
            log.info("上传发生错误: {}！", e.getMessage());
        }
        return null;
    }

    @Override
    public void download(String fileName, String objectName, HttpServletResponse response) {
        InputStream inputStream = null;
        BufferedInputStream bufferedInputStream = null;
        try {
            fileName = new String(fileName.getBytes(), "ISO8859-1");
            // 设置response的Header
            response.setContentType("application/octet-stream;charset=ISO8859-1");
            response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
            response.setHeader("Pargam", "no-cache");
            response.addHeader("Cache-Control", "no-cache");

            OutputStream os = response.getOutputStream();
            MinioClient minioClient = getMinioClient();
            // 获取文件输入流
            inputStream = minioClient.getObject(BUCKET_NAME, objectName);

            bufferedInputStream = new BufferedInputStream(inputStream);
            byte[] bytes = new byte[2048];
            int len;
            while((len = bufferedInputStream.read(bytes)) != -1) {
                os.write(bytes, 0, len);
            }
        } catch (Exception e) {
            log.error("下载文件发生错误: ", e);
        }finally {
            if(bufferedInputStream != null) {
                try {
                    bufferedInputStream.close();
                } catch (IOException e) {
                    log.error("下载文件发生错误: ", e);
                }
            }
            if(inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    log.error("下载文件发生错误: ", e);
                }
            }
        }
    }

    @Override
    public void deleteFile(List<String> objectNames) {
        try {
            MinioClient minioClient = getMinioClient();
            Iterable<Result<DeleteError>> results = minioClient.removeObjects(BUCKET_NAME, objectNames);
            for (Result<DeleteError> result : results) {
                DeleteError deleteError = result.get();
                log.debug(deleteError.message());
            }
        } catch (Exception e) {
            log.error("删除文件发生错误: ", e);
        }
    }

    @Override
    public List<FileVO> listFile() {
        List<FileVO> fileVOS = new ArrayList<>();
        try {
            MinioClient minioClient = getMinioClient();
            Iterable<Result<Item>> results = minioClient.listObjects(BUCKET_NAME);
            for (Result<Item> result : results) {
                Item item = result.get();
                ZonedDateTime zonedDateTime = item.lastModified().withZoneSameInstant(ZoneId.of("Asia/Shanghai"));
                FileVO fileVO = new FileVO();
                fileVO.setObjectName(item.objectName());
                fileVO.setUploadTime(zonedDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
                if(ImageUtils.isImage(item.objectName())) {
                    fileVO.setPreviewUrl(ENDPOINT + "/" + BUCKET_NAME + "/" + item.objectName());
                }
                fileVOS.add(fileVO);

            }
        } catch (Exception e) {
            log.error("查询文件列表发生错误: ", e);
        }
        return fileVOS;
    }
}
```