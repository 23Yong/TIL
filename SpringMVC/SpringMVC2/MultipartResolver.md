# MultipartResolver

`org.springframework.web.multipart` 패키지에 있는 MulltipartResolver는 파일 업로드를 포함해, http multipart request 요청을 파싱할 수 있는 인터페이스이다. 이 인터페이스의 구현체로는 두 가지가 존재한다.
- CommonsMultipartResolver
- StandardServletMultipartResolver

이런 multipart 데이터를 핸들링하기 위해서는 DispatcherServlet에 MultipartResolver 빈을 등록할 필요가 있다. 그러면 DispatcherServlet은 이를 감지해 들어오는 요청에 대해 MultipartResolver를 적용할 수 있게 된다. ContentType이 multipart/form-data인 POST 요청이 들어왔을 때, MultipartResolver가 동작해 HttpServletRequest 대신에 MultipartHttpServletRequest를 주입받게 된다. 이를 사용하면 multipart와 관련된 여러가지 처리를 편하게 할 수 있다.

## 스프링에서 제공해주는 MultipartFile
MultipartHttpServletRequest를 컨트롤러에서 사용하는 대신 스프링에서 제공하는 `MultipartFile`을 사용하자.

다음과 같이 적용할 수 있다.
```java
@PostMapping("/upload")
public String saveFile(@RequestPart MultipartFile file) {
    if (file.isEmpty()) { return null; }
    
    Path fileStorageLocation = Paths.get("~/uploads")
        .toAbsolutePath().normalize();

     try {
        String fileName = file.getOriginalFileName();
        Path targetLocation = fileStorageLocation.resolve(fileName);

        Files.copy(file.getBytes(), targetLocation, StandardCopyOption.REPLACE_EXISTING);

        return fileName;
    } catch (IOException e) {
        throw new IllegalArgumentException();
    }
}
```

이때 MultipartResolver를 적용하기 위해 .yml 파일에 관련 설정을 `true`로 설정한다.
```yml
spring:
  servlet:
    multipart:
      enabled: true             # MultipartFile 업로드 가능
```

![](/img/2022-11-30-22-17-43.png)

이미지 파일을 http request body에 담아 content-type을 multipart/form-data로 설정한 요청을 날려 컨트롤러로 넘어오는 파일을 확인해보면 위와 같다. 여러 개의 데이터를 넘기고 싶은 경우에는 @RequestPart 어노테이션을 추가하고 키 값을 명시해주면 된다.

