---
lab:
  title: DALL-E 모델로 이미지 생성
---

# DALL-E 모델로 이미지 생성

Azure OpenAI Service에는 DALL-E라는 이미지 생성 모델이 포함되어 있습니다. 이 모델을 사용하여 원하는 이미지를 설명하는 자연어 프롬프트를 제출할 수 있으며, 모델은 제공한 설명을 기반으로 원본 이미지를 생성합니다.

이 연습에서는 DALL-E 버전 3 모델을 사용하여 자연어 프롬프트를 기반으로 이미지를 생성합니다.

이 연습은 약 **25**분 정도 소요됩니다.

## Azure OpenAI 리소스 프로비전

Azure OpenAI를 사용하여 이미지를 생성하려면 먼저 Azure 구독에서 Azure OpenAI 리소스를 프로비전해야 합니다. 리소스는 DALL-E 모델이 지원되는 지역에 있어야 합니다.

1. `https://portal.azure.com`에서 **Azure Portal**에 로그인합니다.
1. 다음 설정을 사용하여 **Azure OpenAI** 리소스를 만듭니다.
    - **구독**: *DALL-E를 포함하여 Azure OpenAI 서비스에 대한 액세스가 승인된 Azure 구독 선택*
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: ***미국 동부**, **오스트레일리아 동부** 또는 **스웨덴 중부*** 선택\*
    - **이름**: ‘원하는 고유한 이름’**
    - **가격 책정 계층**: 표준 S0

    > \*DALL-E 3 모델은 **미국 동부**, **오스트레일리아 동부** 및 **스웨덴 중부** 지역의 Azure OpenAI 서비스 리소스에서만 사용할 수 있습니다.

1. 배포가 완료될 때까지 기다립니다. 그런 다음, Azure Portal에서 배포된 Azure OpenAI 리소스로 이동합니다.

## 모델 배포

다음으로 CLI에서 **dalle3** 모델의 배포를 만듭니다. Azure Portal에서 위쪽 메뉴 모음에서 **Cloud Shell** 아이콘을 선택하고 터미널이 **Bash**로 설정되어 있는지 확인합니다. 이 예제를 사용하여 다음 변수를 사용자 고유의 값으로 바꿉니다.

```dotnetcli
az cognitiveservices account deployment create \
   -g *your resource group* \
   -n *your Open AI resource* \
   --deployment-name dall-e-3 \
   --model-name dall-e-3 \
   --model-version 3.0  \
   --model-format OpenAI \
   --sku-name "Standard" \
   --sku-capacity 1
```

    > \* Sku-capacity is measured in thousands of tokens per minute. A rate limit of 1,000 tokens per minute is more than adequate to complete this exercise while leaving capacity for other people using the same subscription.


## REST API를 사용하여 이미지 생성

Azure OpenAI 서비스는 DALL-E 모델에서 생성된 이미지를 포함하여 콘텐츠 생성을 위한 프롬프트를 제출하는 데 사용할 수 있는 REST API를 제공합니다.

### Visual Studio Code에서 앱 개발 준비

이제 Azure OpenAI 서비스를 사용하여 이미지를 생성하는 사용자 지정 앱을 빌드하는 방법을 살펴보겠습니다. 여기서는 Visual Studio Code를 사용하여 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-openai** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-openai` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **참고**: Visual Studio Code에서 열려는 코드를 신뢰하라는 팝업 메시지가 표시되면 팝업에서 **예, 작성자를 신뢰합니다.** 옵션을 클릭합니다.

4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

### 애플리케이션 사용

C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure OpenAI 리소스에 대한 엔드포인트와 키를 앱의 구성 파일에 추가합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/03-image-generation** 폴더를 찾아 언어 선택에 맞게 **CSharp** 또는 **Python** 폴더를 확장합니다. 각 폴더에는 Azure OpenAI 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
2. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env
    
3. 만든 Azure OpenAI 리소스의 **엔드포인트** 및 **키**를 포함하도록 구성 값을 업데이트합니다(Azure Portal의 Azure OpenAI 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능).
4. 구성 파일을 저장합니다.

### 애플리케이션 코드 보기

이제 REST API를 호출하고 이미지를 생성하는 데 사용되는 코드를 탐색할 준비가 되었습니다.

1. **탐색기** 창에서 애플리케이션의 기본 코드 파일을 선택합니다.

    - C#: `Program.cs`
    - Python: `generate-image.py`

2. 파일에 포함된 코드를 검토하고 다음 주요 기능을 확인합니다.
    - 이 코드는 헤더에 서비스 키를 포함하여 서비스 엔드포인트에 https 요청을 보냅니다. 이 두 값은 모두 구성 파일에서 가져옵니다.
    - 요청에는 이미지 기반 프롬프트, 생성할 이미지 수, 생성된 이미지 크기 등 일부 매개 변수가 포함됩니다.
    - 응답에는 DALL-E 모델이 사용자가 제공한 프롬프트에서 추정하여 더 설명적으로 표현한 수정된 프롬프트와 생성된 이미지의 URL이 포함됩니다.
    
    > **중요**: 권장 되는 *dalle3* 이외의 배포 이름을 지정한 경우 배포 이름을 사용하도록 코드를 업데이트해야 합니다.

### 앱 실행

이제 코드를 검토했으므로 코드를 실행하고 일부 이미지를 생성할 차례입니다.

1. 코드 파일이 포함된 **CSharp** 또는 **Python** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음 적절한 명령을 입력하여 애플리케이션을 실행합니다.

   **C#**
   ```
   dotnet run
   ```
   
   **Python**
   ```
   pip install requests
   python generate-image.py
   ```

3. 메시지가 표시되면 이미지에 대한 설명을 입력합니다. 예: *연을 날리는 기린*.

4. 이미지가 생성될 때까지 기다립니다. 하이퍼링크가 터미널 창에 표시됩니다. 그런 다음 하이퍼링크를 선택하여 새 브라우저 탭을 열고 생성된 이미지를 검토합니다.

   > **팁**: 앱이 응답을 반환하지 않으면 잠시 기다렸다가 다시 시도합니다. 새로 배포된 리소스를 사용할 수 있게 되기까지 최대 5분이 걸릴 수 있습니다.

5. 생성된 이미지가 포함된 브라우저 탭을 닫고 앱을 다시 실행하여 다른 프롬프트로 새 이미지를 생성합니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 **Azure Portal**의 `https://portal.azure.com`에서 리소스를 삭제해야 합니다.
