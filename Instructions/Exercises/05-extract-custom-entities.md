---
lab:
  title: 사용자 지정 엔터티 추출
  module: Module 3 - Getting Started with Natural Language Processing
---

# 사용자 지정 엔터티 추출

다른 자연어 처리 기능 외에도 Azure AI 언어 서비스를 사용하면 사용자 지정 엔터티를 정의하고 텍스트에서 해당 엔터티의 인스턴스를 추출할 수 있습니다.

사용자 지정 엔터티 추출을 테스트하기 위해 모델을 만들고 Azure AI 언어 스튜디오를 통해 학습시킨 다음 명령줄 애플리케이션을 사용하여 테스트합니다.

## *Azure AI 언어* 리소스를 프로비전합니다.

구독에 아직 없는 경우 **Azure AI 언어 서비스** 리소스를 프로비전해야 합니다. 또한 사용자 지정 텍스트 분류를 사용하려면 **사용자 지정 텍스트 분류 및 추출** 기능을 사용하도록 설정해야 합니다.

1. 브라우저의 `https://portal.azure.com`에서 Azure Portal을 열고 Microsoft 계정에 로그인합니다.
1. **리소스 만들기** 단추를 선택하고, *언어*를 검색하고, **언어 서비스** 리소스를 만듭니다. *추가 기능 선택* 페이지에서 **사용자 지정 명명된 엔터티 인식 추출**이 포함된 사용자 지정 기능을 선택합니다. 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: *사용 가능한 지역 선택*
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **스토리지 계정**: 새 스토리지 계정:
      - **스토리지 계정 이름**: 고유한 이름을 입력합니다.
      - **스토리지 계정 유형**: 표준 LRS
    - **책임 있는 AI 알림**: 선택됨.

1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## 샘플 광고 업로드

Azure AI 언어 서비스 및 스토리지 계정을 만든 후에는 나중에 모델을 학습하기 위해 예 광고를 업로드해야 합니다.

1. 새 브라우저 탭에서 `https://aka.ms/entity-extraction-ads`의 샘플 분류 광고를 다운로드하고 원하는 폴더에 파일을 추출합니다.

2. Azure Portal에서 생성된 스토리지 계정으로 이동하고 선택합니다.

3. 스토리지 계정에서 **설정** 아래에 있는 **구성**을 선택하고 화면에서 **Blob 익명 액세스 허용** 옵션을 사용하도록 설정한 다음 **저장**을 선택합니다.

4. **데이터 스토리지** 아래에 있는 왼쪽 메뉴에서 **컨테이너**를 선택합니다. 표시된 화면에서 **+ 컨테이너**를 선택합니다. 컨테이너에 이름을 `classifieds`로 지정하고 **익명 액세스 수준**을 **컨테이너(컨테이너 및 Blob에 대한 익명 읽기 권한)** 로 설정합니다.

    > **참고**: 실제 솔루션에 대한 스토리지 계정을 구성하는 경우 적절한 액세스 수준을 할당하도록 주의해야 합니다. 각 액세스 수준에 대해 자세히 알아보려면 [Azure Storage 설명서](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)를 참조하세요.

5. 컨테이너를 만든 후 이를 선택하고 **업로드** 단추를 클릭한 후 다운로드한 샘플 광고를 업로드합니다.

## 사용자 지정 명명된 엔터티 인식 프로젝트 만들기

이제 사용자 지정 명명된 엔터티 인식 프로젝트를 만들 준비가 되었습니다. 이 프로젝트는 모델을 빌드, 학습 및 배포할 수 있는 작업 위치를 제공합니다.

> **참고**: REST API를 통해 모델을 만들고, 빌드하고, 학습시키고, 배포할 수도 있습니다.

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

1. 포털 상단의 **새로 만들기** 메뉴에서 **사용자 지정 명명된 엔터티 인식**을 선택합니다.

1. 다음 설정을 사용하여 새 프로젝트를 만듭니다.
    - **스토리지 연결**: *이 값은 이미 입력된 것 같습니다. 아직 스토리지 계정이 아닌 경우 스토리지 계정으로 변경합니다.*
    - **기본 정보:**
    - **이름**: `CustomEntityLab`
        - **텍스트 기본 언어**: 영어(미국)
        - **데이터 세트에 동일한 언어로 작성되지 않은 문서가 포함되어 있나요?** : ‘아니요’**
        - **설명**: `Custom entities in classified ads`
    - **컨테이너**:
        - **Blob 저장소 컨테이너**: 분류 항목
        - **파일 레이블이 클래스로 지정되어 있지 않은가요?**: 네, 파일 레이블을 이 프로젝트의 일부로 지정해야 합니다.

## 데이터에 레이블 지정

이제 프로젝트를 만들었으므로 데이터에 레이블을 지정하여 엔터티 식별 방법을 모델에 학습시켜야 합니다.

1. **데이터 레이블 지정** 페이지가 아직 열려 있지 않으면 왼쪽 창에서 **데이터 레이블 지정**을 선택합니다. 스토리지 계정에 업로드한 파일 목록이 표시됩니다.
1. 오른쪽의 **작업** 창에서 **엔터티 추가**를 선택하고 `ItemForSale`라는 새 엔터티를 추가합니다.
1.  이전 단계를 반복하여 다음 엔터티를 만듭니다.
    - `Price`
    - `Location`
1. 세 개의 항목을 만든 후 읽을 수 있도록 **Ad 1.txt**를 선택합니다.
1. *Ad 1.txt*에서: 
    1. *face cord of firewood* 텍스트를 강조 표시하고 **ItemForSale** 엔터티를 선택합니다.
    1. *Denver, CO* 텍스트를 강조 표시하고 **Location** 엔터티를 선택합니다.
    1. *$90* 텍스트를 강조 표시하고 **Price** 엔터티를 선택합니다.
1. **작업** 창에서 이 문서가 모델 학습을 위한 데이터 세트에 추가된다는 점에 유의해야 합니다.
1. **다음 문서** 단추를 사용하여 다음 문서로 이동하고, 전체 문서 집합의 적절한 항목에 텍스트를 계속 할당하여 학습 데이터 세트에 모두 추가합니다.
1. 마지막 문서(*Ad 9.txt*)에 레이블을 지정했으면 레이블을 저장합니다.

## 모델 학습

데이터에 레이블을 지정한 후에는 모델을 학습시켜야 합니다.

1. 왼쪽 창에서 **학습 작업**을 선택합니다.
2. **학습 작업 시작** 선택
3. `ExtractAds`라는 새 모델을 학습시킵니다.
4. **학습 데이터에서 테스트 집합 자동 분할** 선택

    > **팁**: 사용자의 추출 프로젝트에서 데이터에 가장 적합한 테스트 분할을 사용합니다. 보다 일관된 데이터와 더 큰 데이터 세트를 위해 Azure AI 언어 서비스는 자동으로 테스트 집합을 백분율로 분할합니다. 데이터 세트가 작은 경우, 가능한 다양한 입력 문서를 사용하여 학습시키는 것이 중요합니다.

5. **학습**을 클릭합니다.

    > **중요**: 모델을 학습시키는 데 몇 분 정도 걸릴 수 있습니다. 완료되면 알림이 표시됩니다.

## 모델 평가

실제 애플리케이션에서는 모델을 평가하고 개선하여 모델이 예상대로 작동하는지 확인하는 것이 중요합니다. 왼쪽의 두 페이지에는 학습된 모델의 세부 정보와 실패한 테스트가 표시됩니다.

왼쪽 메뉴에서 **모델 성능**을 선택하고 `ExtractAds` 모델을 선택합니다. 모델 점수 매기기, 성능 메트릭 및 학습시킨 시간을 볼 수 있습니다. 테스트 문서가 실패했는지 확인할 수 있으며 이러한 오류를 통해 개선해야 할 부분을 파악할 수 있습니다.

## 모델 배포

모델 학습에 만족하는 경우 배포하여 API를 통해 엔터티 추출을 시작할 수 있습니다.

1. 왼쪽 창에서 **모델 배포**를 선택합니다.
2. **배포 추가**를 선택한 다음 이름 `AdEntities`를 입력하고 **ExtractAds** 모델을 선택합니다.
3. **배포**를 클릭하여 모델을 배포합니다.

## Visual Studio Code에서 앱 개발 준비

Azure AI 언어 서비스의 사용자 지정 엔터티 추출 기능을 테스트하기 위해 Visual Studio Code에서 간단한 콘솔 애플리케이션을 개발합니다.

> **팁**: **mslearn-ai-language** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-language` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **참고**: Visual Studio Code에서 열려는 코드를 신뢰하라는 팝업 메시지가 표시되면 팝업에서 **예, 작성자를 신뢰합니다.** 옵션을 클릭합니다.

4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure AI 언어 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/05-custom-entity-recognition** 폴더를 찾아 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장하고, 포함되어 있는 **custom-entities** 폴더를 확장합니다. 각 폴더에는 Azure AI 언어 텍스트 분류 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
1. 코드 파일이 포함된 **custom-entities** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 언어 Text Analytics SDK 패키지를 설치합니다.

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. **탐색기** 창의 **custom-entities** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env
    
1. 만든 Azure 언어 리소스의 **엔드포인트** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal의 Azure AI 언어 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능). 파일에는 사용자 지정 엔터티 추출 모델에 대한 프로젝트 및 배포 이름이 이미 포함되어 있어야 합니다.
1. 구성 파일을 저장합니다.

## 엔터티를 추출하는 코드 추가

이제 Azure AI 언어 서비스를 사용하여 텍스트에서 사용자 지정 엔터티를 추출할 준비가 되었습니다.

1. 애플리케이션에서 분석할 분류된 광고를 보려면 **custom-entities** 폴더의 **ads** 폴더를 확장합니다.
1. **custom-entities** 폴더에서 클라이언트 애플리케이션용 코드 파일을 엽니다.

    - **C#**: Program.cs
    - **Python**: custom-entities.py

1. **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Text Analytics SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. **Main** 함수에는 Azure AI 언어 서비스 엔드포인트와 키를 로드하는 코드와 구성 파일의 프로젝트 및 배포 이름이 이미 제공되어 있습니다. 그런 후 **엔드포인트와 키를 사용하여 클라이언트 만들기** 주석을 찾아서 다음 코드를 추가하여 Text Analytics API용 클라이언트를 만듭니다.

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python**: custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. **Main** 함수에서 기존 코드는 **ads** 폴더의 모든 파일을 읽고 해당 콘텐츠가 포함된 목록을 만듭니다. C# 코드의 경우 파일 이름을 ID와 언어로 포함하기 위해 **TextDocumentInput** 개체 목록을 사용합니다. Python에서는 텍스트 콘텐츠의 간단한 목록이 사용됩니다.
1. **엔터티 추출** 주석을 찾아 다음 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python**: custom-entities.py

    ```Python
    # Extract entities
    operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. 코드 파일에 변경 내용을 저장합니다.

## 애플리케이션 테스트

이제 애플리케이션을 테스트할 준비가 되었습니다.

1. **classify-text** 폴더의 통합 터미널에서 다음 명령을 입력하여 프로그램을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python custom-entities.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

1. 출력을 확인합니다. 애플리케이션은 각 텍스트 파일에 있는 엔터티 세부 정보를 나열해야 합니다.

## 정리

프로젝트가 더 이상 필요하지 않으면 Language Studio의 **프로젝트** 페이지에서 삭제할 수 있습니다. [Azure Portal](https://portal.azure.com)에서 Azure AI 언어 서비스 및 관련 스토리지 계정을 제거할 수도 있습니다.
