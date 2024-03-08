---
lab:
  title: Azure OpenAI Service를 사용하여 코드 생성 및 개선
---

# Azure OpenAI Service를 사용하여 코드 생성 및 개선

Azure OpenAI Service 모델은 자연어 프롬프트를 사용하여 코드를 생성하고, 작성한 코드의 버그를 수정하고, 코드 주석을 제공할 수 있습니다. 이러한 모델은 기존 코드에 대해서도 그 기능과 개선 방법을 이해할 수 있도록 설명하고 간소화할 수 있습닌다.

이 연습은 약 **25**분 정도 소요됩니다.

## 시작하기 전에

Azure OpenAI 서비스에 액세스하려면 승인된 Azure 구독이 필요합니다.

- 무료 Azure 구독에 등록하려면 [https://azure.microsoft.com/free](https://azure.microsoft.com/free)를 참조하세요.
- Azure OpenAI 서비스에 대한 액세스를 요청하려면 [https://aka.ms/oaiapply](https://aka.ms/oaiapply)를 참조하세요.

## Azure OpenAI 리소스 프로비전

Azure OpenAI 모델을 사용하려면 먼저 Azure 구독에서 Azure OpenAI 리소스를 프로비전해야 합니다.

1. [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 다음 설정을 사용하여 **Azure OpenAI** 리소스를 만듭니다.
    - **구독**: Azure OpenAI 서비스에 대한 액세스가 승인된 Azure 구독입니다.
    - **리소스 그룹**: 기존 리소스 그룹을 선택하거나 원하는 이름으로 새 리소스 그룹을 만듭니다.
    - **지역**: 사용 가능한 지역을 선택합니다.
    - **이름**: 원하는 고유한 이름.
    - **가격 책정 계층**: 표준 S0
3. 배포가 완료될 때까지 기다립니다. 그런 다음, Azure Portal에서 배포된 Azure OpenAI 리소스로 이동합니다.
4. **키 및 엔드포인트** 페이지로 이동하여 나중에 사용할 수 있도록 텍스트 파일에 저장합니다.

## 모델 배포

코드 생성에 Azure OpenAI API를 사용하려면 먼저 **Azure OpenAI Studio**를 통해 사용할 모델을 배포해야 합니다. 배포한 후에는 플레이그라운드와 함께 모델을 사용하고 앱에서 해당 모델을 참조합니다.

1. Azure OpenAI 리소스의 **개요** 페이지에서 **탐색** 단추를 사용하여 새 브라우저 탭에서 Azure OpenAI Studio를 엽니다. 또는 [Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true)로 직접 이동합니다.
2. Azure OpenAI Studio의 **배포** 페이지에서 기존 모델 배포를 확인합니다. 아직 없는 경우 다음 설정을 사용하여 **gpt-35-turbo-16k** 모델의 새 배포를 만듭니다.
    - **모델**: gpt-35-turbo-16k
    - **모델 버전**: 기본값으로 자동 업데이트
    - **배포 이름**: ‘원하는 고유한 이름’**
    - **고급 옵션**
        - **콘텐츠 필터**: 기본값
        - **분당 토큰 속도 제한**: 5K\*
        - **동적 할당량 사용**: 사용

    > \* 분당 5,000개 토큰의 속도를 제한하더라도 동일한 구독을 사용하는 다른 사용자에게 용량을 남겨두면서 이 연습을 충분히 완료할 수 있습니다.

> **참고**: 일부 지역에서는 새 모델 배포 인터페이스에 **모델 버전** 옵션이 표시되지 않습니다. 이 경우 옵션을 설정하지 않고도 걱정하지 말고 계속 진행하세요.

## 채팅 플레이그라운드에서 코드 생성

앱에서 사용하기 전에 Azure OpenAI가 채팅 플레이그라운드에서 어떻게 코드를 생성하고 설명하는지 검토합니다.

1. [Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true)에서 왼쪽 창에 있는 **채팅** 플레이그라운드로 이동합니다.
1. 1. **구성**에서 모델 배포가 선택되었는지 확인합니다.
1. 위쪽의 **도우미 설정** 섹션에서 **기본** 시스템 메시지 템플릿을 선택합니다.
1. **채팅 세션** 섹션에서 다음 프롬프트를 입력하고 *Enter*를 누릅니다.

    ```code
   Write a function in python that takes a character and string as input, and returns how many times that character appears in the string
    ```

1. 모델은 함수가 수행하는 작업과 함수를 호출하는 방법에 대한 설명과 함께 함수로 응답할 가능성이 높습니다.
1. 다음으로 프롬프트 `Do the same thing, but this time write it in C#`을(를) 보냅니다.
1. 출력을 확인합니다. 모델은 처음과 매우 유사하게 응답했겠지만 이번에는 C#으로 코딩했습니다. 다른 언어로 다시 요청하거나, 입력 문자열을 되돌리는 것과 같은 다른 작업을 완료하는 함수로 다시 요청해 볼 수 있습니다.
1. 다음으로, Ruby로 작성된 이 임의 함수 예제를 통해 AI를 사용하여 코드를 이해하는 방식을 살펴보겠습니다. 다음 프롬프트를 사용자 메시지로 보냅니다.

    ```code
    What does the following function do?  
    ---  
    def random_func(n)
      start = [0, 1]
      (n - 2).times do
        start << start[-1] + start[-2]
      end
      start.shuffle.each do |num|
        puts num
      end
    end
    ```

1. 자연어로 함수가 어떤 작업을 하는지 보여주는 출력을 확인합니다. 익숙한 언어로 다시 작성하도록 모델에 요청해 보세요.

## Cloud Shell에서 애플리케이션 설정

Azure OpenAI 모델과 통합하는 방법을 보여 주기 위해 Azure의 Cloud Shell에서 실행되는 간단한 명령줄 애플리케이션을 사용할 예정입니다. Cloud Shell을 사용하려면 새 브라우저 탭을 엽니다.

1. [Azure Portal](https://portal.azure.com?azure-portal=true)에서 검색 상자 오른쪽 페이지 맨 위에 있는 **[>_]**(*Cloud Shell*) 단추를 선택합니다. 포털의 맨 아래에서 Cloud Shell 창이 열립니다.

    ![맨 위 검색 상자 오른쪽에 있는 아이콘을 클릭하여 Cloud Shell을 시작하는 스크린샷](../media/cloudshell-launch-portal.png#lightbox)

2. Cloud Shell을 처음 열면 사용할 셸 유형(*Bash* 또는 *PowerShell*)을 선택하라는 메시지가 표시될 수 있습니다. **Bash**를 선택합니다. 이 옵션이 표시되지 않으면 단계를 건너뜁니다.  

3. Cloud Shell용 스토리지를 만들라는 메시지가 표시되면 **고급 설정 표시**를 선택하고 다음 설정을 선택합니다.
    - **구독**: 사용자의 구독
    - **Cloud Shell 지역**: 사용 가능한 지역 선택
    - **VNet 격리 설정 표시**가 선택 취소됨
    - **리소스 그룹**: Azure OpenAI 리소스를 프로비전한 기존 리소스 그룹 사용
    - **스토리지 계정**: 고유한 이름으로 새 스토리지 계정 만들기
    - **파일 공유**: 고유한 이름으로 새 파일 공유 만들기

    그런 다음, 스토리지가 만들어질 때까지 1분 정도 기다립니다.

    > **참고**: Azure 구독에 Cloud Shell이 이미 설정되어 있는 경우 ⚙️ 메뉴에서 **사용자 설정 초기화** 옵션을 사용하여 최신 버전의 Python 및 .NET Framework가 설치되어 있는지 확인해야 할 수 있습니다.

4. Cloud Shell 창 왼쪽 위에 표시된 셸 형식이 *Bash*인지 확인합니다. *PowerShell*인 경우 드롭다운 메뉴를 사용하여 *Bash*로 전환합니다.

5. 터미널이 시작되면 다음 명령을 입력하여 샘플 애플리케이션을 다운로드하고 `azure-openai`(이)라는 폴더에 저장합니다.

    ```bash
    rm -r azure-openai -f
    git clone https://github.com/MicrosoftLearning/mslearn-openai azure-openai
    ```

6. 파일은 **azure-openai**라는 폴더에 다운로드됩니다. 다음 명령을 사용하여 이 연습의 랩 파일로 이동합니다.

    ```bash
    cd azure-openai/Labfiles/04-code-generation
    ```

7. 다음 명령을 실행하여 기본 제공 코드 편집기를 엽니다.

    ```bash
    code .
    ```

8. 코드 편집기에서 **샘플 코드** 폴더를 확장하고 애플리케이션이 모델을 사용하여 개선할 코드 파일을 검토합니다.

    > **팁**: Azure Cloud Shell 환경에서 파일 작업에 사용하는 방법에 대한 자세한 내용은 [Azure Cloud Shell 코드 편집기 설명서](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor)를 참조하세요.

## 애플리케이션 사용

이 연습에서는 Azure OpenAI 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다. C# 및 Python용 애플리케이션이 모두 제공되었습니다.

1. 코드 편집기에서 기본 설정 언어의 언어 폴더를 확장합니다.

2. 해당 언어에 대한 구성 파일을 엽니다.

    - **C#**: `appsettings.json`
    - **Python**: `.env`

3. 배포 이름뿐만 아니라 만든 Azure OpenAI 리소스의 **엔드포인트**와 **키** 배포 이름을 포함하도록 구성 값을 업데이트합니다. 파일을 저장합니다.

4. 콘솔 창에서 다음 명령을 입력하여 기본 설정 언어의 폴더로 이동하고 필요한 패키지를 설치합니다.

    **C#**

    ```bash
    cd CSharp
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.9
    ```

    **Python**

    ```bash
    cd Python
    pip install python-dotenv
    pip install openai==1.2.0
    ```

5. 언어에 대한 이 폴더의 코드 파일을 선택하고 필요한 라이브러리를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**: code-generation.py

    ```python
    # Add OpenAI import
    from openai import AzureOpenAI
    ```

6. 클라이언트를 구성하는 데 필요한 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    ```

    **Python**: code-generation.py

    ```python
    # Set OpenAI configuration settings
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2023-05-15"
            )
    ```

7. Azure OpenAI 모델을 호출하는 함수에서 형식을 지정하는 코드를 추가하고 모델에 요청을 보냅니다.

    **C#**: Program.cs

    ```csharp
    // Create chat completion options
    var chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatMessage(ChatRole.System, systemPrompt),
            new ChatMessage(ChatRole.User, userPrompt)
        },
        Temperature = 0.7f,
        MaxTokens = 1000,
        DeploymentName = oaiModelName
    };

    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);

    ChatCompletions completions = response.Value;
    string completion = completions.Choices[0].Message.Content;
    ```

    **Python**: code-generation.py

    ```python
    # Build the messages array
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    # Call the Azure OpenAI model
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=1000
    )
    ```

## 애플리케이션 실행

이제 앱이 구성되었으므로 이를 실행하여 각 사용 사례에 대한 코드를 생성해 봅니다. 사용 사례는 앱에서 번호가 매겨지며, 어떤 순서로든 실행할 수 있습니다.

> **참고**: 모델을 너무 자주 호출하는 경우 속도 제한이 발생할 수도 있습니다. 토큰 속도 제한에 대한 오류가 발생하면 1분 동안 기다린 다음 다시 시도합니다.

1. 코드 편집기에서 `sample-code` 폴더를 확장하고 언어에 대한 함수와 앱을 간략하게 확인합니다. 이러한 파일은 앱의 작업에 사용됩니다.
1. Cloud Shell bash 터미널에서 기본 설정 언어의 폴더로 이동합니다.
1. 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python code-generation.py`

1. 옵션 **1**을 선택하여 코드에 주석을 추가합니다. 이러한 각 작업에 응답이 몇 초 정도 걸릴 수 있습니다.
1. 결과가 `result/app.txt`에 입력됩니다. 해당 파일을 열고 `sample-code`의 함수 파일과 비교합니다.
1. 그런 다음 옵션 **2**를 선택하여 동일한 함수에 대한 단위 테스트를 작성합니다.
1. 결과는 `result/app.txt`에 있던 내용을 대체하며 해당 함수의 네 가지 단위 테스트를 자세히 설명합니다.
1. 다음으로, **3**을 선택하여 Go Fish를 재생하는 앱의 버그를 수정합니다.
1. 결과는 `result/app.txt`에 있던 내용을 대체하하며 몇 가지 항목이 수정된 매우 유사한 코드를 보여줍니다.

    - **C#**: 30줄과 59줄에서 수정됩니다
    - **Python**: 18줄과 31줄에서 수정됩니다

버그가 있는 줄을 Azure OpenAI가 제공한 응답으로 바꾸면 `sample-code`에 있는 Go Fish 앱을 실행할 수 있습니다. 수정 없이 실행하는 경우 앱이 제대로 작동하지 않습니다.

이 Go Fish 앱의 코드가 일부 구문에서 수정되었지만 이것이 게임을 정확한 나타내지는 않습니다. 자세히 살펴보면 카드를 뽑을 때 데크가 비어 있는지 확인하지 않거나, 페어를 얻을 때 플레이어의 패에서 페어를 제거하지 않는 등의 문제와, 실현할 카드 게임에 대한 이해가 필요한 몇 가지 버그가 있음을 알 수 있습니다. 이 예는 생성형 AI 모델이 코드 생성을 지원하는 데 유용하지만 정확한 것으로 신뢰할 수 없고 개발자의 검증이 필요함을 잘 보여줍니다.

Azure OpenAI의 전체 응답을 보려면 `printFullResponse` 변수를 `True`(으)로 설정하고 앱을 다시 실행하면 됩니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 [Azure Portal](https://portal.azure.com/?azure-portal=true)에서 배포 또는 전체 리소스를 삭제해야 합니다.
