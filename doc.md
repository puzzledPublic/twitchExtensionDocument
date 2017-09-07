Introduction
============
트위치 Extension은 스트리머가 자신의 방송 페이지에 추가적인 컨텐츠를 넣을 수 있게 만듭니다.
스트리머가 설치하는 Extension은 Desktop 웹 브라우저를 이용하는 시청자에게 자동적으로 보여집니다. 시청자는 Extension을 볼 수 있고 규칙에 어긋나는 Extension을 신고할 수 있습니다.

Extension은 몇개의 컴포넌트를 가집니다.
* 스트리머가 설치 및 구성 할 수 있는 HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 일반적인 Extension 설치 과정의 한 부분이 됩니다.
* 시청자가 체험 가능한 HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 스트리머 방송 페이지에 iframe(inline frame)으로 만들어 집니다.
* (Optional) 스트리머가 체험 가능한  HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 스트리머 대시보드 페이지에 iframe으로 만들어집니다. 스트리머는 방송 중 특별한 명령(ex: 투표)를 지시가 가능합니다.
* (Optional) Extension 백엔드 서비스(EBS).<br>
  프론트 엔드와 커뮤니케이션 가능하고 데이터 및 상태를 저장 할 수 있는 백엔드 웹 서비스를 지칭합니다. 모든 시청자에게 데이터를 보내기 위해 AJAX 요청을 받거나 트위치의 PubSub 아키텍처를 사용할 수 있습니다. 

2가지 유형의 Extension:
* Video Player 아래 Panal 영역에 위치하는 Panal Extension.
* 투명한 오버레이로써 Video Player 상에 위치하는 Video-Overlay Extension.

### 스트리머 체험
스트리머는 대시보드의 Extension Manager 탭에서 Extension의 목록을 볼 수 있으며 설치 할 수 있습니다. Extension의 추가, 제거 및 동작 여부를 제어 할 수 있습니다.
<br><br>
Extension 설치를 한다고 해서 작동을 하지는 않습니다. 작동을 위해 스트리머는 Extension 설정(개발자가 필요로 한다면)을 해야합니다. 그 다음 Extension을 활성화 하면 시청자에게 보여지게 됩니다.

### 도움이 필요하다면..
이 문서에 문제가 있거나 정보가 더 필요하거나 질문이 있으시다면 [Extensions category of the Twitch Developer Forums](https://discuss.dev.twitch.tv/c/extensions) 에 방문해주세요.

Architecture Overview
=====================
![Alt text](https://media-elerium.cursecdn.com/attachments/215/369/extensionsguide-architecture.png)
Extension은 iframe 입니다. 일반적으로 Extension은 AJAX를 이용해 Extension Backend Service(EBS)와 커뮤니케이션을 합니다.
EBS는 Extension 개발자가 개발, 배포, 유지보수 하는 웹 서비스입니다.