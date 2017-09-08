Introduction
============
트위치 Extension은 스트리머가 자신의 방송 페이지에 추가적인 컨텐츠를 넣을 수 있게 만듭니다. 스트리머가 설치하는 Extension은 Desktop 웹 브라우저를 이용하는 시청자에게 자동적으로 보여집니다. 시청자는 Extension을 볼 수 있고 규칙에 어긋나는 Extension을 신고할 수 있습니다.

Extension은 몇개의 컴포넌트를 가집니다.
* 스트리머가 설치 및 구성 할 수 있는 HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 일반적인 Extension 설치 과정의 한 부분이 됩니다.
* 시청자가 체험 가능한 HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 스트리머 방송 페이지에 iframe(inline frame)으로 만들어 집니다.
* (Optional) 스트리머가 체험 가능한  HTML/JavaScript를 이용한 프론트 엔드.<br>
  이것은 스트리머 대시보드 페이지에 iframe으로 만들어집니다. 스트리머는 방송 중 특별한 명령(ex: 투표)이 가능합니다.
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
Extension은 front-end **iframe** 입니다. 일반적으로 Extension은 AJAX를 이용해 **Extension Backend Service(EBS)** 와 커뮤니케이션 합니다.
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

* **사이즈 조절(Sizing)** — Extension UI를 디자인 할때 브라우저나 video player 크기를 조절하는 경우 UI도 그에따라 바뀌어야 함을 고려하세요. 기본적으로 Video-Overlay Extension은 스트리머가 방송중인 동안 모든 크기에서 동작합니다. Extension이 동작하지 않을 정도로 크기가 작아지는 경우 Extension을 숨기는 것도 고려하세요.

* **(Covering up player elements)** — Extension UI 요소가 트위치 video player 크기를 넘는다면 작동하지 않을 수 있습니다. 다음은 Video-Overlay Extension을 디자인할때 고려해야하는 영역의 다이어그램입니다.

![Alt text](https://media-elerium.cursecdn.com/attachments/215/371/extensionsguide-videoplayerelements.jpg)

* **iFrame 경계(iFrame boundaries)** — 드래그 앤 드랍 요소를 위해 시청자가 어떤 요소를 video player 화면 밖으로 이동시킬때 Video-Overlay Extension이 어떻게 반응할 것인지 고려하세요. 화면 밖으로 요소가 이동하는 것을 막거나 튕기도록하여(spring) 화면 안으로 들어오도록 만드세요.

* **비활성화(Disabling)** — 스트리머가 방송을 끄거나 다른 채널로 호스팅을 하면 Video-Overlay Extension은 스스로 비활성화됩니다. 시청자가 영상을 정지시키면 Extension iframe은 시청자가 영상 정지를 풀때까지 숨겨진 상태가 됩니다.

* **플레이어 조작 계층(Player control layers)** — Video-Overlay Extension은 모든 video player 조작 계층의 아래에 존재 한다는 것을 명심하세요. video player 조작 계층은 팝업메뉴, 마우스 오버, LIVE 표시자, 화면 좌상단에 나타나는 채널 정보를 포함합니다. 극장 모드, 꽉찬 화면, embed 화면들은 보통의 트위치 player와는 다른 레이아웃 UI를 가지는 것을 명심하세요.

![Alt text](https://media-elerium.cursecdn.com/attachments/215/370/extensionsguide-extensionlayers.jpg)

Extension Life Cycle
====================
각각의 Extension은 개발자 사이트의 **Extensions** 섹션 내에서 독립적으로 관리됩니다. 각각의 버전을 위해 **Version Status** 탭에서는 자신의 Extension 생명주기를 컨트롤 할 수 있도록 도와줍니다.

모든 Extension의 모든 버전은 **Local Test**로 시작합니다. 현재버전이 Local Test중인 동안 모든 자원(HTML, JavaScript, CSS, images, fonts 등)은 미리 정의된 테스트 URI를 통해 제공됩니다. 

개발자가 로컬에서 테스트한 버전에 만족했다면 **Hosted Test**로 변환합니다. 이것은 Extension의 모든 자원들을 트위치 CDN에 업로드하여 트위치에서 제공할때도 Extension이 잘 작동하는지를 개발자가 확인할 수 있습니다. 몇몇 유효검사(sanity check)는 업로드하며 이루어집니다. (ex: 아이콘 확인, 스크린샷이 적절한 크기인지) Local Test 또는 Hosted Test 도중 Extension은 테스트 계정 목록을 제공 받은 개발자와 트위치의 몇몇 스태프에게만 노출됩니다.

Hosted Test가 완료되면 개발자는 **Review**를 위해 Extension을 제출할 수 있습니다. 원하는 횟수만큼 Review를 요청할 수 있습니다. 하지만 한번에 하나의 버전만 Review가 가능합니다. Extension이 제출된 상황에서 Extension을 바꾸고 싶다면 Hosted Test로 되돌린 후 자원을 다시 업로드 하는 방법 밖에 없습니다. Review가 진행되는 동안 모든 테스트 계정은 전처럼 Extension 테스트를 할 수 있습니다. 

트위치가 Extension Review를 마친 후에는 다음 3가지 상태 중 하나의 상태가 됩니다.
* 보류(Pending Action) — 수정이 요구되는 상태입니다. 제공한 제작자 email 주소로 승인되지 않은 이유가 보내집니다. 개발자는 Extension을 테스트 상태로 되돌리고 문제를 수정하여 Review 재요청을 할 수 있습니다. 새로운 버전을 생성할 필요는 없습니다.

* 거부(Rejected) — Extension 부적절하고 어떤 상황에든 받아들여지지 않는 상태입니다. Extension의 클라이언트 ID(고유 식별자) 영구적으로 취소됩니다. 거부는 영구적이며 종결된 상태입니다.

* 승인(Accepted) — 개발자가 언제든 Extension을 상용화 할 수 있음을 알립니다. 승인된 Extension은 필요하다면 테스트 상태로 되돌릴 수 있습니다. 하지만 그럴경우 Review 과정을 다시 거쳐야합니다. 

상용화 하기 위해 개발자는 승인된 버전 Extension에서 Release 버튼을 클릭하면 됩니다. Extension이 공식적으로 노출되며 더 이상 업데이트 할 수 없습니다. 새로운 버전으로 교체만 가능합니다. 새로운 버전이 **출시**되면 이전에 출시된 어느 버전이든 **폐기 대상**으로 이행됩니다. 그리고 Extension 설치도 바로 업그레이드 됩니다.

새로운 버전이 출시되면 개발자는 :
* 몇몇 시청자 또는 스트리머는 잠시 옛 버전을 사용하게 된다는 것을 알아야합니다.
* EBS가 아직 바뀌지 않은 옛 버전의 트래픽을 처리할 수 있음을 보장해야합니다.

Creating Your Extension
=======================
Extension 개발을 시작하기 위해 트위치 개발자 사이트를 이용하게 됩니다. 이곳에서 Extension을 개발, 관리, 리뷰를 하게 됩니다.
1. 자신의 트위치 ID로 [트위치 개발자 사이트](https://dev.twitch.tv/)에 로그인 합니다.
2. [Extensions](https://vulcan.curseforge.com/dashboard/extensions) 페이지로 이동한 후 **Create Extension**을 클릭하세요.
3. Extension 생성 신청서 양식을 완성해주세요.
   * **이름(Name)** — Extension의 이름. 변경할 수 없으니 신중히 작성해주세요.
   * **유형(Type of Extension)** — Panel 또는 Video-Overay 중에 선택하세요.
   * **요약(Summary)** — 스트리머가 Extension Manager 탭 안에 있는 Extension 목록에서 볼 수 있는 문장입니다. Extension이 무엇을 하는지 간략히 1~2 문장으로 작성하세요. 더 많은 정보를 제공하기 위해선 설명(Description) 필드를 사용하세요.
   * **설명(Description)** — 요약보다 더 자세한 Extension의 기능을 작성하세요.
   * **제작자 이름(Author name)** — Extension Manager 탭에서 돈을 받게 될 제작자 또는 회사 성명을 작성하세요. 나중에 변경 가능합니다.
   * **제작자 이메일(Author email)** — Extension 제작자와 연락 정보를 작성하세요. Extension 생명주기(ex: 거부/승인 알림) 정보를 주고 받기 위해 사용됩니다. 트위치는 절대 누구에게도 이메일을 공개하지 않습니다.
   * **보조 이메일(Support email)** — 스트리머로 부터 질의를 받을 공개 이메일을 작성하세요.
4. (Optional) Extension의 로고를 추가하세요. 크기는 100px * 100px여야 합니다. 트위치 또는 Glitch 로고를 사용하지마세요. 만약 로고를 설정하지 않으면 기본 로고가 할당됩니다.
5. **Create Extension**을 눌러 Extension을 생성하세요. 생성 직후 확인 이메일을 받게됩니다.

Extension을 생성했습니다. 축하드립니다! 제공한 제작자 이메일 주소의 소유권자인지 확인하기 위해 이메일을 확인해주세요.

Extension 설명, 요약, 제작자 이름, 로고를 변경하기 원하면 **Settings** 탭을 클릭하세요. 이런 필드들은 특정 버전에 종속되지 않습니다. : Extension의 모든 버전에 적용됩니다.

Managing Extension Versions
===========================
**Create Extension**을 클릭한 후 **Version** 탭의 **Version Status** 섹션으로 이동합니다. 여기서 Extension 생명주기 상에 있는 Extension 상태를 볼 수 있고 필요하면 상태를 변경할 수 있습니다. 왼쪽 네비게이션 바에는 Extension 버전을 관리를 위한 추가적인 섹션을 볼 수 있습니다. 이는 아래에 설명합니다.

### Version Assets
자원(assets)을 트위치 CDN에 업로드할 준비가 됐다면 압축한 후 **Version Assets** 섹션에 업로드 하세요. 자원을 트위치 CDN에 업로드하기 전까지는 Extension Review 요청을 할 수 없습니다. 정보가 더 필요하다면 [Hosted Test](#hosted-test)을 참조하세요.

### Extension Capabilities
각 Extension은 고유합니다. 트위치는 Extension의 능력(potential)을 최대화하기 위한 선택가능한 보조기능(capabilities)을 가지고 있습니다. 이는 **Extension Capabilities** 섹션에 있습니다.

* 자신의 Extension이 시청자의 트위치 숫자 ID를 알아야 한다면 **Request Identity Link** 박스에 체크하세요. 시청자가 트위치 ID 공유에 동의한다면 Extension Helper의 <code>onAuthorized()</code> 콜백 함수로 트위치 숫자 ID가 제공됩니다. 콜백 함수나 JWT 토큰에 대해 자세히 알고 싶다면 [Extensions Reference](https://dev.twitch.tv/docs/extensions/reference)를 참조하세요.

* 스트리머가 Extension을 활성화 하기 전에 적절하게 설정하도록 요구할 수 있습니다. 활성화된 Extension이 잘못 설정되어 시청자에게 혼란스런 에러 메시지를 출력하는 경우 유용합니다. **Required Configurations** 필드에 작성해서 스트리머 설정 요구사항을 강제할 수 있습니다. 이 필드에는 자신이 원하는 문자열 컨텐츠를 작성 가능합니다. [Creating Your Extension Backend Service (EBS)](#creating-your-extension-backend-service-ebs) 에서 더 다룹니다.

* 자신의 어플리케이션이 스트리머를 대신하여 동작을 수행해야 한다면 필수적으로 OAuth 범위가 **Required Broadcaster Ablities** 에 추가돼야 합니다. 콤마로 구분된 OAuth 범위 리스트를 이용하여 명시된 리다이렉트 URI의 인가 요청이 설정됩니다. 그리하여 사용자가 어플리케이션 요청 범위를 허용하면 리다이렉트 URI를 통해 사용자의 토큰이 개발자에게 보내집니다. ([범위 리스트(list of scopes)](https://dev.twitch.tv/docs/v5/guides/authentication/#scopes)에 대한 트위치 인증 가이드 참조), [Creating Your Extension Backend Service (EBS)](#creating-your-extension-backend-service-ebs) 에서 더 다룹니다.

* (Optional) 설정 또는 panel의 주요 기능으로써 외부 URL 접근이 필요하다면 **Whitelisted Config URLs** 또는 **Whitelisted Panel URLs** 내에 URL 리스트를 작성하세요. Video-Overlay Extension에서는 외부 사이트 링크하는 것을 절대 금합니다.

### Asset Hosting