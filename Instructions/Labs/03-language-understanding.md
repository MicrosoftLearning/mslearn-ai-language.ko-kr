---
lab:
  title: Azure AI 언어 서비스를 사용하여 언어 이해 모델 만들기
  description: 입력을 해석하고 의도를 예측하고 엔터티를 식별하는 사용자 지정 언어 이해 모델을 만듭니다.
---

# 언어 서비스를 사용하여 언어 이해 모델 만들기

Azure AI 언어 서비스를 사용하면 애플리케이션이 사용자의 자연어 *발화*(텍스트 또는 음성 입력)를 해석하고, 사용자 *의도*(달성하려는 것)를 예측하고, 의도를 적용해야 하는 *엔터티*를 식별하는 데 사용할 수 있는 *대화 언어 이해* 모델을 정의할 수 있습니다.

예를 들어, 시계 애플리케이션용 대화 언어 모델은 다음과 같은 입력을 처리해야 할 수 있습니다.

*What is the time in London?*

이러한 입력 유형의 예가 발화(사용자가 말하거나 입력하는 내용)입니다. 이 발화에서 원하는 의도는 특정 위치(엔터티), 즉 여기서는 런던의 시간을 확인하는 것입니다.******

> **참고** 대화 언어 모델의 작업은 사용자 의도를 예측하고 해당 의도가 적용되는 엔터티를 식별하는 것입니다. 의도를 충족하는 데 필요한 작업을 실제로 수행하는 것은 대화 언어 모델의 작업이 <u>아닙니다</u>. 예를 들어, 시계 애플리케이션은 대화 언어 모델을 사용하여 사용자가 런던의 시간을 확인하려 한다는 것을 파악할 수는 있습니다. 그러나 이처럼 의도가 파악되고 나면 클라이언트 애플리케이션 자체가 논리를 구현해 정확한 시간을 확인하여 사용자에게 표시해야 합니다.

이 연습에서는 Azure AI 언어 서비스를 사용하여 대화형 언어 이해 모델을 만들고 Python SDK를 사용하여 해당 모델을 사용하는 클라이언트 앱을 구현합니다.

이 연습은 Python을 기준으로 하지만 여러 언어별 SDK를 사용하여 대화 이해 애플리케이션을 개발할 수 있습니다. 포함 사항:

- [Python용 Azure AI 대화 클라이언트 라이브러리](https://pypi.org/project/azure-ai-language-conversations/)
- [.NET용 Azure AI 대화 클라이언트 라이브러리](https://www.nuget.org/packages/Azure.AI.Language.Conversations)
- [JavaScript용 Azure AI 대화 클라이언트 라이브러리](https://www.npmjs.com/package/@azure/ai-language-conversations)

이 연습에는 약 **35**분이 소요됩니다.

## *Azure AI 언어* 리소스를 프로비전합니다.

구독에 아직 리소스가 없는 경우 Azure 구독에 **Azure AI 언어 서비스** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. **리소스 만들기**를 선택합니다.
1. 검색 필드에서 **언어 서비스**를 검색합니다. 그런 다음 결과에서 **언어 서비스** 아래의 **만들기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 프로비전합니다.
    - **구독**: *자신의 Azure 구독*.
    - **리소스 그룹**: *리소스 그룹을 선택하거나 만듭니다*.
    - **지역**: *다음 지역 중 하나를 선택합니다.*\*
        - 오스트레일리아 동부
        - 인도 중부
        - 중국 동부 2
        - 미국 동부
        - 미국 동부 2
        - 북유럽
        - 미국 중남부
        - 스위스 북부
        - 영국 남부
        - 서유럽
        - 미국 서부 2
        - 미국 서부 3
    - **이름**: *고유한 이름을 입력*합니다.
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **담당 AI 알림**: 동의합니다.
1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## 대화 언어 이해 프로젝트 만들기

이제 작성 리소스를 만들었으므로 이 리소스를 사용하여 대화 언어 이해 프로젝트를 만들 수 있습니다.

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

1. 포털 위쪽의 **새로 만들기** 메뉴에서 **대화형 언어 이해**을 선택합니다.

1. **기본 정보 입력** 페이지의 **프로젝트 만들기** 대화 상자에 다음 세부 정보를 입력하고 **다음**을 선택합니다.
    - **이름**: `Clock`
    - **발화 기본 언어**: 영어
    - **프로젝트에서 여러 언어 사용**: 선택 안 함**
    - **설명**: `Natural language clock`

1. **검토 및 완료** 페이지에서 **만들기**를 선택합니다.

### 의도 만들기

새 프로젝트에서 가장 먼저 수행할 작업은 일부 의도를 정의하는 것입니다. 모델은 궁극적으로 자연어 발화를 제출할 때 사용자가 요청하는 의도를 예측합니다.

> **팁**: 프로젝트 작업 시 몇 가지 팁이 표시되면 해당 팁을 읽고 **확인**을 선택하여 닫거나 **모두 건너뛰기**를 선택합니다.

1. **스키마 정의** 페이지의 **의도** 탭에서 **&#65291; 추가**를 선택하여 `GetTime`이라는 새 의도를 추가합니다.
1. **GetTime** 의도가 기본 **None** 의도와 함께 나열되어 있는지 확인합니다. 그런 후 다음 추가 의도를 추가합니다.
    - `GetDay`
    - `GetDate`

### 샘플 발화로 각 의도에 레이블 지정

모델이 사용자가 요청하는 의도를 예측할 수 있도록 하려면 몇 가지 샘플 발화로 각 의도에 레이블을 지정해야 합니다.

1. 왼쪽 창에서 **데이터 레이블 지정** 페이지를 선택합니다.

> **팁**: **>>** 아이콘을 사용하여 창을 확장하여 페이지 이름을 확인하고, **<<** 아이콘을 사용하여 창을 다시 숨길 수 있습니다.

1. 새 **GetTime** 의도를 선택하고 발화 `what is the time?`을 입력합니다. 이렇게 하면 발화가 의도에 대한 샘플 입력으로 추가됩니다.
1. **GetTime** 의도에 대해 다음 추가 발화를 추가합니다.
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

    > **참고** 새 발화를 추가하려면 의도 옆에 있는 텍스트 상자에 발화를 작성한 다음 Enter 키를 누릅니다. 

1. **GetDay** 의도를 선택하고 해당 의도에 대한 입력 예로 다음 발화를 추가합니다.
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. **GetDate** 의도를 선택하고 이에 대한 다음 발화를 추가합니다.
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. 각 의도에 대한 발화를 추가한 후 **변경 내용 저장**을 선택합니다.

### 모델 학습 및 테스트

이제 일부 의도를 추가했으므로 언어 모델을 학습시키고 사용자 입력에서 올바르게 예측할 수 있는지 확인해 보겠습니다.

1. 왼쪽 창에서 **학습 작업**을 선택합니다. 그런 다음 **+ 학습 작업 시작**을 선택합니다.

1. **학습 작업 시작** 대화 상자에서 새 모델을 학습하는 옵션을 선택하고 이름을 `Clock`로 지정합니다. **표준 학습** 모드와 기본 **데이터 분할** 옵션을 선택합니다.

1. 모델 학습 과정을 시작하려면 **학습**을 선택합니다.

1. 학습이 완료되면(몇 분 정도 걸릴 수 있음) 작업 **상태**가 **학습 성공**으로 변경됩니다.

1. **모델 성능** 페이지를 선택한 다음 **클록** 모델을 선택합니다. 전체 및 의도별 평가 메트릭(정밀도, 재현율, F1 점수)과 학습 시 수행된 평가에서 생성된 혼동 행렬을 검토합니다(샘플 발화 수가 적기 때문에 일부 의도가 결과에 포함되지 않을 수 있음).********

    > **참고** 평가 메트릭에 대해 자세히 알아보려면 [설명서](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)를 참조하세요.

1. **모델 배포** 페이지로 이동한 다음 **배포 추가**를 선택합니다.

1. **배포 추가** 대화 상자에서 **새 배포 이름 만들기**를 선택한 다음 `production`을 입력합니다.

1. **모델** 필드에서 **클록** 모델을 선택한 다음 **배포**를 선택합니다. 배포에는 다소 시간이 걸릴 수 있습니다.

1. 모델이 배포되면 **배포 테스트** 페이지를 선택한 다음 **배포 이름** 필드에서 **프로덕션** 배포를 선택합니다.

1. 빈 텍스트 상자에 다음 텍스트를 입력한 후 **테스트 실행**을 선택합니다.

    `what's the time now?`

    반환되는 결과를 검토하여 예측한 의도(**GetTime**이어야 함), 그리고 모델이 예측한 의도를 대상으로 계산한 가능성을 나타내는 신뢰도 점수가 포함되어 있음을 확인합니다. JSON 탭은 각 잠재적 의도에 대한 비교 신뢰도를 표시합니다(신뢰도 점수가 가장 높은 것이 예측된 의도임).

1. 텍스트 상자를 지우고 다음 텍스트로 다른 테스트를 실행합니다.

    `tell me the time`

    이번에도 예측한 의도 및 신뢰도 점수를 검토합니다.

1. 다음 텍스트를 시도합니다.

    `what's the day today?`

    이 경우 모델은 **GetDay** 의도를 예측해야 합니다.

## 엔터티 추가

지금까지 의도에 매핑되는 간단한 발화 몇 가지를 정의했습니다. 대다수 실제 애플리케이션에는 더 복잡한 발화가 포함됩니다. 이러한 발화에서 구체적인 데이터 엔터티를 추출하여 의도 예측을 위한 추가 컨텍스트를 수집해야 합니다.

### {b>학습된<b} 엔터티를 추가합니다.

가장 일반적인 엔터티 종류는 모델이 예제를 기반으로 엔터티 값을 식별하기 위해 학습하는 학습된 엔터티입니다.**

1. Language Studio에서 **스키마 정의** 페이지로 돌아간 다음 **엔터티** 탭에서 **&#65291; 새 엔터티를 추가하려면 **을 추가합니다.

1. **엔터티 추가** 대화 상자에서 엔터티 이름 `Location`을 입력하고 **학습함** 탭이 선택되어 있는지 확인합니다. 그런 다음 **엔터티 추가**를 선택합니다.

1. **위치** 엔터티가 만들어진 후 **데이터 레이블 지정** 페이지로 돌아갑니다.
1. **GetTime** 의도를 선택하고 다음과 같은 새로운 발화 예를 입력합니다.

    `what time is it in London?`

1. 이 발화를 추가한 후 단어 **London**을 선택합니다. 그러면 표시되는 드롭다운 목록에서 **Location**을 선택하여 “London”이 위치의 예임을 지정합니다.

1. **GetTime** 의도에 대한 또 다른 발화 예를 추가합니다.

    `Tell me the time in Paris?`

1. 발화가 추가된 경우 단어 **Paris**를 선택하고 **Location** 엔터티에 매핑합니다.

1. **GetTime** 의도에 대한 또 다른 발화 예를 추가합니다.

    `what's the time in New York?`

1. 이 발화를 추가한 후 단어 **New York**을 선택하여 **Location** 엔터티에 매핑합니다.

1. 새 발화를 저장하려면 **변경 내용 저장**을 선택합니다.

### *list* 엔터티 추가

엔터티의 유효한 값을 특정 단어 및 동의어 목록으로 제한할 수 있는 경우도 있습니다. 이렇게 하면 앱이 발화의 엔터티 인스턴스를 쉽게 식별할 수 있습니다.

1. Language Studio에서 **스키마 정의** 페이지로 돌아간 다음 **엔터티** 탭에서 **&#65291; 새 엔터티를 추가하려면 **을 추가합니다.

1. **엔터티 추가** 대화 상자에서 엔터티 이름 `Weekday`를 입력하고 **목록** 엔터티 탭을 선택합니다. 그런 다음 **엔터티 추가**를 선택합니다.

1. **평일** 엔터티 페이지의 **학습함** 섹션에서 **필요하지 않음**이 선택되어 있는지 확인합니다. 그런 다음 **목록** 섹션에서 **&#65291; 새 목록 추가**를 선택합니다. 그런 후 다음 값과 동의어를 입력하고 **저장**을 선택합니다.

    | 목록 키 | 동의어|
    |-------------------|---------|
    | `Sunday` | `Sun` |

    > **참고** 새 목록의 필드를 입력하려면 텍스트 필드에 `Sunday` 값을 삽입한 다음 '값을 입력하고 Enter 키를 누릅니다...'가 표시되는 필드를 클릭한 다음, 동의어를 입력하고 Enter 키를 누릅니다.

1. 이전 단계를 반복하여 다음 목록 구성 요소를 추가합니다.

    | 값 | 동의어|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. 목록 값을 추가하고 저장한 후 **데이터 레이블 지정** 페이지로 돌아갑니다.
1. **GetDate** 의도를 선택하고 다음과 같은 새로운 발화 예를 입력합니다.

    `what date was it on Saturday?`

1. 발화가 추가된 경우 단어 ***Saturday***를 선택하고 표시되는 드롭다운 목록에서 **Weekday**를 선택합니다.

1. **GetDate** 의도에 대한 또 다른 발화 예를 추가합니다.

    `what date will it be on Friday?`

1. 발화가 추가된 경우 **금요일**을 **Weekday** 엔터티에 매핑합니다.

1. **GetDate** 의도에 대한 또 다른 발화 예를 추가합니다.

    `what will the date be on Thurs?`

1. 발화가 추가된 경우 **Thurs**를 **Weekday** 엔터티에 매핑합니다.

1. 새 발화를 저장하려면 **변경 내용 저장**을 선택합니다.

### 미리 빌드된 엔터티 추가**

Azure AI 언어 서비스는 대화형 애플리케이션에서 일반적으로 사용되는 *미리 빌드됨* 엔터티 집합을 제공합니다.

1. Language Studio에서 **스키마 정의** 페이지로 돌아간 다음 **엔터티** 탭에서 **&#65291; 새 엔터티를 추가하려면 **을 추가합니다.

1. **엔터티 추가** 대화 상자에서 엔터티 이름 `Date`를 입력하고 **미리 빌드됨** 엔터티 탭을 선택합니다. 그런 다음 **엔터티 추가**를 선택합니다.

1. **날짜** 엔터티 페이지의 **학습함** 섹션에서 **필요하지 않음**이 선택되어 있는지 확인합니다. 그런 다음 **미리 빌드됨** 섹션에서 **&#65291; 새 미리 빌드됨 추가**를 선택합니다.

1. **미리 빌드됨 선택** 목록에서 **DateTime**을 선택한 다음 **저장**을 선택합니다.
1. 미리 빌드됨 엔터티를 추가한 후 **데이터 레이블 지정** 페이지로 돌아갑니다.
1. **GetDay** 의도를 선택하고 다음과 같은 새로운 발화 예를 입력합니다.

    `what day was 01/01/1901?`

1. 발화가 추가된 경우 단어 ***01/01/1901***을 선택하고 표시되는 드롭다운 목록에서 **Date**를 선택합니다.

1. **GetDay** 의도에 대한 또 다른 발화 예를 추가합니다.

    `what day will it be on Dec 31st 2099?`

1. 발화가 추가된 경우 **2099년 12월 31일**을 **Date** 엔터티에 매핑합니다.

1. 새 발화를 저장하려면 **변경 내용 저장**을 선택합니다.

### 모델 재교육

이제 스키마를 수정했으므로 모델을 다시 학습시키고 다시 테스트해야 합니다.

1. **학습 작업** 페이지에서 **학습 작업 시작**을 선택합니다.

1. **학습 작업 시작** 대화 상자에서 **기존 모델 덮어쓰기**를 선택하고 **클록** 모델을 지정합니다. 모델을 학습하려면 **학습**을 선택합니다. 메시지가 나타나면 기존 모델을 덮어쓸 것인지 확인합니다.

1. 학습이 완료되면 작업 **상태**가 **학습 성공**으로 업데이트됩니다.

1. **모델 성능** 페이지를 선택한 다음 **클록** 모델을 선택합니다. 평가 메트릭(*정밀도*, *재현율*, *F1 점수*)과 학습 시 수행된 평가에서 생성된 *혼동 행렬*을 검토합니다(샘플 발화 수가 적기 때문에 일부 의도가 결과에 포함되지 않을 수 있음).

1. **모델 배포** 페이지에서 **배포 추가**를 선택합니다.

1. **모델 배포** 대화 상자에서 **기존 배포 이름 재정의**를 선택한 다음, **production**을 선택합니다.

1. **모델** 필드에서 **클록** 모델을 선택한 다음 **배포**를 선택하여 배포합니다. 이 작업에는 약간의 시간이 걸릴 수 있습니다.

1. 모델이 배포되면 **배포 테스트** 페이지의 **배포 이름** 필드에서 **프로덕션** 배포를 선택한 후 다음 텍스트를 사용하여 테스트합니다.

    `what's the time in Edinburgh?`

1. 반환되는 결과를 검토합니다. 여기에서 **GetTime** 의도 및 텍스트 값이 “Edinburgh”인 **Location** 엔터티를 예측할 수 있어야 합니다.

1. 다음 발화를 테스트해 봅니다.

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## 클라이언트 앱에서 모델 사용

실제 프로젝트에서는 만족할 만한 예측 성능이 제공될 때까지 의도와 엔터티를 여러 번 미세 조정하고 모델을 다시 학습시켜 테스트를 다시 진행합니다. 그런 다음 테스트를 마치고 예측 성능에 만족하면 REST 인터페이스 또는 런타임별 SDK를 호출하여 클라이언트 앱에서 사용할 수 있습니다.

### Cloud Shell 앱 개발 준비

Azure Portal에서 Cloud Shell을 사용하여 언어 이해 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

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

1. 리포지토리가 복제된 후 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    ```
   cd mslearn-ai-language/Labfiles/03-language/Python/clock-client
    ```

### 애플리케이션 사용

1. 명령줄 창에서 다음 명령을 실행하여 **clock-client** 폴더의 코드 파일을 봅니다.

    ```
   ls -a -l
    ```

    이러한 파일에는 구성 파일(**.env**) 및 코드 파일(**clock-client.py**)이 포함됩니다.

1. Python 가상 환경을 만들고 다음 명령을 실행하여 Azure AI 언어 대화 SDK 패키지 및 기타 필수 패키지를 설치합니다.

    ```
   python -m venv labenv
    ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-conversations==1.1.0
    ```
1. 다음 명령을 입력하여 구성 파일을 편집합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 만든 Azure 언어 리소스의 **엔드포인트** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal의 Azure AI 언어 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능).
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

### 애플리케이션에 코드 추가

1. 다음 명령을 입력하여 애플리케이션 코드 파일을 편집합니다.

    ```
   code clock-client.py
    ```

1. 기존 코드를 검토합니다. AI 언어 대화 SDK를 사용하기 위한 코드를 추가합니다.

    > **팁**: 코드 파일에 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 코드 파일의 맨 위에 있는 기존 네임스페이스 참조에서 **Import namespaces** 주석을 찾고 다음 코드를 추가하여 AI 언어 대화 SDK를 사용해야 하는 네임스페이스를 가져옵니다.

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. **main** 함수에서 구성 파일의 예측 엔드포인트 및 키를 로드하는 코드가 이미 제공되어 있음을 알 수 있습니다. 그런 후 **Create a client for the Language service model** 주석을 찾고 다음 코드를 추가하여 AI 언어 서비스에 대한 대화 분석 클라이언트를 만듭니다.

    ```python
   # Create a client for the Language service model
   client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. **main** 함수의 코드는 사용자가 "quit"을 입력할 때까지 사용자 입력을 제공하라는 메시지를 표시합니다. 이 루프 내에서 **언어 서비스 모델을 호출하여 의도 및 엔터티 가져오기** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Call the Language service model to get intent and entities
   cls_project = 'Clock'
   deployment_slot = 'production'

   with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

   top_intent = result["result"]["prediction"]["topIntent"]
   entities = result["result"]["prediction"]["entities"]

   print("view top intent:")
   print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
   print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
   print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

   print("view entities:")
   for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

   print("query: {}".format(result["result"]["query"]))
    ```

    대화 이해 모델을 호출하면 예측/결과가 반환됩니다. 여기에는 상위(가장 가능성이 높은) 의도, 그리고 입력 발화에서 검색된 모든 엔터티가 포함됩니다. 이제 클라이언트 애플리케이션은 해당 예측을 사용하여 적절한 작업을 결정한 다음 수행해야 합니다.

1. **적절한 작업 적용** 주석을 찾고 다음 코드를 추가합니다. 이 코드는 애플리케이션에서 지원하는 의도(**GetTime**, **GetDate** 및 **GetDay**)를 확인하고 관련 엔터티가 검색되었는지를 확인한 후 기존 함수를 호출하여 적절한 응답을 생성합니다.

    ```python
   # Apply the appropriate action
   if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

   elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

   elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

   else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. 변경 내용을 저장하고(Ctrl+S) 다음 명령을 입력하여 프로그램을 실행합니다(Cloud Shell 창을 최대화하고 명령줄 창에 더 많은 텍스트를 표시하도록 패널 크기 조정).

    ```
   python clock-client.py
    ```

1. 메시지가 표시되면 발화를 입력하여 애플리케이션을 테스트합니다. 예를 들어 다음을 시도해 보세요.

    *Hello*

    *What time is it?*

    *What's the time in London?*

    *What's the date?*

    *What date is Sunday?*

    *What day is it?*

    *What day is 01/01/2025?*

    > **참고**: 여기서는 애플리케이션에 의도적으로 단순한 논리가 사용되었으며 몇 가지 제한이 적용되었습니다. 예를 들어 시간을 가져올 때는 제한된 도시 세트만 지원되며 일광 절약 시간제는 무시됩니다. 목표는 애플리케이션이 수행해야 하는 일반적인 언어 서비스 사용 패턴의 예를 확인하는 것입니다.
    >   1. 예측 엔드포인트에 연결
    >   2. 발화를 제출하여 예측 가져오기
    >   3. 예측된 의도와 엔터티에 적절하게 응답하는 논리 구현

1. 테스트를 완료한 후 *quit*을 입력합니다.

## 리소스 정리

Azure AI 언어 서비스 탐색을 마쳤으면 이 연습에서 만든 리소스를 삭제할 수 있습니다. 이 경우 가능한 방법은 다음과 같습니다.

1. Azure Cloud Shell 창 닫기
1. Azure Portal에서 이 랩에서 만든 Azure AI 언어 리소스를 찾습니다.
1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

Azure AI 언어의 대화 언어 이해에 대해 자세히 알아보려면 [Azure AI 언어 설명서](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview)를 참조하세요.
