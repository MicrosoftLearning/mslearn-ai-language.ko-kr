---
lab:
  title: 텍스트 번역
  description: Azure AI 번역기를 사용하여 제공된 텍스트를 지원되는 언어 간에 번역합니다.
---

# 텍스트 번역

**Azure AI 번역기**는 언어 간 텍스트를 번역할 수 있는 서비스입니다. 이 연습에서는 이를 사용하여 지원되는 모든 언어의 입력을 선택한 대상 언어로 번역하는 간단한 앱을 만듭니다.

이 연습은 Python을 기준으로 하지만 여러 언어별 SDK를 사용하여 텍스트 번역 애플리케이션을 개발할 수 있습니다. 포함된 내용은 다음과 같습니다.

- [Python용 Azure AI 번역 클라이언트 라이브러리](https://pypi.org/project/azure-ai-translation-text/)
- [.NET용 Azure AI 번역 클라이언트 라이브러리](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [JavaScript용 Azure AI 번역 클라이언트 라이브러리](https://www.npmjs.com/package/@azure-rest/ai-translation-text)

이 연습에는 약 **30**분이 소요됩니다.

## *Azure AI 번역기* 리소스 프로비전

구독에 아직 없는 경우 **Azure AI 번역기** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. 위쪽의 검색 필드에서 **번역기**를 검색한 다음, 결과에서 **번역기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: *사용 가능한 지역 선택*
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## Cloud Shell 앱 개발 준비

Azure AI 번역기의 텍스트 번역 기능을 테스트하려면 Azure Cloud Shell에서 간단한 콘솔 애플리케이션을 개발합니다.

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
   cd mslearn-ai-language/Labfiles/06-translator-sdk/Python/translate-text
    ```

## 애플리케이션 사용

1. 명령줄 창에서 다음 명령을 실행하여 **translate-text** 폴더의 코드 파일을 봅니다.

    ```
   ls -a -l
    ```

    파일에는 구성 파일(**.env**) 및 코드 파일(**translate.py**)이 포함됩니다.

1. Python 가상 환경을 만들고 다음 명령을 실행하여 Azure AI Translation SDK 패키지 및 기타 필수 패키지를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-translation-text==1.0.1
    ```

1. 다음 명령을 입력하여 애플리케이션 구성 파일을 편집합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 만든 Azure AI 번역기 리소스의 **지역** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal에서 Azure AI 번역기 리소스의 **키 및 엔드포인트** 페이지에서 사용 가능).

    > **참고**: 엔드포인트가 <u>아님</u> 리소스에 대한 *지역*을 추가해야 합니다!

1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 종료**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

## 텍스트를 번역하는 코드 추가

1. 다음 명령을 입력하여 애플리케이션 코드 파일을 편집합니다.

    ```
   code translate.py
    ```

1. 기존 코드를 검토합니다. Azure AI Translation SDK를 사용하기 위한 코드를 추가합니다.

    > **팁**: 코드 파일에 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 코드 파일의 맨 위에 있는 기존 네임스페이스 참조에서 **Import namespaces** 주석을 찾고 다음 코드를 추가하여 Translation SDK를 사용해야 하는 네임스페이스를 가져옵니다.

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. **main** 함수에서는 기존 코드가 구성 설정을 참조한다는 점에 유의해야 합니다.
1. **엔드포인트 및 키를 사용하여 클라이언트 만들기** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Create client using endpoint and key
   credential = AzureKeyCredential(translatorKey)
   client = TextTranslationClient(credential=credential, region=translatorRegion)
    ```

1. **Choose target language** 주석을 찾고 다음 코드를 추가합니다. 이 코드는 텍스트 번역기 서비스를 사용하여 번역하도록 지원되는 언어 목록을 반환하고 사용자에게 대상 언어에 대한 언어 코드를 선택하라는 메시지를 표시합니다.

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
   print("{} languages supported.".format(len(languagesResponse.translation)))
   print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
   print("Enter a target language code for translation (for example, 'en'):")
   targetLanguage = "xx"
   supportedLanguage = False
   while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. **Translate text** 주석을 찾고 다음 코드를 추가합니다. 이 코드는 사용자에게 번역할 텍스트를 입력하라는 메시지를 반복적으로 표시하고, Azure AI 번역기 서비스를 사용하여 대상 언어로 번역하고(소스 언어를 자동으로 검색), 사용자가 *quit*을 입력할 때까지 결과를 표시합니다.

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. 변경 내용을 저장하고(Ctrl+S) 다음 명령을 입력하여 프로그램을 실행합니다(Cloud Shell 창을 최대화하고 명령줄 창에 더 많은 텍스트를 표시하도록 패널 크기 조정).

    ```
   python translate.py
    ```

1. 메시지가 표시되면 표시된 목록에서 유효한 대상 언어를 입력합니다.
1. 번역할 구(예: `This is a test` 또는 `C'est un test`)를 입력한 다음 소스 언어를 검색하고 텍스트를 대상 언어로 번역해야 하는 결과를 확인합니다.
1. 완료되면 `quit`를 입력합니다. 애플리케이션을 다시 실행하고 다른 대상 언어를 선택할 수 있습니다.

## 리소스 정리

Azure AI 번역기 서비스 탐색을 마쳤으면 이 연습에서 만든 리소스를 삭제할 수 있습니다. 방법은 다음과 같습니다.

1. Azure Cloud Shell 창 닫기
1. Azure Portal에서 이 랩에서 만든 Azure AI 번역기 리소스를 찾습니다.
1. 리소스 페이지에서 **삭제**를 선택하고 지침에 따라 리소스를 삭제합니다.

## 자세한 정보

**Azure AI 번역기** 사용에 대한 자세한 내용은 [Azure AI 번역기 설명서](https://learn.microsoft.com/azure/ai-services/translator/)를 참조하세요.
