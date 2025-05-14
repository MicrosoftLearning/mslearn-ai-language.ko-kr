---
lab:
  title: 오디오 지원 채팅 앱 개발
  description: Azure AI 파운드리를 사용하여 오디오 입력을 지원하는 생성형 AI 앱을 빌드하는 방법을 알아봅니다.
---

# 오디오 지원 채팅 앱 개발

이 연습에서는 *Phi-4-multimodal-instruct* 생성형 AI 모델을 사용하여 오디오 파일을 포함하는 프롬프트에 대한 응답을 생성합니다. Azure AI 파운드리 및 Azure AI 모델 추론 서비스를 사용하여 식료품점에서 신선 식품으로 AI 지원을 제공하는 앱을 개발합니다.

이 연습에는 약 **30**분이 소요됩니다.

## Azure AI 파운드리 프로젝트 만들기

먼저 Azure AI 파운드리 프로젝트를 만들어 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈 페이지로 이동합니다.

    ![Azure AI Foundry 포털의 스크린샷.](../media/ai-foundry-home.png)

1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서 유효한 프로젝트 이름을 입력하고 기존 허브가 추천되면 새 허브를 만드는 옵션을 선택합니다. 그런 다음 허브 및 프로젝트를 지원하기 위해 자동으로 만들어지는 Azure 리소스를 검토합니다.
1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **허브 이름**: *허브에서 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **위치**: 다음 지역 중 하나를 선택합니다.\*
        - 미국 동부
        - 미국 동부 2
        - 미국 중북부
        - 미국 중남부
        - 스웨덴 중부
        - 미국 서부
        - 미국 서부 3
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: *새 AI 서비스 리소스 만들기*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* 작성 시점에 이 연습에서 사용할 Microsoft *Phi-4-multimodal-instruct* 모델은 이 지역에서 사용할 수 있습니다. [Azure AI 파운드리 설명서](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability)에서 특정 모델에 대한 최신 지역 가용성을 확인할 수 있습니다. 연습 후반부에 지역 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **다음**을 선택하여 구성을 검토합니다. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.
1. 프로젝트를 만들 때 표시되는 팁을 모두 닫고 Azure AI 파운드리 포털에서 프로젝트 페이지를 검토합니다. 이 페이지는 다음 이미지와 유사합니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](../media/ai-foundry-project.png)

## 멀티모달 모델 배포

이제 오디오 기반 입력을 지원할 수 있는 멀티모달 모델을 배포할 준비가 되었습니다. OpenAI *gpt-4o* 모델을 포함하여 선택할 수 있는 여러 모델이 있습니다. 이 연습에서는 *Phi-4-multimodal-instruct* 모델을 사용하겠습니다.

1. Azure AI 파운드리 프로젝트 페이지의 오른쪽 위에 있는 도구 모음에서 **미리 보기 기능**(**&#9215;**) 아이콘을 사용하여 **Azure AI 모델 추론 서비스에 모델 배포** 기능을 사용하도록 설정합니다. 이 기능을 사용하면 애플리케이션 코드에서 사용할 Azure AI 유추 서비스에서 모델 배포를 사용할 수 있습니다.
1. 프로젝트 왼쪽 창의 **내 자산** 섹션에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. **모델 + 엔드포인트** 페이지의 **모델 배포** 탭의 **+ 모델 배포** 메뉴에서 **기본 모델 배포**를 선택합니다.
1. 목록에서 **Phi-4-multimodal-instruct** 모델을 검색한 다음 선택하고 확인합니다.
1. 메시지가 표시되면 사용권 계약에 동의한 다음 배포 세부 정보에서 **사용자 지정**을 선택하여 다음 설정을 사용하여 모델을 배포합니다.
    - **배포 이름**: *모델 배포에 대한 고유한 이름*
    - **배포 유형**: 글로벌 표준
    - **배포 세부 정보**: *기본 설정 사용*
1. 배포 프로비전 상태가 **완료**될 때까지 기다립니다.

## 클라이언트 애플리케이션 생성하기

이제 모델을 배포했으므로 클라이언트 애플리케이션에서 배포를 사용할 수 있습니다.

> **팁**: Python 또는 Microsoft C#을 사용하여 솔루션을 개발하도록 선택할 수 있습니다. 선택한 언어의 해당 섹션에 있는 지침을 따릅니다.

### 애플리케이션 구성 준비

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 봅니다.
1. **프로젝트 세부 정보** 영역에서 **프로젝트 연결 문자열**을 확인합니다. 이 연결 문자열 사용하여 클라이언트 응용 프로그램에서 프로젝트에 연결합니다.
1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

    Azure Portal 홈페이지를 보려면 환영 알림을 닫습니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다. 보다 쉽게 작업할 수 있도록 이 창의 크기를 조정하거나 최대화할 수 있습니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하여 일반 텍스트로 붙여넣습니다).


    ```
    rm -r mslearn-ai-audio -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/c-sharp
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
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

1. 코드 파일에서 **your_project_connection_string** 자리 표시자를 프로젝트의 연결 문자열(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고, **your_model_deployment** 자리 표시자를 Phi-4-multimodal-instruct 모델 배포에 할당한 이름으로 바꿉니다.

1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 프로젝트에 연결하고 모델에 대한 채팅 클라이언트를 가져오는 코드를 작성합니다.

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. 코드 파일에서 파일 상단에 추가된 기존 문에 주목하여 필요한 SDK 네임스페이스를 가져옵니다. 그런 다음 **Add references** 주석을 찾아 이전에 설치한 라이브러리의 네임스페이스를 참조하도록 다음 코드를 추가합니다.

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        AudioContentItem,
        InputAudio,
        AudioContentFormat,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. **main** 함수의 **구성 설정 가져오기** 주석 아래에서 해당 코드가 구성 파일에서 정의한 프로젝트 연결 문자열 및 모델 배포 이름 값을 로드한다는 점에 유의합니다.

1. **Initialize the project client** 주석을 찾아 다음 코드를 추가하여 현재 로그인한 Azure 자격 증명을 사용하여 Azure AI 파운드리 프로젝트에 연결합니다.

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. **Get a chat client** 주석을 찾아 다음 코드를 추가하여 모델과 채팅할 클라이언트 개체를 만듭니다.

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### URL 기반 오디오 프롬프트를 제출하는 코드 작성

1. **audio-chat.py** 파일의 코드 편집기에서 루프 섹션의 **Get a response to audio input** 주석 아래에 다음 코드를 추가하여 다음 오디오가 포함된 프롬프트를 제출합니다.

    <video controls src="../media/manzanas.mp4" title="사과 요청" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다. 원하는 경우 코드 편집기를 닫을 수도 있습니다(**CTRL+Q**).

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 앱을 실행합니다.

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. 메시지가 표시되면 프롬프트(`What is this customer saying in English?`)를 입력합니다.

1. 응답을 검토합니다.

### 다른 프롬프트 사용

1. 앱 코드의 코드 편집기에서 **Get a response to audio input** 주석 아래에서 이전에 추가한 코드를 찾습니다. 그런 다음 다음과 같이 코드를 수정하여 다른 오디오 파일을 선택합니다.

    <video controls src="../media/caomei.mp4" title="딸기 요청" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다. 원하는 경우 코드 편집기를 닫을 수도 있습니다(**CTRL+Q**).

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 앱을 실행합니다.

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. 메시지가 표시되면 다음 프롬프트를 입력합니다.

    ```
    A customer left this voice message, can you summarize it?
    ```

1. 응답을 검토합니다. 그런 다음 `quit`(을)를 입력하여 프로그램을 종료합니다.

    > **참고**: 이 간단한 앱에서는 대화 내용을 유지하는 논리를 구현하지 않았으므로 모델은 각 프롬프트를 이전 프롬프트의 컨텍스트 없이 새 요청으로 처리합니다.

1. 앱을 계속 실행하고, 다른 프롬프트 유형을 선택하고, 다른 프롬프트를 시도할 수 있습니다. 완료되면 `quit`를 입력하여 프로그램을 종료합니다.

    시간이 있다면 다른 시스템 프롬프트와 인터넷에서 액세스할 수 있는 자체 오디오 파일을 사용하도록 코드를 수정할 수 있습니다.

    > **참고**: 이 간단한 앱에서는 대화 내용을 유지하는 논리를 구현하지 않았으므로 모델은 각 프롬프트를 이전 프롬프트의 컨텍스트 없이 새 요청으로 처리합니다.

## 요약

이 연습에서는 Azure AI 파운드리 및 Azure AI 유추 SDK를 사용하여 오디오에 대한 응답을 생성하는 멀티모달 모델 기반의 클라이언트 애플리케이션을 만들었습니다.

## 정리

Azure AI 파운드리 탐색을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
