
# 🚀 Project 관리
> mermaid로 작성된 과제는 마크다운 파일(Project.md)로 올려주시면 됩니다. (md 파일 내에 기존 구조를 넣어주세요) <br>
> 별도 아키택쳐나 모델링 도구를 사용한 경우에는 마크다운 파일(Project.md)과 png, gif, jpg, pdf 파일 형식으로 Project-{gitID}.png 파일명으로 upload 해주세요
# 요구사항
- [ ] 개선하려는 프로젝트의 최종 설계
    - 3주차에 작성한 markdown파일을 그대로 사용
- [ ] task list 도출
    - 3주차에 작성한 markdown파일을 그대로 사용
- [ ] 일정 계획 문서 (WBS)
   - 3주차에 작성한 markdown파일을 그대로 사용
- [ ] issue list
   - 프로젝트를 진행 하는 과정에서 발생한 이슈가 있다면 작성.


# 🚀미션
1. 3주차 미션에 진행된 ToBe 개선 프로젝트 WBS를 기반으로 향후 4주간 개선 작업을 진행한다.
2. 매주 토요일까지 개선된 프로젝트의 진행사항을 Github으로 PR을 요청하고 코치의 리뷰를 받는다.
    1. 미션을 진행하면서 기술적인 어려움이나 이슈사항이 있다면 이슈사항을 작성하고 리뷰를 진행한다.
    2. WBS상에서 진행된 과제들은 'Done'으로 상태를 update한다.
3. 이슈 사항이 있을 경우 설계의 수정이나 일정의 변경이 필요하면 수정 한다.
4. 수정된 계획을 기반으로 다음주 개선작업을 진행하고 1~4의 과정을 4주간 반복한다.


## 실무계선 Project
### 실무계선 Project 기대효과 분석
- timeout 건을 처리하는데 매일 소요되는 2시간의 업무 시간을 30분 내외로 줄일 수 있다.
- 사람이 직접 하는 부분을 자동화 하여 실수를 줄일 수 있다.
    - 가끔 결제가 되었는데 timeout건으로 나왔으나 수기처리시 누락된 경우 고객의 CS 클래임이 인입되고 좋지않은 고객경험을 준다.
    - timeout 갤제 CS인입건 1건/week 을 0건으로 줄일 수 있다.
- 익일 처리되던 프로세스를 5분단위의 batch로 처리하여서 고객만족을 줄 수 있다.
    - 주문은 실패 했지만 결제가 되었다는 CS 건 3건/week를 0건으로 줄일 수 있다.
 
### 실무계선 Project 프로세스
```mermaid
flowchart TB
 G[Start] --주문요청 --> H(승인)
    H --> I(PG요청)
    I -- Success --> J[주문완료]
    I -- Fail --> K[결제취소]
    I -- Timeout --> L[재처리]
    L -- 승인확인 --> M(PG)
    M -- 결제실패 --> Z(완료)
    M -- 결제성공 --> O(승인취소)
    O --> Z

```

### class diagram
- class diagram
```mermaid
classDiagram

    class PaymentMethod {
        +String paymentMethodID
        pay()
        cancel()
    }
    PaymentMethod <|-- Card
    PaymentMethod <|-- Bank

    class PG {
        +String pgID
        pay()
        cancel()
    }
    PG <|-- Card
    PG <|-- Bank


    class Payment {
        +String paymentID
        +String transactionID
        void pay()
    }

    class Cancel {
        +String cancelID
        +PaymentID paymentID
        +String transactionID
        void cancel()
    }

    class CancelDetail {
        +String cancelDetailID
        +String cancelID
    }

    class PaymentDetail {
        +PaymentID paymentID
    }


    class Card {
        CardID
        pay()
        cancel()
        checkTransaction()
    }
    note for Card "checkTransaction() : 결제내역확인"

    class Bank {
        BankID
        pay()
        cancel()
        checkTransaction()
    }
    note for Bank "checkTransaction() : 결제내역확인"

   Payment "1" -- "*" PaymentDetail : 결제수단, 금액, 상품 정보
   Cancel "1" -- "*" CancelDetail : 결제수단, 금액, 상품 정보
   Cancel "0..1" --> "1" Payment : 원결제 정보
   Payment --> PaymentMethod : 결제요청
   Cancel --> PaymentMethod : 취소요청


   class PaymentTiemoutListner {
        +beforeCancelForTimeout()
        -checkLimitRetryCount()
        -isPay()
        +cancelForTimeout()
        +postCancelForTimeout()
   }

   class timeoutResultNotification {
        sendNotification()
   }


    PaymentTiemoutListner "1" -- "1" Payment : 원결제확인
    Payment --> PaymentTiemoutListner : Timeout Event
    PaymentTiemoutListner --> PaymentMethod : cancel 처리

```
    

### ERD
- 실무계선 Project 구조에서 변경되는 ERD를 작성한다.
```mermaid
erDiagram
  Payment {
    Integer id 
    String name
  }
  Payment ||--|{ PaymentDetail : has

  PaymentMethod {
    Integer id
    String name 
  }

  PaymentDetail {
    Integer id
    Integer paymentId
    Integer paymentMethodId
    Long productId
    Integer amount
    Integer quantity
    Integer unitPrice
    String productInfo
  }
  PaymentDetail ||--|{ PaymentMethod : fundingsource

  Cancel {
    Integer id
    Integer paymentId
    String transactionId
  }
  Cancel ||--|{ CancelDetail : has
  CancelDetail ||--|{ PaymentMethod : fundingsource

  CancelDetail {
    Integer id
    Integer cancelId
    Integer amount
    String productInfo
  }

  PaymentDetail {
    String paymentId
    String paymentMethodId
  }

  CancelDetail {
    String cancelId
  }

  payTimeoutRetry {
    String id
    Integer retryCnt
    String status
  }

  payTimeoutRetryHistories {
    String id
    String status
  }

payTimeoutRetry ||--o{ Retry-Process : do
payTimeoutRetryHistories ||--o{ Retry-Process : dohistories

```

## Task List
1. Timeout 발생 시 Event발생 수정- SQS, SNS <br>
2. Timeout event subscription module 작성<br>
3. Timeout log table 설계, 생성<br>
4. Timeout 재처리 service 설개, 구현<br>
&nbsp; &nbsp; 1. transaction 성공여부 확인 <br>
&nbsp; &nbsp; 2. transaction 취소 처리 하기 (결제시)<br>
&nbsp; &nbsp; 3. 재처리 logging(DB) : 처리 횟수(3회), 처리 내역<br>
5. Timeout 재처리 현황 조회 어드민 page.<br>
6. Timeout 재처리 실패시 메일 발송 모듈.<br>

## WBS
1. 요구사항 분석 : 이미수행
2. 설계 : 3d
3. 일정산정: 1d
4. Timeout 발생 시 Event발생 수정- SQS, SNS : 이미 사용하는 SQS가 있고 큐생성 및 기존코드 수정 : 2d
5. Timeout event subscription module 작성 : SQS, SNS : 이미 사용하는 SQS가 있고 신규 class 생성 : 2d
6. Timeout log table 설계, 생성 : 1d
7. Timeout 재처리 service 설개, 구현 : 2d
    1. transaction 성공여부 확인 : 0.5d
    2. transaction 취소 처리 하기 (결제시) : 0.5d
    3. 재처리 logging(DB) : 처리 횟수(3회), 처리 내역 : 1d
8. Timeout 재처리 현황 조회 어드민 page.: 기존 admin에 메뉴 추가 : 5d
9. Timeout 재처리 실패시 메일 발송 모듈: 기존 notification에 method 추가 : 1d

```mermaid
gantt
    dateFormat  YYYY-MM-DD
    title       결제 재처리 WBS
    excludes    weekends, 2023-12-25, 2024-01-01
    %% (`excludes` accepts specific dates in YYYY-MM-DD format, days of the week ("sunday") or "weekends", but not the word "weekdays".)

    section prepare
    요구사항분석                    :done,    des1, 2023-12-01, 10d
    설계                            :done,  des2, 2023-12-11, 3d
    일정산정                        :done     des3, after des2, 1d
    Timeout log table 설계, 생성    :done       des4, 2023-12-27, 1d

    section 기존 모듈 수정
    Payment timeout event 발생          :crit, b1, 2024-01-03,2d
    Cancel timeout용 cancel 추가        :crit, b2, 2024-01-10, 2d

    section 신규 모듈 구현
    Timeout event consumer 모듈작성    :c1, after b1, 2d
    Queue 동작확인                      :milestone, after c1, 0d
    Timeout service 구현                  :c2, after b2  , 2d
    Timeout 재처리 현황 조회 어드민 개발    :c3, after c2  , 5d
    Timeout 재처리 실패시 notification     : c4, after c3, 1d

    section 테스트
    Test & QA                           :after c4, 2d

```

## Issue list
1. ~~일정 시작이 지연됨~~
    1. ~~Follow up : 1일 지연 - 추가 작업으로 완료~~
2. SQS 설정 지연
    1. Follow up : 1주 지연 - 주말 작업으로 1.5일까지 완료 예정

