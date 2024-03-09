---
lab:
  title: 앱에서 프롬프트 엔지니어링 활용
---

# 앱에서 프롬프트 엔지니어링 활용

개발자는 Azure OpenAI Service를 사용하여 자연어를 이해하는 데 탁월한 챗봇, 언어 모델 및 기타 애플리케이션을 만들 수 있습니다. Azure OpenAI는 미리 학습된 AI 모델에 대한 액세스를 제공할 뿐만 아니라 애플리케이션의 특정 요구 사항을 충족하도록 이러한 모델을 사용자 지정하고 미세 조정할 수 있는 API 및 도구 모음도 제공합니다. 이 연습에서는 Azure OpenAI에서 모델을 배포하고 이를 자체 애플리케이션에서 사용하여 텍스트를 요약하는 방법을 알아봅니다.

Azure OpenAI Service를 사용하는 경우 개발자가 프롬프트를 형성하는 방식은 생성 AI 모델이 응답하는 방식에 큰 영향을 줍니다. 명확하고 간결한 방식으로 요청하면 Azure OpenAI 모델은 콘텐츠를 조정하고 서식을 지정할 수 있습니다. 이 연습에서는 유사한 콘텐츠에 대한 다양한 프롬프트가 요구 사항을 더 잘 충족하도록 AI 모델의 응답을 형성하는 데 어떻게 도움이 되는지 알아봅니다.

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

Azure OpenAI API를 사용하려면 먼저 **Azure OpenAI Studio**를 통해 사용할 모델을 배포해야 합니다. 배포되면 앱에서 해당 모델을 참조하세요.

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

## 채팅 플레이그라운드에서 프롬프트 엔지니어링 적용

앱을 사용하기 전에 프롬프트 엔지니어링이 플레이그라운드의 모델 응답을 개선하는 방법을 검토합니다. 이 첫 번째 예제에서는 재미있는 이름을 가진 동물에 대한 Python 앱을 작성하려고 한다고 가정해 봅니다.

1. [Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true)에서 왼쪽 창에 있는 **채팅** 플레이그라운드로 이동합니다.
1. **구성**에서 모델 배포가 선택되었는지 확인합니다.
1. 위쪽의 **도우미 설정** 섹션에서 `You are a helpful AI assistant`을(를) 시스템 메시지로 입력합니다.
1. **채팅 세션** 섹션에서 다음 프롬프트를 입력하고 *Enter*를 누릅니다.

    ```code
   1. Create a list of animals
   2. Create a list of whimsical names for those animals
   3. Combine them randomly into a list of 25 animal and name pairs
    ```

1. 모델은 프롬프트를 충족하기 위해 번호를 매긴 목록으로 나눈 답변으로 응답합니다. 이러한 응답은 좋지만 우리가 찾는 것은 아닙니다.
1. 다음으로, 지침 `You are an AI assistant helping write python code. Complete the app based on provided comments`을(를) 포함하도록 시스템 메시지를 업데이트합니다. **변경 내용 저장**을 클릭합니다.
1. 명령의 서식을 python 주석으로 지정합니다. 모델에 다음 프롬프트를 보냅니다.

    ```code
   # 1. Create a list of animals
   # 2. Create a list of whimsical names for those animals
   # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

1. 모델은 주석이 요청한 작업을 수행하는 전체 Python 코드로 올바르게 응답합니다.
1. 다음으로, 문서를 분류하려고 할 때 프롬프트가 거의 없는 경우의 영향을 살펴보겠습니다. 시스템 메시지로 돌아가서 `You are a helpful AI assistant`을(를) 다시 입력하고 변경 내용을 저장합니다. 그러면 새 채팅 세션이 생성됩니다.
1. 모델에 다음 프롬프트를 보냅니다.

    ```code
   Severe drought likely in California

   Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
   
   In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
   
   Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

1. 이에 대한 응답은 캘리포니아의 가뭄에 대한 정보가 될 것입니다. 나쁜 응답은 아니지만, 우리가 찾는 분류는 아닙니다.
1. 시스템 메시지 근처의 **도우미 설정** 섹션에서 **예 추가** 단추를 선택합니다. 다음 예를 추가합니다.

    **사용자**:

    ```code
   New York Baseballers Wins Big Against Chicago
   
   New York Baseballers mounted a big 5-0 shutout against the Chicago Cyclones last night, solidifying their win with a 3 run homerun late in the bottom of the 7th inning.
   
   Pitcher Mario Rogers threw 96 pitches with only two hits for New York, marking his best performance this year.
   
   The Chicago Cyclones' two hits came in the 2nd and the 5th innings, but were unable to get the runner home to score.
    ```

    **도우미:**

    ```code
   Sports
    ```

1. 다음 텍스트로 다른 예를 추가합니다.

    **사용자**:

    ```code
    Joyous moments at the Oscars
    
    The Oscars this past week where quite something!
    
    Though a certain scandal might have stolen the show, this year's Academy Awards were full of moments that filled us with joy and even moved us to tears.
    These actors and actresses delivered some truly emotional performances, along with some great laughs, to get us through the winter.
    
    From Robin Kline's history-making win to a full performance by none other than Casey Jensen herself, don't miss tomorrows rerun of all the festivities.
    ```

    **도우미:**

    ```code
    Entertainment
    ```

1. 도우미 설정에 대한 변경 내용을 저장하고 편의상 여기에 다시 제공된 캘리포니아 가뭄에 대한 동일한 프롬프트를 보냅니다.

    ```code
   Severe drought likely in California

   Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
   
   In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
   
   Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

1. 이번에는 모델이 지침 없이도 적절한 분류로 응답합니다.

## Cloud Shell에서 애플리케이션 설정

Azure OpenAI 모델과 통합하는 방법을 보여주기 위해 Azure의 Cloud Shell에서 실행되는 간단한 명령줄 애플리케이션을 사용할 예정입니다. Cloud Shell을 사용하려면 새 브라우저 탭을 엽니다.

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
   cd azure-openai/Labfiles/03-prompt-engineering
    ```

7. 다음 명령을 실행하여 기본 제공 코드 편집기를 엽니다.

    ```bash
    code .
    ```

8. 코드 편집기에서 **프롬프트** 폴더를 확장하고, 애플리케이션이 모델에 제출할 프롬프트가 포함된 텍스트 파일을 검토합니다.

    > **팁**: Azure Cloud Shell 환경에서 파일 작업에 사용하는 방법에 대한 자세한 내용은 [Azure Cloud Shell 코드 편집기 설명서](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor)를 참조하세요.

## 애플리케이션 사용

이 연습에서는 Azure OpenAI 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다. C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다.

1. 코드 편집기에서 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장합니다.

2. 해당 언어에 대한 구성 파일을 엽니다.

    - C#: `appsettings.json`
    - Python: `.env`
    
3. 만든 Azure OpenAI 리소스의 **엔드포인트** 및 **키**와 배포한 모델 이름을 포함하도록 구성 값을 업데이트합니다. 파일을 저장합니다.

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

5. 기본 설정 언어 폴더로 이동하여 코드 파일을 선택하고 필요한 라이브러리를 추가합니다.

    **C#**

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**

    ```python
    # Add OpenAI import
    from openai import AzureOpenAI
    ```

6. 언어에 대한 애플리케이션 코드를 열고 클라이언트를 구성하는 데 필요한 코드를 추가합니다.

    **C#**

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    ```

    **Python**

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2023-05-15"
            )
    ```

7. Azure OpenAI 모델을 호출하는 함수에서 형식을 지정하는 코드를 추가하고 모델에 요청을 보냅니다.

    **C#**

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
        MaxTokens = 800,
        DeploymentName = oaiModelName
    };
    
    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);
    
    ChatCompletions completions = response.Value;
    string completion = completions.Choices[0].Message.Content;
    ```

    **Python**

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
        max_tokens=800
    )
    ```

## 애플리케이션 실행

이제 앱이 구성되었으므로 앱을 실행하여 모델에 요청을 보내고 응답을 확인합니다. 서로 다른 옵션 간의 유일한 차이점은 프롬프트의 내용이며, 다른 모든 매개 변수(예: 토큰 수 및 온도)는 각 요청에서 동일하게 유지됩니다.

각 프롬프트는 전송될 때 콘솔에 표시되어 서로 다른 프롬프트 내용이 어떻게 다른 응답을 생성하는지 보여줍니다.

1. Cloud Shell bash 터미널에서 기본 설정 언어의 폴더로 이동합니다.
1. 애플리케이션을 실행하고, 브라우저 창의 대부분을 차지하도록 터미널을 확장합니다.

    - **C#**: `dotnet run`
    - **Python**: `python prompt-engineering.py`

1. 가장 기본적인 프롬프트에 옵션 **1**을 선택합니다.
1. 프롬프트 입력과 생성된 출력을 확인합니다. AI 모델은 야생 동물 구조에 대한 좋은 일반적인 소개를 생성할 가능성이 높습니다.
1. 다음으로, 옵션 **2**를 선택하여 야생 동물 구조에 대한 세부 정보가 포함된 소개 이메일을 요청하는 프롬프트를 제공합니다.
1. 프롬프트 입력과 생성된 출력을 확인합니다. 이번에는 특정 동물 이야기가 포함된 이메일 양식과 함께 기부 요청을 보게 될 수 있습니다.
1. 다음으로, 옵션 **3**을 선택하여 위와 유사한 이메일을 요청하되 추가 동물들이 있는 서식이 지정된 표를 함께 요청합니다.
1. 프롬프트 입력과 생성된 출력을 확인합니다. 이번에는 비슷한 이메일이 특정 방식으로 서식이 지정된 텍스트(이 경우는 끝 부분의 표)가 포함되어 나타날 것이고, 생성형 AI 모델이 요청을 받을 때 출력 형식을 어떻게 지정할 수 있는지 보여줍니다.
1. 다음으로, 옵션 **4**를 선택하여 비슷한 이메일을 요청하되 이번에는 시스템 메시지에서 다른 톤을 지정합니다.
1. 프롬프트 입력과 생성된 출력을 확인합니다. 이번에는 형식은 비슷하지만 어조는 훨씬 비공식적인 이메일을 볼 수 있습니다. 어쩌면 농담도 포함되어 있을 수 있습니다!

온도를 높이면 동일한 프롬프트가 제공되더라도 임의성 증가로 인해 응답이 달라지는 경우가 많습니다. 동일한 프롬프트를 서로 다른 온도 또는 top_p 값으로 여러 번 실행하여 응답이 어떻게 달라지는지 확인할 수 있습니다.

Azure OpenAI의 전체 응답을 보려면 `printFullResponse` 변수를 `True`(으)로 설정하고 앱을 다시 실행하면 됩니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 [Azure Portal](https://portal.azure.com/?azure-portal=true)에서 배포 또는 전체 리소스를 삭제해야 합니다.
