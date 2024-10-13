---

layout: post
title: S3 Presigned URL
date: 2024-10-13 00:00:00 +0800
categories: [CS Study, database]
tags: [Sophomore, database]
pin: true

---


### 0. S3 Presigned URL을 사용하는 이유

1. 서버 부하 감소

	- 서버에 파일을 통째로 저장하는 경우: 서버는 그 파일을 저장하고 처리해야 하므로, 트래픽과 저장 공간에 부담을 준다.  

	- presigned URL을 사용하는 경우: 클라이언트가 S3에 직접 파일을 업로드한다. 서버는 파일 전송이나 저장을 처리할 필요가 없고, 단지 presigned URL을 생성해 제공하는 역할만 한다.

2. 성능

	서버를 거치지 않고 클라이언트가 S3와 직접 통신함으로써 네트워크 대역폭을 줄일 수 있다. 클라이언트와 S3 간의 직접 업로드/다운로드는 서버의 네트워크 대역폭을 사용하지 않는다.

3. 보안

	presigned URL은 일정 시간 동안만 유효한 임시 URL이다. 또한, S3에 저장된 파일에 접근하려면 유효한 presigned URL이 필요하므로, 파일 접근 권한을 동적으로 관리할 수 있다.

4. 속도

	S3는 전 세계적으로 분산된 데이터센터를 통해 파일 전송을 최적화한다. 클라이언트가 서버를 거치지 않고 S3에 직접 접근하면, S3의 **CDN(Content Delivery Network)**을 활용해 더 빠르게 파일을 업로드하거나 다운로드할 수 있다.

5. 그냥 쓰자..

	- 대용량 파일 업로드: presigned URL을 사용하면 클라이언트가 대용량 파일을 서버에 업로드하는 대신 S3에 직접 업로드할 수 있어, 서버의 처리 능력에 영향을 주지 않고도 파일 업로드를 확장할 수 있다.  

	- 분산된 파일 처리: 서버가 모든 파일을 직접 처리하지 않기 때문에, 여러 클라이언트가 동시에 파일을 업로드해도 서버의 성능이 저하되지 않고 효율적으로 파일을 처리할 수 있다.


<br>

### 1. com.amazonaws.services.s3 뜯어보기

- com.amazonaws.services.s3.AmazonS3  

```java
public interface AmazonS3 extends S3DirectSpi {
	// ...
    URL generatePresignedUrl(GeneratePresignedUrlRequest var1) throws SdkClientException;
	// ...
}
```  

GeneratePresignedUrlRequest 형식으로 변수를 넣어주면, Presigned URL을 발급해주는 인터페이스이다.  

<br>

- com.amazonaws.services.s3.model.GeneratePresignedUrlRequest  

```java
public class GeneratePresignedUrlRequest extends AmazonWebServiceRequest implements SSECustomerKeyProvider, Serializable {

	// ...
	
	public GeneratePresignedUrlRequest(String bucketName, String key, HttpMethod method) {
		this.requestParameters = new HashMap();
		this.bucketName = bucketName;
		this.key = key;
		this.method = method;
	}

	public GeneratePresignedUrlRequest withMethod(HttpMethod method) {
		this.setMethod(method);
		return this;
	}

    public GeneratePresignedUrlRequest withExpiration(Date expiration) {
        this.setExpiration(expiration);
        return this;
    }
	
	// ...

}
```  

우리는 GeneratePresignedUrlRequest model에서 이 세 가지 메서드를 이용해볼 예정이다.  

- bucketName: S3에서 파일을 저장하거나 가져올 버킷 이름.  
- key: S3 내에서 파일이 저장될 경로 (파일명 또는 파일 경로).
- method: 클라이언트가 이 presigned URL을 사용하여 수행할 HTTP 메서드. (업로드할 때는 PUT, 다운로드할 때는 GET 메서드를 사용한다.)  

언제 만료될지를 설정해주려면, Date 객체를 withExpiration에 넣어서 설정해줄 수 있다.


<br>

### 2. Presigned URL 발급 로직

- PresignedUrlRequest  

```java
package com.pocket.core.image.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
@Schema(description = "프리사인드 URL 요청 DTO")
public class PresignedUrlRequest {

    private final String prefix;

    private final String fileName;

}
```  

<br>

- PresignedUrlResponse  

```java
package com.pocket.core.image.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
@Schema(description = "프리사인드 URL 응답 DTO")
public class PresignedUrlResponse {

    @Schema(description = "생성된 프리사인드 URL", example = "https://example.com/presigned-url")
    private final String url;

    @Schema(description = "파일 경로", example = "images/example.txt")
    private final String filePath;
}
```  

<br>

- FileService  

```java
package com.pocket.core.image.service;

import com.amazonaws.HttpMethod;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.model.GeneratePresignedUrlRequest;
import com.pocket.core.image.dto.PresignedUrlResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

@Service
@RequiredArgsConstructor
public class FileService {

    @Value("${cloud.aws.s3.bucket}")
    private String bucket;

    @Value("${cloud.aws.s3.expTime}")
    private Long expTime;


    private final AmazonS3 amazonS3;

    public PresignedUrlResponse getUploadPresignedUrl(String prefix, String originalFileName) {
        String filePath = createPath(prefix, originalFileName);
        GeneratePresignedUrlRequest generatePresignedUrlRequest = getGeneratePresignedUrlRequest(bucket, filePath, HttpMethod.PUT);
        URL url = amazonS3.generatePresignedUrl(generatePresignedUrlRequest);

        return new PresignedUrlResponse(url.toString(), filePath);
    }

    private GeneratePresignedUrlRequest getGeneratePresignedUrlRequest(String bucket, String fileName, HttpMethod method) {

        return new GeneratePresignedUrlRequest(bucket, fileName)
                .withMethod(method)
                .withExpiration(getPresignedUrlExpiration());
    }

    private Date getPresignedUrlExpiration() {
        Date expiration = new Date();
        long expTimeMillis = expiration.getTime();
        expTimeMillis += expTime;
        expiration.setTime(expTimeMillis);

        return expiration;
    }

    private String createFileId() {
        return UUID.randomUUID().toString();
    }

    private String createPath(String prefix, String fileName) {
        String fileId = createFileId();
        String timestamp = new SimpleDateFormat("yyyyMMddHHmmss").format(new Date());
        return String.format("%s/%s-%s-%s", prefix, timestamp, fileId, fileName);
    }
}
```

<br>

- FileController  

```java
package com.pocket.inbounds.file.presentation;

import com.pocket.core.exception.common.ApplicationResponse;
import com.pocket.core.image.dto.PresignedUrlRequest;
import com.pocket.core.image.dto.PresignedUrlResponse;
import com.pocket.core.image.service.FileService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/file")
public class FileController implements FileControllerDocs{

    private final FileService fileService;

    @PostMapping
    public ApplicationResponse<PresignedUrlResponse> uploadPhoto(@RequestBody PresignedUrlRequest presignedUrlRequest) {
        PresignedUrlResponse response = fileService.getUploadPresignedUrl(presignedUrlRequest.getPrefix(), presignedUrlRequest.getFileName());
        return ApplicationResponse.ok(response);
    }

}
```

<br>

### 3. Presigned URL 발급 이후

만들어놓은 request의 형식에 맞춰 POST 요청을 보내보면, 아래와 같이 URL(Presigned URL)과 filePath가 날아온다.  

<img alt="1" src="https://github.com/JiinHong/jiinhong.github.io/blob/main/_posts/image-9.png?raw=true">  

이제 클라이언트는 이 URL로 이미지를 업로드하면 되고, filePath는 객체 생성 API에 전달해주면 되는 것이다.

<br>
