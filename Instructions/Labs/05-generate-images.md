---
lab:
  title: DALL-E 모델로 이미지 생성
---

# DALL-E 모델로 이미지 생성

Azure OpenAI Service에는 DALL-E라는 이미지 생성 모델이 포함되어 있습니다. 이 모델을 사용하여 원하는 이미지를 설명하는 자연어 프롬프트를 제출할 수 있으며, 모델은 제공한 설명을 기반으로 원본 이미지를 생성합니다.

이 연습은 약 **25**분 정도 소요됩니다.

## 시작하기 전에

DALL-E를 포함하여 Azure OpenAI 서비스에 액세스하려면 승인된 Azure 구독이 필요합니다. 이전에 Azure openAI 서비스에 대한 액세스를 적용한 경우 DALL-E에 액세스하려면 다른 애플리케이션을 제출해야 할 수도 있습니다.

- 무료 Azure 구독에 등록하려면 [https://azure.microsoft.com/free](https://azure.microsoft.com/free)를 참조하세요.
- Azure OpenAI 서비스에 대한 액세스를 요청하려면 [https://aka.ms/oaiapply](https://aka.ms/oaiapply)를 참조하세요.

## Azure OpenAI 리소스 프로비전

Azure OpenAI 모델을 사용하려면 먼저 Azure 구독에서 Azure OpenAI 리소스를 프로비전해야 합니다.

1. [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 다음 설정을 사용하여 **Azure OpenAI** 리소스를 만듭니다.
    - **구독**: Azure OpenAI 서비스에 대한 액세스가 승인된 Azure 구독입니다.
    - **리소스 그룹**: 기존 리소스 그룹을 선택하거나 원하는 이름으로 새 리소스 그룹을 만듭니다.
    - **지역**: 지역으로 **EastUS**를 선택합니다.
    - **이름**: 원하는 고유한 이름.
    - **가격 책정 계층**: 표준 S0
3. 배포가 완료될 때까지 기다립니다. 그런 다음, Azure Portal에서 배포된 Azure OpenAI 리소스로 이동합니다.
4. **키 및 엔드포인트** 페이지로 이동합니다. 여기에서 서비스에 대한 고유 엔드포인트 및 인증 키를 검색할 수 있습니다. 나중에 필요합니다!

## DALL-E 플레이그라운드에서 이미지 생성을 살펴봅니다.

**Azure OpenAI Studio**의 DALL-E 플레이그라운드를 사용하여 이미지 생성을 실험할 수 있습니다.

1. Azure Portal의 Azure OpenAI 리소스에 대한 **개요** 페이지에서 **탐색** 단추를 사용하여 새 브라우저 탭에서 Azure OpenAI Studio를 엽니다. 또는 [Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true)로 직접 이동합니다.
2. **DALL-E 플레이그라운드**를 선택합니다.
3. **프롬프트** 상자에 생성하려는 이미지에 대한 설명을 입력합니다. 예: *스케이트보드를 탄 코끼리*. 그런 다음 **생성**을 선택하고 생성된 이미지를 확인합니다.

    ![생성된 이미지가 있는 Azure OpenAI Studio의 DALL-E 플레이그라운드.](../media/dall-e-playground.png)

4. 보다 구체적인 설명을 제공하도록 프롬프트를 수정합니다. 예: *피카소 스타일의 스케이트보드를 탄 코끼리*. 그런 다음 새 이미지를 생성하고 결과를 검토합니다.

    ![두 개의 생성된 이미지가 있는 Azure OpenAI Studio의 DALL-E 플레이그라운드.](../media/dall-e-playground-new-image.png)

## REST API를 사용하여 이미지 생성

Azure OpenAI 서비스는 DALL-E 모델에서 생성된 이미지를 포함하여 콘텐츠 생성을 위한 프롬프트를 제출하는 데 사용할 수 있는 REST API를 제공합니다.

### 앱 환경 준비

이 연습에서는 간단한 Python 또는 Microsoft C# 앱을 사용하여 REST API를 호출하여 이미지를 생성합니다. Azure Portal의 Cloud Shell 콘솔 인터페이스에서 코드를 실행합니다.

1. [Azure Portal](https://portal.azure.com?azure-portal=true)에서 검색 상자 오른쪽 페이지 맨 위에 있는 **[>_]**(*Cloud Shell*) 단추를 선택합니다. 포털의 맨 아래에서 Cloud Shell 창이 열립니다. 

    ![맨 위 검색 상자 오른쪽에 있는 아이콘을 클릭하여 Cloud Shell을 시작하는 스크린샷](../media/cloudshell-launch-portal.png#lightbox)

2. Cloud Shell을 처음 열면 사용할 셸 유형(*Bash* 또는 *PowerShell*)을 선택하라는 메시지가 표시될 수 있습니다. **Bash**를 선택합니다. 이 옵션이 표시되지 않으면 단계를 건너뜁니다.  

3. Cloud Shell용 스토리지를 만들라는 메시지가 표시되면 **고급 설정 표시**를 선택하고 다음 설정을 선택합니다.
    - **구독**: 사용자의 구독
    - **Cloud Shell 지역**: 사용 가능한 지역 선택
    - **VNet 격리 설정 표시**가 선택 취소됨
    - **리소스 그룹**: Azure OpenAI 리소스를 프로비전한 기존 리소스 그룹을 사용합니다.
    - **스토리지 계정**: 고유한 이름으로 새 스토리지 계정 만들기
    - **파일 공유**: 고유한 이름으로 새 파일 공유 만들기

    그런 다음, 스토리지가 만들어질 때까지 1분 정도 기다립니다.

    > **참고**: Azure 구독에 Cloud Shell이 이미 설정되어 있는 경우 ⚙️ 메뉴에서 **사용자 설정 초기화** 옵션을 사용하여 최신 버전의 Python 및 .NET Framework가 설치되어 있는지 확인해야 할 수 있습니다.

4. Cloud Shell 창 왼쪽 상단에 표시된 셸 형식이 *Bash*인지 확인합니다. *PowerShell*인 경우 드롭다운 메뉴를 사용하여 *Bash*로 전환합니다.

5. 터미널이 시작되면 다음 명령을 입력하여 작업할 애플리케이션 코드를 다운로드합니다.

    ```bash
    rm -r azure-openai -f
    git clone https://github.com/MicrosoftLearning/mslearn-openai azure-openai
    ```

    파일은 **azure-openai**라는 폴더에 다운로드됩니다. C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다.

6. 적절한 명령을 실행하여 사용하려는 언어의 폴더로 이동합니다.

    **Python**

    ```bash
    cd azure-openai/Labfiles/05-image-generation/Python
    ```

    **C#**

    ```bash
    cd azure-openai/Labfiles/05-image-generation/CSharp
    ```

7. 다음 명령을 사용하여 기본 제공 코드 편집기를 열고 작업할 코드 파일을 확인합니다.

    ```bash
    code .
    ```

    > **팁**: Azure Cloud Shell 환경에서 파일 작업에 사용하는 방법에 대한 자세한 내용은 [Azure Cloud Shell 코드 편집기 설명서](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor)를 참조하세요.

### 애플리케이션 사용

애플리케이션은 구성 파일을 사용하여 Azure OpenAI 서비스 계정에 연결하는 데 필요한 세부 정보를 저장합니다.

1. 코드 편집기에서 언어 선택에 따라 앱의 구성 파일을 선택합니다.

    - C#: `appsettings.json`
    - Python: `.env`
    
2. Azure OpenAI 서비스에 대한 **엔드포인트** 및 **키1**을 포함하도록 구성 값을 업데이트한 다음 파일을 저장합니다.

    > **팁**: Cloud Shell 창 상단에서 분할을 조정하여 Azure Portal을 확인하고 Azure OpenAI 서비스의 **키 및 엔드포인트** 페이지에서 엔드포인트와 키 값을 가져올 수 있습니다.

3. **Python**을 사용하는 경우 구성 파일을 읽는 데 사용되는 **python-dotenv** 패키지도 설치해야 합니다. 콘솔 프롬프트 창에서 현재 폴더가 **~/azure-openai/Labfiles/05-image- Generation/Python**인지 확인합니다. 그런 후 다음 명령을 입력합니다.

    ```bash
    pip install python-dotenv
    ```

### 애플리케이션 코드 보기

이제 REST API를 호출하고 이미지를 생성하는 데 사용되는 코드를 탐색할 준비가 되었습니다.

1. 코드 편집기 창에서 애플리케이션의 기본 코드 파일을 선택합니다.

    - C#: `Program.cs`
    - Python: `generate-image.py`

2. 파일에 포함된 코드를 검토하고 다음 주요 기능을 확인합니다.
    - 이 코드는 헤더의 서비스 키를 포함하여 서비스 엔드포인트에 대한 https 요청을 만듭니다. 이 두 값은 모두 구성 파일에서 가져옵니다.
    - 이 프로세스는 <u>두 가지</u> REST 요청으로 구성됩니다. 하나는 이미지 생성 요청을 시작하고 다른 하나는 결과를 검색합니다.
    초기 요청에는 다음 데이터가 포함됩니다.
        - 생성할 이미지를 설명하는 사용자 제공 프롬프트
        - 생성할 이미지 수(이 경우 1)
        - 생성할 이미지의 해상도(크기).
    - 초기 요청의 응답 헤더에는 결과를 가져오기 위한 후속 콜백에 사용되는 **작업 위치** 값이 포함되어 있습니다.
    - 코드는 이미지 생성 작업 상태가 *성공*이 될 때까지 콜백 URL을 폴링한 다음 생성된 이미지에 대한 URL을 추출하여 표시합니다.

### 앱 실행

이제 코드를 검토했으므로 코드를 실행하고 일부 이미지를 생성할 차례입니다.

1. 콘솔 프롬프트 창에서 적절한 명령을 입력하여 애플리케이션을 실행합니다.

    **Python**

    ```bash
    python generate-image.py
    ```

    **C#**

    ```bash
    dotnet run
    ```

2. 메시지가 표시되면 이미지에 대한 설명을 입력합니다. 예: *연을 날리는 기린*.

3. 이미지가 생성될 때까지 기다립니다. 콘솔 창에 하이퍼링크가 표시됩니다. 그런 다음 하이퍼링크를 선택하여 새 브라우저 탭을 열고 생성된 이미지를 검토합니다.

4. 생성된 이미지가 포함된 탭을 닫고 앱을 다시 실행하여 다른 프롬프트로 새 이미지를 생성합니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 [Azure Portal](https://portal.azure.com/?azure-portal=true)에서 리소스를 삭제해야 합니다.
