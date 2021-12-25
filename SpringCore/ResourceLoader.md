# ResourceLoader
리소스를 로딩해주는 인터페이스

## ResourceLoader 사용
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Resource resource = resourceLoader.getResource("classpath:test.txt");
        System.out.println(resource.exists());
        System.out.println(Files.readString(Path.of(resource.getURI())));
    }
}
```
- ```Resource resource = resourceLoader.getResource("classpath:test.txt")```
    - resourceLoader의 getResource 메서드를 호출해 Resource를 얻는다. 
    - 리소스 디렉토리에 있는 것들이 빌드시 target 디렉토리 밑으로 들어가면서 classes에 들어가게 된다. 그리고 classpath를 기준으로 리소스를 찾기 시작한다.
- ```System.out.println(resource.exists());```
    - 해당 리소스가 존재하는지 출력해준다.

- ```System.out.println(Files.readString(Path.of(resource.getURI())));```
    - 읽어들인 리소스의 내용을 출력한다.

## 참고자료
[스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core/dashboard)