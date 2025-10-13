# 요구 사항

## Cloud Shell에서 실행

* OpenAI 액세스 권한이 있는 Azure 구독
* Azure Cloud Shell에서 실행되는 경우 Bash 셸을 선택합니다. Azure CLI 및 Azure Developer CLI는 Cloud Shell에 포함됩니다.

## 로컬 실행

* 배포 스크립트를 실행한 후 웹앱을 로컬로 실행할 수 있습니다.
    * [Azure 개발자 CLI(azd)](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd)
    * [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    * OpenAI 액세스 권한이 있는 Azure 구독


## 환경 변수

`.env` 파일은 *azdeploy.sh* 스크립트에 의해 만들어집니다. AI 모델 엔드포인트, API 키 및 모델 이름은 리소스를 배포하는 동안 추가됩니다.

## Azure 리소스 배포

제공된 `azdeploy.sh`는 Azure에서 필요한 리소스를 만듭니다.

* 스크립트 맨 위에 있는 두 변수를 요구 사항에 맞게 변경하고 다른 변수는 변경하지 마세요.
* 이 스크립트는
    * AZD를 사용하여 *gpt-4o* 모델을 배포합니다.
    * Azure Container Registry 서비스를 만듭니다.
    * ACR 작업을 사용하여 Dockerfile 이미지를 빌드하고 ACR에 배포합니다.
    * App Service 요금제를 만듭니다.
    * App Service 웹앱을 만듭니다.
    * ACR에서 컨테이너 이미지에 대한 웹앱을 구성합니다.
    * 웹앱 환경 변수를 구성합니다.
    * 스크립트는 App Service 엔드포인트를 제공합니다.

스크립트는 다음과 같은 두 가지 배포 옵션을 제공합니다. 1. 전체 배포. 2. 이미지만 다시 배포. 옵션 2는 애플리케이션의 변경 내용을 실험해 보려는 배포 이후용 옵션입니다. 

> 참고: `bash azdeploy.sh` 명령을 사용하여 PowerShell 또는 Bash에서 스크립트를 실행할 수 있습니다. 또한 이 명령을 실행 파일로 만들지 않고 Bash에서 스크립트를 실행해 보겠습니다.

## 로컬 개발

### Azure에 AI 모델 프로비전

프로젝트를 로컬로 실행하고 다음 단계에 따라 AI 모델만 프로비전할 수 있습니다.

1. **환경 초기화**(설명이 포함된 이름 선택):

   ```bash
   azd env new gpt-realtime-lab --confirm
   # or: azd env new your-name-gpt-experiment --confirm
   ```
   
   **중요**: 이 이름은 Azure 리소스 이름의 일부가 됩니다.  
   `--confirm` 플래그는 확인 메시지를 표시하지 않고 이를 기본 환경으로 설정합니다.

1. **리소스 그룹을 설정합니다**.

   ```bash
   azd env set AZURE_RESOURCE_GROUP "rg-your-name-gpt"
   ```

1. **로그인 및 AI 리소스를 프로비전합니다**.

   ```bash
   az login
   azd provision
   ```

    > **중요**: `azd deploy`는 실행하지 마세요. 이 앱은 AZD 템플릿에서 구성되지 않습니다.

`azd provision` 메서드를 사용하여 모델을 프로비전한 경우에만 다음 항목을 사용하여 디렉터리의 루트에 `.env` 파일을 만들어야 합니다.

```
AZURE_VOICE_LIVE_ENDPOINT=""
AZURE_VOICE_LIVE_API_KEY=""
VOICE_LIVE_MODEL=""
VOICE_LIVE_VOICE="en-US-JennyNeural"
VOICE_LIVE_INSTRUCTIONS="You are a helpful AI assistant with a focus on world history. Respond naturally and conversationally. Keep your responses concise but engaging."
VOICE_LIVE_VERBOSE="" #Suppresses excessive logging to the terminal if running locally
```

참고:

1. 엔드포인트는 모델의 엔드포인트이며 `https://<proj-name>.cognitiveservices.azure.com`을 포함해야 합니다.
1. API 키는 모델의 키입니다.
1. 모델은 배포 중에 사용되는 모델 이름입니다.
1. AI Foundry 포털에서 이러한 값을 검색할 수 있습니다.

### 로컬로 프로젝트 실행

**uv**를 사용하여 프로젝트를 만들고 관리했지만 실행할 필요는 없습니다. 

**uv**를 설치한 경우:

* `uv venv`를 실행하여 환경 만들기
* `uv sync`를 실행하여 패키지 추가
* 웹앱에 대해 만든 별칭: `flask_app.py` 스크립트를 시작하기 위한 `uv run web`.
* `uv pip compile pyproject.toml -o requirements.txt`로 만든 requirements.txt 파일

**uv**를 설치하지 않은 경우:

* 환경 만들기: `python -m venv .venv`
* 환경 활성화: `.\.venv\Scripts\Activate.ps1`
* 종속성 설치: `pip install -r requirements.txt`
* 애플리케이션 실행(프로젝트 루트에서): `python .\src\real_time_voice\flask_app.py`
