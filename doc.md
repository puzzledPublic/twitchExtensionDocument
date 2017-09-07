Introduction
============
트위치 Extension은 스트리머가 자신의 방송 페이지에 추가적인 컨텐츠를 넣을 수 있게 만듭니다. 스트리머가 설치하는 Extension은 Desktop 웹 브라우저를 이용하는 시청자에게 자동적으로 보여집니다. 시청자는 Extension을 볼 수 있고 규칙에 어긋나는 Extension을 신고할 수 있습니다.

Extension은 몇개의 컴포넌트를 가집니다.
* 스트리머가 설치 및 구성 할 수 있는 HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 일반적인 Extension 설치 과정의 한 부분이 됩니다.
* 시청자가 체험 가능한 HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 스트리머 방송 페이지에 iframe(inline frame)으로 만들어 집니다.
* (Optional) 스트리머가 체험 가능한  HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 스트리머 대시보드 페이지에 iframe으로 만들어집니다. 스트리머는 방송 중 특별한 명령(ex: 투표)를 지시가 가능합니다.
* (Optional) Extension Backend Service(EBS).<br>
  프론트 엔드와 커뮤니케이션 가능하고 데이터 및 상태를 저장 할 수 있는 backend 웹 서비스를 지칭합니다. 모든 시청자에게 데이터를 보내기 위해 AJAX 요청을 받거나 트위치의 PubSub 아키텍처를 사용할 수 있습니다. 

2가지 유형의 Extension:
* Video Player 아래 Panal 영역에 위치하는 Panal Extension.
* 투명한 오버레이로써 Video Player 상에 위치하는 Video-Overlay Extension.

### 스트리머 체험
스트리머는 대시보드의 Extension Manager 탭에서 Extension의 목록을 볼 수 있으며 설치 할 수 있습니다. Extension의 추가, 제거 및 동작 여부를 제어 할 수 있습니다.
<br><br>
Extension 설치를 한다고 해서 작동을 하지는 않습니다. 작동을 위해 스트리머는 Extension 설정(개발자가 필요로 한다면)을 해야합니다. 그 다음 Extension을 활성화 하면 시청자에게 보여지게 됩니다.

### 도움이 필요하다면..
이 문서에 문제가 있거나 정보가 더 필요하거나 질문이 있다면 [Extensions category of the Twitch Developer Forums](https://discuss.dev.twitch.tv/c/extensions) 에 방문해주세요.

Architecture Overview
=====================
![Alt text](https://media-elerium.cursecdn.com/attachments/215/369/extensionsguide-architecture.png)
Extension은 front-end **iframe** 입니다. 일반적으로 Extension은 AJAX를 이용해 **Extension Backend Service(EBS)**와 커뮤니케이션을 합니다.
(EBS는 Extension 개발자가 개발, 배포, 유지보수 하는 웹 서비스입니다.)
<br>

Extension를 테스트 하는 중에는 iframe 및 Extension의 관련한 자원들은 개발자가 제공하는 URL로서 이루어집니다. 이것은 개발자가 개발 중 수정을 더 빠르게 해줍니다. Extension이 완성돼 트위치가 검토 중이거나 후에 상용화 되면 해당 자원들은 트위치가 제공하는 CDN(Content Delivery Network)에 복사해서 제공됩니다. 
<br>

Extension의 iframe은 반드시 **Extension Helper** Javascript 파일을 포함(import)해야 합니다. Extension Helper는 트위치에서 만들어져 제공합니다. **PubSub** 이벤트, 방송 정보, 인증을 다루는 메소드들을 지원합니다. 트위치 PubSub는 backend 서비스가 클라이언트에 실시간 메시지를 전달 할 수 있게끔 만드는 시스템입니다. 

### Opaque IDs

Extension Helper는 다음을 제공합니다.
* 방송 채널에 대한 Context 정보(ex: Video 화질, 스트리머 지연속도, 채널 ID..)로 호출되는 콜백 함수.
* Opaque identifier는 트위치에 로그인 하지 않은 시청자를 식별합니다. opaque ID를 사용하면 개발자는 시청자의 인증 상태를 알아낼 수 있습니다. 또한 로그아웃한 유저도 opaque ID를 갖습니다. 하지만 채널들과 세션들 사이에서 동일 하다고 보장할 수 없습니다.

Opaque ID는 모든 채널에서 지속됩니다. ID를 명시적으로 바꿔달라고 요청하지 않는 이상 변경되지 않습니다. 개발자는 opaque ID를 키로 사용하여 각각의 사용자 정보를 EBS에 저장하는 것을 권장합니다. 만약 Extension 시청자들의 트위치 숫자 ID를 알아야 할 필요가 있다면, Extension의 **Extension Capbilities** 섹션에 **Request Identify Link** 아래 박스를 체크 하세요. 시청자가 트위치 identify를 공유하겠다고 결정하면 트위치 숫자 ID는 Extension Helper의 onAuthorized() 콜백 함수로 제공됩니다. 이 콜백 함수 및 JWT Token에 대한 자세한 정보는 여길 [Extensions Reference](https://dev.twitch.tv/docs/extensions/reference) 확인하세요. 

### 인증 토큰 및 범위
Extension Helper는 Authentication JWT([JSON Web Token](https://jwt.io/))을 가진 iframe을 제공합니다. iframe이 EBS와 커뮤니케이션을 하고 싶다면 HTTP 헤더에 토큰을 실어 보냅니다. JWT는 Extension 개발자와 트위치만 아는 비밀키로 트위치가 서명합니다. EBS는 전송된 JWT를 비밀키로 확인할 수 있고 메시지가 타당한 곳으로 부터 전송된 것을 확신 할 수 있습니다. 또한 JWT 자체는 전송자의 역할에 대해 신뢰성있는 정보를 포함합니다.(ex: 메시지가 시청자로 부터 전송 됐는지 스트리머로 부터 전송 됐는지)

## Focus
시청자가 Extension을 클릭한 경우 video player 키보드 단축키가 계속 작동되도록 video player에게 포커스를 보냅니다. 만약 Extension이 시청자에게 form field element(ex: "field", "select", "textarea")를 클릭하도록 요구하고 시청자가 클릭한다면 그 form element에 포커스 됩니다. 

Firefox 브라우저에서는 포커스가 안되는 문제가 있습니다.

Design Best Practice
====================
디자인은 주관적인면이 있습니다. 그래서 트위치 플랫폼에서는 "좋은 디자인은 무엇이다!"를 강요하지는 않습니다. 하지만 Extension을 이용하는 시청자들에게 좋은 경험할 수 있도록 몇가지 모범사례가 있습니다. Extension 개발을 하는 언제든 다음 사항들을 고려해주세요. 

* **브랜딩(Branding)** — Extension 브랜딩은 깔끔하고 인식하기 쉬우며 독특해야합니다. 로고를 적당히 사용하고 색상을 사용하여 트위치에서 자신의 브랜드를 강화하세요. Extension에는 트위치, Glitch 로고를 포함한 트위치 브랜드 요소를 포함할 수 없습니다.

* **색상(Color)** — 색상을 너무 많이 사용하지 마세요. 만약 특정 게임의 Extension이라면 적절한 곳에 무료 색상 표를 사용하세요. Video-Overlay Extension는 후면 Extension 컨텐츠(비디오, 게임 데이터, 그외 방송 Overlay components)와 어떻게 결합될지 고려하세요. 강조 및 동작 호출을 위해 주요 색상을 사용하세요. 상호작용하는 요소와 비상호작용하는 요소에 같은 색상을 사용하는 것을 피하세요.

* **뚜렷한 접근성(Contrast and accessbility)** — 자신의 디자인을 가능하면 이용하기 쉽게 항상 색상간 대비되도록 만드세요. 배경 색상과 유사한 링크를 피하세요. 색맹 시청자를 고려하세요.

* **레이아웃(Layout)** — 눈으로 보기 쉽게 정렬하거나 계층적으로 만드세요. 또한 아래에 "Video-overlay consideration"을 참고하세요.

* **글자(Typography)** — 글자 강조를 위해 weight, size 그리고 color를 사용하세요. 가능하다면 하나의 글자체(font)만 사용하세요. 여러개 글자체를 사용하면 Extension이 따로노는 느낌을 주게 됩니다. 대신에 글자체 스타일(bold, italic)을 사용하고 몇개의 글자 크기를 사용하세요. 브라우저 내장 글자체를 사용하세요: 효율적이며 모든 브라우저에서 동작합니다. 다음은 웹에서 안전한 글자체들입니다.
  * Serif: Georgia, Palatino Linotype, Times New Roman
  * Sans serif: Arial, Helvetica, Comic Sans MS, Impact, Lucida Sans Unicode, Lucida Grande, Tahoma, Geneva, Trebuchet MS, Verdana

* **모바일 Extension(Extensions on mobile)** — Overlay, Video-Panel Extension은 트위치 앱, 트위치 웹사이트에서 지원하지 않습니다.

* **패널 고려사항(Panel considerations)** — Panel Extension은 iframe 스크롤을 피하기 위해 너비 320px, 높이 500px로 제한됩니다. 이 크기내에서 최대한의 가독성을 위해 어느 텍스트건 10px 내부 padding을 설정하도록 노력하세요.

* **상태 피드백(Stateful feedback)** — 어디서든 가능하면 사전로드하세요. 특히 Overlay Extension에서는 더욱이 권장합니다. Overlay Extension은 시청에 불편을 줄 수 있는 "loading" 표시자 사용을 자제해야 합니다. 만약 Panel Extension에서 로딩 상태를 표시할 필요가 있다면 가능한 명확하고 간결하게 디자인 하세요. loading 표시자 추가를 고려하세요. 
자신의 Extension에 필요할 것 같은 상태 표시자(커뮤니케이션 업데이트, 에러, 다른 상태 등..)를 사용하세요. 자신의 Extension 사용자가 현재 Extension이 무슨 상황인지 추측해야만 해서는 안됩니다.

* **네비게이션(Navigation)** —  일반적으로 Extension은 여러겹의 네비게이션을 자제해야합니다. 만약 계층적인 네비게이션을 반드시 사용해야 한다면 사용자 자신이 어느 상황에 있는지 알 수 있도록 항상 명확한 경로를 제시하세요. 각각의 네비게이션이 필요한지 본인 스스로에게 물어보세요.

### Video Overlay Design Considerations
Video-Overay Extension은 시청자의 경험을 향상시키는데 있습니다. 그렇기에 각 Extension 요소는 중요한 real estate를 포함한다는 것을 알고 있어야 합니다.