---
lab:
  title: 질문 답변 솔루션 만들기
  description: Azure AI 언어를 사용하여 사용자 지정 질문 답변 솔루션을 만듭니다.
---

# 질문 답변 솔루션 만들기

가장 흔히 진행되는 대화 시나리오 중 하나는 FAQ(질문과 대답) 기술 자료를 통한 지원 제공입니다. 대다수 조직은 FAQ를 문서나 웹 페이지로 게시합니다. 질문과 대답 쌍의 수가 적은 경우에는 이러한 방식에 문제가 없지만, 문서가 크면 검색이 어려우며 시간이 많이 걸릴 수 있습니다.

**Azure AI 언어**는 자연어 입력을 사용하여 쿼리할 수 있는 질문과 대답 기술 자료를 만들 수 있는 *질문 답변* 기능을 포함하며, 사용자가 제출한 질문의 대답을 조회하기 위해 봇이 사용할 수 있는 리소스로 가장 많이 사용됩니다. 이 연습에서는 텍스트 분석에 Azure AI 언어 Python SDK를 사용하여 간단한 질문 답변 애플리케이션을 구현합니다.

이 연습은 Python을 기반으로 하지만 여러 언어별 SDK를 사용하여 질문 답변 애플리케이션을 개발할 수 있습니다. 포함 사항:

- [Python용 Azure AI 언어 서비스 질문 답변 클라이언트 라이브러리](https://pypi.org/project/azure-ai-language-questionanswering/)
- [.NET용 Azure AI 언어 서비스 질문 답변 클라이언트 라이브러리](https://www.nuget.org/packages/Azure.AI.Language.QuestionAnswering)

이 연습에는 약 **20**분이 소요됩니다.

## *Azure AI 언어* 리소스를 프로비전합니다.

구독에 아직 없는 경우 **Azure AI 언어 서비스** 리소스를 프로비전해야 합니다. 또한 질문 답변을 위한 기술 자료를 만들고 호스팅하려면 **질문 답변** 기능을 사용하도록 설정해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. **리소스 만들기**를 선택합니다.
1. 검색 필드에서 **언어 서비스**를 검색합니다. 그런 다음 결과에서 **언어 서비스** 아래의 **만들기**를 선택합니다.
1. **사용자 지정 질문 답변** 블록을 선택합니다. **계속해서 리소스 만들기**를 선택합니다. 다음 설정을 입력해야 합니다.

    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: ** 사용 가능한 위치 선택
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **Azure Search 지역**: *언어 리소스와 같은 글로벌 지역에서 위치를 선택합니다.*
    - **Azure Search 가격 책정 계층**: 무료(F)(이 계층을 사용할 수 없으면 기본(B) 선택**)
    - **책임 있는 AI 알림**: 동의**

1. **만들기 + 검토**를 선택한 다음 **만들기**를 선택합니다.

    > **참고** 맞춤형 질의응답은 Azure Search를 사용해 질문과 대답 기술 자료를 인덱싱 및 쿼리합니다.

1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **리소스 관리** 섹션의 **키 및 엔드포인트**를 봅니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## 질문 답변 프로젝트 만들기

Azure AI 언어 리소스에서 질문 답변에 대한 기술 자료를 만들려면 Language Studio 포털을 사용하여 질문 답변 프로젝트를 만들 수 있습니다. 여기서는 [Microsoft Learn](https://docs.microsoft.com/learn) 관련 질문과 대답이 포함된 기술 자료를 만듭니다.

1. 새 브라우저 탭에서 [https://language.cognitive.azure.com/](https://language.cognitive.azure.com/)의 Language Studio 포털로 이동하고 Azure 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
1. 언어 리소스를 선택하라는 메시지가 표시되면 다음 설정을 선택합니다.
    - **Azure Directory**: 구독이 포함된 Azure Directory입니다.
    - **Azure 구독**: Azure 구독입니다.
    - **리소스 종류**: 언어
    - **리소스 이름**: 이전에 만든 Azure AI 언어 리소스입니다.

    언어 리소스를 선택하라는 메시지가 표시되지 <u>않으면</u> 구독에 여러 언어 리소스가 있기 때문일 수 있습니다. 이 경우 다음을 수행합니다.

    1. 페이지 상단 표시줄에서 **설정(&#9881;)** 단추를 선택합니다.
    2. **설정** 페이지에서 **리소스** 탭을 봅니다.
    3. 방금 만든 언어 리소스를 선택하고 **리소스 전환**을 클릭합니다.
    4. 페이지 맨 위에서 **Language Studio**를 클릭하여 Language Studio 홈페이지로 돌아갑니다.

1. 포털의 위쪽에 있는 **새로 만들기** 메뉴에서 **사용자 지정 질문 답변**을 선택합니다.
1. ***프로젝트 만들기** 마법사의 **언어 설정 선택** 페이지에서 **모든 프로젝트에 대한 언어 설정** 옵션을 선택하고 언어로 **영어**를 선택합니다. 그런 후 **다음**을 선택합니다.
1. **기본 정보 입력** 페이지에서 다음 세부 정보를 입력합니다.
    - **이름** `LearnFAQ`
    - **설명**: `FAQ for Microsoft Learn`
    - **답변이 반환되지 않는 경우의 기본 답변**: `Sorry, I don't understand the question`
1. **다음**을 선택합니다.
1. **검토 및 완료** 페이지에서 **프로젝트 만들기**를 선택합니다.

## 기술 자료에 원본 추가

기술 자료를 처음부터 만들어도 되지만 일반적으로는 기존 FAQ 페이지나 문서에서 질문과 대답을 가져오는 작업부터 시작합니다. 이 경우 Microsoft Learn에 대한 기존 FAQ 웹 페이지에서 데이터를 가져오고, 미리 정의된 “잡담” 질문과 대답을 가져와 일반적인 대화형 교환을 지원합니다.

1. 질문 답변 프로젝트의 **원본 관리** 페이지의 **&#9547; 원본 추가** 목록에서 **URL**을 선택합니다. 그런 다음 **URL 추가** 대화 상자에서 **&#9547; URL을 추가하고** 다음 이름과 URL을 설정한 후 **모두 추가**를 선택하여 기술 자료에 추가합니다.
    - **이름**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
1. 질문 답변 프로젝트의 **원본 관리** 페이지의 **&#9547; 원본 추가** 목록에서 **잡담**을 선택합니다. **잡담 추가** 대화 상자에서 **친절함**을 선택하고 **잡담 추가**를 선택합니다.

## 기술 자료 테스트

앞에서 만든 기술 자료에는 Microsoft Learn FAQ의 질문과 대답 쌍이 입력되었으며 대화형 *잡담* 질문과 대답 쌍 세트가 추가되었습니다. 질문과 대답 쌍을 더 추가하여 기술 자료를 확장할 수 있습니다.

1. Language Studio의 **LearnFAQ** 프로젝트에서 **기술 자료 편집** 페이지를 선택하여 기존 질문 및 답변 쌍을 확인합니다(일부 팁이 표시되는 경우 팁을 읽고 **확인**을 선택하여 해제하거나, **모두 건너뛰기** 선택).
1. 기술 자료의 **질문 답변 쌍** 탭에서 **&#65291;** 을 선택하고 다음 설정으로 새 질문 답변 쌍을 만듭니다.
    - **원본**:  `https://docs.microsoft.com/en-us/learn/support/faq`
    - **질문**: `What are Microsoft credentials?`
    - **응답**: `Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. **완료**를 선택합니다.
1. 만들어진 **Microsoft 자격 증명이란?** 질문 페이지에서 **대체 질문**을 확장합니다. 그런 다음 대체 질문 `How can I demonstrate my Microsoft technology skills?`를 추가합니다.

    사용자가 대답을 확인한 후 추가 작업으로 *멀티 턴* 대화를 작성하면 효율적인 경우도 있습니다. 그러면 사용자가 질문을 여러 번 구체화하여 필요한 대답을 확인할 수 있습니다.

1. 인증 질문에 입력한 답변 아래에서 **후속 프롬프트**를 확장하고 다음 후속 프롬프트를 추가합니다.
    - **사용자에게 프롬프트에 표시되는 텍스트**: `Learn more about credentials`.
    - **새 쌍에 대한 링크 만들기** 탭을 선택하고 다음 텍스트를 입력합니다. `You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).` 
    - **컨텍스트 흐름에만 표시**를 선택합니다. 이 옵션을 선택하면 원래 인증 관련 질문의 후속 질문 컨텍스트에서만 대답이 반환됩니다.
1. **프롬프트 추가**를 선택합니다.

## 기술 자료 학습 및 테스트

이제 기술 자료가 있으므로, Language Studio에서 이것을 테스트할 수 있습니다.

1. 왼쪽의 **질문 답변 쌍** 탭 아래에 있는 **저장** 단추를 선택하여 기술 자료의 변경 내용을 저장합니다.
1. 변경 내용이 저장되면 **테스트** 단추를 선택하여 테스트 창을 엽니다.
1. 테스트 창 상단에서 **단답형 답변 포함**을 선택 해제합니다(아직 선택 해제하지 않은 경우). 그런 다음 아래쪽에 `Hello`라는 메시지를 입력합니다. 적절한 응답이 반환되어야 합니다.
1. 테스트 창의 맨 아래에 `What is Microsoft Learn?` 메시지를 입력합니다. FAQ에서 적절한 응답이 반환되어야 합니다.
1. `Thanks!` 메시지를 입력합니다. 적절한 잡담 응답이 반환되어야 합니다.
1. 메시지 `Tell me about Microsoft credentials`를 입력합니다. 앞에서 작성한 대답이 반환되고 후속 프롬프트 링크가 표시되어야 합니다.
1. **사용자 인증 정보에 대해 자세히 알아보기** 후속 링크를 선택합니다. 인증 페이지 링크가 포함된 후속 질문의 대답이 반환되어야 합니다.
1. 기술 자료 테스트를 마쳤으면 테스트 창을 닫습니다.

## 기술 자료 배포

기술 자료는 클라이언트 애플리케이션이 질문의 대답을 찾는 데 사용할 수 있는 백 엔드 서비스를 제공합니다. 이제 기술 자료를 게시하고 클라이언트에서 기술 자료 REST 인터페이스에 액세스할 수 있습니다.

1. Language Studio의 **LearnFAQ** 프로젝트에서 왼쪽 탐색 메뉴에 있는 **기술 자료 배포** 페이지를 선택합니다.
1. 페이지 상단에서 **배포**를 선택합니다. 그런 다음 **배포**를 선택하여 기술 자료 배포를 확인합니다.
1. 배포가 완료되면 **예측 URL 가져오기**를 선택하여 기술 자료에 대한 REST 엔드포인트를 확인하고 샘플 요청에 다음에 대한 매개 변수가 포함되어 있는지 확인합니다.
    - **projectName**: 프로젝트 이름(*LearnFAQ*이어야 함)
    - **deploymentName**: 배포 이름(*production*이어야 함)
1. 예측 URL 대화 상자를 닫습니다.

## Cloud Shell 앱 개발 준비

Azure Portal에서 Cloud Shell을 사용하여 질문 답변 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

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
    cd mslearn-ai-language/Labfiles/02-qna/Python/qna-app
    ```

## 애플리케이션 사용

1. 명령줄 창에서 다음 명령을 실행하여 **qna-app** 폴더의 코드 파일을 봅니다.

    ```
   ls -a -l
    ```

    파일에는 구성 파일(**.env**) 및 코드 파일(**qna-app.py**)이 포함됩니다.

1. Python 가상 환경을 만들고 다음 명령을 실행하여 Azure AI 언어 질문 답변 SDK 패키지 및 기타 필수 패키지를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-questionanswering
    ```

1. 다음 명령을 입력하여 구성 파일을 편집합니다.

    ```
    code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 사용자가 만든 Azure 언어 리소스에 대한 **엔드포인트** 및 인증 **키**를 반영하도록 해당 파일에 포함된 구성 값을 업데이트합니다(Azure Portal의 Azure AI 언어 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능). 배포된 기술 자료의 프로젝트 이름과 배포 이름도 이 파일에 있어야 합니다.
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

## 기술 자료를 사용하기 위한 코드 추가

1. 다음 명령을 입력하여 애플리케이션 코드 파일을 편집합니다.

    ```
    code qna-app.py
    ```

1. 기존 코드를 검토합니다. 기술 자료를 사용하기 위한 코드를 추가합니다.

    > **팁**: 코드 파일에 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 코드 파일에서 **Import namespaces** 주석을 찾습니다. 그런 다음, 이 주석 아래에 다음 언어별 코드를 추가하여 질문 답변 SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. **main** 함수에 Azure AI 언어 서비스 엔드포인트와 구성 파일의 키를 로드하는 코드가 이미 제공되어 있습니다. 그런 후 **Create client using endpoint and key** 주석을 찾고 다음 코드를 추가하여 질문 답변 클라이언트를 만듭니다.

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 코드 파일에서 **Submit a question and display the answer** 주석을 찾고, 다음 코드를 추가하여 명령줄에서 질문을 반복적으로 읽고, 서비스에 제출하고, 답변의 세부 정보를 표시합니다.

    ```Python
   # Submit a question and display the answer
   user_question = ''
   while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. 변경 내용을 저장하고(Ctrl+S) 다음 명령을 입력하여 프로그램을 실행합니다(Cloud Shell 창을 최대화하고 명령줄 창에 더 많은 텍스트를 표시하도록 패널 크기 조정).

    ```
   python qna-app.py
    ```

1. 메시지가 나타나면 질문 답변 프로젝트에 제출할 질문을 입력합니다. 예를 들어, `What is a learning path?`입니다.
1. 반환된 답변을 검토합니다.
1. 더 많은 질문을 해보세요. 완료되면 `quit`를 입력합니다.

## 리소스 정리

Azure AI 언어 서비스 탐색을 마쳤으면 이 연습에서 만든 리소스를 삭제할 수 있습니다. 이 경우 가능한 방법은 다음과 같습니다.

1. Azure Cloud Shell 창 닫기
1. Azure Portal에서 이 랩에서 만든 Azure AI 언어 리소스를 찾습니다.
1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

Azure AI 언어의 질문 답변에 대해 자세히 알아보려면 [Azure AI 언어 설명서](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview)를 참조하세요.
