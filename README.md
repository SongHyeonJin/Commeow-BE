### **Commeow!** 🐈‍⬛
___

>📺 **프로젝트 소개**

커뮤(Commeow!)는 실시간 방송 스트리밍과 채팅을 통해 사용자들의 컨텐츠 생산/소비를 제공하는 서비스입니다.
<p align="center"><img src="https://github.com/jeonga127/Commeow-BE/assets/71822288/8b1f8152-46f7-4961-afbb-b36f128b7ac3" width = "60%" height = "30%"></p>

___

>🔥 **서비스 핵심 기능**

(1). 방송 스트리밍
    - 스트리머는 OBS Studio 프로그램을 사용하여 방송을 송출할 수 있습니다.
    - 유저 플로우
    <p align="center"><img src="https://github.com/jeonga127/Commeow-BE/assets/71822288/bb54d595-01e2-440f-8fb1-703d1b37e43d" width = "60%" height = "30%"></p>

1. 스트리머는 OBS Studio를 통해 Streaming Service 서버로 방송 시작 요청을 보냅니다.
2. Streaming Service 서버에서는 Channel Service를 제공하는 서버로, 현재 방송 시작을 요청한 스트리머가 유효한 사용자인지 확인합니다.
3. 유효한 사용자인지 확인이 되었다면, Streaming Service에서는 RTMP 스트림을 만들고, 그 스트림을 통해 영상 데이터(Chunk)가 Transcoding Service 서버로 전송되도록 합니다.
4. Transcoding Service 서버에서는 전달된 영상 데이터를 HTTP Live Streaming(HLS) 스트림을 구성하고 실행할 수 있도록 ffmpeg을 활용해 .ts 파일과 .m3u8 파일을 생성한 후, 이를 서버의 로컬 스토리지에 저장합니다.

      위 과정을 통해 방송을 송출할 준비가 완료됩니다.

---

(2). 방송 시청
    - 시청자는 웹 페이지에서 스트리머의 방송을 선택하여 시청할 수 있습니다.
    - 유저 플로우
<p align="center"><img src="https://github.com/jeonga127/Commeow-BE/assets/71822288/1f7222b9-4287-4ae7-a2a8-fc02f8712ae8" width = "60%" height = "30%"></p>

1. 시청자가 방송 화면에 접속하게 되면, Channel Service를 제공하는 서버로 로컬 스토리지에 있는 영상 데이터를 요청합니다.
2. Channel Service에서는 로컬 스토리지에서 HLS 스트림을 통해 전송될 파일을 찾습니다.
3. HLS 스트림을 통해 영상 데이터인 .m3u8 파일과 .ts 파일이 전송되고, 시청자는 그 데이터로 방송을 시청합니다.
        
  - 부가 기능
    - 채팅
      - 각각의 시청자들은 방송 화면에 접속할 때 모두 RSocketRequester을 할당받습니다. 이 RSocketRequester은 해당 방송에 구독이 되어 있어, 해당 방송에 접속한 시청자만이 스트리머와 소통할 수 있습니다.
      - 채팅에 참여하려면 인증 토큰이 필요합니다. 인증된 (로그인 한) 시청자는 채팅에 참여할 수 있지만, 미인증된 시청자는 채팅을 볼 수만 있습니다.
    - 후원하기
      - 시청자들은 방송 중인 스트리머에게 후원할 수 있습니다. Rsocket을 이용해 후원 내역이 해당 채팅창에 전송되어 모든 시청자들이 확인할 수 있습니다.
      - 동시성 제어를 통해 후원받는 스트리머의 데이터 무결성이 보장되었습니다.

---

(3). 재화 충전
    - 휴대폰으로 QR 인식 또는 핸드폰 번호 입력을 통해 카카오 결제 화면으로 이동합니다.
    - 모든 사용자는 카카오페이 테스트 결제를 통해 후원에 필요한 재화를 충천할 수 있습니다.
    - 거래 금액과 상품 금액의 일치 여부를 서버에서 검증합니다.
___

>💡**기술 스택 및 기술적 의사 결정**

- 프레임 워크
    - **Spring WebFlux**
        - 마이크로 서비스 아키텍처: Spring Webflux는 마이크로 서비스 아키텍처에 적합한 프레임워크입니다. 각 마이크로서비스는 독립적으로 동작하고, 비동기 방식으로 통신할 수 있습니다.
        - 비동기 처리: 라이브 스트리밍 서비스는 많은 수의 요청을 처리해야 합니다. Spring Webflux는 비동기 방식으로 요청을 처리하여 높은 처리량을 지원하며, 이를 통해 응답성을 향상시킬 수 있습니다.
        - 높은 처리량: 라이브 스트리밍 서비스에서는 대량의 데이터를 실시간으로 처리해야합니다. Spring Webflux는 넌 블로킹 I/O와 백프레셔 기능을 지원하여 처리량을 향상 시킬 수 있습니다.
        - 저희는 라이브 스트리밍의 특성상 서비스간 데이터를 주고 받는 요청이 많고, 각 요청들에 대한 처리가 효율적으로 되어야하기 때문에 Spring WebFlux가 가장 적합한 프레임 워크라고 생각했습니다.
- 운영 환경
    - **Docker**

      저희는 이번 프로젝트에서 MSA(Micro Service Architecture)를 구현해보고자 했습니다. AWS EC2 클라우드에 Docker를 설치하고, 3개의 컨테이너를 실행해서 서비스를 제공하고 있습니다.(Content Service, Transcoding Service, Redis)

    - 모니터링: K6, Grafana
- DB
    - **R2DBC**
      Spring WebFlux는 비동기 처리가 장점인 만큼 리액티브 데이터 베이스를 사용하는 것이 적합합니다. 저희는 그 중 PostgreSQL을 선택했습니다.
        - PostgreSQL
            - **다중 버전 동시성 제어**: PostgreSQL은 다중 버전 동시성 제어(MVCC) 기능을 제공합니다. 이를 통해 동시에 여러 클라이언트가 데이터를 읽고 쓸 수 있고, 동시성 충돌을 최소화하면서 데이터 일관성을 유지할 수 있습니다.
    - **Redis**

      Refresh Token을 저장하기 위해 사용했습니다.

- 개발
    - **RTMP(Real Time Messaging Protocol)**

      OBS(스트리밍/녹화 프로그램)에서 방송을 송출할 때 사용되는 기본적인 프로토콜 입니다. 저희의 서비스는 이 프로토콜을 통해 전달되는 데이터들을 서버에서 처리합니다.

    - **HLS(HTTP Live Streaming)**

      OBS로부터 전달받은 데이터들을 변환한 뒤에 시청자에게 전송할 때 사용하는 프로토콜입니다. RTMP 프로토콜은 Flash 기반의 프로토콜이므로 현대 웹에서는 사용할 수 없습니다. HLS 프로토콜은 HTTP 기반의 프로토콜이고, 대부분의 웹 브라우저에서 지원되어서 영상을 시청할 수 있게 해줍니다.

    - **FFmpeg**

      RTMP 프로토콜을 통해 전달된 데이터를 HLS 프로토콜을 사용할 수 있도록 transcoding하기 위해서 사용합니다. 게다가 라이브 스트리밍 등과 같이 비디오 전송에 크게 의존하는 콘텐츠의 경우 transcoding은 프로세스의 필수적인 부분이기 때문에 적응 비트 전송률 비디오 품질을 저하시키지 않으면서 많은 사용자에게 접근할 수 있는 일반적이고 효과적인 방법입니다.

    - **RSocket**

      RSocket은 메시지 지향 프로토콜로, 경량화되어 있어 최적화된 성능을 제공합니다. RSocket은 요청-응답 패턴과 스트리밍 패턴을 효율적으로 처리할 수 있어 대용량 데이터 전송이나 대규모 시스템에서 유용합니다.
      Websocket은 클라이언트와 서버 간에 지속적인 연결 유지가 필요하기 때문에, 애플리케이션에 대량의 동시 접속자가 발생하는 시나리오에서는 연결 유지를 위한 리소스 소비가 증가할 수 있는 반면, RSocket은 스트리밍, 백프레셔 제어 기능 등을 제공하여 네트워크 대역폭 및 리소스 사용을 최적화할 수 있습니다.

___
>🔧 **서버 아키텍처**

<p align="center"><img src="https://github.com/jeonga127/Commeow-BE/assets/71822288/fec01513-fe79-4611-954c-dc3d5ea48b71" width = "60%" height = "30%"></p>

___
### 구현 기능

- **트랜스코딩**
    - OBS를 통해 받아온 방송을 FFmpeg를 이용하여 트랜스코딩을 진행합니다.
    - 스트리밍 서비스를 통해 전송된 영상 파일들을 처리합니다.
    - 4초 간격으로 영상 파일을 생성하여 저장합니다.
- **HLS(Http Live Streaming)**
    - Spring WebFlux를 활용하여 HLS 프로토콜을 지원하는 웹 서비스를 개발하여, 효율적인 멀티미디어 콘텐츠 전송 및 원활한 사용자 경험을 제공하였습니다.
- **파일 삭제 로직 구현**
    - 원활한 스트리밍 서비스 제공받기 위해 영상 순서 정보 제공하는 매니페스트 파일과 그에 상응되는 영상 데이터 파일을 지속적으로 수신하게 하기 위해 불필요한 파일을 삭제해줘야겠다고 판단했습니다.
    - **VUser 1000명, Duration 5분** 이라는 동일한 테스트 환경에서 **성공률**이 **23% ~ 100%** 로 비약적으로 상승되었습니다.

<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/428aa733-a0b3-42dc-826f-9b842c0514d4" width = "60%" height = "30%"></p>
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/487cbea8-5f07-4722-adb1-b0ecedb25dc5" width = "60%" height = "30%">
<p align="center">(Before) 자동 파일 삭제 기능이 없을 때의 VUser = 1000명, Duration = 5분 스트리밍 테스트</p>
        
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/60f84770-5a8c-4603-98fe-947cbf44be36" width = "60%" height = "30%"></p>
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/dc6fa302-a010-431f-a045-0a5a9edfaea8" width = "60%" height = "30%"></p>
<p align="center">(After)  자동 파일 삭제 기능 추가 후 VUser = 1000명, Duration = 5분 스트리밍 테스트</p>    

- **파일 삭제 로직 비동기화**
    - 파일 삭제 로직을 동기식으로 작성했을 때 스트리밍 특성상 여러 명의 스트리머가 동시에 방송 송출하기 때문에 성능 개선을 위해 비동기 방식으로 변경해 **요청 실패율**이 **11.09%** -> **0.49%** 로 개선되었습니다.
        
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/3a7259d1-9032-4cdb-8424-e1388fef12c7" width = "60%" height = "30%"></p>
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/891625ca-6a2c-4a7a-814e-a8c2890d35ca" width = "60%" height = "30%"></p>
  
- **서버 아키텍처 변경**
    - 처음엔 AWS EC2 클라우드 서버에 Docker를 설치하고, 4개의 컨테이너를 사용했습니다. 스트리밍 특성상 Streaming Service 서버는 RTMP 프로토콜을 사용하여 OBS로부터 지속적으로 전송되는 데이터를 디코딩, 인코딩해야 합니다. 그렇기 때문에, 서버의 메모리나 처리량의 측면에서 Streaming Service 서버를 새로운 서버로 분리하는 것이 효율적이라고 생각했습니다. 실제로 **20,000명**의 가상 시청자가 방송을 시청하는 상황을 테스트했을 때, **요청 처리 실패율이 33.49% 감소**하는 것을 확인할 수 있었습니다.

<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/37d1f63d-ebe0-4698-bd05-876b3ef8d60b" width = "60%" height = "30%"></p>
<p align="center">서비스 초기의 아키텍처</p>    

      
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/674ae13f-9c7a-44ba-a395-4b22e4451f84" width = "60%" height = "30%"></p>
<p align="center"> 이후 변경된 아키텍처</p>  
  
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/c28c886f-76f3-4148-bdda-0dc1403c0ae9" width = "60%" height = "30%"></p>
<p align="center">  (Before) 초기 아키텍처의 VUser = 20000명, Duration = 30분 스트리밍 테스트</p>  

<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/447c89bf-32eb-46e1-b8db-43739890b760" width = "60%" height = "30%"></p>
<p align="center"> (After) 아키텍처 변경 후의 VUser = 20000명, Duration = 30분 스트리밍 테스트</p>     

___


>⛔ **트러블 슈팅**

1️⃣ **스트리머가 방송을 잠시 중단한 뒤에 다시 방송을 시작했을 때, 현재 스트리밍 중인 영상 데이터를 찾지 못하는 문제**

✅ **해결**

- 로컬 스토리지의 메모리 가용성을 위해 오래된 영상 데이터를 주기적으로 삭제하는 로직을 추가했었습니다. 실제 영상 데이터인 .ts 파일과 영상 데이터의 재생 순서를 저장해두는 .m3u8 파일입니다. 새로 시작한 방송의 영상 데이터를 찾지 못하는 현상은 서버의 로컬 스토리지에 있는 .m3u8 파일이 제대로 삭제되지 않아서 발생하는 현상이었습니다. 방송이 종료되는 시점에 모든 .ts 파일과 .m3u8 파일을 삭제하는 로직을 추가하여 문제를 해결할 수 있었습니다.
___
2️⃣ **스트리머가 여러 사람에게 동시에 후원을 받을 경우, 데이터 무결성이 보장되지 않는 문제**

- 테스트 시나리오: 회원 5명이 스트리머에게 100포인트씩 동시 후원
- 예상 결과: 스트리머 포인트가 500 포인트 증가
- 실제 결과: 스트리머 포인트가 300포인트 증가

✅ **해결**

- 동시성 충돌이 빈번하게 발생하는 경우, 보다 신뢰할 수 있는 방식인 비관적 락을 선택했습니다. 낙관적 락은 충돌이 발생할 경우 롤백 처리를 하고 사용자가 다시 후원을 요청해야 하는 반면, 비관적 락은 후원 요청 처리 중 다른 요청이 발생하면 락을 얻을 때까지 대기합니다.
  이러한 방식을 사용하여, 데이터의 일관성과 무결성을 보장하였습니다.
- 테스트 시나리오: 회원 1000명이 스트리머에게 100포인트씩 동시 후원
- 예상 결과 및 실제 결과: 스트리머 포인트가 100000포인트 증가

___

>🏆 수상
- 항해99 최고의 인기 프로젝트 상
<p align="center"><img src="https://github.com/SongHyeonJin/Commeow-BE/assets/101760007/86c12dce-9bb4-4436-8d54-ad28e1cd51e2" width = "60%" height = "30%"></p>

