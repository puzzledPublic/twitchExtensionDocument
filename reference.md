<style>
table th{
    background-color : #dad8de;
    color : black;
}
</style>
Extension Reference
===================
이 레퍼런스는 Extension endpoints, [Javascript Helper](#javascript-helper), [JWT schema](#jwt-schema)에 대해 설명합니다.

Endpoint|Description
--------|-----------
[Create Extension Secret](#create-extension-secret)|명시된 Extension의 새로운 비밀키를 생성합니다. 또한 Extension 클라이언트가 새로운 비밀키로 자연스레 변환하도록 충분한 시간을 줘서 현재 비밀키 서비스를 중단합니다. 새로운 비밀키의 생성과 트위치가 새로운 비밀키를 사용하는 사이의 딜레이 시간은 필수 파라미터인 `activation_delay_secs` 에 명시됩니다. <br><br>기본 딜레이 시간은 300초(5분) 입니다. 만약 300 보다 적게 명시되는 경우 300초를 사용합니다. <br><br>새로운 비밀키를 설치할 준비가 됐을때만 이 함수를 사용하세요.<br><br> 트위치 개발자 사이트를 통해 최초 비밀키를 가집니다. (Extension 가이드의 [Creating Your First Secret](https://dev.twitch.tv/docs/extensions/#creating-your-first-secret) 참조) 필요하면 나중에 비밀키를 변환하는데 이 Endpoint를 사용하세요. 
[Get Extension Secret](#get-extension-secret)|명시된 Extension의 비밀키 데이터를 전달받습니다. (버전, 비밀키 객채의 배열) 각각의 비밀키 객체는 base64로 인코딩된 비밀키, 비밀키가 활성화되고 만료되는 UTC 시간을 포함합니다. 
[Revoke Extension Secrets](#revoke-extension-secrets)|명시된 Extension과 관련된 모든 비밀키를 삭제합니다. <br><br> 새로운 [Create Extension Secret](#create-extension-secret) 이 실행되고 클라이언트가 수동으로 새로고침 할때까지 모든 클라이언트를 즉시 중단합니다. 비밀키가 유출되고 즉시 제거해야 할때만 이 함수를 사용하세요.
[Get Live Channels with Extension Activated](#get-live-channels-with-extension-activated)|설치되고 활성화 중인 명시된 Extension의 Live 채널을 반환합니다. Live를 시작한지 얼마 안지난 채널은 리스트에 나타나는데 몇 분 정도 걸릴 수 있습니다. 막 스트리밍을 종료한 후의 채널은 몇 분 정도 동안 리스트에 나타날 수 있습니다. 
[Set Extension Required Configuration](#set-extension-required-configuration)|필수적인 스트리머 설정이 맞는지 확인한 후 명시된 Extension을 활성화 합니다. Extension 활성화 전 스트리머 설정이 필요한 경우에 사용합니다.
[Set Extension Broadcaster OAuth Receipt](#set-extension-broadcaster-oauth-receipt)|Extension이 요청하는 권한을 스트리머가 수락했는지 여부에 대해 표시합니다. 필수 `permissions_received` 파라미터를 통해 알 수 있습니다. Endpoint URL은 Extension iframe이 내장된 페이지의 채널 ID를 포함합니다.
[Send Extension PubSub Message](#send-extension-pubsub-message)|EBS(Extension Back-end Service)가 시청자, 스트리머와 통신할 수 있게 트위치는 PubSub(publish- subscribe) 시스템을 제공합니다. 이 endpoint 호출은 JavaScript Helper API에 있는 [send()](#send-functiontarget-contenttype-message) 함수와 같은 메커니즘을 사용하여 메시지를 전송합니다.

<h1 id="create-extension-secret">Create Extension Secret</h1>

명시된 Extension의 새로운 비밀키를 생성합니다. 또한 Extension 클라이언트가 새로운 비밀키로 자연스레 변환하도록 충분한 시간을 줘서 현재 비밀키 서비스를 중단합니다. 새로운 비밀키의 생성과 트위치가 새로운 비밀키를 사용하는 사이의 딜레이 시간은 필수 파라미터인 `activation_delay_secs` 에 명시됩니다. <br><br>기본 딜레이 시간은 300초(5분) 입니다. 만약 300 보다 적게 명시되는 경우 300초를 사용합니다. <br><br>새로운 비밀키를 설치할 준비가 됐을때만 이 함수를 사용하세요.<br><br> 트위치 개발자 사이트를 통해 최초 비밀키를 가집니다. (Extension 가이드의 [Creating Your First Secret](https://dev.twitch.tv/docs/extensions/#creating-your-first-secret) 참조) 필요하면 나중에 비밀키를 변환하는데 이 Endpoint를 사용하세요.

### Authentication
EBS에서 생성한 서명된 JWT. Extension 가이드의 [Signed the JWT](https://dev.twitch.tv/docs/extensions/#signing-the-jwt) 에서 설명하는 필요사항을 따르세요. 

### URL
```HTML
POST https://api.twitch.tv/extensions/<extension ID>/auth/secret
```

### Optional Query String Parameters
없음

### Example Request
Extension 'pxifeyz7vxk9v6yb202nq4cwsnsp1t' 의 새 비밀키를 생성합니다.
```d
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDIyOTM3MzUsInVzZXJfaWQiOiIyNzQxOTAxMSIsInJvbGUiOiJleHRlcm5hbCJ9.JLQWkGh86g-3Q1Ki-aWABeNQYgYYQIOYmpmNq3bTAVM' \
-H 'Client-Id: pxifeyz7vxk9v6yb202nq4cwsnsp1t' \
-H 'Content-Type: application/json' \
-d '{"activation_delay_secs": 300}' \
-X POST https://api.twitch.tv/extensions/pxifeyz7vxk9v6yb202nq4cwsnsp1t/auth/secret
```

### Example Response
```JSON
{
  "format_version": 1,
  "secrets": [
    {
      "active": "2017-08-09T15:49:41.553002151Z",
      "content": "TK4/j3AEJ9f14jODdxhX0kVcD/h6lKA9Sxz/CCUtVLI=",
      "expires": "2017-08-09T16:55:35.894628749Z"
    },
    {
      "active": "2017-08-09T15:50:35.90304258Z",
      "content": "B6OJ/WWA/bTsCbGYzaZwq3KdgDsC1485h4BKdnIBGOE=",
      "expires": "2117-07-16T15:50:35.90304258Z"
    }
  ]
}
```

<h1 id="get-extension-secret">Get Extension Secret</h1>

명시된 Extension의 비밀키 데이터를 전달받습니다. (버전, 비밀키 객채의 배열) 각각의 비밀키 객체는 base64로 인코딩된 비밀키, 비밀키가 활성화되고 만료되는 UTC 시간을 포함합니다. 

### Authentication
EBS에서 생성한 서명된 JWT. Extension 가이드의 [Signed the JWT](https://dev.twitch.tv/docs/extensions/#signing-the-jwt) 에서 설명하는 필요사항을 따르세요. 

### URL
```HTML
GET https://api.twitch.tv/extensions/<extension ID>/auth/secret
```

### Optional Query String Parameters
없음

### Example Request
Extension 'pxifeyz7vxk9v6yb202nq4cwsnsp1t' 의 비밀키 데이터를 요청합니다.
```d
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDIyOTM3MzUsInVzZXJfaWQiOiIyNzQxOTAxMSIsInJvbGUiOiJleHRlcm5hbCJ9.s_uedGCjGzlD5CpTQ_IpqUiQuB12dGsUIdbarLn3LrU' \
-H 'Client-Id: pxifeyz7vxk9v6yb202nq4cwsnsp1t' \
-X GET https://api.twitch.tv/extensions/pxifeyz7vxk9v6yb202nq4cwsnsp1t/auth/secret
```

### Example Response
```JSON
{
  "format_version": 1,
  "secrets": [
    {
      "active": "2017-08-09T15:49:41.553002151Z",
      "content": "TK4/j3AEJ9f14jODdxhX0kVcD/h6lKA9Sxz/CCUtVLI=",
      "expires": "2017-08-09T16:55:35.894628749Z"
    },
    {
      "active": "2017-08-09T15:50:35.90304258Z",
      "content": "B6OJ/WWA/bTsCbGYzaZwq3KdgDsC1485h4BKdnIBGOE=",
      "expires": "2117-07-16T15:50:35.90304258Z"
    }
  ]
}
```

<h1 id="revoke-extension-secrets">Revoke Extension Secrets</h1>

명시된 Extension과 관련된 모든 비밀키를 삭제합니다. <br><br> 새로운 [Create Extension Secret](#create-extension-secret) 이 실행되고 클라이언트가 수동으로 새로고침 할때까지 모든 클라이언트를 즉시 중단합니다. 비밀키가 유출되고 즉시 제거해야 할때만 이 함수를 사용하세요.

### Authentication
EBS에서 생성한 서명된 JWT. Extension 가이드의 [Signed the JWT](https://dev.twitch.tv/docs/extensions/#signing-the-jwt) 에서 설명하는 필요사항을 따르세요.

### URL
```HTML
DELETE https://api.twitch.tv/extensions/<extension ID>/auth/secret
```

### Optional Query String Parameters
없음

### Example Request
Extension 'pxifeyz7vxk9v6yb202nq4cwsnsp1t' 와 관련된 모든 비밀키를 삭제합니다.
```d
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDIyOTM3MzUsInVzZXJfaWQiOiIyNzQxOTAxMSIsInJvbGUiOiJleHRlcm5hbCJ9.s_uedGCjGzlD5CpTQ_IpqUiQuB12dGsUIdbarLn3LrU' \
-H 'Client-Id: pxifeyz7vxk9v6yb202nq4cwsnsp1t' \
-X DELETE https://api.twitch.tv/extensions/pxifeyz7vxk9v6yb202nq4cwsnsp1t/auth/secret
```

### Example Response
```d
204 Success
```

<h1 id="get-live-channels-with-extension-activated">Get Live Channels with Extension Activated</h1>

설치되고 활성화 중인 명시된 Extension의 Live 채널을 반환합니다. Live를 시작한지 얼마 안지난 채널은 리스트에 나타나는데 몇 분 정도 걸릴 수 있습니다. 막 스트리밍을 종료한 후의 채널은 몇 분 정도 동안 리스트에 나타날 수 있습니다. 

### Authentication
없음

### URL
```HTML
GET https://api.twitch.tv/extensions/<extension ID>/live_channels
```

### Optional Query String Parameters
없음

### Example Request
설치되고 활성화 중인 Extension 'pxifeyz7vxk9v6yb202nq4cwsnsp1t' 의 Live 채널을 요청합니다.
```d
curl -H "Client-Id: pxifeyz7vxk9v6yb202nq4cwsnsp1t" \
-X GET https://api.twitch.tv/extensions/pxifeyz7vxk9v6yb202nq4cwsnsp1t/live_channels
```

### Example Response
```JSON
{
  "channels": [
    {
      "game": "PLAYERUNKNOWN'S BATTLEGROUNDS",
      "id": "27419011",
      "title": "Still hunting for chicken!",
      "username": "TravistyOJ",
      "view_count": "943"
    }
  ]
}
```

<h1 id="set-extension-required-configuration">Set Extension Required Configuration</h1>

필수적인 스트리머 설정이 맞는지 확인한 후 명시된 Extension을 활성화 합니다. Extension 활성화 전 스트리머 설정이 필요한 경우에 사용합니다.

Extension manifest에 있는 `required_configuration` 으로 문자열을 사용해 필수 스트리머 설정을 강제할 수 있습니다. 문자열의 내용은 원하는 대로 작성 가능합니다. EBS가 채널에 대한 Extension 설정이 올바르게 됐다고 판단하면 Extension이 활성화 할 수 있게하는 설정 문자열을 이 endpoint를 사용해 제공하세요. Endpoint URL은 Extension iframe이 내장된 페이지의 채널 ID를 포함합니다.   

새 버전에서 다른 설정이 필요하다면 manifest에서 `required_configuration` 의  문자열을 변경하세요. 새 버전이 출시되면 스트리머는 재설정이 요구됩니다.

### Authentication
EBS에서 생성한 서명된 JWT. Extension 가이드의 [Signed the JWT](https://dev.twitch.tv/docs/extensions/#signing-the-jwt) 에서 설명하는 필요사항을 따르세요.

### URL
```HTML
PUT https://api.twitch.tv/extensions/<extension ID>/<extension version>/required_configuration?channel_id=<channel ID>
```

### Optional Query String Parameters
없음

### Example Request
Extension 'pxifeyz7vxk9v6yb202nq4cwsnsp1t' 을 활성화 합니다.
```d
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDIyOTg3ODksInVzZXJfaWQiOiIyNzQxOTAxMSIsInJvbGUiOiJleHRlcm5hbCJ9.sKjspl1jw0XGcpjF8vo2IcS2dQLQU_rSsmRCnrohCP4' \
-H 'Client-Id: pxifeyz7vxk9v6yb202nq4cwsnsp1t' \
-H 'Content-Type: application/json' \
-d '{"required_configuration": "RCS-1"}' \
-X PUT https://api.twitch.tv/extensions/pxifeyz7vxk9v6yb202nq4cwsnsp1t/0.0.1/required_configuration?channel_id=27419011
```

### Example Response
```d
204 Success
```

<h1 id="set-extension-broadcaster-oauth-receipt">Set Extension Broadcaster OAuth Receipt</h1>

Extension이 요청하는 권한을 스트리머가 수락했는지 여부에 대해 표시합니다. 필수 `permissions_received` 파라미터를 통해 알 수 있습니다. Endpoint URL은 Extension iframe이 내장된 페이지의 채널 ID를 포함합니다.

Extension이 스트리머를 대신해 트위치 API 사용이 필요하다면 manifest의 `required_broadcaster_abilities` 파트에 OAuth 범위(scope) 리스트를 작성할 수 있습니다. 

리스트가 존재하면 스트리머가 Extension을 활성화를 시도할때 스트리머는 보통의 트위치 OAuth 흐름(flow)상에 존재하게 됩니다. OAuth는 스트리머와 EBS간의 계약이기 때문에 Extension 시스템은 Extension이 요구하는 권한에 대한 스트리머의 결정을 알지 못합니다. 그러므로 스트리머가 요구 권한을 허락하거나 거부할때 스트리머가 결정하는 Extension API를 반드시 알려야합니다. 그리하면 트위치는 활성화 완료를 허용할 수 있습니다.

### Authentication
EBS에서 생성한 서명된 JWT. Extension 가이드의 [Signed the JWT](https://dev.twitch.tv/docs/extensions/#signing-the-jwt) 에서 설명하는 필요사항을 따르세요.

### URL
```HTML
PUT https://api.twitch.tv/extensions/<extension ID>/<extension version>/oauth_receipt?channel_id=<channel ID>
```

### Optional Query String Parameters
없음

### Example Request
Extension 'pxifeyz7vxk9v6yb202nq4cwsnsp1t'가 요구하는 권한을 스트리머가 허락했는지 표시합니다.
```d
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDIzMDQ0OTMsInVzZXJfaWQiOiIyNzQxOTAxMSIsInJvbGUiOiJleHRlcm5hbCJ9.byerAAo-UygWhzsVYciEAJlqdqDoJ1ihCBPTu11UwCU' \
-H 'Client-Id: pxifeyz7vxk9v6yb202nq4cwsnsp1t' \
-H 'Content-Type: application/json' \
-d '{"permissions_received": true}' \
-X PUT https://api.twitch.tv/extensions/pxifeyz7vxk9v6yb202nq4cwsnsp1t/0.0.1/oauth_receipt?channel_id=27419011
```

### Example Response
```d
204 Success
```

<h1 id="send-extension-pubsub-message">Send Extension PubSub Message</h1>

EBS(Extension Back-end Service)가 시청자, 스트리머와 통신할 수 있게 트위치는 PubSub(publish- subscribe) 시스템을 제공합니다. 이 endpoint 호출은 JavaScript Helper API에 있는 [send()](#send-functiontarget-contenttype-message) 함수와 같은 메커니즘을 사용하여 메시지를 전송합니다.

### Authentication
서명된 JWT (트위치 또는 EBS의 JWT)

**Note**: 서명된 JWT는 반드시 `channel_id` 그리고 `pubsub_perms` 필드를 포함해야 합니다. ([JWT Schema](#jwt-schema) 참조) 다음은 서명된 JWT의 페이로드 예제입니다.
```JSON
{
  "exp": 1503343947,
  "user_id": "27419011",
  "role": "external",
  "channel_id": "27419011",  
  "pubsub_perms": {
    "send":[
      "*"
    ]
  }
}
```

### URL
```HTML
POST https://api.twitch.tv/extensions/message/<channel ID>
```

### Optional Query String Parameters
없음

### Example Request
채널 27419011에 명시된 메시지를 보냅니다.
```d
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDMzNDM5NDcsInVzZXJfaWQiOiIyNzQxOTAxMSIsImNoYW5uZWxfaWQiOiIyNzQxOTAxMSIsInJvbGUiOiJleHRlcm5hbCIsInB1YnN1Yl9wZXJtcyI6eyJzZW5kIjpbIioiXX19.TiDAzrq58XczdymAozwsdVilRkjr9KN8C0pCv7px-FM" \
-H "Client-Id: pxifeyz7vxk9v6yb202nq4cwsnsp1t" \
-H "Content-Type: application/json" \
-d '{"content_type":"application/json", "message":"{\"foo\":\"bar\"}", "targets":["broadcast"]}' \
-X POST https://api.twitch.tv/extensions/message/27419011
```

### Example Response
```d
204	Success
```

<h1 id="javascript-helper">JavaScript Helper</h1>

Extension JavaScript Helper는 `window.Twitch.ext` 인터페이스를 추가합니다. 이것은 다음의 문자열을 사용합니다.

   * `version` — 1.1.1 ([semantic versioning](http://semver.org/)) 형식의 Helper 버전을 인코딩합니다.
   * `environment` — environment를 인코딩합니다. 외부 사용자를 위해 이것은 항상 `production` 입니다.

JavaScript Helper는 아래의 함수를 포함합니다.
   * [onAuthorized: function(authCallback)](#onauthorized-functionauthcallback)
   * [onError: function(errorCallback)](#onerror-functionerrorcallback)
   * [onContext: function(contextCallback)](#oncontext-functioncontextcallback)
   * [send: function(target, contentType, message)](#send-functiontarget-contenttype-message)
   * [listen: function(target, callback)](#listen-functiontarget-callback)
   * [unlisten: function(target, callback)](#unlisten-functiontarget-callback)

<h3 id="onauthorized-functionauthcallback">onAuthorized: function(authCallback)</h3>

JWT가 갱신될 때마다 콜백이 실행됩니다.

`authCallback` 은 아래의 속성들을 포함하는 객체를 인자로 하는 함수입니다.

Property    |Type|  Description
--------    |----|  -----------
`channelId`|string| Extension iframe을 내장하는 페이지의 채널 ID
`clientId` |string| Extension의 클라이언트 ID
`token`    | jwt  | 인증을 위한 EBS 호출에 전달돼야 하는 JWT
`userId`   |string| Opaque 사용자(user) ID

예를들면
```Javascript
window.Twitch.ext.onAuthorized(function(auth) {
  console.log('The JWT that will be passed to the EBS is', auth.token);
  console.log('The channel ID is', auth.channelId);
});
```

보통 이 함수로 얻은 JWT는 AJAX 상황에서 헤더로 EBS에 전달됩니다. 예를들면
```Javascript
window.Twitch.ext.onAuthorized(function(auth) {
  $.ajax({
    url: '/<some backend path>',
    type: 'GET',
    headers: {
      'x-extension-jwt': auth.token,
    }
  });
});
```

<h3 id="onerror-functionerrorcallback">onError: function(errorCallback)</h3>

Extension Helper에서 에러가 발생하면 콜백이 실행됩니다.

`errorCallback` 은 helper 내에서 발생한 에러 값을 인자로 하는 함수입니다.

<h3 id="oncontext-functioncontextcallback">onContext: function(contextCallback)</h3>

Extension의 context가 실행될때 콜백이 실행됩니다.

`contextCallback` 은 `context` 객체와 변경된 `context` 속성을 갖는 문자열 배열을 인자로 하는 함수입니다. `context` 객체는 아래의 속성을 포함합니다.

Property     |Type|  Description
--------     |----|  -----------
`mode`      |string| 유효한 값: <br> * `viewer` — 트위치 채널 페이지 같은 viewer context에 helper가 로드 됐습니다. <br> * `dashboard` — 라이브 대시보드 같은 스트리머 제어 context에 helper가 로드 됐습니다. 스트리머가 방송 중 Extension의 행동 업데이트를 제어 할 수 있음을 표시하도록 이 모드를 사용하세요. <br> * `config` — 초기 설정을 포함한 Extension 설정 대시보드에 helper가 로드 됐습니다. Extension을 실행하기 위한 모든 설정은 이 모드에 있어야 합니다.
`bitrate`    |number| 방송 비트레이트
`bufferSize` |number| 방송 버퍼 크기
`displayResolution` |string| 플레이어의 화면 크기
`game` |string| 방송 중인 게임
`hlsLatencyBroadcaster` |number| 스트리머와 시청자간에 지연시간(초)
`isFullScreen` |boolean| `true` 인 경우 시청자는 전체화면 모드로 시청 중입니다.
`isPaused` |boolean| `true` 인 경우 시청자는 방송을 일시정지했습니다.
`isTheatreMode` |boolean| `true` 인 경우 시청자는 극장 모드로 시청 중입니다.
`language` |string| 방송시 사용 언어 (ex : `"en"`)
`videoResolution` |string| 방송 화질

<h3 id="send-functiontarget-contenttype-message">send: function(target, contentType, message)</h3>