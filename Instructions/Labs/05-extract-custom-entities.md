---
lab:
  title: 사용자 지정 엔터티 추출
  description: Azure AI 언어를 사용하여 텍스트 입력에서 사용자 지정된 엔터티를 추출하도록 모델을 학습시킵니다.
---

# 사용자 지정 엔터티 추출

다른 자연어 처리 기능 외에도 Azure AI 언어 서비스를 사용하면 사용자 지정 엔터티를 정의하고 텍스트에서 해당 엔터티의 인스턴스를 추출할 수 있습니다.

사용자 지정 엔터티 추출을 테스트하기 위해 모델을 만들고 Azure AI 언어 스튜디오를 통해 학습시킨 다음, Python 애플리케이션을 사용하여 테스트합니다.

이 연습은 Python을 기준으로 하지만 여러 언어별 SDK를 사용하여 텍스트 분류 애플리케이션을 개발할 수 있습니다. 포함된 내용은 다음과 같습니다.

- [Python용 Azure AI Text Analytics 클라이언트 라이브러리 사용](https://pypi.org/project/azure-ai-textanalytics/)
- [.NET용 Azure AI Text Analytics 클라이언트 라이브러리 사용](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [JavaScript용 Azure AI Text Analytics 클라이언트 라이브러리 사용](https://www.npmjs.com/package/@azure/ai-text-analytics)

이 연습에는 약 **35**분이 소요됩니다.

## *Azure AI 언어* 리소스를 프로비전합니다.

구독에 아직 없는 경우 **Azure AI 언어 서비스** 리소스를 프로비전해야 합니다. 또한 사용자 지정 텍스트 분류를 사용하려면 **사용자 지정 텍스트 분류 및 추출** 기능을 사용하도록 설정해야 합니다.

1. 브라우저의 `https://portal.azure.com`에서 Azure Portal을 열고 Microsoft 계정에 로그인합니다.
1. **리소스 만들기** 단추를 선택하고, *언어*를 검색하고, **언어 서비스** 리소스를 만듭니다. *추가 기능 선택* 페이지에서 **사용자 지정 명명된 엔터티 인식 추출**이 포함된 사용자 지정 기능을 선택합니다. 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: *다음 지역 중 하나를 선택합니다.*\*
        - 오스트레일리아 동부
        - 인도 중부
        - 미국 동부
        - 미국 동부 2
        - 북유럽
        - 미국 중남부
        - 스위스 북부
        - 영국 남부
        - 서유럽
        - 미국 서부 2
        - 미국 서부 3
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **스토리지 계정**: 새 스토리지 계정:
      - **스토리지 계정 이름**: 고유한 이름을 입력합니다.
      - **스토리지 계정 유형**: 표준 LRS
    - **책임 있는 AI 알림**: 선택됨.

1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## 사용자에 대한 역할 기반 액세스 구성

> **참고:** 이 단계를 건너뛰면 사용자 지정 프로젝트에 연결하려고 할 때 403 오류가 발생합니다. 현재 사용자는 스토리지 계정의 소유자인 경우에도 스토리지 계정 Blob 데이터에 액세스하기 위해 이 역할을 맡는 것이 중요합니다.

1. Azure Portal에서 스토리지 계정 페이지로 이동합니다.
2. 왼쪽 탐색 메뉴에서 **액세스 제어(IAM)** 를 선택합니다.
3. **추가**를 선택하여 역할 할당을 추가하고, 스토리지 계정에 대한 **Storage Blob 데이터 기여자** 역할을 선택합니다.
4. **다음에 대한 액세스 권한 할당** 내에서 **사용자, 그룹 또는 서비스 주체**를 선택합니다.
5. **멤버 선택**을 선택합니다.
6. 사용자를 선택합니다. **선택** 필드에서 사용자 이름을 검색할 수 있습니다.

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

> **팁**: 이 작업을 수행할 권한이 없다는 오류가 발생하면 역할 할당을 추가해야 합니다. 이 문제를 해결하려면 랩을 실행하는 사용자의 스토리지 계정에 "Storage Blob 데이터 기여자" 역할을 추가합니다. 자세한 내용은 [설명서](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource)를 참조하세요.

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
1. **작업** 창에서 이 문서가 모델 학습을 위한 데이터 세트에 추가된다는 점에 유의합니다.
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

## Cloud Shell 앱 개발 준비

Azure AI 언어 서비스의 사용자 지정 엔터티 추출 기능을 테스트하기 위해 Azure Cloud Shell에서 간단한 콘솔 애플리케이션을 개발합니다.

1. Azure Portal에서 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 버튼으로 Azure Portal에서 새 Cloud Shell을 생성하고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.
    ```

1. After the repo has been cloned, navigate to the folder containing the application code files:  

    ```
    cd mslearn-ai-language/Labfiles/05-custom-entity-recognition/Python/custom-entities
    ```

## Configure your application

1. In the command line pane, run the following command to view the code files in the **custom-entities** folder:

    ```
   ls -a -l
    ```

    The files include a configuration file (**.env**) and a code file (**custom-entities.py**). The text your application will analyze is in the **ads** subfolder.

1. Create a Python virtual environment and install the Azure AI Language Text Analytics SDK package and other required packages by running the following command:

    ```
   python -m venv labenv ./labenv/bin/Activate.ps1 pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Enter the following command to edit the application configuration file:

    ```
   code .env
    ```

    The file is opened in a code editor.

1. Update the configuration values to include the  **endpoint** and a **key** from the Azure Language resource you created (available on the **Keys and Endpoint** page for your Azure AI Language resource in the Azure portal).The file should already contain the project and deployment names for your custom entity extraction model.
1. After you've replaced the placeholders, within the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

## Add code to extract entities

1. Enter the following command to edit the application code file:

    ```
    code custom-entities.py
    ```

1. Review the existing code. You will add code to work with the AI Language Text Analytics SDK.

    > **Tip**: As you add code to the code file, be sure to maintain the correct indentation.

1. At the top of the code file, under the existing namespace references, find the comment **Import namespaces** and add the following code to import the namespaces you will need to use the Text Analytics SDK:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. **main** 함수에는 Azure AI 언어 서비스 엔드포인트와 키를 로드하는 코드와 구성 파일의 프로젝트 및 배포 이름이 이미 제공되어 있습니다. 그런 후 **Create client using endpoint and key** 주석을 찾고 다음 코드를 추가하여 Text Analytics 클라이언트를 만듭니다.

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 기존 코드는 **ads** 폴더의 모든 파일을 읽고 해당 콘텐츠가 포함된 목록을 만듭니다. 그런 후 **Extract entities** 주석을 찾아 다음 코드를 추가합니다.

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

1. 변경 내용을 저장하고(Ctrl+S) 다음 명령을 입력하여 프로그램을 실행합니다(Cloud Shell 창을 최대화하고 명령줄 창에 더 많은 텍스트를 표시하도록 패널 크기 조정).

    ```
   python custom-entities.py
    ```

1. 출력을 확인합니다. 애플리케이션은 각 텍스트 파일에 있는 엔터티 세부 정보를 나열해야 합니다.

## 정리

프로젝트가 더 이상 필요하지 않으면 Language Studio의 **프로젝트** 페이지에서 삭제할 수 있습니다. [Azure Portal](https://portal.azure.com)에서 Azure AI 언어 서비스 및 관련 스토리지 계정을 제거할 수도 있습니다.
