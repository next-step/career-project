
# 🚀 Project 관리
> mermaid로 작성된 과제는 마크다운 파일(Project-{gitID}.md)로 올려주시면 됩니다. (md 파일 내에 기존 구조를 넣어주세요) <br>
> 별도 아키택쳐나 모델링 도구를 사용한 경우에는 마크다운 파일(Project-{gitID}.md)과 png, gif, jpg, pdf 파일 형식으로 Project-{gitID}.png 파일명으로 upload 해주세요
# 요구사항
- [ ] 서비스환경의 특정 설정 파일 분석
    - DB connection pool configuration, Redis configuration, HTTP client configuration 등 서비스 하고 있는 시스템에서 분석 하고 싶은 설정을 선택한다. (복수선택가능)
- [ ] 해당 설정에 대한 내용을 분석한다.
    - 민감한 내용(id, password, ip, host-name...)을 제외한 설정 내용을 분석해보고 각각의 설정이 왜 그렇게 되어있는지 작성.
- [ ] 설정 내용에 대한 PR을 요청하고 리뷰진행.


# 🚀미션
1.  서비스 하고 있는 시스템에서 분석 하고 싶은 설정을 분석하고 이해도를 높인다.
2.  서비스 하고 있는 시스템의 연결 설정이나 thread pool 설정 같은 다양한 설정 파일 중 빈번히 사용하는 모듈의 설정 파일을 선정 한다.
3.  해당 설정 파일(들)에 설정 되어있는 속성들을 이해하고 각각의 속성에 대한 값을 왜 그렇게 했을지 고민해보고 설정파일의 주석에 해당 내용을 작성한다.
4.  분석한 내용을 PR로 요청 하고 리뷰어와 리뷰를 진행 한다.
5.  1~4의 과정을 2주간 반복하면서 설정에 대한 이해도를 높인다.


# 이름 : 장진달
## 설정분석 
### 설정파일
#### [HicariCP](https://github.com/brettwooldridge/HikariCP) 
- JPA기반 코드에서 사용하는 DB Connection Pool
- Mysql을 사용하고 있으며 서버는 Master/Slave 구조로 되어있다.
```
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/example?useSSL=false&autoReconnect=true&useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Seoul
    username: nextstep
    password: next1step
    driver-class-name: com.mysql.jdbc.Driver
    hikari:
      connection-timeout: 5000        
      validation-timeout: 3000         ### traffic이 높지 않은 상황이라 default값보다 작게 잡음
      minimum-idle: 5                   ### 서비스 TPS가 1미만이며 peak 트래픽도 2~3정도, 요청마다 사용하는 커넥션은 1개 정도 이며 멀티스레드로 동작하는 부분이 없음.
      max-lifetime: 1000             ### 설정 이유 작성
      maximum-pool-size: 20          ### 설정 이유 작성
```
- connection-timeout : 데이터 건수가 많지 않고(100만 row미만)
- validation-timeout : traffic이 높지 않은 상황이라 default값보다 작게 잡음
- minimum-idle :
  - 서비스 TPS가 1미만이며 peak 트래픽도 2~3정도
  - 요청마다 사용하는 커넥션은 1개 정도 이며 멀티스레드로 동작하는 부분이 없음.
- max-lifetime : 
  - 동일 network대역에 db서버가 있다.
  - 대부분의 traffic이 적은 편이다.
- 
#### [OpenFeign](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)
- 결제를 위한 외부 PG사 연동을 위해사용
```
feign:
  client:
    config:
      feignName:
        connectTimeout: 5000
        readTimeout: 10000
        loggerLevel: full
        errorDecoder: com.example.SimpleErrorDecoder
        retryer: com.example.SimpleRetryer
        requestInterceptors:
          - com.example.FooRequestInterceptor
          - com.example.BarRequestInterceptor
        decode404: false
        encoder: com.example.SimpleEncoder
        decoder: com.example.SimpleDecoder
        contract: com.example.SimpleContract
```
- connectTimeout : PG사의 설정 가이드 수치 적용
- readTimeout :PG사의 설정 가이드 수치 적용
- retryer : 1초의 간격으로 시작해 최대 5초의 간격으로 점점 증가하며, 최대5번 재시도한다.
```
@Configuration
@EnableFeignClients("net.nextstep.openfeign")
class OpenFeignConfig {

    @Bean
    retryer() {
        // 1초의 간격으로 시작해 최대 5초의 간격으로 점점 증가하며, 최대5번 재시도한다.
        return new Retryer.Default(1000L, TimeUnit.SECONDS.toMillis(5L), 5);
    }
}
```

 



