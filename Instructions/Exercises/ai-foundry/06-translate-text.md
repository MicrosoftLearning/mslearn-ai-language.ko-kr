---
lab:
  title: 텍스트 번역
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# 텍스트 번역

**Azure AI 번역기**는 언어 간 텍스트를 번역할 수 있는 서비스입니다. 이 연습에서는 이를 사용하여 지원되는 모든 언어의 입력을 선택한 대상 언어로 번역하는 간단한 앱을 만듭니다.

## *Azure AI 번역기* 리소스 프로비전

구독에 아직 없는 경우 **Azure AI 번역기** 리소스를 프로비전해야 합니다.

1. `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연관된 Microsoft 계정을 사용하여 로그인합니다.
1. 상단의 검색 필드에서 **Azure AI 서비스**를 검색하고 **Enter** 키를 누른 다음 결과의 **번역기**에서 **만들기**를 선택합니다.
1. 다음 설정을 사용하여 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: *사용 가능한 지역 선택*
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: F를 사용할 수 없는 경우 **F0**(*무료*) 또는 **S**(*표준*)를 선택합니다.
    - **담당 AI 알림**: 동의합니다.
1. **검토 + 만들기를** 선택한 다음, **만들기**를 선택하여 리소스를 프로비전합니다.
1. 배포가 완료될 때까지 기다린 다음, 배포된 리소스로 이동합니다.
1. **키 및 엔드포인트** 페이지를 확인합니다. 연습 후반부에서 이 페이지의 정보가 필요합니다.

## Visual Studio Code에서 앱 개발 준비

Visual Studio Code를 사용하여 텍스트 번역 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-ai-language** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-language` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **참고**: Visual Studio Code에서 열려는 코드를 신뢰하라는 팝업 메시지가 표시되면 팝업에서 **예, 작성자를 신뢰합니다.** 옵션을 클릭합니다.

4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure AI 번역기 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/06b-translator-sdk** 폴더를 찾아 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장하고, 포함되어 있는 **translate-text** 폴더를 확장합니다. 각 폴더에는 Azure AI 번역기 기능을 통합할 앱에 대한 언어별 코드 파일이 포함되어 있습니다.
2. 코드 파일이 포함된 **translate-text** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure AI 번역기 SDK 패키지를 설치합니다.

    **C#:**

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**:

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. **탐색기** 창의 **translate-text** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. 만든 Azure AI 번역기 리소스의 **지역** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal에서 Azure AI 번역기 리소스의 **키 및 엔드포인트** 페이지에서 사용 가능).

    > **참고**: 엔드포인트가 <u>아님</u> 리소스에 대한 *지역*을 추가해야 합니다!

5. 구성 파일을 저장합니다.

## 텍스트를 번역하는 코드 추가

이제 Azure AI 번역기를 사용하여 텍스트를 번역할 준비가 되었습니다.

1. **translate-text** 폴더에는 클라이언트 애플리케이션용 코드 파일이 포함되어 있습니다.

    - **C#**: Program.cs
    - **Python**: translate.py

    코드 파일을 열고 파일 맨 윗부분의 기존 네임스페이스 참조 아래에 있는 **네임스페이스 가져오기** 주석을 찾습니다. 그런 다음 이 주석 아래에 다음 언어별 코드를 추가하여 Text Analytics SDK를 사용하는 데 필요한 네임스페이스를 가져옵니다.

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python**: translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. **Main** 함수에서 기존 코드가 구성 설정을 참조한다는 점에 유의해야 합니다.
1. **엔드포인트 및 키를 사용하여 클라이언트 만들기** 주석을 찾아 다음 코드를 추가합니다.

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python**: translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. **대상 언어 선택** 주석을 찾아 텍스트 Translator 서비스를 사용하여 번역에 지원되는 언어 목록을 반환하고 사용자에게 대상 언어에 대한 언어 코드를 선택하라는 메시지를 표시하는 다음 코드를 추가합니다.

    **C#**: Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python**: translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
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

1. **텍스트 번역** 주석을 찾아 사용자에게 번역할 텍스트를 반복적으로 묻고, Azure AI 번역기 서비스를 사용하여 대상 언어로 번역하고(원어를 자동으로 검색), 사용자가 *quit*을 입력할 때까지 결과를 표시하는 다음 코드를 추가합니다.

    **C#**: Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python**: translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. 코드 파일에 변경 내용을 저장합니다.

## 애플리케이션 테스트

이제 애플리케이션을 테스트할 준비가 되었습니다.

1. **텍스트 번역** 폴더의 통합 터미널에서 다음 명령을 입력하여 프로그램을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python translate.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

1. 메시지가 표시되면 표시된 목록에서 유효한 대상 언어를 입력합니다.
1. 번역할 구(예: `This is a test` 또는 `C'est un test`)를 입력한 다음 소스 언어를 검색하고 텍스트를 대상 언어로 번역해야 하는 결과를 확인합니다.
1. 완료되면 `quit`를 입력합니다. 애플리케이션을 다시 실행하고 다른 대상 언어를 선택할 수 있습니다.

## 정리

프로젝트가 더 이상 필요하지 않으면 [Azure Portal](https://portal.azure.com)에서 Azure AI 번역기 리소스를 삭제할 수 있습니다.
