# SpringBoot: AWS S3에 파일 업로드하기

프로젝트를 진행하면서 이미지 파일 업로더가 필요했다.
웹 서버의 로컬에 저장하는 방법도 있지만, S3에 저장하는 편이 파일을 관리하기 더 쉽고 EC2 서버에 비해(서버의 로컬에 저장하는 것에 비해) 비용이 굉장히 저렴하다.
목표는 업로드 하고자 하는 이미지의 원본과 크기를 줄인 썸네일을 함께 S3에 업로드 하고 업로드 된 경로를 응답으로 돌려주는 것이다.

## 과정

### S3 생성 및 키 발급

가장 먼저 S3 bucket 을 생성하고, `credentials.accessKey` 와 `credentials.secretKey` 를 미리 발급받아야 한다. 이미 키를 발급했기 때문에 해당 과정은 이 글에서 다루지 않겠다. 잘 설명된 글이 굉장히 많다.
추후에 S3를 또 다룰일이 있다면 추가하도록 하겠음!

### 의존성 추가

**build.gradle**
```
...

dependencies {
	...
    implementation 'org.springframework.cloud:spring-cloud-starter-aws:2.2.6.RELEASE'
    implementation 'net.coobird:thumbnailator:0.4.18'
    ...
}
```

S3 연동을 위해 `org.springframework.cloud:spring-cloud-starter-aws` 그리고
썸네일 변환을 위해`net.coobird:thumnailator` 를 추가했다.
`thumnailator` 가 이미지 파일을 다루기 쉬워보여서 사용했지만, 다른 라이브러리나 API를 사용해도 된다.

### 설정 값 입력
**application.properties**
```
cloud.aws.credentials.accessKey=?????
cloud.aws.credentials.secretKey=?????

cloud.aws.s3.bucket=버킷 이름
cloud.aws.region.static=버킷 region
```

[Service: AmazonCloudFormation Error](https://stackoverflow.com/questions/37161989/disable-cloudformation-in-spring-cloud-aws) 에러가 발생시에는 `cloud.aws.stack.auto=false` 를 추가하면 된다.

**파일 업로드 예외 발생시**

`org.apache.tomcat.util.http.fileupload.impl.SizeLimitExceededException`  
파일 업로드 시에 크기 문제로 톰캣에서 오류가 발생할 수 있다. 기본 최대 업로드 용량은 2MB로, 더 큰 크기의 파일을 받을 예정이 있다면, `application.properties`에 아래의 값을 추가로 설정하자.
```
spring.servlet.multipart.max-file-size=15MB
spring.servlet.multipart.max-request-size=15MB
```

### 코드
**FileController.java**
```java
@RequiredArgsConstructor
@RequestMapping("/api/v1/file")
@RestController
public class FileController {

    private final FileService fileService;

    @PostMapping("/image")
    public ResponseEntity<UploadImageFileResponseDTO> uploadImageFile(
        @RequestPart MultipartFile imageFile) throws IOException {

        return ResponseEntity
            .status(HttpStatus.OK)
            .body(fileService.uploadImageFile(imageFile));
    }
}
```

파일은 `@RequestPart` 어노테이션과 `MultipartFile` 타입으로 입력받는다. Controller 에서는 그 외 특이사항은 없다.

**FileService**
```java
@RequiredArgsConstructor
@Service
public class FileService {

    private static final String SUFFIX = ".png";
    private static final String THUMBNAIL_TAIL = "-th";
    private static final int IMAGE_WIDTH = 1000;
    private static final int IMAGE_HEIGHT = 1000;
    private static final double OUTPUT_QUALITY = 0.7;

    private final ImageFileRepository imageFileRepository;
    private final AmazonS3Client amazonS3Client;

    @Value("${cloud.aws.s3.bucket}") // application.properties 의 해당 값을 가져온다.
    private String bucket;

    public UploadImageFileResponseDTO uploadImageFile(MultipartFile imageFile) throws IOException {
        String uuid = UUID.randomUUID().toString();
        String originalUrl = putS3(imageFile, uuid);
        String thumbnailUrl = putS3(convertToThumbnail(imageFile), uuid);

        ImageFile savedImageFile = ImageFile.builder()
            .originalUrl(originalUrl)
            .thumbnailUrl(thumbnailUrl)
            .build();

        imageFileRepository.save(savedImageFile);

        return new UploadImageFileResponseDTO(savedImageFile);
    }

    private String putS3(MultipartFile imageFile, String fileName) throws IOException {
        fileName += SUFFIX;

        ObjectMetadata objectMetadata = new ObjectMetadata();
        objectMetadata.setContentLength(imageFile.getInputStream().available());
        objectMetadata.setContentType("image/png");

        amazonS3Client.putObject(bucket, fileName, imageFile.getInputStream(), objectMetadata);

        return amazonS3Client.getUrl(bucket, fileName).toString();
    }

    private String putS3(File file, String fileName) {
        fileName += THUMBNAIL_TAIL + SUFFIX;

        amazonS3Client.putObject(bucket, fileName, file);
        deleteFile(file);

        return amazonS3Client.getUrl(bucket, fileName).toString();
    }

    private File convertToThumbnail(MultipartFile imageFile) throws IOException {
        File file = File.createTempFile(imageFile.getName() + THUMBNAIL_TAIL, SUFFIX);
        Thumbnails.of(imageFile.getInputStream())
            .size(IMAGE_WIDTH, IMAGE_HEIGHT) // 변환 시 사이즈
            .outputQuality(OUTPUT_QUALITY) // 변환 시 퀄리티
            .toFile(file);

        return file;
    }

    private void deleteFile(File file) {
        if (!file.delete()) {
            throw new RuntimeException("썸네일 파일 제거에 실패했습니다.");
        }
    }
}
```

1. `MultipartFile` 파일을 받아서 S3에 업로드한다.
2. 파일을 변환해서(크기와 퀄리티를 낮춘다) 마찬가지로 S3에 업로드 한다.
3. 이후 파일들의 URL을 `ImageFile`의 `Entity`에 저장한다.
4. 해당 내용을 DTO에 담아 반환한다.

**putS3()**  
`MultipartFile`과 `File`두 타입을 받을 수 있도록 오버로딩해 작성했다. `MultiPartFile`은 S3에 바로 업로드 할 수 없으므로 파일로 변환하거나 `InputStream`과 `Metadata`를 포함해 업로드 해야한다. `InputStream`을 사용해 업로드 할 때 `contentType`을 설정하지 않으면 `application/octet-stream`로 업로드가 되는데 이 경우에 링크를 클릭 시 브라우저에서 이미지 파일이 열리는 것이 아닌, 다운로드가 시작된다.

**convertToThumbnail()**  
`Thumbnails`를 사용한 파일 변환에는 반환값이 존재하는 메서드가 없어 비어있는 임시 파일을 생성한 뒤에 해당 파일에 원본 파일의 `InputStream`을 담아 변환했다. `File.createTempFile()` 로 생성된 임시 파일은 추후 명시적으로 삭제를 해야하니 주의하자.

### 응답
```json
{
    "fileId": 15,
    "originalUrl": "https://XXXXX.s3.ap-northeast-2.amazonaws.com/74ea3bca-a03b-4fd1-a995-9939e801da41.png",
    "thumbnailUrl": "https://XXXXX.s3.ap-northeast-2.amazonaws.com/74ea3bca-a03b-4fd1-a995-9939e801da41-th.png"
}
```
의도한대로 원본 파일의 주소와 변환된 파일의 주소가 응답됐다.

## 참고자료

> [SpringBoot & AWS S3 연동하기](https://jojoldu.tistory.com/300)  
> [Thumbnailator](https://github.com/coobird/thumbnailator/wiki/Examples)



