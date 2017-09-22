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

Opaque ID는 모든 채널에서 지속됩니다. ID를 명시적으로 바꿔달라고 요청하지 않는 이상 변경되지 않습니다. 개발자는 opaque ID를 키로 사용하여 각각의 사용자 정보를 EBS에 저장하는 것을 권장합니다. 만약 Extension 시청자들의 트위치 숫자 ID를 알아야 할 필요가 있다면, Extension의 **Extension Capabilities** 섹션에 **Request Identify Link** 아래 박스를 체크 하세요. 시청자가 트위치 identify를 공유하겠다고 결정하면 트위치 숫자 ID는 Extension Helper의 onAuthorized() 콜백 함수로 제공됩니다. 이 콜백 함수 및 JWT Token에 대한 자세한 정보는 여길 [Extensions Reference](https://dev.twitch.tv/docs/extensions/reference) 확인하세요. 

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
각 Extension은 개발자 사이트의 **Extensions** 섹션 내에서 독립적으로 관리됩니다. 각각의 버전을 위해 **Version Status** 탭에서는 자신의 Extension 생명주기를 컨트롤 할 수 있도록 도와줍니다.

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

<h3 id="extension-capabilities"> Extension Capabilities</h3>
각 Extension은 고유합니다. 트위치는 Extension의 능력(potential)을 최대화하기 위한 선택가능한 보조기능(capabilities)을 가지고 있습니다. 이는 **Extension Capabilities** 섹션에 있습니다.

* 자신의 Extension이 시청자의 트위치 숫자 ID를 알아야 한다면 **Request Identity Link** 박스에 체크하세요. 시청자가 트위치 ID 공유에 동의한다면 Extension Helper의 <code>onAuthorized()</code> 콜백 함수로 트위치 숫자 ID가 제공됩니다. 콜백 함수나 JWT 토큰에 대해 자세히 알고 싶다면 [Extensions Reference](https://dev.twitch.tv/docs/extensions/reference)를 참조하세요.

* 스트리머가 Extension을 활성화 하기 전에 적절하게 설정하도록 요구할 수 있습니다. 활성화된 Extension이 잘못 설정되어 시청자에게 혼란스런 에러 메시지를 출력하는 경우 유용합니다. **Required Configurations** 필드에 작성해서 스트리머 설정 요구사항을 강제할 수 있습니다. 이 필드에는 자신이 원하는 문자열 컨텐츠를 작성 가능합니다. [Creating Your Extension Backend Service (EBS)](#creating-your-extension-backend-service-ebs) 에서 더 다룹니다.

* 자신의 어플리케이션이 스트리머를 대신하여 동작을 수행해야 한다면 필수적으로 OAuth 범위가 **Required Broadcaster Ablities** 에 추가돼야 합니다. 콤마로 구분된 OAuth 범위 리스트를 이용하여 명시된 리다이렉트 URI의 인가 요청이 설정됩니다. 그리하여 사용자가 어플리케이션 요청 범위를 허용하면 리다이렉트 URI를 통해 사용자의 토큰이 개발자에게 보내집니다. ([범위 리스트(list of scopes)](https://dev.twitch.tv/docs/v5/guides/authentication/#scopes)에 대한 트위치 인증 가이드 참조), [Creating Your Extension Backend Service (EBS)](#creating-your-extension-backend-service-ebs) 에서 더 다룹니다.

* (Optional) 설정 또는 panel의 주요 기능으로써 외부 URL 접근이 필요하다면 **Whitelisted Config URLs** 또는 **Whitelisted Panel URLs** 내에 URL 리스트를 작성하세요. Video-Overlay Extension에서는 외부 사이트 링크하는 것을 절대 금합니다.

### Asset Hosting
Extension을 처음 생성한 후 해당 Extension **Asset Hosting** 섹션에 기본(default) 자원 경로를 업데이트 해야합니다.
* 해당 Extension 버전과 관련된 모든 자원의 루트 URI를 가리키도록 **Testing Base URI**를 변경하세요. URI는 반드시 '/'로 끝나야합니다. URI는 전적으로 본인에게 달렸습니다. 버전과 관련될 필요는 없습니다. 테스트 동안 모든 자원은 이 URI로 부터 얻게됩니다. 그래서 아무것도 다시 제출할 필요없이 자신의 코드를 업데이트 할 수 있습니다. 로컬 테스트 방법에 대한 정보는 [Local Test](#local-test) 를 참조하세요.
* **Viewer Path**는 방송 페이지에서 시청자에게 보여질 화면인 HTML 파일을 포함합니다. 이 페이지는 지정된 Extension 유형에 따라 panel 영역 또는 Video-Overlay 영역에 표시됩니다.
* **Config Path**는 스트리머가 Extension을 활성화하기 전에 나타나는 설정화면인  HTML 파일을 포함합니다. 이 페이지는 동적 너비와 고정 높이(720px)인 iframe에 표시됩니다. Testing Base URI의 상대 경로여야합니다. 드물게 사용됩니다. (ex: 설치 중 설정)
* (Optional) **Live Config Path**는 대시보드 Live 모듈에서 스트리머에게 보여지는 화면인 HTML 파일을 포함합니다. Extension이 활성화된 동안 스트리머의 행동(ex: 투표 만들기)을 입력받는데 사용됩니다. Testing Base URI의 상대 경로여야합니다.

### Access
**Access** 섹션내 **Testing Accounts** 에 Extension을 테스트하는데 사용되는 모든 계정의 계정 ID를 추가하세요. 해당 Extension 버전을 테스트하는 중 접근하는 계정 ID (이름이 아닙니다.)의 리스트를 콤마로 구분하여 작성하세요.

특정 스트리머의 계정 ID를 **BroadCaster Whitelist**에 추가할 수도 있습니다. Whitelist가 승인되면 Whitelist를 제외한 스트리머는 Extension을 설치할 수 없습니다. Whitelist가 비어있거나 없는 경우 모든 스트리머는 Extension 사용이 가능합니다. 계정 이름을 계정 ID로 변환하기 위해서는 [Translating from User Names to User IDs](https://dev.twitch.tv/docs/v5/guides/using-the-twitch-api/#translating-from-user-names-to-user-ids) 를 참조하세요.

Creating Your Extension Front End
=================================
이전에 서술했듯이 Extension은 front-end iframe입니다. 지금까지 자신의 Extension을 생성하고 첫번째 버전의 설정을 정의했으므로 iframe 안에 위치할 자원을 생성할 준비가 됐습니다. 

### The Extensions Boilerplate
개발자가 가능한 빨리 Extension을 개발할 수 있도록 Extensions Boilerplate를 제공합니다. 이는 Extension 개발을 위한 시작점과 쉽고 반복적인 배포가 가능한 로컬 테스트 환경을 제공합니다.

 [The Extensions Boilerplate Github page](https://github.com/twitchdev/extensions-samples/tree/master/boilerplate) 을 방문해서 clone 및 로컬 버전 deploy 지시를 따르세요.

 ### Extension Helper Library
 Extension의 iframe은 반드시 **Extension Helper** Javascript 파일을 포함(import)해야 합니다. Extension Helper는 트위치에서 만들어져 제공합니다. PubSub 이벤트, 방송 정보, 인증을 다루는 메소드들을 지원합니다. Asset Hosting 섹션에 정의된 HTML(Viewer, Config, Live Config) 파일은 반드시 Extension Helper를 포함해야 합니다. 다음과 같이 포함할 수 있습니다.

```javascript
<script src="https://extension-files.twitch.tv/helper/v1/twitch-ext.min.js"></script>
```

Extension Helper가 제공하는 콜백 및 함수들에 대한 자세한 정보는 [Extensions Reference](https://dev.twitch.tv/docs/extensions/reference) 를 참조하세요.

Managing Extension Secrets
==========================
각 Extension은 비밀키를 유지합니다. 비밀키는 사용자의 identity를 제공하는 JSON Web Token(JWT)를 서명하거나 확인하는데 사용됩니다. EBS(비밀키 인증을 지원하는 종단점)에서 Extension API 호출을 구현할때 이 인증 방법을 사용하세요.

트위치 Extension 기술은 JWT 유효성 검사를 위해 EBS와 트위치 API간에 공유되는 비밀키에 의존합니다. 비밀키는 긴 수명(100년)을 갖습니다. 하지만 더 나은 보안을 위해 비밀키를 자주 변경할 것을 강력히 권장합니다.

### JWT Roles
EBS, 트위치 모두 JWT를 생성합니다.

EBS는 API 호출을 위해 `external` 역할(role)로 JWT를 생성하고 서명해야합니다. 트위치는 그외의 역할로 JWT를 생성하기에 EBS는 사용자 인증 수행이 가능합니다. 두 사용법 모두(<code>external</code> 역할, 그외의 역할) 똑같은 비밀키를 사용합니다. (역할에 대해 더 알고 싶다면 [Extensions Reference]() 의 "JWT Schema"를 참조하세요.)

### Creating Your First Secret
1. **Extensions Dashboard**에 있는 **Settings** 페이지에 접속합니다.
2. 왼쪽 패널에 **Secret Keys** 를 클릭합니다.
3. 새 비밀키를 생성하기 위해 **Create New Secret** 을 클릭하세요. 다음 항목을 볼 수 있습니다.
   * **Key** — base64로 인코딩된 비밀키
   * **Active** — 비밀키가 활성화되는 시간(UTC)입니다. 비밀키를 사용하기 전 트위치 서버와 EBS간에 비밀키가 전파됩니다.
   * **Expires** — 비밀키가 만료되는 시간입니다. 이것은 비밀키로 JWT를 확인하기 위한 마지막 시간입니다. 이 시간 이후의 JWT는 폐기되어야 합니다.

### Rotating Secrets
비밀키는 만료되기 전에 반드시 교체해야합니다. 교체하려면 **Secret Keys** 아래 **Settings** 페이지에서 새로운 비밀키를 생성하세요. 사용하던 비밀키가 만료되고 새로운 비밀키가 활성화돼 표에 표시될것입니다.

활성화 딜레이 때문에 일정 시간동안 여러개의 활성화된 비밀키를 가질 수 있습니다. 그 중 서명을 위해서 만료시간이 제일 늦은 활성화된 비밀키를 사용하세요.

원한다면 [Create Extension Secret](https://dev.twitch.tv/docs/extensions/reference/#create-extension-secret) endpoint로 새로운 비밀키 생성이 가능합니다. 더 높은 수준의 보안을 위해 스케줄링 코드로 비밀키를 교체할 수도 있습니다.

### Revoking All Secrets
언제든지 자신의 비밀키가 노출됐다면 **Secret Keys** 내 **Settings** 페이지에서 **Revoke All Secrets** 옵션을 사용할 수 있습니다. 킬 스위치라고 생각할 수 있습니다. 명시된 Extension과 관련된 모든 비밀키가 즉시 삭제됩니다.

원한다면 [Revoke Extension Secrets](https://dev.twitch.tv/docs/extensions/reference/#revoke-extension-secrets) endpoint 을 통해 모든 비밀키를 폐기할 수 있습니다.

<h1 id="creating-your-extension-backend-service-ebs">Creating Your Extension Backend Service (EBS)</h1>
EBS는 자신의 Extension을 지원하기 위한 backend 서비스입니다. EBS는 자신이 원하는 프로그래밍 언어로 작성 가능합니다. Extension 본질에 충실하도록 일반적으로 다음과 같은 기능이 요구됩니다.

### Verifying the JWT
EBS는 Extension으로부터 오는 AJAX 통신을 확인할 수 있어야합니다. 예를들면 스트리머만 설정할 수 있는 특정 작업의 통신 또는 EBS와 현재 연결중이고 비확인 시청자가 아닌 확인된 트위치 시청자의 통신이 될 수 있습니다. 이전에 서술했듯이 front-end iframe은 Extension Helper에서 <code>onAuthorized()</code> 콜백 함수를 통해 서명된 JWT를 포함합니다. Extension 개발자는 EBS에 AJAX 요청시 이 토큰을 헤더에 포함할 수 있습니다.

JWT 서명, 검사 라이브러리는 [https://jwt.io](https://jwt.io/) 에서 여러 프로그래밍 언어로 사용가능합니다. 보통 다음과 같은 인터페이스를 호출하게 됩니다.
<pre><code>verify(&ltjwt&gt, &ltsecret&gt)</code></pre>

인자
* `<jwt>` 는 Extension Helper을 통한 트위치 backend로부터 전송된 토큰이며 이 토큰은 헤더에 실려 EBS로 전달됩니다. 
* `<secret>` 은 사전에 설정된 공유 비밀키입니다.

트위치 Extension에 의해 사용된 JWT는 만료되며 만료기간 후의 토큰 확인은 실패합니다. Extension Helper는 자동적으로 토큰을 다시 만들며 <code>onAuthorized()</code> 콜백을 재호출 합니다. 항상 Extension Helper가 제공하는 최신 JWT를 사용하세요. 

JWT의 전체 스키마 혹은 각 필드에 대한 정보를 자세히 알고 싶다면 [Extensions Reference](https://dev.twitch.tv/docs/extensions/reference)의 "JWT Schema"를 참조하세요.

### Signing the JWT
EBS는 Extension Helper가 서명한 토큰을 확인하는 것 외에도 JWT 인증 방식을 사용하는 다양한 Extension endpoint를 호출하기 위해 새로운 JWT에 서명을 할 수 있어야합니다. EBS가 JWT를 서명할 수 있도록 다음과 같은 형식을 사용하세요. <pre><code>{
  "exp": 1502646259,
  "user_id": "27419011",
  "role": "external"
}</code></pre>

필드 : 
* `exp` 는 토큰이 만료되는 시간입니다. (UNIX epoch timestamp : 1970년 1월 1일을 기준으로 하는 1초 단위 시간) 잠재적 positive time drift를 다루기 위해서 버퍼를 제공해야 함을 명심하세요.
* `user_id` 는 Extension을 사용하는(owns) 트위치 사용자 ID입니다. 
* `role` 는 `external`로 설정합니다.

이 필드들에 대한 자세한 정보는 Extensions Reference의 ["JWT Schema"](https://dev.twitch.tv/docs/extensions/reference#jwt-schema) 섹션을 참조하세요.

JWT 라이브러리를 사용해 토큰을 서명하세요. 보통 다음와 같은 인터페이스를 호출합니다.
<pre><code>sign(&lttoken&gt, &ltsecret&gt)</code></pre>

인자 :
* `<token>` 은 이전 단계에서 만든 토큰 객체입니다.
* `<secret>` 은 사전에 설정된 공유 비밀키입니다.

다음과 같은 형식으로 서명된 JWT를 요청 헤더에 담아 전송하세요.
<pre><code>Authorization: Bearer &ltsigned JWT&gt</code></pre>

### Broadcasting via PubSub
EBS에서 PubSub를 통해 실시간 메시지, 상태를 전송하고 싶을때는 [Send Extension PubSub Message](https://dev.twitch.tv/docs/extensions/reference/#send-extension-pubsub-message) endpoint 를 사용하게됩니다. 이 endpoint를 사용하여 해당 방송의 모든 시청자 또는 귓속말을 통한 특정 사용자에게 메시지 전달이 가능합니다. 

트위치 PubSub 사용을 위한 다음의 기술 가이드라인을 준수하세요.
* 한 채널에 초당 하나의 메시지
* 5KB 메시지 크기

이 가이드라인은 트위치 시스템의 확장성, 안정성을 보장하는 기준입니다.

### Requesting Broadcaster Abilities
자신의 어플리케이션이 스트리머를 대신하여 동작을 수행해야 한다면 필수적으로 OAuth 범위가 **Required Broadcaster Ablities** 에 추가돼야 합니다. ([Extension Capabilities](#extension-capabilities) 참조) 

EBS가 OAuth 콜백을 받고 접큰(access) 토큰을 받으면 [Set Extension Broadcaster OAuth Receipt](https://dev.twitch.tv/docs/extensions/reference/#set-extension-broadcaster-oauth-receipt) endpoint 를 호출하세요.

Extension을 생성할때 리다이렉트 URI가 https://localhost/ 로 설정된 OAuth 어플리케이션이 자동으로 등록됩니다. Extension을 hosted test 상태로 변경하면 이 리다이렉트 URI를 자신의 EBS가 호스팅되는 곳으로 설정해야 합니다. 

<h3 id="required-configurations">Required Configurations</h3>
원한다면 스트리머가 Extension을 활성화 하기 전에 적절하게 설정하도록 요구할 수 있습니다. 활성화된 Extension이 잘못 설정되어 시청자에게 혼란스런 에러 메시지를 출력하는 경우 유용합니다. **Required Configurations** 필드에 작성해서 스트리머 설정 요구사항을 강제할 수 있습니다. 이 필드에는 자신이 원하는 문자열 컨텐츠를 작성 가능합니다. 이 필드에 작성한 문자열을 사용해 버전마다 다른 설정을 쉽게 요구할 수 있습니다. 그래서 만약 새버전에서 재설정이 필요하면 스트리머는 새버전을 활성화하기 전에 재설정을 해야할 것입니다.

채널에서 Extension에 대한 설정이 제대로 됐다고 EBS가 판단하면 [Set Extension Required Configuration](https://dev.twitch.tv/docs/extensions/reference/#set-extension-required-configuration) endpoint 를 호출하세요.

Testing Your Extension
======================
Extension을 생성하고 보조기능(capabilities) 세팅이 완료됐다면 개발 및 테스트가 준비된 것입니다. Extension 개발은 일반적으로 로컬에서 Extension과 EBS를 반복하며 개발합니다. 그 다음 hosted 상태로 전환하여 추가적인 테스트 및 확인을 합니다.

<h3 id="local-test">Local Test</h3>
로컬에서 테스트를 위해 Extension Boilerplate를 사용하거나 로컬 웹 서버 같은 것으로 본인 마음대로 할 수 있습니다. 몇몇 명령은 HTTPS를 필요로 하므로 자신의 시스템에 개인 인증서(self-signed certificate) 생성 및 설치를 해야합니다. 

스트리머 대시보드에 있는 **Extension Manager** 에서 자신의 방송 채널에 Extension을 설치할 수 있습니다. Extension이 Local Test 또는 Hosted Test 모드인 동안 테스트 whitelist에 등록된 시청자들만 볼 수 있습니다. 그외 시청자는 Extension이 보이지 않습니다.

다음 단계를 따르세요.
* 트위치에 로그인 한 상태에서 https://www.twitch.tv/dashboard 를 접속하세요.
* **Extension Manager** 탭을 클릭하세요.
* Extension을 계정에 설치하세요.
* Extension 설정을 해야하면 [Required Configurations](#required-configurations) 을 참조하세요.
* Activate를 클릭하세요.
* Extension을 테스트하세요.

<h3 id="hosted-test">Hosted Test</h3>
트위치 CDN으로 Extension을 테스트할 준비가 됐다면   

1. 디렉토리 구조를 유지하여 front-end 자원들을 모두 압축형식으로 만듭니다. 
    * 파일들은 압축폴더 루트안에 있어야 합니다.
    * Extension이 아닌 파일은 포함해선 안됩니다.
2. **Extension Dashboard**에서 해당 Extension의 **Versions** 탭으로 이동하세요.
3. 왼쪽 패널에서 **Version Assets**을 클릭하세요.
4. 알맞은 압축파일을 선택하세요.
5. **Upload Assets**를 클릭하세요.

이제부터 모든 자원은 트위치 CDN에서 호스팅됩니다. 하지만 review 과정은 시작된게 아닙니다. CDN으로 실행되는 Extension을 테스트할 기회가 주어지는 것입니다.

업로드에 대한 문제 발생시 (파일 접근 불가, 부적절한 이미지 크기 등) 이메일로 알림이 보내집니다. 알림을 받기 위해서는 Extension 생성시 반드시 제작자 이메일(Author email)을 작성하세요. Extension을 생성하면 확인 메일을 받게됩니다. 확인 메일 안에 링크를 클릭하세요. 그렇지 않으면 파일업로드 문제, Extension 승인 등의 알림 이메일을 받을 수 없습니다. 

### Submitting Your Extension for Review
트위치 CDN으로 Extension이 잘 작동한다면 review를 위한 제출 준비가 됐습니다. 제출하기 전 해당 Extension 버전의 **Version Details** 에 있는 **EULA or Terms of Service URL** 과 **Privacy Policy URL** 필드를 작성해주세요. 

**Screenshots** 섹션에 Extension이 작동하는 이미지를 업로드 하도록 권장합니다. 이미지는 PNG, JPG 또는 GIF 중에 가능합니다. 최소(권장) 크기는 1024 * 768 입니다. 이미지는 반드시 4:3 비율이어야 합니다. Extension을 제출하기전 최소 하나 이상의 이미지를 업로드 하도록 권장합니다.

Extension을 제출하기 전 반드시 해당 버전의 **Review Details** 섹션을 작성해주세요. 다음의 필드와 문제에 대해 각별히 주목하세요.
* **Name of Channel for Review** — review 과정에 사용할 채널 URL를 작성하세요. 트위치가 리뷰를 마치기 위해선 승인이 될때까지는 트위치 채널 페이지에서 시연되고 전체 기능이 동작하는 버전이 요구됩니다. 또한 Extension이 작동하는데 실제 데이터(live data)가 요구된다면 트위치 리뷰어들이 Extension의 전체적인 기능을 사용할 수 있도록 필수 데이터(necessary data)로 시뮬레이션 해주세요. 
* **Walkthrough Guide and Change Log** — 빠른 리뷰를 위해 Extension 기능에 대한 명확하고 철저한 검토(Walkthrough) 가이드 문서 제공을 매우 권장합니다.

   만약 새로운 버전의 출시라면 변경 로그(change log)를 포함해주세요.

   검토 가이드 및 변경 로그를 제공하지 않으면 리뷰 시간이 늘어나거나 거부 당할 수 있습니다. 

**Review Details** 섹션을 작성한 후 자원을 업로드한 버전의 **Version Status** 섹션에서 **Mark In Review** 을 클릭하면 Extension 제출이 가능합니다.

### Do’s and Don’ts
스트리머와 시청자들이 Extension으로 좋은 체험할 수 있도록 트위치는 개발자가 반드시 지켜야 할 정책을 가집니다. 리뷰 과정동안 트위치는 Extension이 정책을 위반하는지 검사합니다. *정책을 위반하는 Extension은 거부됩니다.*

모든 정책 리스트는 **Guidelines and Policies** 를 참조하세요. 리뷰 하기 전에 대한 대부분의 공통 이슈는 리스트 아래쪽에 있습니다. 리뷰 시간과 거부 가능성을 최소화 하기 위해 다음의 모든 단계를 따라주세요.

해야 하는 것:
* 난독화 되지 않은, 사람이 읽을 수 있는 코드로 제출하세요.
* 첫 JavaScript 파일로서 트위치 Extension Helper를 포함하세요.
* 리뷰 기간동안 테스트 방송에서 Extension의 모든 기능들에 접근할 수 있도록 하세요.
* Extension에 대한 요약과 설명을 정확히 작성하세요.
* 사용자들이 Extension에서 클릭 가능한, 보여지는 URL들은 모두 [Extension Capabilities](#extension-capabilities) 섹션에 작성하세요.
* 요구되는 모든 곳에 HTTPS 프로토콜을 사용하세요.
* 자신의 Extension이 OAuth가 필요하다면 Extension 설정에서 `required_broadcaster_abilities` 필드에 필수 OAuth 범위(scope)를 작성하세요.
* 트위치 Extension CDN으로만 JavaScript, CSS를 로드하세요.
* 리뷰 도중엔 Extension과 Extension backend는 안정적 상태를 유지해야 합니다.

다음과 같은 Extension은 제출하지 마세요 : 
* Video-Overlay Extension에서 브라우저 창을 여는 링크를 가지는 경우
* Extension에서 소리(음악, 음성)가 재생되는 경우
* 광고나 스폰서 컨텐츠를 포함하는 경우(개발자 자신의 브랜딩 제외)
* 트위치, 아마존이 소유, 운영하지 않는 사이트에 접속이나 행동을 유도하는 경우
* 트위치 플랫폼 밖에서 업그레이드 판을 판매하는 경우
* Extension 링크로 이동한 페이지에서 물건을 판매하는 경우
* 플래시, 브라우저 Extension 또는 브라우저 플러그인을 사용하는 경우
* 어떠한 방법으로 pop-up 경고를 사용하는 경우
* JavaScript 코드에서 <code>eval</code> 을 사용하는 경우
* HTML 파일 내에서 iframe을 생성하는 경우
* 브라우저 창에서 곧바로 AJAX를 렌더링하는 경우
* <code>console.log</code> 명령이 있는 경우
* 아래 항목을 제외한 트위치 사용자 데이터를 수집하거나 저장하는 경우 
   * 트위치 플랫폼에 독점적인 어플리케이션, 트위치 사용자 경험을 향상시킬 수 있는 명백한 혜택을 만들 경우
   * 운영상의 커뮤니케이션이 필요한 경우, 예를들면 사용자들이게 디지털 아이템을 상환하는 경우 또는 고객서비스 목적 업데이트를 사용자에게 알리는 경우
   * 정기적으로 홍보용 자료를 전송하거나 어플리케이션의 기능, 혜택 알림을 모든 트위치 사용자에게 보내는 경우 (사용자가 홍보용 자료를 거절할 수 있는 메커니즘을 제공하고 통신의 주요 목적이 경쟁 제품 또는 플랫폼을 유도하거나 홍보하는 것이 아닌 경우에 한해서 가능)
   * 트랜잭션 과정에서 필요한 경우

### Making Changes
Extension이 리뷰인 중에는 어떠한 버전의 자원이나 설명을 수정할 수 없습니다. 수정이 필요한 경우 **Return to Testing**을 클릭하여 테스트 버전으로 되돌리세요. 수정을 한 후에는 트위치 CDN으로 수정된 자원을 업로드하세요. 그 다음 리뷰를 재신청하세요. 이 경우 리뷰 순서는 초기화 됩니다.

Releasing Your Extension
========================
Extension이 승인됐다면 승인된 버전의 **Release** 버튼을 클릭하여 상용화(release) 할 수 있습니다. 이전 버전의 Extension이 상용화 상태라면 자동적으로 폐기대상(Deprecated) 상태로 변경됩니다. Extension은 동시에 하나의 버전만 상용화 상태에 있게됩니다.
 
Updating Your Extension
=======================
상용화 된 Extension을 업데이트 하기 위해선 반드시 Extension의 새로운 버전을 만들고 리뷰 제출을 해야합니다. 새로운 버전을 만들기 위해 관리하고자 하는 Extension에서 **Versions** 탭을 클릭하세요. Extension의 모든 버전과 상태들이 표에 표시됩니다.

새 Extension 버전이 승인됐다면 스트리머가 사용할 수 있게 상용화 할 수 있습니다. 이전 버전은 사용 중지가 되고 **Version Status** 페이지 하단에서 확인할 수 있습니다.

Deleting Your Extension
=======================
Extension을 더 이상 지원하지 않겠다고 결정했다면 해당 Extension **Settings** 섹션에서 **Delete Extension**을 클릭하세요. 이는 모든 스트리머의 Extension Manager 페이지에서 자신의 Extension을 제거합니다.

특정 버전의 Extension을 삭제하는 것은 불가능합니다. **삭제된 Extension은 복구할 수 없습니다.**

Guidelines and Policies
=======================
### Technical Guidelines
상용화된 Extension은 반드시 트위치 CDN으로 부터 모든 HTML, JavaScript, CSS 파일이 배포 돼야 합니다. 트위치 서버로 배포될 스트리머, 시청자의 HTML, JS, CSS 파일을 제작한다면 다음 가이드 라인을 준수하세요. 그렇지 않으면 Extension은 승인되지 않을 것입니다.
* <code>eval()</code> 을 사용하지마세요. 만약 <code>eval()</code> 를 사용하는 라이브러리에 의존하고 있다면 코드를 축소한(minified) JavaScript로 만들지 말고 <code>eval()</code> 의 출처가 어딘지 트위치 리뷰어가 알 수 있도록 따로 두세요.
* 리뷰를 위해 Extension을 제출하기 전에 Javascript의 모든 콘솔에 로깅하는 코드(console.log)를 제거하세요.
* 가능하면 언제라도 <code>.html</code> 파일에 모든 텍스트를 넣으세요. 문자열에서 동적으로 문자를 변경해야 하는 경우 가능하면 JavaScript 파일에 동적 문자열을 열거하세요. (불가능한 경우도 있습니다.)
* AJAX를 통해 동적으로 얻은 데이터를 즉시 DOM안에 삽입하지 마세요. (ex : JSON은 가능, HTML은 불가)
* HTML5와 UTF-8을 사용하세요.
* 다음의 브라우저를 지원하세요.
   * IE11(Windows)
   * 최신 2개 버전의 Chrome(OS X, Windows)
   * 최신 2개 버전의 Firefox(OS X, Windows)
   * Safari 9(OS X)
* HTML 파일에 첫 JavaScript 파일로 트위치 Extension Helper를 포함하세요. 트위치 CDN으로 얻을 수 있습니다. https://extension-files.twitch.tv/helper/v1/twitch-ext.min.js
* Extension이 시청자를 위한 동작을 위해 써드파티 다운로드를 요구할 순 없습니다. (스트리머를 위해 추가적으로 요구되는 소프트웨어는 허용됩니다.)
* Extension의 모든 Javascript와 CSS 자원은 업로드 돼야 합니다.
* 리뷰를 위해 제출된 JavaScript 파일은 코드 축소(minified) 할 수 있지만 난독화(obfuscated) 하면 안됩니다.
* Extension은 소리를 재생하거나 그럴의도가 있는 코드를 포함할 수 없습니다.
* HTTPS를 사용하여 모든 자원(images, fonts 등)을 로드하세요.
* Extension은 Flash를 사용할 수 없습니다.
* Extension은 입력으로 더블클릭을 사용할 수 없습니다.
* Extension은 iframe을 사용할 수 없습니다.
* PubSub 트래픽은 다음의 제한을 준수해야 합니다.
   * 한 채널에 초당 하나의 메시지
   * 5KB 메시지 크기

### Localization Guidelines
개발자는 Extension의 front-end에 적절한 지역화(localization) 할 책임이 있습니다. 다음과 같은 방식으로 클라이언트 언어를 명시하세요.
* Extension의 URL 로딩에서 `language` 쿼리 파라미터를 사용 (ex : <code>https://<ext-id>.ext-twitch.tv/<ext-id>/<version>/viewer.html?language=en</code>) 
* `context` 콜백에 있는 `language` 필드 사용

### Content Policies 
모든 Extension은 콘텐츠 정책을 반드시 준수해야 합니다. Extension 리뷰 중 콘텐츠 정책에 반하는지 검사하며 준수하지 않는 경우 거부될 수 있습니다. 
* Extension 개발자는 [Developer Services Agreement](https://www.twitch.tv/p/developer-agreement) 와 [Terms of Service](https://www.twitch.tv/p/terms-of-service) 를 반드시 준수해야 합니다. 
* 회사 또는 개인으로 제출된 모든 Extension은 하나의 트위치 계정으로 생성해야 합니다. 
* 상업(Commerce) :
   * Extension은 스트리머에게 보상을 받는 대가로 차별화된 경험 및 기능을 제공할 수 있습니다. (ex : 특정 기능에 권한 별 접근(tiered access), 구매가능한 기능 추가 플러그인)
   * Extension은 시청자에게 보상을 받는 대가로 차별화된 경험 및 기능을 제공할 수 없습니다. (트위치/아마존 commerce instruments 사용과 관련된 것은 제외)
   * Extension은 트위치/아마존 commerce instruments와 관련되지 않은 곳과의 금전 거래 및 금전 거래 유도를 할 수 없습니다.
* 어느 Extension에서든 정적 또는 동적의 광고 컨텐츠를 게시할 수 없습니다.
* 사이트 밖으로의 링크(Off-site linking)
   * Video-Overlay Extension은 어떠한 종류의 링크든지 포함할 수 없습니다.
   * Panel Extension은 반드시 사이트 밖의 링크를 사용하기 위해 도메인 whitelist를 작성해 제출해야 합니다.
   * 사이트 밖으로의 링크는 반드시 Extension의 핵심 기능과 연관돼야 합니다.
   * 트위치 tv와 실질적으로 유사한 기능을 제공하는 사이트로의 링크는 사용 불가능 합니다. 
* Extension의 기본설정, 기능설정은 반드시 각각 Extension 설정 페이지, live 대시보드 iframe 페이지에서 관리돼야 합니다.
* Extension은 사용자가 트위치 외의 사이트에서 트위치의 컨텐츠를 사용하도록 유도하거나 사용에 대해 보상할 수 없습니다.
* Extension은 사용자가 트위치/아마존 소유물 외에 대한 특정 행동을 하도록 유도하거나 그 행동에 대해 보상할 수 없습니다.
* Extension은 시청자들에게 OAuth 권한를 요청하거나 요구할 수 없습니다.
* Extension은 사용자가 서비스 규약을 위반 하도록 유도하거나 위반 가능하게 할 수 없습니다. 예를 들어 : 
   * 성적 컨텐츠 명시
   * 폭력, 괴롭힘, 욕설 유도
   * 사기, 타인 사칭
   * 불법적 활동 
* Extension은 키보드 단축키로 기능을 수행할 수 없습니다.
* Extension의 컨텐츠 내에서 트위치 브랜드, 트위치 로고, 트위치 Glitch를 사용할 수 없습니다.
* Extension은 트위치의 보안, 안전 예방책을 무력화 하는 악의적인 코드를 포함할 수 없습니다.
* Extension은 지적재산권을 존중해야 합니다. Extension 개발자는 지적재산권 위반으로 제기되는 주장(claim)에 대해 책임이 있습니다. 그 주장이 해결될때까지는 Extension은 제거된 상태가 될 것입니다.
* Extension의 타이틀, 설명, 메타데이터에 Extension과 관련없거나 오해할만한, 그리고 과도한 표현을 사용할 수 없습니다. 
* Extension 목록(listing)에 Extension의 기능을 정확하고 완벽하게 설명해야 합니다.
* 모든 Extension은 하나의 아이콘과 최소 하나 이상의 front-end를 나타내는 스크린샷을 포함해야 합니다.
* 개발자는 Extension Manager 페이지 또는 훗날 Extension 검색 메커니즘을 가진 페이지에서 Extension의 위치 순서를 조작하려 시도해서는 안됩니다.
* 트위치는 언제든, 어떤 이유로든 Extension을 제거 할 권리를 가집니다.

### Content Security Policies
시청자, 스트리머의 프라이버시와 보안을 위해 트위치는 컨텐츠 보안 정책을 적용합니다. 개발 도중에 콘솔에서 다음과 같은 문구를 볼 수 있습니다.
<pre></code>Refused to load the stylesheet 'https://somedomain.net/bad.css' because it violates the following Content Security Policy directive: "style-src 'self' https://fonts.googleapis.com".</code></pre>
요청이 거부된 이유를 보고 자원(resource)을 로드하기 위해 그에따라 Extension을 수정 하세요.

기본적으로 다음의 경우를 제외하고 모든 자원은(assets) 트위치 CDN으로 로드돼야 합니다.
   * HTTPS 또는 WSS endpoint와 연결할때 (ex : XHR의 목적으로)
   * 이미지를 로드할때 (data URI 등 어디든)
   * Google 글자체는(fonts) 글자체, stylesheet를 위해 유효한 자원으로서 허용됨

또한 다음과 같은 제약사항을 따릅니다.
   * 인라인(inline) JavaScript를 사용할 수 없습니다.
   * HTTPS 또는 WSS와 같은 암호화된 프로토콜을 사용하지 않는 자원에는 접근이나 로드를 할 수 없습니다.
   * Extension이 존재하는 곳 외의 공간에서 Extension을 내장할 수 없습니다.

더 자세한 사항은 https://content-security-policy.com 을 참조하세요.