---
lab:
  title: Azure AI Voice Live 음성 에이전트 개발
  description: Azure AI Voice Live 에이전트와 실시간 음성 상호 작용을 사용하도록 설정하는 웹앱을 만드는 방법을 알아봅니다.
---

# Azure AI Voice Live 음성 에이전트 개발

이 연습에서는 에이전트와의 실시간 음성 상호 작용을 가능하게 하는 Flask 기반 Python 웹앱을 완료합니다. 세션을 초기화하고 세션 이벤트를 처리하는 코드를 추가합니다. AI 모델을 배포하고, ACR(Azure Container Registry)에서 ACR 작업을 사용하여 앱 이미지를 만든 다음, 이미지를 끌어오는 Azure App Service 인스턴스를 만드는 배포 스크립트를 사용합니다. 앱을 테스트하려면 마이크 및 스피커 기능이 있는 오디오 디바이스가 필요합니다.

이 연습은 Python을 기준으로 하지만 다음을 포함하여 다른 언어별 SDK와 유사한 애플리케이션을 개발할 수 있습니다.

- [.NET용 Azure VoiceLive 클라이언트 라이브러리](https://www.nuget.org/packages/Azure.AI.VoiceLive/)

이 연습에서 수행된 작업:

* 앱의 기본 파일 다운로드
* 웹앱을 완료하는 코드 추가
* 전체 코드 베이스 검토
* 배포 스크립트 업데이트 및 실행
* 애플리케이션 보기 및 테스트

이 연습을 완료하는 데 약 **30**분이 걸립니다.

## Azure Cloud Shell 시작 및 파일 다운로드

연습의 이 섹션에서는 앱의 기본 파일이 포함된 압축된 파일을 다운로드합니다.

1. 브라우저에서 Azure Portal[https://portal.azure.com](https://portal.azure.com)로 이동한 다음, 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***Bash*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *PowerShell* 환경을 사용하는 Cloud Shell을 만든 경우 ***Bash***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

1. **Bash** 셸에서 다음 명령을 실행하여 연습 파일을 다운로드하고 압축을 풉니다. 두 번째 명령은 연습 파일의 디렉터리로도 변경합니다.

    ```bash
    wget https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/downloads/python/voice-live-web.zip
    ```

    ```
    unzip voice-live-web.zip && cd voice-live-web
    ```

## 웹앱을 완료하는 코드 추가

이제 연습 파일이 다운로드되었으므로 다음 단계는 애플리케이션을 완료하는 코드를 추가하는 것입니다. Cloud Shell에서 다음 단계를 수행합니다. 

>**팁:** 위쪽 테두리를 끌어서 Cloud Shell의 크기를 조절하면 더 많은 정보와 코드가 표시됩니다. 최소화 및 최대화 단추를 사용하여 Cloud Shell과 기본 포털 인터페이스 사이를 전환할 수도 있습니다.

연습을 계속하기 전에 다음 명령을 실행하여 *src* 디렉터리로 변경합니다.

```bash
cd src
```

### 음성 라이브 도우미를 구현하는 코드 추가

이 섹션에서는 음성 라이브 도우미를 구현하는 코드를 추가합니다. **\_\_init\_\_** 메서드는 Azure VoiceLive 연결 매개 변수(엔드포인트, 자격 증명, 모델, 음성 및 시스템 지침)를 저장하고 런타임 상태 변수를 설정하여 연결 수명 주기를 관리하고 대화 중에 사용자 중단을 처리함으로써 음성 도우미를 초기화합니다. **start** 메서드는 WebSocket 연결을 설정하고 실시간 음성 세션을 구성하는 데 사용할 필요한 Azure VoiceLive SDK 구성 요소를 가져옵니다.

1. 다음 명령을 실행하여 편집할 *flask_app.py* 파일을 엽니다.

    ```bash
    code flask_app.py
    ```

1. 코드에서 **# BEGIN VOICE LIVE ASSISTANT IMPLEMENTATION - ALIGN CODE WITH COMMENT** 주석을 검색합니다. 아래 코드를 복사하고 주석 바로 아래에 입력합니다. 들여쓰기를 확인해야 합니다.

    ```python
    def __init__(
        self,
        endpoint: str,
        credential,
        model: str,
        voice: str,
        instructions: str,
        state_callback=None,
    ):
        # Store Azure Voice Live connection and configuration parameters
        self.endpoint = endpoint
        self.credential = credential
        self.model = model
        self.voice = voice
        self.instructions = instructions
        
        # Initialize runtime state - connection established in start()
        self.connection = None
        self._response_cancelled = False  # Used to handle user interruptions
        self._stopping = False  # Signals graceful shutdown
        self.state_callback = state_callback or (lambda *_: None)

    async def start(self):
        # Import Voice Live SDK components needed for establishing connection and configuring session
        from azure.ai.voicelive.aio import connect  # type: ignore
        from azure.ai.voicelive.models import (
            RequestSession,
            ServerVad,
            AzureStandardVoice,
            Modality,
            InputAudioFormat,
            OutputAudioFormat,
        )  # type: ignore
    ```

1. **ctrl+s**를 입력하여 변경 내용을 저장하고, 다음 섹션을 위해 편집기는 열어 둡니다.

### 음성 라이브 도우미를 구현하는 코드 추가

이 섹션에서는 음성 라이브 세션을 구성하는 코드를 추가합니다. 이렇게 하면 형식(API에서 오디오 전용은 지원되지 않음), 도우미의 동작을 정의하는 시스템 지침, 응답에 대한 Azure TTS 음성, 입력 및 출력 스트림 모두에 대한 오디오 형식, 사용자가 말하기를 시작하고 중지할 때 모델이 감지하는 방법을 지정하는 VAD(서버 쪽 음성 활동 검색)가 지정됩니다.

1. 코드에서 **# BEGIN CONFIGURE VOICE LIVE SESSION - ALIGN CODE WITH COMMENT** 주석을 검색합니다. 아래 코드를 복사하고 주석 바로 아래에 입력합니다. 들여쓰기를 확인해야 합니다.

    ```python
    # Configure VoiceLive session with audio/text modalities and voice activity detection
    session_config = RequestSession(
        modalities=[Modality.TEXT, Modality.AUDIO],
        instructions=self.instructions,
        voice=voice_cfg,
        input_audio_format=InputAudioFormat.PCM16,
        output_audio_format=OutputAudioFormat.PCM16,
        turn_detection=ServerVad(threshold=0.5, prefix_padding_ms=300, silence_duration_ms=500),
    )
    await conn.session.update(session=session_config)
    ```

1. **ctrl+s**를 입력하여 변경 내용을 저장하고, 다음 섹션을 위해 편집기는 열어 둡니다.

### 세션 이벤트를 처리하는 코드 추가

이 섹션에서는 음성 라이브 세션에 대한 이벤트 처리기를 추가하는 코드를 추가합니다. 이벤트 처리기는 세션이 대화 수명 주기 동안 주요 VoiceLive 세션 이벤트에 응답합니다. 즉, **_handle_session_updated**는 세션이 사용자 입력을 받을 준비가 되면 신호로 알리고, **_handle_speech_started**는 사용자가 말하기를 시작할 때를 감지한 후 진행 중인 도우미 오디오 재생을 중지하고 진행 중인 응답을 취소하여 중단 논리를 구현함으로써 자연스러운 대화 흐름을 허용하고, **_handle_speech_stopped**는 사용자가 말하기를 완료하고 도우미가 입력 처리를 시작할 때 작업을 처리합니다.

1. 코드에서 **# BEGIN HANDLE SESSION EVENTS - ALIGN CODE WITH COMMENT** 주석을 검색합니다. 아래 코드를 복사하고 주석 바로 아래에 입력합니다. 들여쓰기를 확인해야 합니다.

    ```python
    async def _handle_event(self, event, conn, verbose=False):
        """Handle Voice Live events with clear separation by event type."""
        # Import event types for processing different Voice Live server events
        from azure.ai.voicelive.models import ServerEventType
        
        event_type = event.type
        if verbose:
            _broadcast({"type": "log", "level": "debug", "event_type": str(event_type)})
        
        # Route Voice Live server events to appropriate handlers
        if event_type == ServerEventType.SESSION_UPDATED:
            await self._handle_session_updated()
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED:
            await self._handle_speech_started(conn)
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED:
            await self._handle_speech_stopped()
        elif event_type == ServerEventType.RESPONSE_AUDIO_DELTA:
            await self._handle_audio_delta(event)
        elif event_type == ServerEventType.RESPONSE_AUDIO_DONE:
            await self._handle_audio_done()
        elif event_type == ServerEventType.RESPONSE_DONE:
            # Reset cancellation flag but don't change state - _handle_audio_done already did
            self._response_cancelled = False
        elif event_type == ServerEventType.ERROR:
            await self._handle_error(event)

    async def _handle_session_updated(self):
        """Session is ready for conversation."""
        self.state_callback("ready", "Session ready. You can start speaking now.")

    async def _handle_speech_started(self, conn):
        """User started speaking - handle interruption if needed."""
        self.state_callback("listening", "Listening… speak now")
        
        try:
            # Stop any ongoing audio playback on the client side
            _broadcast({"type": "control", "action": "stop_playback"})
            
            # If assistant is currently speaking or processing, cancel the response to allow interruption
            current_state = assistant_state.get("state")
            if current_state in {"assistant_speaking", "processing"}:
                self._response_cancelled = True
                await conn.response.cancel()
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"Interrupted assistant during {current_state}"})
            else:
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"User speaking during {current_state} - no cancellation needed"})
        except Exception as e:
            _broadcast({"type": "log", "level": "debug", 
                      "msg": f"Exception in speech handler: {e}"})

    async def _handle_speech_stopped(self):
        """User stopped speaking - processing input."""
        self.state_callback("processing", "Processing your input…")

    async def _handle_audio_delta(self, event):
        """Stream assistant audio to clients."""
        if self._response_cancelled:
            return  # Skip cancelled responses
            
        # Update state when assistant starts speaking
        if assistant_state.get("state") != "assistant_speaking":
            self.state_callback("assistant_speaking", "Assistant speaking…")
        
        # Extract and broadcast Voice Live audio delta as base64 to WebSocket clients
        audio_data = getattr(event, "delta", None)
        if audio_data:
            audio_b64 = base64.b64encode(audio_data).decode("utf-8")
            _broadcast({"type": "audio", "audio": audio_b64})

    async def _handle_audio_done(self):
        """Assistant finished speaking."""
        self._response_cancelled = False
        self.state_callback("ready", "Assistant finished. You can speak again.")

    async def _handle_error(self, event):
        """Handle Voice Live errors."""
        error = getattr(event, "error", None)
        message = getattr(error, "message", "Unknown error") if error else "Unknown error"
        self.state_callback("error", f"Error: {message}")

    def request_stop(self):
        self._stopping = True
    ```

1. **ctrl+s**를 입력하여 변경 내용을 저장하고, 다음 섹션을 위해 편집기는 열어 둡니다.

### 앱에서 코드 검토

지금까지 에이전트를 구현하고 에이전트 이벤트를 처리하는 코드를 앱에 추가했습니다. 앱이 클라이언트 상태 및 작업을 처리하는 방법을 더 잘 이해하기 위해 약간의 시간을 할애해서 전체 코드 및 주석을 검토하세요.

1. 완료되면 **ctrl+q**를 입력하여 편집기에서 종료합니다. 

## 배포 스크립트 업데이트 및 실행

이 섹션에서는 **azdeploy.sh** 배포 스크립트를 약간 변경한 다음, 배포를 실행합니다. 

### 배포 스크립트 업데이트

**azdeploy.sh** 배포 스크립트의 맨 위에서 변경해야 하는 값은 두 개뿐입니다. 

* **rg** 값은 배포를 포함할 리소스 그룹을 지정합니다. 기본값을 적용하거나 특정 리소스 그룹에 배포해야 하는 경우 고유한 값을 입력할 수 있습니다.

* **위치** 값은 배포의 지역을 설정합니다. 연습에 사용되는 *gpt-4o* 모델은 다른 지역에 배포할 수 있지만 특정 지역에는 제한이 있을 수 있습니다. 선택한 지역에서 배포가 실패하면 **eastus2** 또는 **swedencentral**을 사용해 보세요. 

    ```
    rg="rg-voicelive" # Replace with your resource group
    location="eastus2" # Or a location near you
    ```

1. Cloud Shell에서 다음 명령을 실행하여 배포 스크립트 편집을 시작합니다.

    ```bash
    cd ~/voice-live-web
    ```
    
    ```bash
    code azdeploy.sh
    ```

1. 요구 사항에 맞게 **rg** 및 **location** 값을 업데이트한 다음, **ctrl+s**를 입력하여 변경 내용을 저장하고 **Ctrl+q**를 입력하여 편집기를 종료합니다.

### 배포 스크립트 실행

배포 스크립트는 AI 모델을 배포하고 App Service에서 컨테이너화된 앱을 실행하는 데 필요한 리소스를 Azure에 만듭니다.

1. Cloud Shell에서 다음 명령을 실행하여 Azure 리소스 및 애플리케이션 배포를 시작합니다.

    ```bash
    bash azdeploy.sh
    ```

1. 초기 배포를 위한 **옵션 1**을 선택합니다.

    배포는 5-10분 후에 완료됩니다. 배포하는 동안 다음 정보/작업을 묻는 메시지가 표시될 수 있습니다.
    
    * Azure에 인증하라는 메시지가 표시되면 제시된 지침을 따릅니다.
    * 구독을 선택하라는 메시지가 표시되면 화살표 키를 사용하여 구독을 강조 표시하고 **Enter** 키를 누릅니다. 
    * 배포하는 동안 몇 가지 경고가 표시될 수 있으며 이러한 경고는 무시될 수 있습니다.
    * AI 모델 배포 중에 배포가 실패하면 배포 스크립트에서 지역을 변경하고 다시 시도합니다. 
    * Azure의 지역은 때때로 사용량이 많아지고 배포 타이밍을 방해할 수 있습니다. 모델 배포 후 배포가 실패하면 배포 스크립트를 다시 실행합니다.

## 앱 보기 및 테스트

배포가 완료되면 "Deployment complete!" 메시지가 웹앱에 대한 링크와 함께 셸에 표시됩니다. 해당 링크를 선택하거나 App Service 리소스로 이동하여 해당 위치에서 앱을 시작할 수 있습니다. 애플리케이션이 로드되는 데 몇 분 정도 걸릴 수 있습니다. 

1. **세션 시작** 단추를 선택하여 모델에 연결합니다.
1. 애플리케이션에 오디오 디바이스에 대한 액세스 권한을 부여하라는 메시지가 표시됩니다.
1. 앱에서 말하기를 시작하라는 메시지가 표시되면 모델과 소통을 시작합니다.

문제 해결:

* 앱에서 누락된 환경 변수를 보고하는 경우 App Service에서 애플리케이션을 다시 시작합니다.
* 애플리케이션에 표시된 로그에 과도한 *오디오 청크* 메시지가 표시되면 **세션 중지**를 선택했다가 세션을 다시 시작합니다. 
* 앱이 전혀 작동하지 않는 경우 모든 코드를 추가하고 적절한 들여쓰기를 추가했는지 다시 확인합니다. 변경이 필요한 경우 배포를 다시 실행하고 **옵션 2**를 선택하여 이미지만 업데이트합니다.

## 리소스 정리

Cloud Shell에서 다음 명령을 실행하여 이 연습에 배포된 모든 리소스를 제거합니다. 리소스 삭제를 확인하라는 메시지가 표시됩니다.

```
azd down --purge
```