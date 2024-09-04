---
lab:
  title: 사용자 지정 텍스트 분류
  module: Module 3 - Getting Started with Natural Language Processing
---

# 사용자 지정 텍스트 분류

Azure AI 언어는 핵심 구 식별, 텍스트 요약 및 감정 분석을 비롯한 여러 NLP 기능을 제공합니다. 언어 서비스는 사용자 지정 질문 답변 및 사용자 지정 텍스트 분류와 같은 사용자 지정 기능도 제공합니다.

Azure AI 언어 서비스의 사용자 지정 텍스트 분류를 테스트하기 위해 Language Studio를 사용하여 모델을 구성한 다음, Cloud Shell에서 실행되는 작은 명령줄 애플리케이션을 사용하여 테스트합니다. 여기에서 사용되는 동일한 패턴과 기능은 실제 애플리케이션을 따를 수 있습니다.

## *Azure AI 언어* 리소스를 프로비전합니다.

구독에 아직 없는 경우 **Azure AI 언어 서비스** 리소스를 프로비전해야 합니다. 또한 사용자 지정 텍스트 분류를 사용하려면 **사용자 지정 텍스트 분류 및 추출** 기능을 사용하도록 설정해야 합니다.

1. 브라우저의 `https://portal.azure.com`에서 Azure Portal을 열고 Microsoft 계정에 로그인합니다.
1. 포털 상단의 검색 필드를 선택하고 `Azure AI services`를 검색한 후 **언어 서비스** 리소스를 만듭니다.
1. **사용자 지정 텍스트 분류**가 포함된 상자를 선택합니다. **계속해서 리소스 만들기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: *자신의 Azure 구독*.
    - **리소스 그룹**: *리소스 그룹 선택하거나 만듭니다*.
    - **지역**: *사용 가능한 지역을 선택*합니다.
    - **이름**: *고유한 이름을 입력*합니다.
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **스토리지 계정**: 새 스토리지 계정
      - **스토리지 계정 이름**: 고유한 이름을 입력합니다.
      - **스토리지 계정 유형**: 표준 LRS
    - **책임 있는 AI 알림**: 선택됨.

1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## 샘플 문서 업로드

Azure AI 언어 서비스 및 스토리지 계정을 만든 후에는 나중에 모델을 학습시키기 위한 예제 문서를 업로드해야 합니다.

1. 새 브라우저 탭에서 `https://aka.ms/classification-articles`의 샘플 문서를 다운로드하고 원하는 폴더에 파일을 추출합니다.

1. Azure Portal에서 생성된 스토리지 계정으로 이동하고 선택합니다.

1. 스토리지 계정에서 **설정** 아래에 있는 **구성**을 선택합니다. 구성 화면에서 **Blob 익명 액세스 허용** 옵션을 사용하도록 설정한 다음 **저장**을 선택합니다.

1. **데이터 스토리지** 아래에 있는 왼쪽 메뉴에서 **컨테이너**를 선택합니다. 표시된 화면에서 **+ 컨테이너**를 선택합니다. 컨테이너에 이름을 `articles`로 지정하고 **익명 액세스 수준**을 **컨테이너(컨테이너 및 Blob에 대한 익명 읽기 권한)** 로 설정합니다.

    > **참고**: 실제 솔루션에 대한 스토리지 계정을 구성하는 경우 적절한 액세스 수준을 할당하도록 주의해야 합니다. 각 액세스 수준에 대해 자세히 알아보려면 [Azure Storage 설명서](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)를 참조하세요.

1. 컨테이너를 만든 후 이를 선택한 다음 **업로드** 단추를 선택합니다. **파일 찾아보기**를 선택하여 다운로드한 샘플 문서를 찾습니다. 그런 다음, **업로드**를 선택합니다.

## 사용자 지정 텍스트 분류 프로젝트 만들기

구성이 완료되면 사용자 지정 텍스트 분류 프로젝트를 만듭니다. 이 프로젝트는 모델을 빌드, 학습 및 배포할 수 있는 작업 위치를 제공합니다.

> **참고**: 이 랩은 **Language Studio**를 활용하지만 REST API를 통해 모델을 생성, 빌드, 학습 및 배포할 수도 있습니다.

1. 새 브라우저 탭에서 `https://language.cognitive.azure.com/`의 Azure AI 언어 스튜디오 포털을 열고 Azure 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1. 언어 리소스를 선택하라는 메시지가 표시되면 다음 설정을 선택합니다.

    - **Azure Directory**: 구독이 포함된 Azure Directory입니다.
    - **Azure 구독**: Azure 구독입니다.
    - **리소스 종류**: 언어.
    - **언어 리소스**: 이전에 만든 Azure AI 언어 리소스입니다.

    언어 리소스를 선택하라는 메시지가 표시되지 <u>않으면</u> 구독에 여러 언어 리소스가 있기 때문일 수 있습니다. 이 경우 다음을 수행합니다.

    1. 페이지 상단 표시줄에서 **설정(&#9881;)** 단추를 선택합니다.
    2. **설정** 페이지에서 **리소스** 탭을 봅니다.
    3. 방금 만든 언어 리소스를 선택하고 **리소스 전환**을 클릭합니다.
    4. 페이지 맨 위에서 **Language Studio**를 클릭하여 Language Studio 홈페이지로 돌아갑니다.

1. 포털 상단의 **새로 만들기** 메뉴에서 **사용자 지정 텍스트 분류**를 선택합니다.
1. **스토리지 연결** 페이지가 나타납니다. 모든 값이 이미 채워져 있습니다. 그러므로 **다음**을 선택합니다.
1. **프로젝트 유형 선택** 페이지에서 **단일 레이블 분류**를 선택합니다. 그런 후 **다음**을 선택합니다.
1. **기본 정보 입력** 창에서 다음을 설정합니다.
    - **이름**: `ClassifyLab`  
    - **텍스트 기본 언어**: 영어(미국)
    - **설명**: `Custom text lab`

1. **다음**을 선택합니다.
1. **컨테이너 선택** 페이지에서 **Blob 저장소 컨테이너** 드롭다운을 *아티클* 컨테이너로 설정합니다.
1. **아니요, 이 프로젝트의 일부로 파일에 레이블을 지정해야 합니다** 옵션을 선택합니다. 그런 후 **다음**을 선택합니다.
1. **프로젝트 만들기**를 선택합니다.

> **팁**: 이 작업을 수행할 권한이 없다는 오류가 발생하면 역할 할당을 추가해야 합니다. 이 문제를 해결하려면 랩을 실행하는 사용자의 스토리지 계정에 "Storage Blob 데이터 기여자" 역할을 추가합니다. 자세한 내용은 [설명서](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource)를 참조하세요.

## 데이터에 레이블 지정

이제 프로젝트를 만들었으므로 데이터에 레이블을 지정하여 텍스트 분류 방법을 모델에 학습시켜야 합니다.

1. 아직 선택하지 않은 경우 왼쪽에서 **데이터 레이블 지정**을 선택합니다. 스토리지 계정에 업로드한 파일 목록이 표시됩니다.
1. 오른쪽에 있는 **작업** 창에서 **+ 클래스 추가**를 선택합니다.  이 랩의 문서는 만들어야 하는 네 가지 클래스(`Classifieds`, `Sports`, `News` 및 `Entertainment`)로 분류됩니다.

    ![태그 데이터 페이지와 클래스 추가 단추를 보여 주는 스크린샷.](../media/tag-data-add-class-new.png#lightbox)

1. 네 개 클래스를 만든 후 **문서 1**을 선택하여 시작합니다. 여기에서 문서를 읽고, 이 파일이 어떤 클래스인지, 어떤 데이터 세트(학습 또는 테스트)에 할당할지 정의할 수 있습니다.
1. 오른쪽에 있는 **작업** 창에서 각 문서에 적절한 클래스와 데이터 세트(학습 또는 테스트)를 할당합니다.  오른쪽의 레이블 목록에서 레이블을 선택하고 작업 창 아래쪽의 옵션을 사용하여 각 문서를 **학습** 또는 **테스트**로 설정할 수 있습니다. **다음 문서**를 선택하여 그 다음 문서로 이동합니다. 이 랩의 목적을 위해 모델을 학습시키는 데 사용할 문서와 모델 테스트에 사용할 문서를 정합니다.

    | 아티클  | 클래스  | 데이터 세트  |
    |---------|---------|---------|
    | 문서 1 | 스포츠 | 학습 |
    | 문서 10 | 뉴스 | 학습 |
    | 문서 11 | Entertainment | 테스트 |
    | 문서 12 | 뉴스 | 테스트 |
    | 문서 13 | 스포츠 | 테스트 |
    | 문서 2 | 스포츠 | 학습 |
    | 문서 3 | 분류 | 학습 |
    | 문서 4 | 분류 | 학습 |
    | 문서 5 | Entertainment | 학습 |
    | 문서 6 | Entertainment | 학습 |
    | 문서 7 | 뉴스 | 학습 |
    | 문서 8 | 뉴스 | 학습 |
    | 문서 9 | Entertainment | 학습 |

    > **참고** Language Studio의 파일은 사전순으로 나열되므로 위의 목록이 순차적으로 나열되지 않습니다. 문서에 레이블을 붙일 때 문서의 두 페이지를 모두 방문했는지 확인합니다.

1. **레이블 저장**을 선택하여 레이블을 저장합니다.

## 모델 학습

데이터에 레이블을 지정한 후에는 모델을 학습시켜야 합니다.

1. 왼쪽 메뉴에서 **학습 작업**을 선택합니다.
1. **학습 작업 시작**을 선택합니다.
1. `ClassifyArticles`라는 새 모델을 학습합니다.
1. **학습 및 테스트 데이터의 수동 분할 사용**을 선택합니다.

    > **팁** 자체 분류 프로젝트에서는 Azure AI 언어 서비스가 테스트 집합을 백분율 기준으로 자동으로 분할하므로 대규모 데이터 세트에 유용합니다. 데이터 세트가 작을수록 올바른 클래스 분포를 사용하여 학습시키는 것이 중요합니다.

1. **학습**을 선택합니다.

> **중요** 모델을 학습하는 데 몇 분 정도 걸릴 수 있습니다. 완료되면 알림이 표시됩니다.

## 모델 평가

텍스트 분류의 실제 애플리케이션에서는 모델을 평가하고 개선하여 모델이 예상대로 작동하는지 확인하는 것이 중요합니다.

1. **모델 성능을** 선택하고 **ClassifyArticles** 모델을 선택합니다. 모델 점수 매기기, 성능 메트릭 및 학습시킨 시간을 볼 수 있습니다. 모델 점수가 100%가 아닌 경우 테스트에 사용된 문서 중 하나가 지정된 레이블로 평가되지 않았음을 의미합니다. 이러한 실패는 개선점을 파악하는 데 도움될 수 있습니다.
1. **테스트 집합 세부 정보** 탭을 선택합니다. 오류가 있는 경우 이 탭을 사용하면 테스트로 표시한 문서, 모델이 문서에 대해 예측한 것, 그리고 문서의 테스트 레이블과의 충돌 여부를 확인할 수 있습니다. 탭은 기본적으로 잘못된 예측만 표시합니다. **불일치만 표시** 옵션을 전환하면 테스트로 표시한 모든 문서와 함께 각 문서가 무엇으로 예측되는지 확인할 수 있습니다.

## 모델 배포

모델 학습에 만족하면 배포할 시간이므로 API를 통해 텍스트 분류를 시작할 수 있습니다.

1. 왼쪽 창에서 **모델 배포**를 선택합니다.
1. **배포 추가**를 선택한 다음, **새 배포 이름 만들기** 필드에 `articles`를 입력하고 **모델** 필드에서 **ClassifyArticles**를 선택합니다.
1. **배포**를 선택하여 모델을 배포합니다.
1. 모델이 배포되면 해당 페이지를 열어 둡니다. 다음 단계에서 프로젝트 및 배포 이름이 필요합니다.

## Visual Studio Code에서 앱 개발 준비

Azure AI 언어 서비스의 사용자 지정 텍스트 분류 기능을 테스트하기 위해 Visual Studio Code에서 간단한 콘솔 애플리케이션을 개발합니다.

> **팁**: **mslearn-ai-language** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-language` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **참고**: Visual Studio Code에서 열려는 코드를 신뢰하라는 팝업 메시지가 표시되면 팝업에서 **예, 작성자를 신뢰합니다.** 옵션을 클릭합니다.

4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션과 요약을 테스트하는 데 사용할 샘플 텍스트 파일이 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure AI 언어 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/04-text-classification** 폴더를 찾아 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장하고, 포함되어 있는 **classify-text** 폴더를 확장합니다. 각 폴더에는 Azure AI 언어 텍스트 분류 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
1. 코드 파일이 포함된 **classify-text** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 언어 Text Analytics SDK 패키지를 설치합니다.

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. **탐색기** 창의 **classify-text** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env
    
1. 만든 Azure 언어 리소스의 **엔드포인트** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal의 Azure AI 언어 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능). 파일에는 텍스트 분류 모델에 대한 프로젝트 및 배포 이름이 이미 포함되어 있어야 합니다.
1. 구성 파일을 저장합니다.

## 문서를 분류하는 코드 추가

이제 Azure AI 언어 서비스를 사용하여 문서를 분류할 준비가 되었습니다.

1. 애플리케이션이 분류할 텍스트 문서를 보려면 **classify-text** 폴더의 **articles** 폴더를 확장합니다.
1. **classify-text** 폴더에서 클라이언트 애플리케이션용 코드 파일을 엽니다.

    - **C#**: Program.cs
    - **Python**: classify-text.py

1. **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Text Analytics SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. **Main** 함수에는 Azure AI 언어 서비스 엔드포인트와 키를 로드하는 코드와 구성 파일의 프로젝트 및 배포 이름이 이미 제공되어 있습니다. 그런 후 **엔드포인트와 키를 사용하여 클라이언트 만들기** 주석을 찾아서 다음 코드를 추가하여 Text Analytics API용 클라이언트를 만듭니다.

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. **Main** 함수에서 기존 코드는 **articles** 폴더의 모든 파일을 읽고 해당 콘텐츠가 포함된 목록을 만듭니다. 그런 다음 **Get Classifications** 주석을 찾아 다음 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python**: classify-text.py

    ```Python
    # Get Classifications
    operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. 코드 파일에 변경 내용을 저장합니다.

## 애플리케이션 테스트

이제 애플리케이션을 테스트할 준비가 되었습니다.

1. **classify-text** 폴더의 통합 터미널에서 다음 명령을 입력하여 프로그램을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python classify-text.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

1. 출력을 확인합니다. 애플리케이션은 각 텍스트 파일에 대한 분류 및 신뢰도 점수를 나열해야 합니다.


## 정리

프로젝트가 더 이상 필요하지 않으면 Language Studio의 **프로젝트** 페이지에서 삭제할 수 있습니다. [Azure Portal](https://portal.azure.com)에서 Azure AI 언어 서비스 및 관련 스토리지 계정을 제거할 수도 있습니다.
