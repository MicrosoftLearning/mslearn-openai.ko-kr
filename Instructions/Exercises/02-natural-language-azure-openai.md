---
lab:
  title: 앱에서 Azure OpenAI SDK 사용
---

# 앱에서 Azure OpenAI API 사용

개발자는 Azure OpenAI Service를 사용하여 자연어를 이해하는 데 탁월한 챗봇, 언어 모델 및 기타 애플리케이션을 만들 수 있습니다. Azure OpenAI는 미리 학습된 AI 모델에 대한 액세스를 제공할 뿐만 아니라 애플리케이션의 특정 요구 사항을 충족하도록 이러한 모델을 사용자 지정하고 미세 조정할 수 있는 API 및 도구 모음도 제공합니다. 이 연습에서는 Azure OpenAI에서 모델을 배포하고 이를 자체 애플리케이션에서 사용하는 방법을 알아봅니다.

이 연습의 시나리오에서 생성형 AI를 사용하여 하이킹 권장 사항을 제공할 수 있는 앱을 구현하는 임무를 맡은 소프트웨어 개발자의 역할을 수행하게 됩니다. 연습에 사용된 기술은 Azure OpenAI API를 사용하려는 모든 앱에 적용될 수 있습니다.

이 연습은 약 **30**분 정도 소요됩니다.

## Azure OpenAI 리소스 프로비전

아직 없는 경우 Azure 구독에서 Azure OpenAI 리소스를 프로비전합니다.

1. `https://portal.azure.com`에서 **Azure Portal**에 로그인합니다.
2. 다음 설정을 사용하여 **Azure OpenAI** 리소스를 만듭니다.
    - **구독**: *Azure OpenAI 서비스에 대한 액세스가 승인된 Azure 구독 선택*
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: *다음 지역 중 하나를 **임의로** 선택합니다.*\*
        - 오스트레일리아 동부
        - 캐나다 동부
        - 미국 동부
        - 미국 동부 2
        - 프랑스 중부
        - 일본 동부
        - 미국 중북부
        - 스웨덴 중부
        - 스위스 북부
        - 영국 남부
    - **이름**: ‘원하는 고유한 이름’**
    - **가격 책정 계층**: 표준 S0

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 제한됩니다. 나열된 지역에는 이 연습에 사용된 모델 형식에 대한 기본 할당량이 포함되어 있습니다. 지역을 임의로 선택하면 다른 사용자와 구독을 공유하는 시나리오에서 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

3. 배포가 완료될 때까지 기다립니다. 그런 다음, Azure Portal에서 배포된 Azure OpenAI 리소스로 이동합니다.

## 모델 배포

Azure는 모델을 배포, 관리 및 탐색하는 데 사용할 수 있는 **Azure AI Foundry 포털**이라는 웹 기반 포털을 제공합니다. Azure AI Foundry 포털을 사용하여 모델을 배포함으로써 Azure OpenAI 탐색을 시작합니다.

> **참고**: Azure AI Foundry 포털을 사용하면 수행할 작업을 제안하는 메시지 상자가 표시될 수 있습니다. 이를 닫고 이 연습의 단계를 따를 수 있습니다.

1. Azure Portal의 Azure OpenAI 리소스에 대한 **개요** 페이지에서 **시작** 섹션까지 아래로 스크롤하여 **AI Foundry 포털**(이전에는 AI 스튜디오)로 이동하는 단추를 선택합니다.
1. Azure OpenAI Foundry 의 왼쪽 창에서 **배포** 페이지를 선택하고 기존 모델 배포를 확인합니다. 아직 없는 경우 다음 설정을 사용하여 **gpt-35-turbo-16k** 모델의 새 배포를 만듭니다.
    - **배포 이름**: ‘원하는 고유한 이름’**
    - **모델**: gpt-35-turbo-16k *(16k 모델을 사용할 수 없는 경우 gpt-35-turbo 선택)*
    - **모델 버전**: *기본 버전 사용*
    - **배포 유형**: 표준
    - **분당 토큰 속도 제한**: 5K\*
    - **콘텐츠 필터**: 기본값
    - **동적 할당량 사용**: 사용할 수 없음

    > \* 분당 5,000개 토큰의 속도를 제한하더라도 동일한 구독을 사용하는 다른 사용자에게 용량을 남겨두면서 이 연습을 충분히 완료할 수 있습니다.

## Visual Studio Code에서 앱 개발 준비

Visual Studio Code를 사용하여 Azure OpenAI 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-openai** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-openai` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **참고**: Visual Studio Code에서 열려는 코드를 신뢰하라는 팝업 메시지가 표시되면 팝업에서 **예, 작성자를 신뢰합니다.** 옵션을 클릭합니다.

4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure OpenAI 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/02-azure-openai-api** 폴더를 찾아 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장합니다. 각 폴더에는 Azure OpenAI 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
2. 코드 파일이 포함된 **CSharp** 또는 **Python** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure OpenAI SDK 패키지를 설치합니다.

    **C#:**

    ```
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.14
    ```

    **Python**:

    ```
    pip install openai==1.55.3
    ```

3. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. 다음을 포함하도록 구성 값을 업데이트합니다.
    - 만든 Azure OpenAI 리소스의 **엔드포인트** 및 **키**(Azure Portal의 Azure OpenAI 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능)
    - 모델 배포에 대해 지정한 **배포 이름**(Azure AI Foundry 포털의 **배포** 페이지에서 사용 가능).
5. 구성 파일을 저장합니다.

## Azure OpenAI 서비스를 사용하기 위한 코드 추가

이제 Azure OpenAI SDK를 사용하여 배포된 모델을 사용할 준비가 되었습니다.

1. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 코드 파일을 열고 ***Azure OpenAI 패키지 추가*** 주석을 Azure OpenAI SDK 라이브러리를 추가하는 코드로 바꿉니다.

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```
    
    **Python**: test-openai-model.py
    
    ```python
    # Add Azure OpenAI package
    from openai import AzureOpenAI
    ```

1. 해당 언어의 애플리케이션 코드에서 ***Azure OpenAI 클라이언트 초기화...*** 주석을 다음 코드로 바꿔 클라이언트를 초기화하고 시스템 메시지를 정의합니다.

    **C#**: Program.cs

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    
    // System message to provide context to the model
    string systemMessage = "I am a hiking enthusiast named Forest who helps people discover hikes in their area. If no area is specified, I will default to near Rainier National Park. I will then provide three suggestions for nearby hikes that vary in length. I will also share an interesting fact about the local nature on the hikes when making a recommendation.";
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2024-02-15-preview"
            )
    
    # Create a system message
    system_message = """I am a hiking enthusiast named Forest who helps people discover hikes in their area. 
        If no area is specified, I will default to near Rainier National Park. 
        I will then provide three suggestions for nearby hikes that vary in length. 
        I will also share an interesting fact about the local nature on the hikes when making a recommendation.
        """
    ```

1. ***요청을 보낼 코드 추가...*** 주석을 요청 빌드에 필요한 코드로 바꿉니다. `messages` 및 `temperature`와 같은 모델에 대한 다양한 매개 변수를 지정합니다.

    **C#**: Program.cs

    ```csharp
    // Add code to send request...
    // Build completion options object
    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(systemMessage),
            new ChatRequestUserMessage(inputText),
        },
        MaxTokens = 400,
        Temperature = 0.7f,
        DeploymentName = oaiDeploymentName
    };

    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);

    // Print the response
    string completion = response.Choices[0].Message.Content;
    Console.WriteLine("Response: " + completion + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=400,
        messages=[
            {"role": "system", "content": system_message},
            {"role": "user", "content": input_text}
        ]
    )
    generated_text = response.choices[0].message.content

    # Print the response
    print("Response: " + generated_text + "\n")
    ```

1. 코드 파일에 변경 내용을 저장합니다.

## 애플리케이션 테스트

이제 앱이 구성되었으므로 앱을 실행하여 모델에 요청을 보내고 응답을 확인합니다.

1. 대화형 터미널 창에서 폴더 컨텍스트가 기본 설정 언어의 폴더인지 확인합니다. 그런 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

1. 메시지가 나타나면 `What hike should I do near Rainier?` 텍스트를 입력합니다.
1. 출력을 관찰하면서 응답이 *messages* 배열에 추가한 시스템 메시지에 제공된 지침을 따른다는 점에 유의해야 합니다.
1. 프롬프트 `Where should I hike near Boise? I'm looking for something of easy difficulty, between 2 to 3 miles, with moderate elevation gain.`을 제공하고 출력을 관찰합니다.
1. 기본 설정 언어의 코드 파일에서 요청의 *온도* 매개 변수 값을 **1.0**으로 변경하고 파일을 저장합니다.
1. 위의 프롬프트를 사용하여 애플리케이션을 다시 실행하고 출력을 관찰합니다.

온도를 높이면 동일한 텍스트가 제공되더라도 임의성 증가로 인해 응답이 달라지는 경우가 많습니다. 여러 번 실행하여 출력이 어떻게 변경되는지 확인할 수 있습니다. 동일한 입력으로 온도에 다른 값을 사용해 보세요.

## 대화 기록 유지

대부분의 실제 애플리케이션에서 대화의 이전 부분을 참조하는 기능을 사용하면 AI 에이전트와 보다 현실적인 상호 작용이 가능합니다. Azure OpenAI API는 기본적으로 상태 비저장이지만 프롬프트에 대화 기록을 제공하면 AI 모델이 과거 메시지를 참조할 수 있습니다.

1. 앱을 다시 실행하고 프롬프트 `Where is a good hike near Boise?`를 제공합니다.
1. 출력을 관찰한 후 `How difficult is the second hike you suggested?` 프롬프트를 표시합니다.
1. 모델의 응답은 언급한 하이킹을 이해할 수 없음을 나타낼 가능성이 높습니다. 이 문제를 해결하기 위해 모델이 참조용으로 과거 대화 메시지를 갖도록 할 수 있습니다.
1. 애플리케이션에서는 이전 메시지를 추가하고 앞으로 보낼 메시지에 대한 응답을 추가해야 합니다. **시스템 메시지** 정의 아래에 다음 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Initialize messages list
    var messagesList = new List<ChatRequestMessage>()
    {
        new ChatRequestSystemMessage(systemMessage),
    };
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize messages array
    messages_array = [{"role": "system", "content": system_message}]
    ```

1. ***요청을 보낼 코드 추가...*** 주석 아래에서 주석부터 **while** 루프 끝까지의 모든 코드를 다음 코드로 바꾼 후 파일을 저장합니다. 코드는 대부분 동일하지만 이제 메시지 배열을 사용하여 대화 기록을 저장합니다.

    **C#**: Program.cs

    ```csharp
    // Add code to send request...
    // Build completion options object
    messagesList.Add(new ChatRequestUserMessage(inputText));

    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        MaxTokens = 1200,
        Temperature = 0.7f,
        DeploymentName = oaiDeploymentName
    };

    // Add messages to the completion options
    foreach (ChatRequestMessage chatMessage in messagesList)
    {
        chatCompletionsOptions.Messages.Add(chatMessage);
    }

    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);

    // Return the response
    string completion = response.Choices[0].Message.Content;

    // Add generated text to messages list
    messagesList.Add(new ChatRequestAssistantMessage(completion));

    Console.WriteLine("Response: " + completion + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    messages_array.append({"role": "user", "content": input_text})

    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=1200,
        messages=messages_array
    )
    generated_text = response.choices[0].message.content
    # Add generated text to messages array
    messages_array.append({"role": "assistant", "content": generated_text})

    # Print generated text
    print("Summary: " + generated_text + "\n")
    ```

1. 파일을 저장합니다. 추가한 코드에서 이제 모델이 대화 기록을 이해할 수 있도록 프롬프트 배열에 이전 입력과 응답을 추가합니다.
1. 터미널 창에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

1. 앱을 다시 실행하고 프롬프트 `Where is a good hike near Boise?`를 제공합니다.
1. 출력을 관찰한 후 `How difficult is the second hike you suggested?` 프롬프트를 표시합니다.
1. 모델이 제안한 두 번째 하이킹에 대한 응답을 가져올 가능성이 높으며 이는 훨씬 더 현실적인 대화를 제공합니다. 이전 답변을 참조하여 추가 후속 질문을 할 수 있으며 기록이 모델이 답변할 컨텍스트를 제공할 때마다 가능합니다.

    > **팁**: 토큰 수는 1200으로만 설정되므로 대화가 너무 오래 지속되면 애플리케이션에 사용 가능한 토큰이 부족해 불완전한 프롬프트가 표시됩니다. 프로덕션 용도에서는 기록 길이를 가장 최근의 입력 및 응답으로 제한하면 필요한 토큰 수를 제어하는 데 도움이 됩니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 **Azure Portal**의 `https://portal.azure.com`에서 배포 또는 전체 리소스를 삭제해야 합니다.
