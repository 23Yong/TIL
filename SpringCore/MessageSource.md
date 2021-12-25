# MessageSource
국제화(I18n) 기능을 제공하는 인터페이스
(메시지를 다국화 하는 방법)

## MessageSource 사용

먼저 message들을 만들기 위해 resources 디렉토리에 ```messages.properties``` 파일과 ```messages_ko_KR.properties``` 파일을 생성한다.

각 파일에 ```greeting=Hi {0}```, ```greeting=안녕 {0}``` 을 적고 

```java
@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception { 
        System.out.println(messageSource.getMessage("greeting", new String[]{"jeongyong"}, Locale.getDefault()));
        System.out.println(messageSource.getMessage("greeting", new String[]{"jeongyong"}, Locale.KOREA));
    }
}
```
스프링 부트를 사용한다면 ResourceBundleMessageSource가 빈으로 등록이 되어있어서 이 빈이 messages라는 리소스 번들을 읽어 별다른 설정없이 messages를 읽을 수 있기 때문에 다음과 같이 getMessage 메서드를 통해 호출하면 각각 Hi jeongyong, 안녕 jeongyong이 나오는 것을 확인할 수 있었다.


## 참고자료
[스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core/dashboard)