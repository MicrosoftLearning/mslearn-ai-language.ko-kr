---
lab:
  title: 텍스트 분석
  description: 'Azure AI 언어를 사용하여 언어 감지, 감정 분석, 핵심 구 추출 및 엔터티 인식을 포함한 텍스트 분석을 지원합니다.'
---

# 텍스트 분석

**Azure AI 언어**는 언어 인식, 감정 분석, 핵심 구 추출 및 엔터티 인식을 포함한 텍스트 분석을 지원합니다.

회사 웹 사이트로 제출된 호텔 리뷰를 처리하려는 여행사의 경우를 예로 들어 보겠습니다. 이 여행사는 Azure AI 언어를 사용하여 각 리뷰를 작성한 언어, 리뷰의 감정(긍정적, 중립, 부정적), 리뷰에 설명되어 있는 주요 토픽을 나타낼 수 있는 핵심 구, 그리고 리뷰에 언급되어 있는 장소, 주요 건물, 사람 등의 명명된 엔터티를 확인할 수 있습니다. 이 연습에서는 텍스트 분석에 Azure AI 언어 Python SDK를 사용하여 이 예제를 기준으로 간단한 호텔 리뷰 애플리케이션을 구현합니다.

이 연습은 Python을 기준으로 하지만 여러 언어별 SDK를 사용하여 텍스트 분석 애플리케이션을 개발할 수 있습니다. 포함된 내용은 다음과 같습니다.

- [Python용 Azure AI Text Analytics 클라이언트 라이브러리 사용](https://pypi.org/project/azure-ai-textanalytics/)
- [.NET용 Azure AI Text Analytics 클라이언트 라이브러리 사용](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [JavaScript용 Azure AI Text Analytics 클라이언트 라이브러리 사용](https://www.npmjs.com/package/@azure/ai-text-analytics)

이 연습에는 약 **30**분이 소요됩니다.

## *Azure AI 언어* 리소스를 프로비전합니다.

구독에 아직 리소스가 없는 경우 Azure 구독에 **Azure AI 언어 서비스** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. **리소스 만들기**를 선택합니다.
1. 검색 필드에서 **언어 서비스**를 검색합니다. 그런 다음 결과에서 **언어 서비스** 아래의 **만들기**를 선택합니다.
1. **계속하여 리소스 만들기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 프로비전합니다.
    - **구독**: *자신의 Azure 구독*.
    - **리소스 그룹**: *리소스 그룹을 선택하거나 만듭니다*.
    - **지역**: *사용 가능한 지역을 선택합니다*.
    - **이름**: 고유한 이름을 입력합니다.
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **담당 AI 알림**: 동의합니다.
1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **리소스 관리** 섹션의 **키 및 엔드포인트**를 봅니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## 이 과정용 리포지토리 복제

Azure Portal에서 Cloud Shell을 사용하여 코드를 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

1. Azure Portal에서 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 버튼으로 Azure Portal에서 새 Cloud Shell을 생성하고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
    rm -r mslearn-ai-language -f
    git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **팁**: CloudShell에 명령을 입력하면 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    ```
    cd mslearn-ai-language/Labfiles/01-analyze-text/Python/text-analysis
    ```

## 애플리케이션 사용

1. 명령줄 창에서 다음 명령을 실행하여 **text-analysis** 폴더의 코드 파일을 봅니다.

    ```
   ls -a -l
    ```

    파일에는 구성 파일(**.env**) 및 코드 파일(**text-analysis.py**)이 포함됩니다. 애플리케이션에서 분석할 텍스트는 **reviews** 하위 폴더에 있습니다.

1. Python 가상 환경을 만들고 다음 명령을 실행하여 Azure AI 언어 Text Analytics SDK 패키지 및 기타 필수 패키지를 설치합니다.

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. 다음 명령을 입력하여 애플리케이션 구성 파일을 편집합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 만든 Azure 언어 리소스의 **엔드포인트** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal의 Azure AI 언어 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능).
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

## Azure AI 언어 리소스에 연결하기 위한 코드 추가

1. 다음 명령을 입력하여 애플리케이션 코드 파일을 편집합니다.

    ```
    code text-analysis.py
    ```

1. 기존 코드를 검토합니다. AI 언어 Text Analytics SDK를 사용하기 위한 코드를 추가합니다.

    > **팁**: 코드 파일에 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 코드 파일의 맨 위에 있는 기존 네임스페이스 참조에서 **Import namespaces** 주석을 찾고 다음 코드를 추가하여 Text Analytics SDK를 사용해야 하는 네임스페이스를 가져옵니다.

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. **main** 함수에 Azure AI 언어 서비스 엔드포인트와 구성 파일의 키를 로드하는 코드가 이미 제공되어 있습니다. 그런 후 **엔드포인트와 키를 사용하여 클라이언트 만들기** 주석을 찾아서 다음 코드를 추가하여 Text Analytics API용 클라이언트를 만듭니다.

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 변경 내용을 저장하고(Ctrl+S) 다음 명령을 입력하여 프로그램을 실행합니다(Cloud Shell 창을 최대화하고 명령줄 창에 더 많은 텍스트를 표시하도록 패널 크기 조정).

    ```
   python text-analysis.py
    ```

1. 오류가 발생하지 않고 코드가 실행되어 **reviews** 폴더에 있는 각 리뷰 텍스트 파일의 내용이 표시되는지 확인합니다. 애플리케이션은 Text Analytics API용 클라이언트를 만들었지만 해당 클라이언트를 사용하지는 않습니다. 다음 섹션에서 이 문제를 해결합니다.

## 언어를 검색하는 코드 추가

이제 API용 클라이언트를 만들었으므로 이를 사용하여 각 검토가 작성되는 언어를 검색해 보겠습니다.

1. 코드 편집기에서 **Get language** 주석을 찾습니다. 그런 다음, 각 리뷰 문서의 언어를 감지하는 데 필요한 코드를 추가합니다.

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **참고**: *이 예제에서는 각 리뷰를 개별적으로 분석하므로 각 파일 분석을 위해 서비스를 각기 별도로 호출합니다. 이 방식 대신 문서 컬렉션을 만든 후 단일 호출에서 서비스에 컬렉션을 전달하는 방식을 사용할 수 있습니다. 두 방식에서 모두 서비스의 응답은 문서 컬렉션으로 구성되어 있습니다. 따라서 위 Python 코드의 응답에는 첫 번째(유일한) 문서의 인덱스([0])만 지정되어 있습니다.*

1. 변경 내용을 저장합니다. 그런 다음, 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 이번에는 각 리뷰의 언어가 식별되었습니다.

## 감정을 평가하는 코드 추가

감정 분석은 텍스트를 긍정적 또는 부정적으로 분류할 때 흔히 사용되는 기술입니다(중립적 또는 혼합으로 분류할 수도 있음).********** 감정 분석은 소셜 미디어 게시물, 제품 리뷰, 그리고 텍스트 감정이 유용한 인사이트를 제공할 수 있는 기타 항목을 분석하는 데 흔히 사용됩니다.

1. 코드 편집기에서 **Get sentiment** 주석을 찾습니다. 그런 다음, 각 리뷰 문서의 감정을 감지하는 데 필요한 코드를 추가합니다.

    ```python
   # Get sentiment
   sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
   print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. 변경 내용을 저장합니다. 그런 다음, 코드 편집기를 닫고 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 리뷰의 감정이 감지되었습니다.

## 핵심 구를 식별하는 코드 추가

텍스트 본문에서 핵심 구를 식별하면 텍스트에 설명되어 있는 주요 토픽을 확인하는 데 유용할 수 있습니다.

1. 코드 편집기에서 **Get key phrases** 주석을 찾습니다. 그런 다음, 각 리뷰 문서의 핵심 구를 감지하는 데 필요한 코드를 추가합니다.

    ```python
   # Get key phrases
   phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
   if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. 변경 내용을 저장하고 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 각 문서에는 리뷰 내용과 관련한 몇 가지 인사이트를 제공하는 핵심 구가 포함되어 있습니다.

## 엔터티를 추출하는 코드 추가

문서나 기타 텍스트 본문에서는 사람, 장소, 기간 또는 기타 엔터티를 언급하는 경우가 많습니다. Text Analytics API는 텍스트 내 엔터티의 여러 범주(및 하위 범주)를 감지할 수 있습니다.

1. 코드 편집기에서 **Get entities** 주석을 찾습니다. 그런 다음, 각 리뷰에 언급되어 있는 엔터티를 식별하는 데 필요한 코드를 추가합니다.

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. 변경 내용을 저장하고 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 텍스트에서 엔터티가 감지되었습니다.

## 연결된 엔터티를 추출하는 코드 추가

Text Analytics API는 범주화된 엔터티뿐 아니라 Wikipedia 등의 데이터 원본에 대한 알려진 링크가 있는 엔터티도 감지할 수 있습니다.

1. 코드 편집기에서 **Get linked entities** 주석을 찾습니다. 그런 다음, 각 리뷰에 언급되어 있는 연결된 엔터티를 식별하는 데 필요한 코드를 추가합니다.

    ```python
   # Get linked entities
   entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. 변경 내용을 저장하고 프로그램을 다시 실행합니다.
1. 출력을 확인합니다. 연결된 엔터티가 식별되었습니다.

## 리소스 정리

Azure AI 언어 서비스 탐색을 마쳤으면 이 연습에서 만든 리소스를 삭제할 수 있습니다. 이 경우 가능한 방법은 다음과 같습니다.

1. Azure Cloud Shell 창 닫기
1. Azure Portal에서 이 랩에서 만든 Azure AI 언어 리소스를 찾습니다.
1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

**Azure AI 언어** 사용에 대한 자세한 내용은 [설명서](https://learn.microsoft.com/azure/ai-services/language-service/)를 참조하세요.
