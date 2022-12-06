![image](https://user-images.githubusercontent.com/487999/79708354-29074a80-82fa-11ea-80df-0db3962fb453.png)

# 예제 - 음식배달

본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다.
이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.
- 체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW


# 서비스 시나리오

기능적 요구사항
1. 고객이 메뉴를 선택하여 주문한다
1. 고객이 선택한 메뉴에 대해 결제한다
1. 결제가 완료되면 주문 내역이 입점상점주인에게 주문정보가 전달된다
1. 상점주인이 주문을 수락하거나 거절할 수 있다
1. 상점주인은 요리시작때와 완료 시점에 시스템에 상태를 입력한다
1. 고객은 아직 요리가 시작되지 않은 주문을 취소할 수 있다
1. 요리가 완료되면 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다
1. 고객이 주문상태를 중간중간 조회한다
1. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다
1. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다.


비기능적 요구사항 
1. 장애격리
    1. 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    1. 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시후에 하도록 유도한다  Circuit breaker, fallback
1. 성능
    1. 고객이 자주 상점관리에서 확인할 수 있는 배달상태를 주문시스템(프론트엔드)에서 확인할 수 있어야 한다  CQRS
    1. 배달상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다  Event driven


# 체크포인트

## Event Storming 결과
* MSAEz 로 모델링한 이벤트스토밍 결과
![이벤트 스토밍](https://user-images.githubusercontent.com/43160394/205813317-41eb4187-4dff-41f6-ad34-b4ae4d50f713.png)
1. 고객이 메뉴를 선택하여 주문한다 (Ok)
1. 고객이 선택한 메뉴에 대해 결제한다 (Ok)
1. 결제가 완료되면 주문 내역이 입점상점주인에게 주문정보가 전달된다 (Ok)
1. 상점주인이 주문을 수락하거나 거절할 수 있다 (Ok)
1. 상점주인은 요리시작때와 완료 시점에 시스템에 상태를 입력한다 (Ok)
1. 고객은 아직 요리가 시작되지 않은 주문을 취소할 수 있다 (Ok)
1. 요리가 완료되면 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다 (Ok)
1. 고객이 주문상태를 중간중간 조회한다 (Ok)
1. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다 (Ok)
1. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다 (Ok)

## 1. Saga (Pub/Sub)
Pub/Sub에 해당하는 front에서 음식을 주문하고 delibery에서 받는것을 확인한다.
![image](https://user-images.githubusercontent.com/43160394/205781907-fc537ba7-be88-4864-a280-39fea0828537.png)
![image](https://user-images.githubusercontent.com/43160394/205785572-76f00cc7-cb2c-4514-869b-4270db74fb7c.png)


## 2. CQRS
Read Model CRUD 상세설계
![image](https://user-images.githubusercontent.com/43160394/205777738-f7c7d056-6576-4d4b-adf2-e86301b47e8e.png)


## 3. Compensation/Correlation
ordercancel의 compensation에 해당하는 Paycancal에 대한 correlation 확인한다.
![image](https://user-images.githubusercontent.com/43160394/205788163-c8bac1be-b2dd-43d8-a9d4-d6695e956ce7.png)
![image](https://user-images.githubusercontent.com/43160394/205788204-8602e498-8f7d-447d-b6be-ba562dbfd656.png)

## 4. Request/Response

동기 호출
![image](https://user-images.githubusercontent.com/43160394/205778619-6c9458d0-c548-4797-a596-595e6e3b247e.png)

비동기 호출
![image](https://user-images.githubusercontent.com/43160394/205778719-f0a94ad1-fbd1-429e-b51f-e8846b266d38.png)


## 5. Circuit Breaker
호출선의 설정에서 Circuit breaker 옵션을 On 한다

![image](https://user-images.githubusercontent.com/43160394/205560198-bd1d95b2-95b1-422b-8a2c-8064beb63b2c.png)

생성 코드 확인과 구현
서킷브레이커 설정
front 서비스의 application.yaml 파일의 다음 설정을 true 로 하고, 임계치를 500ms으로 변경
![image](https://user-images.githubusercontent.com/43160394/205835525-a2a787d4-4715-4589-b18e-098b5616741c.png)


성능이 느려지도록 딜레이 발생 코드를 넣는다.


@PostLoad
 public void makeDelay(){
     try {
         Thread.currentThread().sleep((long) (400 + Math.random() * 220));
     } catch (InterruptedException e) {
         e.printStackTrace();
     }

 }
 
 
실행 결과

![image](https://user-images.githubusercontent.com/43160394/205835249-be807634-aa71-4d7c-9209-b11598c12170.png)


## 6. Gateway/Ingress
![image](https://user-images.githubusercontent.com/43160394/205779174-46d4987d-d2f6-4d74-8a70-ee396f7a66d1.png)
![image](https://user-images.githubusercontent.com/43160394/205778999-1744858b-dbe3-4039-939f-8adee9efde21.png)


