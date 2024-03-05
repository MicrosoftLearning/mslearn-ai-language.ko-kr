---
lab:
  title: 텍스트 분석
  module: Module 3 - Develop natural language processing solutions
---

# 텍스트 분석

**Azure 언어**는 언어 인식, 감정 분석, 핵심 구 추출 및 엔터티 인식을 포함한 텍스트 분석을 지원합니다.

회사 웹 사이트로 제출된 호텔 리뷰를 처리하려는 여행사의 경우를 예로 들어 보겠습니다. 이 여행사는 Azure AI 언어를 사용하여 각 리뷰를 작성한 언어, 리뷰의 감정(긍정적, 중립, 부정적), 리뷰에 설명되어 있는 주요 토픽을 나타낼 수 있는 핵심 구, 그리고 리뷰에 언급되어 있는 장소, 주요 건물, 사람 등의 명명된 엔터티를 확인할 수 있습니다.

## *Azure AI 언어* 리소스를 프로비전합니다.

구독에 아직 리소스가 없는 경우 Azure 구독에 **Azure AI 언어 서비스** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. 상단 검색 창에서 **Azure AI 서비스**를 검색합니다. 그런 다음 결과에서 **언어 서비스** 아래의 **만들기**를 선택합니다.
1. **계속하여 리소스 만들기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 프로비전합니다.
    - **구독**: *자신의 Azure 구독*.
    - **리소스 그룹**: *리소스 그룹을 선택하거나 만듭니다*.
    - **지역**: *사용 가능한 지역을 선택합니다*.
    - **이름**: 고유한 이름을 입력합니다.
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **담당 AI 알림**: 동의합니다.
1. **검토 + 만들기**를 선택합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## Visual Studio Code에서 앱 개발 준비

Visual Studio Code를 사용하여 텍스트 분석 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-ai-language** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-language` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션과 요약을 테스트하는 데 사용할 샘플 텍스트 파일이 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure AI 언어 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/01-analyze-text** 폴더를 찾아 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장하고, 포함되어 있는 **text-analytics** 폴더를 확장합니다. 각 폴더에는 Azure AI 언어 텍스트 분석 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
2. 코드 파일이 포함된 **text-analytics** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 언어 Text Analytics SDK 패키지를 설치합니다. Python 연습을 위해 `dotenv` 패키지도 설치합니다.

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. **탐색기** 창의 **text-analytics** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. 만든 Azure 언어 리소스의 **엔드포인트** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal의 Azure AI 언어 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능).
5. 구성 파일을 저장합니다.

6. **text-analysis** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: text-analysis.py

    코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Text Analytics SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. **Main** 함수에 Azure AI 언어 서비스 엔드포인트와 구성 파일의 키를 로드하는 코드가 이미 제공되어 있습니다. 그런 후 **엔드포인트와 키를 사용하여 클라이언트 만들기** 주석을 찾아서 다음 코드를 추가하여 Text Analytics API용 클라이언트를 만듭니다.

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. 변경 내용을 저장하고 **text-analysis** 폴더의 통합 터미널로 돌아와서 다음 명령을 입력하여 프로그램을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python text-analysis.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

9. 오류가 발생하지 않고 코드가 실행되어 **reviews** 폴더에 있는 각 리뷰 텍스트 파일의 내용이 표시되는지 확인합니다. 애플리케이션은 Text Analytics API용 클라이언트를 만들었지만 해당 클라이언트를 사용하지는 않습니다. 다음 절차에서 애플리케이션이 서비스를 사용하도록 수정합니다.

## 언어를 검색하는 코드 추가

이제 API용 클라이언트를 만들었으므로 이를 사용하여 각 검토가 작성되는 언어를 검색해 보겠습니다.

1. 프로그램의 **Main** 함수에서 **언어 가져오기** 주석을 찾습니다. 그 후에 이 주석 아래에 각 리뷰 문서의 언어를 감지하는 데 필요한 코드를 추가합니다.

    **C#**: Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **참고**: *이 예제에서는 각 리뷰를 개별적으로 분석하므로 각 파일 분석을 위해 서비스를 각기 별도로 호출합니다. 이 방식 대신 문서 컬렉션을 만든 후 단일 호출에서 서비스에 컬렉션을 전달하는 방식을 사용할 수 있습니다. 두 방식에서 모두 서비스의 응답은 문서 컬렉션으로 구성되어 있습니다. 따라서 위 Python 코드의 응답에는 첫 번째(유일한) 문서의 인덱스([0])만 지정되어 있습니다.*

1. 변경 내용을 저장합니다. 그런 다음 **text-analytic** 폴더의 통합 터미널로 돌아가서 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 이번에는 각 리뷰의 언어가 식별되었습니다.

## 감정을 평가하는 코드 추가

감정 분석은 텍스트를 긍정적 또는 부정적으로 분류할 때 흔히 사용되는 기술입니다(중립적 또는 혼합으로 분류할 수도 있음).********** 감정 분석은 소셜 미디어 게시물, 제품 리뷰, 그리고 텍스트 감정이 유용한 인사이트를 제공할 수 있는 기타 항목을 분석하는 데 흔히 사용됩니다.

1. 프로그램의 **Main** 함수에서 **감정 가져오기** 주석을 찾습니다. 그 후에 이 주석 아래에 각 리뷰 문서의 감정을 감지하는 데 필요한 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. 변경 내용을 저장합니다. 그런 다음 **text-analytic** 폴더의 통합 터미널로 돌아가서 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 리뷰의 감정이 감지되었습니다.

## 핵심 구를 식별하는 코드 추가

텍스트 본문에서 핵심 구를 식별하면 텍스트에 설명되어 있는 주요 토픽을 확인하는 데 유용할 수 있습니다.

1. 프로그램의 **Main** 함수에서 **핵심 구 가져오기** 주석을 찾습니다. 그 후에 이 주석 아래에 각 리뷰 문서의 핵심 구를 감지하는 데 필요한 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. 변경 내용을 저장합니다. 그런 다음 **text-analytic** 폴더의 통합 터미널로 돌아가서 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 각 문서에는 리뷰 내용과 관련한 몇 가지 인사이트를 제공하는 핵심 구가 포함되어 있습니다.

## 엔터티를 추출하는 코드 추가

문서나 기타 텍스트 본문에서는 사람, 장소, 기간 또는 기타 엔터티를 언급하는 경우가 많습니다. Text Analytics API는 텍스트 내 엔터티의 여러 범주(및 하위 범주)를 감지할 수 있습니다.

1. 프로그램의 **Main** 함수에서 **엔터티 가져오기** 주석을 찾습니다. 그 후에 이 주석 아래에 각 리뷰에 언급되어 있는 엔터티를 식별하는 데 필요한 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. 변경 내용을 저장합니다. 그런 다음 **text-analytic** 폴더의 통합 터미널로 돌아가서 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 텍스트에서 엔터티가 감지되었습니다.

## 연결된 엔터티를 추출하는 코드 추가

Text Analytics API는 범주화된 엔터티뿐 아니라 Wikipedia 등의 데이터 원본에 대한 알려진 링크가 있는 엔터티도 감지할 수 있습니다.

1. 프로그램의 **Main** 함수에서 **연결된 엔터티 가져오기** 주석을 찾습니다. 그 후에 이 주석 아래에 각 리뷰에 언급되어 있는 연결된 엔터티를 식별하는 데 필요한 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. 변경 내용을 저장합니다. 그런 다음 **text-analytic** 폴더의 통합 터미널로 돌아가서 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 연결된 엔터티가 식별되었습니다.

## 리소스 정리

Azure AI 언어 서비스 탐색을 마쳤으면 이 연습에서 만든 리소스를 삭제할 수 있습니다. 이 경우 가능한 방법은 다음과 같습니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.

2. 이 랩에서 만든 Azure AI 언어 리소스를 찾습니다.

3. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

**Azure AI 언어** 사용에 대한 자세한 내용은 [설명서](https://learn.microsoft.com/azure/ai-services/language-service/)를 참조하세요.
