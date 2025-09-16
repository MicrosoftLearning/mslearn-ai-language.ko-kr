---
lab:
  title: 음성 번역
  description: 언어별 음성을 번역하고 사용자 고유의 앱에서 구현합니다.
---

# 음성 번역

Azure AI 음성에는 음성 언어를 번역하는 데 사용할 수 있는 음성 번역 API가 포함되어 있습니다. 예를 들어 현지어를 모르는 곳을 여행할 때 사용할 수 있는 번역기 애플리케이션을 개발하려는 등의 경우에 이 API를 사용할 수 있습니다. 그들은 “역은 어디에 있습니까?” 와 같은 문구를 말할 수 있을 것입니다. 또는 자신의 언어로 “약국을 찾아야”하고 현지 언어로 번역해야 합니다. 이 연습에서는 Python용 Azure AI 음성 SDK를 사용하여 이 예제를 기준으로 하는 간단한 애플리케이션을 만듭니다.

이 연습은 Python을 기준으로 하지만 여러 언어별 SDK를 사용하여 음성 번역 애플리케이션을 개발할 수 있습니다. 포함된 내용은 다음과 같습니다.

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

## Cloud Shell 앱 개발 준비

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

1. 리포지토리가 복제된 후 코드 파일이 들어 있는 폴더로 이동합니다.

    ```
   cd mslearn-ai-language/Labfiles/08-speech-translation/Python/translator
    ```

1. 명령줄 창에서 다음 명령을 실행하여 **translator** 폴더의 코드 파일을 봅니다.

    ```
   ls -a -l
    ```

    파일에는 구성 파일(**.env**) 및 코드 파일(**translator.py**)이 포함됩니다.

1. Python 가상 환경을 만들고 다음 명령을 실행하여 Azure AI 음성 SDK 패키지 및 기타 필수 패키지를 설치합니다.

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

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
   code translator.py
    ```

1. 코드 파일 상단의 기존 네임스페이스 참조 아래에서 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 음성 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. **main** 함수의 **Get config settings** 주석 아래에서 코드는 구성 파일에 정의한 키와 영역을 로드합니다.

1. **Configure translation** 주석 아래에서 다음 코드를 찾고, 다음 코드를 추가하여 Azure AI 서비스 음성 엔드포인트에 대한 연결을 구성합니다.

    ```python
   # Configure translation
   translation_config = speech_sdk.translation.SpeechTranslationConfig(speech_key, speech_region)
   translation_config.speech_recognition_language = 'en-US'
   translation_config.add_target_language('fr')
   translation_config.add_target_language('es')
   translation_config.add_target_language('hi')
   print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. **SpeechTranslationConfig**를 사용하여 음성을 텍스트로 번역하며, **SpeechConfig**도 사용하여 번역을 음성으로 합성합니다. **음성 구성** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Configure speech
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. 변경 내용(*Ctrl+S*)을 저장하지만 코드 편집기를 열어 둡니다.

## 앱 실행

지금까지 앱은 Azure AI 음성 리소스에 연결하는 것 외에는 아무 작업도 수행하지 않지만, 음성 기능을 추가하기 전에 실행하여 작동하는지 확인하는 것이 유용합니다.

1. 명령줄 창에서 다음 명령을 입력하여 번역기 앱을 실행합니다.

    ```
   python translator.py
    ```

    코드는 애플리케이션에서 사용할 음성 서비스 리소스의 지역, en-US에서 번역할 준비가 되었다는 메시지를 표시하고 대상 언어를 묻는 메시지를 표시해야 합니다. 실행이 성공하면 앱이 Azure AI 음성 서비스에 연결되었음을 나타냅니다. ENTER 키를 눌러 프로그램을 종료합니다.

## 음성 번역 구현

이제 Azure AI 음성 서비스에 대한 **SpeechTranslationConfig**가 있으므로 Azure AI 음성 번역 API를 사용하여 음성을 인식하고 번역할 수 있습니다.

1. 코드 파일에서 코드는 **Translate** 함수를 사용하여 음성 입력을 번역합니다. 그런 다음에 **Translate** 함수의 **음성 번역** 주석 아래에 다음 코드를 추가하여 **TranslationRecognizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 파일로부터 음성을 인식하고 번역할 수 있습니다.

    ```python
   # Translate speech
   current_dir = os.getcwd()
   audioFile = current_dir + '/station.wav'
   audio_config_in = speech_sdk.AudioConfig(filename=audioFile)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Getting speech from file...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

1. 변경 내용을 저장하고(*Ctrl+S*) 프로그램을 다시 실행합니다.

    ```
   python translator.py
    ```

1. 메시지가 표시되면 유효한 언어 코드(*fr*, *es* 또는 *hi*)를 입력합니다. 그러면 프로그램이 입력 파일을 필사한 다음, 지정한 언어(프랑스어, 스페인어 또는 힌디어)로 번역합니다. 이 프로세스를 반복하여 음성을 애플리케이션이 지원하는 각 언어로 번역해 봅니다.

    > **참고**: 언어 인코딩 문제로 인해 힌디어 번역이 콘솔 창에 올바르게 표시되지 않는 경우도 있습니다.

1. 프로세스를 완료한 후 Enter 키를 눌러 프로그램을 종료합니다.

> **참고**: 애플리케이션의 코드를 한 번만 호출하면 입력이 3개 언어로 번역됩니다. 출력에는 특정 언어의 번역만 표시되지만 결과의 **translations** 컬렉션에서 대상 언어 코드를 지정하면 원하는 번역을 검색할 수 있습니다.

## 번역을 음성으로 합성

지금까지 애플리케이션은 음성 입력을 텍스트로 번역했습니다. 여행 중에 도움을 요청할 때는 텍스트 번역으로도 충분할 수 있습니다. 그러나 애플리케이션이 적절한 음성으로 번역을 '말해' 준다면 더 효율적일 것입니다.

> **참고**: Cloud Shell의 하드웨어 제한으로 인해 합성된 음성 출력을 파일로 전달할 것입니다.

1. **Translate** 함수에서 **Synthesize translation** 주석을 찾고 다음 코드를 추가하여 **SpeechSynthesizer** 클라이언트를 통해 번역 내용을 음성으로 합성하고 .wav 파일로 저장합니다.

    ```python
   # Synthesize translation
   output_file = "output.wav"
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. 변경 내용을 저장하고(*Ctrl+S*) 프로그램을 다시 실행합니다.

    ```
   python translator.py
    ```

1. 애플리케이션의 출력을 검토하여 음성 출력 번역이 파일에 저장되었음을 표시해야 합니다. 프로세스를 완료한 후 **Enter** 키를 눌러 프로그램을 종료합니다.
1. .wav 오디오 파일을 재생할 수 있는 미디어 플레이어가 있는 경우 다음 명령을 입력하여 생성된 파일을 다운로드합니다.

    ```
   download ./output.wav
    ```

    다운로드 명령은 브라우저의 오른쪽 아래에 팝업 링크를 만듭니다. 이 링크를 선택하여 파일을 다운로드하고 열 수 있습니다.

> **참고**
> *이 예제에서는 **SpeechTranslationConfig**를 사용하여 음성을 텍스트로 번역한 다음 **SpeechConfig**를 사용하여 번역을 음성으로 합성했습니다. 실제로는 **SpeechTranslationConfig**를 사용해 번역을 직접 합성할 수 있습니다. 하지만 이 방식은 한 가지 언어로 번역할 때만 사용 가능하며, 일반적으로 파일로 저장되는 오디오 스트림이 생성됩니다.*

## 리소스 정리

Azure AI 음성 서비스 탐색을 마쳤으면 이 연습에서 만든 리소스를 삭제할 수 있습니다. 방법은 다음과 같습니다.

1. Azure Cloud Shell 창 닫기
1. Azure Portal에서 이 랩에서 만든 Azure AI 음성 리소스를 찾습니다.
1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 마이크와 스피커가 있는 경우 어떻게 해야 할까요?

이 연습에서는 음성 입력 및 출력에 오디오 파일을 사용했습니다. 오디오 하드웨어를 사용하도록 코드를 수정하는 방법을 살펴보겠습니다.

### 마이크와 함께 음성 번역 사용

1. 마이크가 있는 경우 다음 코드를 사용하여 음성 번역을 위한 음성 입력을 캡처할 수 있습니다.

    ```python
   # Translate speech
   audio_config_in = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Speak now...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

> **참고**: 시스템 기본 마이크가 기본 오디오 입력이므로 AudioConfig를 아예 생략할 수도 있습니다!

### 스피커로 음성 합성 사용

1. 스피커가 있는 경우 다음 코드를 사용하여 음성을 합성할 수 있습니다.
    
    ```python
   # Synthesize translation
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(use_default_speaker=True)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

> **참고**: 시스템 기본 스피커가 기본 오디오 출력이므로 AudioConfig를 아예 생략할 수도 있습니다!

## 자세한 정보

Azure AI 음성 번역 API 사용에 대한 자세한 내용은 [음성 번역 설명서](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation)를 참조하세요.
