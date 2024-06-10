# 🚀 Project 관리

> mermaid로 작성된 과제는 마크다운 파일(Project-{gitID}.md)로 올려주시면 됩니다. (md 파일 내에 기존 구조를 넣어주세요) <br>
> 별도 아키택쳐나 모델링 도구를 사용한 경우에는 마크다운 파일(Project-{gitID}.md)과 png, gif, jpg, pdf 파일 형식으로 Project-{gitID}.png 파일명으로 upload 해주세요

# 요구사항

-   [ ] 서비스환경의 특정 설정 파일 분석
    -   DB connection pool configuration, Redis configuration, HTTP client configuration 등 서비스 하고 있는 시스템에서 분석 하고 싶은 설정을 선택한다. (복수선택가능)
-   [ ] 해당 설정에 대한 내용을 분석한다.
    -   민감한 내용(id, password, ip, host-name...)을 제외한 설정 내용을 분석해보고 각각의 설정이 왜 그렇게 되어있는지 작성.
-   [ ] 설정 내용에 대한 PR을 요청하고 리뷰진행.

# 🚀미션

1.  서비스 하고 있는 시스템에서 분석 하고 싶은 설정을 분석하고 이해도를 높인다.
2.  서비스 하고 있는 시스템의 연결 설정이나 thread pool 설정 같은 다양한 설정 파일 중 빈번히 사용하는 모듈의 설정 파일을 선정 한다.
3.  해당 설정 파일(들)에 설정 되어있는 속성들을 이해하고 각각의 속성에 대한 값을 왜 그렇게 했을지 고민해보고 설정파일의 주석에 해당 내용을 작성한다.
4.  분석한 내용을 PR로 요청 하고 리뷰어와 리뷰를 진행 한다.
5.  1~4의 과정을 2주간 반복하면서 설정에 대한 이해도를 높인다.

# 이름 : 김호섭

## 설정분석

### 설정파일

#### [HicariCP](https://github.com/brettwooldridge/HikariCP)

-   JPA기반 코드에서 사용하는 DB Connection Pool
-   Postgresql을 사용하고 있습니다.

```
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      driver-class-name: org.postgresql.Driver
      pool-name: HikariCP4Master
      jdbc-url: jdbc:postgresql://${postgresqldb.cluster}/${postgresqldb.database}${postgresqldb.option}
      username: ${postgresqldb.username}
      password: ${postgresqldb.password}
      maximum-pool-size: 20
      connection-timeout: 5000 #5s
      idle-timeout: 50000 #50s
      max-lifetime: 50000 #50s
      auto-commit: false
```

-   maximum-pool-size: 기존에 기본값인 10을 사용하닥 접속자가 2배로 늘게되어 기본값의 2배인 20으로 설정함
-   connection-timeout: tomcat의 timeout이 5s라 connection-timeout도 이와 같은 5s로 설정함
-   max-lifetime : postgresql tcp 유지시간인 10분보다 작게 설정함
