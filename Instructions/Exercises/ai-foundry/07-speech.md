---
lab:
  title: 음성 인식 및 합성(Azure AI 파운드리 버전)
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# 음성 인식 및 합성

**Azure AI 음성**은 다음을 포함한 음성 관련 기능을 제공하는 서비스입니다.

- *음성 텍스트 변환* API - 음성 인식(가청 음성 단어를 텍스트로 변환)을 구현할 수 있습니다.
- *텍스트 음성 변환* API - 음성 합성(텍스트를 가청 음성으로 변환)을 구현할 수 있습니다.

이 연습에서는 두 API를 모두 사용하여 음성으로 시간을 안내하는 시계 애플리케이션을 구현합니다.

> **참고** 이 연습은 컴퓨터의 사운드 하드웨어에 대한 직접 액세스가 지원되지 않는 Azure Cloud Shell에서 완료하도록 설계되었습니다. 따라서 랩은 음성 입력 및 출력 스트림에 오디오 파일을 사용합니다. 마이크와 스피커를 사용하여 동일한 결과를 얻기 위한 코드가 참조용으로 제공됩니다.

## Azure AI 파운드리 프로젝트 만들기

먼저 Azure AI 파운드리 프로젝트를 만들어 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈 페이지로 이동합니다.

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서 적절한 프로젝트 이름(예`my-ai-project`: )을 입력한 다음, 프로젝트를 지원하기 위해 자동으로 만들어지는 Azure 리소스를 검토합니다.
1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **허브 이름**: *고유한 이름 - 예: `my-ai-hub`*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *고유한 이름(예: `my-ai-resources`)으로 새 리소스 그룹을 만들거나 기존 리소스 그룹 선택*
    - **위치**: 사용 가능한 지역 선택
    - **Azure AI Services 또는 Azure OpenAI** 연결: *적절한 이름으로 새 AI Services 리소스를 만들거나(예 `my-ai-services`:) 기존 리소스를 사용합니다.*
    - **Azure AI 검색 연결**: 연결 건너뛰기

1. **다음**을 선택하여 구성을 검토합니다. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.
1. 프로젝트를 만들 때 표시되는 팁을 모두 닫고 Azure AI 파운드리 포털에서 프로젝트 페이지를 검토합니다. 이 페이지는 다음 이미지와 유사합니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](./media/ai-foundry-project.png)

## 음성 시계 앱 준비 및 구성

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 봅니다.
1. **프로젝트 세부 정보** 영역에서 **프로젝트 연결 문자열** 및 프로젝트의 **위치**를 확인합니다. 이 연결 문자열 사용하여 클라이언트 애플리케이션에서 프로젝트에 연결하고 Azure AI 서비스 Speech 엔드포인트에 연결할 위치가 필요합니다.
1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.
1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***선택한 프로그래밍 언어에 대한 단계를 따릅니다.***

1. 리포지토리가 복제된 후 음성 시계 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    **Python**

    ```
   pip install python-dotenv azure-identity azure-ai-projects azure-cognitiveservices-speech==1.42.0
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Microsoft.CognitiveServices.Speech --version 1.42.0
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 및 **your_location** 자리 표시자를 프로젝트의 연결 문자열 및 위치로 바꿉니다(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사함).
1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

## Azure AI 음성 SDK를 사용하기 위한 코드 추가

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 코드 파일 상단의 기존 네임스페이스 참조 아래에서 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 Azure Ai 파운드리 프로젝트의 Azure AI 서비스 리소스와 함께 Azure AI 음성 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **Python**

    ```python
   # Import namespaces
   from dotenv import load_dotenv
   from azure.ai.projects.models import ConnectionType
   from azure.identity import DefaultAzureCredential
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.projects import AIProjectClient
   import azure.cognitiveservices.speech as speech_sdk
    ```

    **C#**

    ```csharp
   // Import namespaces
   using Azure.Identity;
   using Azure.AI.Projects;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
    ```

1. **main** 함수의 **구성 설정 가져오기** 주석 아래에서 코드가 구성 파일에 정의한 프로젝트 연결 문자열과 위치를 로드한다는 점에 유의합니다.

1. **프로젝트에서 AI 음성 엔드포인트 및 키 가져오기** 주석 아래에 다음 코드를 추가합니다.

    **Python**

    ```python
   # Get AI Services key from the project
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())

   ai_svc_connection = project_client.connections.get_default(
      connection_type=ConnectionType.AZURE_AI_SERVICES,
      include_credentials=True, 
    )

   ai_svc_key = ai_svc_connection.key

    ```

    **C#**

    ```csharp
   // Get AI Services key from the project
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());

   ConnectionResponse aiSvcConnection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureAIServices, true);

   var apiKeyAuthProperties = aiSvcConnection.Properties as ConnectionPropertiesApiKeyAuth;

   var aiSvcKey = apiKeyAuthProperties.Credentials.Key;
    ```

    이 코드는 Azure AI 파운드리 프로젝트에 연결하고, 기본 AI Services 연결된 리소스를 가져오고, 이를 사용하는 데 필요한 인증 키를 검색합니다.

1. **음성 서비스 구성** 주석 아래에 다음 코드를 추가하여 AI Services 키와 프로젝트의 지역을 사용하여 Azure AI 서비스 음성 엔드포인트에 대한 연결을 구성합니다.

   **Python**

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(ai_svc_key, location)
   print('Ready to use speech service in:', speech_config.region)
    ```

    **C#**

    ```csharp
   // Configure speech service
   speechConfig = SpeechConfig.FromSubscription(aiSvcKey, location);
   Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    ```

1. 변경 내용(*Ctrl+S*)을 저장하지만 코드 편집기를 열어 둡니다.

## 앱 실행

지금까지 앱은 Speech Service를 사용하는 데 필요한 세부 정보를 검색하기 위해 Azure AI 파운드리 프로젝트에 연결하는 것 외에는 아무 작업도 수행하지 않지만, 음성 기능을 추가하기 전에 이를 실행하고 작동하는지 확인하는 것이 유용합니다.

1. 코드 편집기 아래의 명령줄에서 다음 Azure CLI 명령을 입력하여 세션에 로그인한 Azure 계정을 확인합니다.

    ```
   az account show
    ```

    결과 JSON 출력에는 Azure 계정 및 작업 중인 구독의 세부 정보가 포함되어야 합니다(Azure AI 파운드리 프로젝트를 만든 구독과 동일해야 합니다.)

    앱은 실행되는 컨텍스트에 Azure 자격 증명을 사용하여 프로젝트에 대한 연결을 인증합니다. 프로덕션 환경에서는 관리 ID를 사용하여 앱을 실행하도록 구성할 수 있습니다. 이 개발 환경에서는 인증된 Cloud Shell 세션 자격 증명을 사용합니다.

    > **참고**: `az login`Azure CLI 명령을 사용하여 개발 환경에서 Azure에 로그인할 수 있습니다. 이 경우 Cloud Shell은 포털에 로그인한 Azure 자격 증명을 사용하여 이미 로그인했습니다. 따라서 명시적으로 로그인할 필요가 없습니다. Azure CLI를 사용하여 Azure에 인증하는 방법에 대한 자세한 내용은 [Azure CLI를 사용하여 Azure에 인증](https://learn.microsoft.com/cli/azure/authenticate-azure-cli)을 참조하세요.

1. 명령줄에서 다음 언어별 명령을 입력하여 음성 시계 앱을 실행합니다.

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. C#을 사용 중인 경우 비동기 메서드의 **await** 연산자 사용 관련 경고는 무시해도 됩니다. 뒷부분에서 해당 부분을 수정할 것입니다. 이 코드는 애플리케이션이 사용할 Speech 서비스 리소스의 지역을 표시합니다. 실행이 성공하면 앱이 Azure AI 파운드리 프로젝트에 연결되고 Azure AI 음성 서비스를 사용하는 데 필요한 키를 검색했음을 나타냅니다.

## 음성을 인식하는 코드 추가

이제 프로젝트의 Azure AI 서비스 리소스에 음성 서비스에 대한 **SpeechConfig**가 있으므로 **음성 텍스트 변환** API를 사용하여 음성을 인식하고 텍스트로 변환할 수 있습니다.

이 절차에서 음성 입력은 오디오 파일에서 캡처되며 여기서 재생할 수 있습니다.

<video controls src="media/Time.mp4" title="What time is it?" width="150"></video>

1. **Main** 함수에서 코드가 **TranscribeCommand** 함수를 사용해 음성 입력을 수락함을 확인합니다. 그런 다음에 **TranscribeCommand** 함수의 **음성 인식 구성** 주석 아래에 다음의 적절한 코드를 추가하여 **SpeechRecognizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 오디오 파일로부터 음성을 인식하고 필사할 수 있습니다.

    **Python**

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

    **C#**

    ```csharp
   // Configure speech recognition
   string audioFile = "time.wav";
   using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
   using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

1. **TranscribeCommand** 함수의 **음성 입력 처리** 주석 아래에 다음 코드를 추가하여 음성 입력을 청취합니다. 이때 명령을 반환하는 함수 끝부분의 코드를 바꾸지 않도록 주의하세요.

    **Python**

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

    **C#**

    ```csharp
   // Process speech input
   Console.WriteLine("Listening...");
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

1. 변경 사항을 저장(*CTRL+S*)한 후 코드 편집기 아래 명령줄에 다음 명령을 입력하여 프로그램을 실행합니다.

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 애플리케이션의 출력을 검토하여 오디오 파일의 음성을 성공적으로 "듣고" 적절한 응답을 반환해야 합니다(Azure Cloud Shell이 사용자와 다른 시간대에 있는 서버에서 실행 중일 수 있음에 유의합니다!).

    > **팁**: SpeechRecognizer에 오류가 발생하면 "취소됨"이라는 결과를 생성합니다. 그러면 애플리케이션의 코드가 오류 메시지를 표시합니다. 가장 가능성이 높은 원인은 구성 파일의 잘못된 지역 값입니다.

## 음성 합성

음성 시계 애플리케이션은 음성 입력을 수락하지만 실제로 시간을 직접 말로 알려 주지는 않습니다. 이제 음성을 합성하는 코드를 추가하여 응답 방식을 수정해 보겠습니다.

다시 한 번, Cloud Shell의 하드웨어 제한으로 인해 합성된 음성 출력을 파일로 전달할 것입니다.

1. 프로그램의 **Main** 함수에서 코드가 **TellTime** 함수를 사용하여 사용자에게 현재 시간을 알려 줌을 확인합니다.
1. **TellTime** 함수의 **음성 합성 구성** 주석 아래에 다음 코드를 추가하여 **SpeechSynthesizer** 클라이언트를 만듭니다. 이 클라이언트를 사용하면 음성 출력을 생성할 수 있습니다.

    **Python**

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

    **C#**

    ```csharp
   // Configure speech synthesis
   var outputFile = "output.wav";
   speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
   using var audioConfig = AudioConfig.FromWavFileOutput(outputFile);
   using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    ```

1. **TellTime** 함수의 **음성 출력 합성** 주석 아래에 다음 코드를 추가하여 음성 출력을 생성합니다. 이때 응답을 인쇄하는 함수 끝부분의 코드를 바꾸지 않도록 주의하세요.

    **Python**

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

    **C#**

    ```csharp
   // Synthesize spoken output
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
       Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. 변경 사항을 저장(*CTRL+S*)한 후 코드 편집기 아래 명령줄에 다음 명령을 입력하여 프로그램을 실행합니다.

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 애플리케이션의 출력을 검토하여 음성 출력이 파일에 저장되었음을 표시해야 합니다.
1. .wav 오디오 파일을 재생할 수 있는 미디어 플레이어가 있는 경우, Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 버튼을 사용하여 앱 폴더에 오디오 파일을 다운로드한 다음 재생합니다.

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    파일은 이와 비슷하게 들릴 것입니다.

    <video controls src="./media/Output.mp4" title="시간은 2시 15분입니다." width="150"></video>

## Speech Synthesis Markup Language 사용

SSML(Speech Synthesis Markup Language)을 사용하면 XML 기반 형식을 통해 음성 합성 방식을 사용자 지정할 수 있습니다.

1. **TellTime** 함수에서 **음성 출력 합성** 현재 주석 아래에 있는 모든 코드를 다음 코드로 바꿉니다(**응답 인쇄** 주석 아래의 코드는 그대로 유지).

    **Python**

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
       print("Spoken output saved in " + outputFile)
    ```

   **C#**

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
   else
   {
        Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. 변경 내용을 저장하고 **speaking-clock** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 애플리케이션의 출력을 검토하여 음성 출력이 파일에 저장되었음을 표시해야 합니다.
1. 다시 한 번 .wav 오디오 파일을 재생할 수 있는 미디어 플레이어가 있는 경우 Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 버튼을 사용하여 앱 폴더에 오디오 파일을 다운로드한 다음 재생합니다.

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    파일은 이와 비슷하게 들릴 것입니다.
    
    <video controls src="./media/Output2.mp4" title="시간은 5:30입니다. 이 랩을 종료할 시간입니다." width="150"></video>

## 정리

Azure AI 음성 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.

## 마이크와 스피커가 있는 경우 어떻게 해야 할까요?

이 연습에서는 음성 입력 및 출력에 오디오 파일을 사용했습니다. 오디오 하드웨어를 사용하도록 코드를 수정하는 방법을 살펴보겠습니다.

### 마이크를 사용한 음성 인식 사용

마이크가 있는 경우 다음 코드를 사용하여 음성 인식을 위한 음성 입력을 캡처할 수 있습니다.

**Python**

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

**C#**

```csharp
// Configure speech recognition
using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
Console.WriteLine("Speak now...");

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

> **참고**: 시스템 기본 마이크가 기본 오디오 입력이므로 AudioConfig를 아예 생략할 수도 있습니다!

### 스피커로 음성 합성 사용

스피커가 있는 경우 다음 코드를 사용하여 음성을 합성할 수 있습니다.

**Python**

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

**C#**

```csharp
var now = DateTime.Now;
string responseText = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");

// Configure speech synthesis
speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
using var audioConfig = AudioConfig.FromDefaultSpeakerOutput();
using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Synthesize spoken output
SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
{
    Console.WriteLine(speak.Reason);
}
```

> **참고**: 시스템 기본 스피커가 기본 오디오 출력이므로 AudioConfig를 아예 생략할 수도 있습니다!

## 자세한 정보

**음성 텍스트 변환** 및 **텍스트 음성 변환** API 사용에 대한 자세한 내용은 [음성 텍스트 변환 설명서](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) 및 [텍스트 음성 변환 설명서](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech)를 참조하세요.
