---
lab:
  title: 음성 인식 및 합성
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# 음성 인식 및 합성

**Azure AI 음성**은 다음을 포함한 음성 관련 기능을 제공하는 서비스입니다.

- *음성 텍스트 변환* API - 음성 인식(가청 음성 단어를 텍스트로 변환)을 구현할 수 있습니다.
- *텍스트 음성 변환* API - 음성 합성(텍스트를 가청 음성으로 변환)을 구현할 수 있습니다.

이 연습에서는 두 API를 모두 사용하여 음성으로 시간을 안내하는 시계 애플리케이션을 구현합니다.

> **참고** 이 연습을 수행하려면 스피커/헤드폰이 있는 컴퓨터를 사용해야 합니다. 최상의 경험을 위해서는 마이크도 필요합니다. 일부 호스트 가상 환경에서는 로컬 마이크로부터 오디오를 캡처할 수 있지만, 이 기능이 작동하지 않거나 마이크가 아예 없는 경우에는 제공된 오디오 파일을 음성 입력으로 사용할 수 있습니다. 마이크를 사용하는지 아니면 오디오 파일을 사용하는지에 따라 서로 다른 옵션을 선택해야 하므로 주의하여 지침을 따르세요.

## *Azure AI 음성* 리소스 프로비전

구독에 아직 리소스가 없으면 **Azure AI 음성** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. 상단 검색 창에서 **Azure AI 서비스**를 검색하고 **Enter** 키를 누른 다음 결과의 **음성 서비스**에서 **만들기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: *사용 가능한 지역 선택*
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **담당 AI 알림**: 동의합니다.
1. **검토 + 만들기**를 선택합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## Visual Studio Code에서 앱 개발 준비

Visual Studio Code를 사용하여 음성 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-ai-language** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
1. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-language` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
1. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
1. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure AI 음성 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/07-speech** 폴더를 찾아 **CSharp** 또는 **Python** 폴더를 확장합니다. 언어 선택과 여기에 포함된 **speaking-clock** 폴더에 따라 다릅니다. 각 폴더에는 Azure AI 음성 기능을 통합할 앱에 대한 언어별 코드 파일이 포함되어 있습니다.
1. 코드 파일이 포함된 **speaking-clock** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 음성 SDK 패키지를 설치합니다.

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. **탐색기** 창의 **speaking-clock** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env

1. 만든 Azure AI 음성 리소스의 **지역** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal에서 Azure AI 음성 리소스의 **키 및 엔드포인트** 페이지에서 사용 가능).

    > **참고**: 엔드포인트가 <u>아님</u> 리소스에 대한 *지역*을 추가해야 합니다!

1. 구성 파일을 저장합니다.

## Azure AI 음성 SDK를 사용하기 위한 코드 추가

1. **speaking-clock** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: speaking-clock.py

    코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Azure AI 음성 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**: speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. **Main** 함수에는 구성 파일에서 서비스 키와 지역을 로드하는 코드가 이미 제공되어 있습니다. Azure AI 음성 리소스에 대한 **SpeechConfig**를 만들려면 이러한 변수를 사용해야 합니다. **Speech 서비스 구성** 주석 아래에 다음 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. 변경 내용을 저장하고 **speaking-clock** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. C#을 사용 중인 경우 비동기 메서드의 **await** 연산자 사용 관련 경고는 무시해도 됩니다. 뒷부분에서 해당 부분을 수정할 것입니다. 이 코드는 애플리케이션이 사용할 Speech 서비스 리소스의 지역을 표시합니다.

## 음성을 인식하는 코드 추가

이제 Azure AI 음성 리소스에 음성 서비스에 대한 **SpeechConfig**가 있으므로 **음성 텍스트 변환** API를 사용하여 음성을 인식하고 텍스트로 변환할 수 있습니다.

> **중요**: 이 섹션에는 두 가지 대체 절차에 대한 지침이 포함되어 있습니다. 마이크가 작동하는 경우 첫 번째 절차를 따릅니다. 오디오 파일을 사용하여 음성 입력을 시뮬레이션하려면 두 번째 절차를 따릅니다.

### 작동하는 마이크가 있는 경우

1. 프로그램의 **Main** 함수에서 코드가 **TranscribeCommand** 함수를 사용해 음성 입력을 수락함을 확인합니다.
1. **TranscribeCommand** 함수의 **음성 인식 구성** 주석 아래에 다음의 적절한 코드를 추가하여 **SpeechRecognizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 기본 시스템 마이크를 사용해 음성을 인식하고 필사할 수 있습니다.

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. 이제 아래의 **필사된 명령을 처리하기 위한 코드 추가** 섹션으로 넘어갑니다.

---

### 또는 파일에서 오디오 입력 사용

1. 터미널 차에서 다음 명령을 입력하여 오디오 파일을 재생하는 데 사용할 수 있는 라이브러리를 설치합니다.

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. 프로그램의 코드 파일에서, 기존 네임스페이스 가져오기 아래에 다음 코드를 추가하여 방금 설치한 라이브러리를 가져옵니다.

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. **Main** 함수에서 코드가 **TranscribeCommand** 함수를 사용해 음성 입력을 수락함을 확인합니다. 그런 다음에 **TranscribeCommand** 함수의 **음성 인식 구성** 주석 아래에 다음의 적절한 코드를 추가하여 **SpeechRecognizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 오디오 파일로부터 음성을 인식하고 필사할 수 있습니다.

    **C#**: Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### 필사된 명령을 처리하기 위한 코드 추가

1. **TranscribeCommand** 함수의 **음성 입력 처리** 주석 아래에 다음 코드를 추가하여 음성 입력을 청취합니다. 이때 명령을 반환하는 함수 끝부분의 코드를 바꾸지 않도록 주의하세요.

    **C#**: Program.cs

    ```csharp
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**: speaking-clock.py

    ```python
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

1. 변경 내용을 저장하고 **speaking-clock** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 마이크를 사용하는 경우에는 "what time is it?"라고 크고 분명하게 말합니다. 프로그램이 음성 입력을 필사한 다음 시간(코드를 실행 중인 컴퓨터의 현지 시간 기준. 현재 위치에서는 정확한 시간이 아닐 수도 있음)을 표시해야 합니다

    SpeechRecognizer에서는 말할 수 있는 시간이 약 5초입니다. 이 시간 동안 음성 입력이 감지되지 않으면 "No match" 결과가 생성됩니다.

    SpeechRecognizer에서 오류가 발생하면 "Cancelled" 결과가 생성됩니다. 그러면 애플리케이션의 코드가 오류 메시지를 표시합니다. 오류의 원인은 구성 파일의 지역이나 키가 정확하지 않기 때문일 가능성이 가장 높습니다.

## 음성 합성

음성 시계 애플리케이션은 음성 입력을 수락하지만 실제로 시간을 직접 말로 알려 주지는 않습니다. 이제 음성을 합성하는 코드를 추가하여 응답 방식을 수정해 보겠습니다.

1. 프로그램의 **Main** 함수에서 코드가 **TellTime** 함수를 사용하여 사용자에게 현재 시간을 알려 줌을 확인합니다.
1. **TellTime** 함수의 **음성 합성 구성** 주석 아래에 다음 코드를 추가하여 **SpeechSynthesizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 음성 출력을 생성할 수 있습니다.

    **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **참고** 기본 오디오 구성에서는 출력에 기본 시스템 오디오 디바이스를 사용하므로 **AudioConfig**는 명시적으로 제공하지 않아도 됩니다. 오디오 출력을 파일로 리디렉션해야 하는 경우에는 파일 경로가 포함된 **AudioConfig**를 사용하면 됩니다.

1. **TellTime** 함수의 **음성 출력 합성** 주석 아래에 다음 코드를 추가하여 음성 출력을 생성합니다. 이때 응답을 인쇄하는 함수 끝부분의 코드를 바꾸지 않도록 주의하세요.

    **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 변경 내용을 저장하고 **speaking-clock** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 메시지가 표시되면 마이크에 "what time is it"이라고 또렷하게 말합니다. 그러면 프로그램이 음성으로 시간을 알려 줍니다.

## 다른 음성 사용

음성 시계 애플리케이션은 기본 음성을 사용하는데, 이 음성을 변경할 수 있습니다. Speech 서비스에서는 광범위한 *표준* 음성은 물론 인간의 음성과 더욱 비슷한 *신경망* 음성도 지원합니다. *사용자 지정* 음성을 만들 수도 있습니다.

> **참고**: 신경 및 표준 음성 목록을 보려면 Speech Studio의 [음성 갤러리](https://speech.microsoft.com/portal/voicegallery)를 참조하세요.

1. **TellTime** 함수의 **음성 합성 구성** 주석 아래에서 코드를 다음과 같이 수정하여 **SpeechSynthesizer** 클라이언트를 만들기 전에 대체 음성을 지정합니다.

   **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. 변경 내용을 저장하고 **speaking-clock** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 메시지가 표시되면 마이크에 "what time is it"이라고 또렷하게 말합니다. 그러면 프로그램이 지정된 음성으로 시간을 알려 줍니다.

## Speech Synthesis Markup Language 사용

SSML(Speech Synthesis Markup Language)을 사용하면 XML 기반 형식을 통해 음성 합성 방식을 사용자 지정할 수 있습니다.

1. **TellTime** 함수에서 **음성 출력 합성** 현재 주석 아래에 있는 모든 코드를 다음 코드로 바꿉니다(**응답 인쇄** 주석 아래의 코드는 그대로 유지).

   **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

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
    ```

1. 변경 내용을 저장하고 **speaking-clock** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 메시지가 표시되면 마이크에 "what time is it"이라고 또렷하게 말합니다. 프로그램이 SSML에 지정된 음성으로 시간을 알려 준 다음(SpeechConfig에 지정된 음성이 재정의됨) 잠시 후에 종료하려면 "Time to end this lab!"을 말하라고 합니다.

## 자세한 정보

**음성 텍스트 변환** 및 **텍스트 음성 변환** API 사용에 대한 자세한 내용은 [음성 텍스트 변환 설명서](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) 및 [텍스트 음성 변환 설명서](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech)를 참조하세요.
