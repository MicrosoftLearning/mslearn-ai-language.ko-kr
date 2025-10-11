---
lab:
  title: 음성 인식 및 합성
  description: 음성을 텍스트로 변환하고 텍스트를 음성으로 변환하는 음성 시계를 구현합니다.
---

# 음성 인식 및 합성

**Azure AI 음성**은 다음을 포함한 음성 관련 기능을 제공하는 서비스입니다.

- *음성 텍스트 변환* API - 음성 인식(가청 음성 단어를 텍스트로 변환)을 구현할 수 있습니다.
- *텍스트 음성 변환* API - 음성 합성(텍스트를 가청 음성으로 변환)을 구현할 수 있습니다.

이 연습에서는 두 API를 모두 사용하여 음성으로 시간을 안내하는 시계 애플리케이션을 구현합니다.

이 연습은 Python을 기반으로 하지만 여러 언어별 SDK를 사용하여 음성 애플리케이션을 개발할 수 있습니다. 포함된 내용은 다음과 같습니다.

- [Python용 Azure AI 음성 SDK](https://pypi.org/project/azure-cognitiveservices-speech/)
- [.NET용 Azure AI 음성 SDK](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [JavaScript용 Azure AI 음성 SDK](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

이 연습에는 약 **30**분이 소요됩니다.

> **참고** 이 연습은 컴퓨터의 사운드 하드웨어에 대한 직접 액세스가 지원되지 않는 Azure Cloud Shell에서 완료하도록 설계되었습니다. 따라서 랩은 음성 입력 및 출력 스트림에 오디오 파일을 사용합니다. 마이크와 스피커를 사용하여 동일한 결과를 얻기 위한 코드가 참조용으로 제공됩니다.

## Azure AI 음성 리소스 만들기

먼저 Azure AI 음성 리소스를 만들어 보겠습니다.

1. `https://portal.azure.com`의 [Azure Portal](https://portal.azure.com)을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. 위쪽 검색 필드에서 **Speech Service**를 검색합니다. 목록에서 선택한 다음, **만들기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 프로비전합니다.
    - **구독**: *자신의 Azure 구독*.
    - **리소스 그룹**: *리소스 그룹을 선택하거나 만듭니다*.
    - **지역**: *사용 가능한 지역을 선택합니다*.
    - **이름**: 고유한 이름을 입력합니다.
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **리소스 관리** 섹션의 **키 및 엔드포인트**를 봅니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## 음성 시계 앱 준비 및 구성

1. **키 및 엔드포인트** 페이지를 열어두고 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **팁**: CloudShell에 명령을 입력하면 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 음성 시계 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    ```
   cd mslearn-ai-language/Labfiles/07-speech/Python/speaking-clock
    ```

1. 명령줄 창에서 다음 명령을 실행하여 **speaking-clock** 폴더의 코드 파일을 봅니다.

    ```
   ls -a -l
    ```

    이러한 파일에는 구성 파일(**.env**) 및 코드 파일(**speaking-clock.py**)이 포함됩니다. 애플리케이션에서 사용할 오디오 파일은 **audio** 하위 폴더에 있습니다.

1. Python 가상 환경을 만들고 다음 명령을 실행하여 Azure AI 음성 SDK 패키지 및 기타 필수 패키지를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. 다음 명령을 입력하여 구성 파일을 편집합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 만든 Azure AI 음성 리소스의 **지역** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal에서 Azure AI 번역기 리소스의 **키 및 엔드포인트** 페이지에서 사용 가능).
1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

## Azure AI 음성 SDK를 사용하기 위한 코드 추가

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code speaking-clock.py
    ```

1. 코드 파일 상단의 기존 네임스페이스 참조 아래에서 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 음성 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. **main** 함수의 **Get config settings** 주석 아래에서 코드는 구성 파일에 정의한 키와 영역을 로드합니다.

1. **Configure speech service** 주석을 찾고 다음 코드를 추가하여 Azure AI 서비스 키와 사용자 지역을 통해 Azure AI 서비스 음성 엔드포인트에 대한 연결을 구성합니다.

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. 변경 내용(*Ctrl+S*)을 저장하지만 코드 편집기를 열어 둡니다.

## 앱 실행

지금까지 앱은 Azure AI 음성 서비스에 연결하는 것 외에는 아무 작업도 수행하지 않지만, 음성 기능을 추가하기 전에 실행하여 작동하는지 확인하는 것이 유용합니다.

1. 명령줄에서 다음 명령을 입력하여 음성 시계 앱을 실행합니다.

    ```
   python speaking-clock.py
    ```

    이 코드는 애플리케이션이 사용할 Speech 서비스 리소스의 지역을 표시합니다. 실행이 성공하면 앱이 Azure AI 음성 리소스에 연결되었음을 나타냅니다.

## 음성을 인식하는 코드 추가

이제 프로젝트의 Azure AI 서비스 리소스에 음성 서비스에 대한 **SpeechConfig**가 있으므로 **음성 텍스트 변환** API를 사용하여 음성을 인식하고 텍스트로 변환할 수 있습니다.

이 절차에서 음성 입력은 오디오 파일에서 캡처되며 여기서 재생할 수 있습니다.

<video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Time.mp4" title="What time is it?" width="150"></video>

1. 코드 파일에서 이 코드는 **TranscribeCommand** 함수를 사용하여 음성 입력을 수락합니다. 그런 다음에 **TranscribeCommand** 함수의 **Configure speech recognition** 주석을 찾고 아래에 적절한 코드를 추가하여 **SpeechRecognizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 오디오 파일로부터 음성을 인식하고 전사할 수 있습니다.

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

1. **TranscribeCommand** 함수의 **음성 입력 처리** 주석 아래에 다음 코드를 추가하여 음성 입력을 청취합니다. 이때 명령을 반환하는 함수 끝부분의 코드를 바꾸지 않도록 주의하세요.

    ```python
   # Process speech input
   print("Listening...")
   speech = speech_recognizer.recognize_once_async().get()
   if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
   else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

1. 변경 내용을 저장하고(*Ctrl+S*) 코드 편집기 아래의 명령줄에서 프로그램을 다시 실행합니다.
1. 출력을 검토하여 오디오 파일의 음성을 성공적으로 "듣고" 적절한 응답을 반환해야 합니다(Azure Cloud Shell이 사용자와 다른 시간대에 있는 서버에서 실행 중일 수 있음에 유의합니다!).

    > **팁**: SpeechRecognizer에 오류가 발생하면 "취소됨"이라는 결과를 생성합니다. 그러면 애플리케이션의 코드가 오류 메시지를 표시합니다. 가장 가능성이 높은 원인은 구성 파일의 잘못된 지역 값입니다.

## 음성 합성

음성 시계 애플리케이션은 음성 입력을 수락하지만 실제로 시간을 직접 말로 알려 주지는 않습니다. 이제 음성을 합성하는 코드를 추가하여 응답 방식을 수정해 보겠습니다.

다시 한 번, Cloud Shell의 하드웨어 제한으로 인해 합성된 음성 출력을 파일로 전달할 것입니다.

1. 코드 파일에서 이 코드는 **TellTime** 함수를 사용하여 사용자에게 현재 시간을 알려줍니다.
1. **TellTime** 함수의 **음성 합성 구성** 주석 아래에 다음 코드를 추가하여 **SpeechSynthesizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 음성 출력을 생성할 수 있습니다.

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

1. **TellTime** 함수의 **음성 출력 합성** 주석 아래에 다음 코드를 추가하여 음성 출력을 생성합니다. 이때 응답을 인쇄하는 함수 끝부분의 코드를 바꾸지 않도록 주의하세요.

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. 변경 내용을 저장하고(*Ctrl+S*) 프로그램을 다시 실행합니다. 그러면 음성 출력이 파일에 저장되었다고 나타납니다.

1. .wav 오디오 파일을 재생할 수 있는 미디어 플레이어가 있는 경우 다음 명령을 입력하여 생성된 파일을 다운로드합니다.

    ```
   download ./output.wav
    ```

    다운로드 명령은 브라우저의 오른쪽 아래에 팝업 링크를 만듭니다. 이 링크를 선택하여 파일을 다운로드하고 열 수 있습니다.

    파일은 이와 비슷하게 들릴 것입니다.

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output.mp4" title="시간은 2시 15분입니다." width="150"></video>

## Speech Synthesis Markup Language 사용

SSML(Speech Synthesis Markup Language)을 사용하면 XML 기반 형식을 통해 음성 합성 방식을 사용자 지정할 수 있습니다.

1. **TellTime** 함수에서 **음성 출력 합성** 현재 주석 아래에 있는 모든 코드를 다음 코드로 바꿉니다(**응답 인쇄** 주석 아래의 코드는 그대로 유지).

    ```python
   # Synthesize spoken output
   responseSsml = " \
       <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
           <voice name='en-GB-LibbyNeural'> \
               {} \
               <break strength='weak'/> \
               Time to end this lab! \
           </voice> \
       </speak>".format(response_text)
   speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + output_file)
    ```

1. 변경 내용을 저장하고 프로그램을 다시 실행합니다. 그러면 음성 출력이 파일에 저장되었다고 다시 한 번 나타납니다.
1. 생성된 파일을 다운로드하여 재생합니다. 이 파일은 다음과 유사합니다.
    
    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output2.mp4" title="시간은 5:30입니다. 이 랩을 종료할 시간입니다." width="150"></video>

## 정리

Azure AI 음성 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Cloud Shell 창 닫기
1. Azure Portal에서 이 랩에서 만든 Azure AI 음성 리소스를 찾습니다.
1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 마이크와 스피커가 있는 경우 어떻게 해야 할까요?

이번 연습에 사용한 Azure Cloud Shell 환경은 오디오 하드웨어를 지원하지 않으므로 음성 명령 입력 및 출력에 오디오 파일을 사용했습니다. 사용할 수 있는 경우 오디오 하드웨어를 사용하도록 코드를 수정하는 방법을 살펴보겠습니다.

### 마이크를 사용한 음성 인식 사용

마이크가 있는 경우 다음 코드를 사용하여 음성 인식을 위한 음성 입력을 캡처할 수 있습니다.

```python
# Configure speech recognition
audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
print('Speak now...')

# Process speech input
speech = speech_recognizer.recognize_once_async().get()
if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
    command = speech.text
    print(command)
else:
    print(speech.reason)
    if speech.reason == speech_sdk.ResultReason.Canceled:
        cancellation = speech.cancellation_details
        print(cancellation.reason)
        print(cancellation.error_details)

```

> **참고**: 시스템 기본 마이크가 기본 오디오 입력이므로 AudioConfig를 아예 생략할 수도 있습니다!

### 스피커로 음성 합성 사용

스피커가 있는 경우 다음 코드를 사용하여 음성을 합성할 수 있습니다.

```python
response_text = 'The time is {}:{:02d}'.format(now.hour,now.minute)

# Configure speech synthesis
speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config)

# Synthesize spoken output
speak = speech_synthesizer.speak_text_async(response_text).get()
if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print(speak.reason)
```

> **참고**: 시스템 기본 스피커가 기본 오디오 출력이므로 AudioConfig를 아예 생략할 수도 있습니다!

## 자세한 정보

**음성 텍스트 변환** 및 **텍스트 음성 변환** API 사용에 대한 자세한 내용은 [음성 텍스트 변환 설명서](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) 및 [텍스트 음성 변환 설명서](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech)를 참조하세요.
