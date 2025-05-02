---
lab:
  title: Azure OpenAI Service를 사용하여 애플리케이션 개발
  status: new
---

# Azure OpenAI Service를 사용하여 애플리케이션 개발

개발자는 Azure OpenAI Service REST API 또는 언어별 SDK를 사용하여 자연어를 이해하는 데 탁월한 챗봇 및 기타 애플리케이션을 만들 수 있습니다. 이 언어 모델을 사용하는 경우 개발자가 프롬프트를 형성하는 방식은 생성형 AI 모델이 응답하는 방식에 큰 영향을 줍니다. 명확하고 간결한 방식으로 요청하면 Azure OpenAI 모델은 콘텐츠를 조정하고 서식을 지정할 수 있습니다. 이 연습에서는 애플리케이션을 Azure OpenAI에 연결하는 방법과 유사한 콘텐츠에 대한 다양한 프롬프트가 요구 사항을 더 잘 충족하도록 AI 모델의 응답을 형성하는 데 어떻게 도움이 되는지 알아봅니다.

이 연습 시나리오에서는 야생동물 마케팅 캠페인을 진행하는 소프트웨어 개발자의 역할을 수행하게 됩니다. 사생성형 AI를 사용하여 보급 이메일을 개선하고 팀에 적용할 수 있는 문서를 분류하는 방법을 탐색하고 있습니다. 연습에 사용된 프롬프트 엔지니어링 기술은 다양한 사용 사례에 유사하게 적용될 수 있습니다.

이 연습은 약 **30**분 정도 소요됩니다.

## 이 과정용 리포지토리 복제

이 과정용 코드 리포지토리를 아직 복제하지 않았으면 복제해야 합니다.

1. Visual Studio Code 시작
2. 명령 팔레트(SHIFT+CTRL+P 또는 **보기** > **명령 팔레트...**)를 열고 **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-openai` 리포지토리를 로컬 폴더에 복제합니다(어떤 폴더든 상관없습니다).
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## Azure OpenAI 리소스 프로비전

아직 없는 경우 Azure 구독에서 Azure OpenAI 리소스를 프로비전합니다.

1. `https://portal.azure.com`에서 **Azure Portal**에 로그인합니다.

1. 다음 설정을 사용하여 **Azure OpenAI** 리소스를 만듭니다.
    - **구독**: *Azure OpenAI 서비스에 대한 액세스가 승인된 Azure 구독 선택*
    - **리소스 그룹**: *리소스 그룹 선택 또는 만들기*
    - **지역**: *다음 지역 중 하나를 **임의로** 선택합니다.*\*
        - 미국 동부
        - 미국 동부 2
        - 미국 중북부
        - 미국 중남부
        - 스웨덴 중부
        - 미국 서부
        - 미국 서부 3
    - **이름**: ‘원하는 고유한 이름’**
    - **가격 책정 계층**: 표준 S0

    > \* Azure OpenAI 리소스는 지역 할당량에 따라 제한됩니다. 나열된 지역에는 이 연습에 사용된 모델 형식에 대한 기본 할당량이 포함되어 있습니다. 지역을 임의로 선택하면 다른 사용자와 구독을 공유하는 시나리오에서 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. 배포가 완료될 때까지 기다립니다. 그런 다음, Azure Portal에서 배포된 Azure OpenAI 리소스로 이동합니다.

## 모델 배포

다음으로 Cloud Shell에서 Azure OpenAI 모델 리소스를 배포합니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***Bash*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *PowerShell* 환경을 사용하는 Cloud Shell을 만든 경우 ***Bash***로 전환합니다.

1. 이 예제를 사용하여 다음 변수를 사용자 고유의 값으로 바꿉니다.

    ```dotnetcli
    az cognitiveservices account deployment create \
       -g <your_resource_group> \
       -n <your_OpenAI_service> \
       --deployment-name gpt-4o \
       --model-name gpt-4o \
       --model-version 2024-05-13 \
       --model-format OpenAI \
       --sku-name "Standard" \
       --sku-capacity 5
    ```

> **참고**: Sku 용량은 분당 수천 개의 토큰 단위로 측정됩니다. 분당 5,000토큰의 속도 제한은 동일한 구독을 사용하는 다른 사용자들을 위해 용량을 남겨두면서 이 연습을 충분히 완료할 수 있습니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션이 모두 제공되었으며 두 앱 모두 동일한 기능을 제공합니다. 먼저, 비동기 API 호출로 Azure OpenAI 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/01-app-develop** 폴더를 찾은 다음 선택한 언어에 맞게 **CSharp** 또는 **Python** 폴더를 확장합니다. 각 폴더에는 Azure OpenAI 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
2. 코드 파일이 포함된 **CSharp** 또는 **Python** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure OpenAI SDK 패키지를 설치합니다.

    **C#:**

    ```powershell
    dotnet add package Azure.AI.OpenAI --version 2.1.0
    ```

    **Python**:

    ```powershell
    pip install openai==1.65.2
    ```

3. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env

4. 다음을 포함하도록 구성 값을 업데이트합니다.
    - 만든 Azure OpenAI 리소스의 **엔드포인트** 및 **키**(Azure Portal의 Azure OpenAI 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능)
    - 모델 배포에 대해 지정한 **배포 이름**입니다.
5. 구성 파일을 저장합니다.

## Azure OpenAI 서비스를 사용하기 위한 코드 추가

이제 Azure OpenAI SDK를 사용하여 배포된 모델을 사용할 준비가 되었습니다.

1. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 코드 파일을 열고 ***Azure OpenAI 패키지 추가*** 주석을 Azure OpenAI SDK 라이브러리를 추가하는 코드로 바꿉니다.

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI packages
    using Azure.AI.OpenAI;
    using OpenAI.Chat;
    ```

    **Python**: application.py

    ```python
    # Add Azure OpenAI package
    from openai import AsyncAzureOpenAI
    ```

2. 코드 파일에서 ***Azure OpenAI 클라이언트 구성*** 주석을 찾아 Azure OpenAI 클라이언트를 구성하는 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Configure the Azure OpenAI client
    AzureOpenAIClient azureClient = new (new Uri(oaiEndpoint), new ApiKeyCredential(oaiKey));
    ChatClient chatClient = azureClient.GetChatClient(oaiDeploymentName);
    ```

    **Python**: application.py

    ```python
    # Configure the Azure OpenAI client
    client = AsyncAzureOpenAI(
        azure_endpoint = azure_oai_endpoint, 
        api_key=azure_oai_key,  
        api_version="2024-02-15-preview"
    )
    ```

3. Azure OpenAI 모델을 호출하는 함수에서 ***Azure OpenAI 응답을 가져오는*** 주석 아래에 형식을 지정하는 코드를 추가하고 모델에 요청을 보냅니다.

    **C#**: Program.cs

    ```csharp
    // Get response from Azure OpenAI
    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        [
            new SystemChatMessage(systemMessage),
            new UserChatMessage(userMessage)
        ],
        chatCompletionOptions
    );

    Console.WriteLine($"{completion.Role}: {completion.Content[0].Text}");
    ```

    **Python**: application.py

    ```python
    # Get response from Azure OpenAI
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    print("\nSending request to Azure OpenAI model...\n")

    # Call the Azure OpenAI model
    response = await client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=800
    )
    ```

4. 변경 내용을 코드 파일에 저장합니다.

## 애플리케이션 실행

이제 앱이 구성되었으므로 앱을 실행하여 모델에 요청을 보내고 응답을 확인합니다. 서로 다른 옵션 간의 유일한 차이점은 프롬프트의 내용이며, 다른 모든 매개 변수(예: 토큰 수 및 온도)는 각 요청에서 동일하게 유지됩니다.

1. Visual Studio Code에서 기본 설정 언어 폴더의 `system.txt`를 엽니다. 각 상호 작용에 대해 이 파일에 **시스템 메시지**를 입력하고 저장합니다. 각 반복은 시스템 메시지를 변경할 수 있도록 먼저 일시 중지됩니다.
1. 대화형 터미널 창에서 폴더 컨텍스트가 기본 설정 언어의 폴더인지 확인합니다. 그런 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python application.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

1. 첫 번째 반복의 경우 다음 프롬프트를 입력합니다.

    **시스템 메시지**

    ```prompt
    You are an AI assistant
    ```

    **사용자 메시지:**

    ```prompt
    Write an intro for a new wildlife Rescue
    ```

1. 출력을 확인합니다. AI 모델은 야생 동물 구조에 대한 좋은 일반적인 소개를 생성할 가능성이 높습니다.
1. 그런 다음, 응답 형식을 지정하는 다음 프롬프트를 입력합니다.

    **시스템 메시지**

    ```prompt
    You are an AI assistant helping to write emails
    ```

    **사용자 메시지:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants 
    - Call for donations to be given at our website
    ```

    > **팁**: 다중 라인 프롬프트에서 VM의 자동 입력이 제대로 작동하지 않을 수 있습니다. 이 문제가 있는 경우 전체 프롬프트를 복사한 다음 Visual Studio Code에 붙여넣습니다.

1. 출력을 확인합니다. 이번에는 특정 동물 이야기가 포함된 이메일 양식과 함께 기부 요청을 보게 될 수 있습니다.
1. 그런 다음, 콘텐츠를 추가로 지정하는 다음 프롬프트를 입력합니다.

    **시스템 메시지**

    ```prompt
    You are an AI assistant helping to write emails
    ```

    **사용자 메시지:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

1. 출력을 관찰하고 명확한 지침에 따라 이메일이 어떻게 변경되었는지 확인합니다.
1. 그런 다음, 시스템 메시지에 톤에 대한 세부 정보를 추가하는 다음 프롬프트를 입력합니다.

    **시스템 메시지**

    ```prompt
    You are an AI assistant that helps write promotional emails to generate interest in a new business. Your tone is light, chit-chat oriented and you always include at least two jokes.
    ```

    **사용자 메시지:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

1. 출력을 확인합니다. 이번에는 형식은 비슷하지만 어조는 훨씬 비공식적인 이메일을 볼 수 있습니다. 어쩌면 농담도 포함되어 있을 수 있습니다!

## 그라운딩 컨텍스트 사용 및 채팅 기록 유지 관리

1. 마지막 반복에서는 이메일 생성에서 벗어나 *그라운딩 컨텍스트*를 탐색하고 채팅 기록을 유지합니다. 여기에서 간단한 시스템 메시지를 제공하고, 채팅 기록의 시작 부분으로 그라운딩 컨텍스트를 제공하도록 앱을 변경합니다. 그런 다음 앱은 사용자 입력을 추가하고 근거 있는 컨텍스트에서 정보를 추출하여 사용자 프롬프트에 답변합니다.
1. 파일 `grounding.txt`를 열고 삽입할 근거 있는 컨텍스트를 간략하게 읽습니다.
1. 앱에서 ***메시지 목록 초기화*** 주석 바로 뒤와 기존 코드 앞에 다음 코드 스니펫을 추가하여 `grounding.txt`에서 텍스트를 읽어와서 채팅 기록을 그라운딩 컨텍스트로 초기화합니다.

    **C#**: Program.cs

    ```csharp
    // Initialize messages list
    Console.WriteLine("\nAdding grounding context from grounding.txt");
    string groundingText = System.IO.File.ReadAllText("grounding.txt");
    var messagesList = new List<ChatMessage>()
    {
        new UserChatMessage(groundingText),
    };
    ```

    **Python**: application.py

    ```python
    # Initialize messages array
    print("\nAdding grounding context from grounding.txt")
    grounding_text = open(file="grounding.txt", encoding="utf8").read().strip()
    messages_array = [{"role": "user", "content": grounding_text}]
    ```

1. ***요청을 포맷하고 모델로 전송*** 주석 아래에서 **while** 루프 끝까지의 코드를 아래 코드로 교체합니다. 코드는 대부분 동일하지만 이제 메시지 배열을 사용하여 모델에 요청을 전송합니다.

    **C#**: Program.cs
   
    ```csharp
    // Format and send the request to the model
    messagesList.Add(new SystemChatMessage(systemMessage));
    messagesList.Add(new UserChatMessage(userMessage));
    GetResponseFromOpenAI(messagesList);
    ```

    **Python**: application.py

    ```python
    # Format and send the request to the model
    messages_array.append({"role": "system", "content": system_text})
    messages_array.append({"role": "user", "content": user_text})
    await call_openai_model(messages=messages_array, 
        model=azure_oai_deployment, 
        client=client
    )
    ```

1. ***Azure OpenAI 엔드포인트로부터 응답을 가져오는 함수 정의*** 주석 아래의 함수 선언 부분을 다음 코드로 교체하여, C#의 `GetResponseFromOpenAI` 또는 Python의 `call_openai_model` 함수 호출 시 채팅 기록 리스트를 사용하도록 합니다.

    **C#**: Program.cs
   
    ```csharp
    // Define the function that gets the response from Azure OpenAI endpoint
    private static void GetResponseFromOpenAI(List<ChatMessage> messagesList)
    ```

    **Python**: application.py

    ```python
    # Define the function that will get the response from Azure OpenAI endpoint
    async def call_openai_model(messages, model, client):
    ```
    
1. 마지막으로 ***Azure OpenAI에서 응답 받기*** 아래의 모든 코드를 교체합니다. 코드는 대부분 동일하지만 이제 메시지 배열을 사용하여 대화 기록을 저장합니다.

    **C#**: Program.cs
   
    ```csharp
    // Get response from Azure OpenAI
    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        messagesList,
        chatCompletionOptions
    );

    Console.WriteLine($"{completion.Role}: {completion.Content[0].Text}");
    messagesList.Add(new AssistantChatMessage(completion.Content[0].Text));
    ```

    **Python**: application.py

    ```python
    # Get response from Azure OpenAI
    print("\nSending request to Azure OpenAI model...\n")

    # Call the Azure OpenAI model
    response = await client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=800
    )   

    print("Response:\n" + response.choices[0].message.content + "\n")
    messages.append({"role": "assistant", "content": response.choices[0].message.content})
    ```
    
1. 파일을 저장하고 앱을 다시 실행합니다.
1. 다음 프롬프트를 입력합니다(**시스템 메시지**가 계속 입력되어 `system.txt`에 저장되어 있음).

    **시스템 메시지**

    ```prompt
    You're an AI assistant who helps people find information. You'll provide answers from the text provided in the prompt, and respond concisely.
    ```

    **사용자 메시지:**

    ```prompt
    What animal is the favorite of children at Contoso?
    ```

   모델이 그라운딩 텍스트 정보를 사용하여 질문에 답변하는 것을 확인할 수 있습니다.

1. 시스템 메시지를 변경하지 않고 사용자 메시지에 대해 다음 프롬프트를 입력합니다.

    **사용자 메시지:**

    ```prompt
    How can they interact with it at Contoso?
    ```

    이제는 채팅 기록에서 이전 질문에 액세스할 수 있으므로 모델은 "그들"을 아이들로 인식하고 "그것"을 아이들이 좋아하는 동물로 인식합니다.
   
## 정리

Azure OpenAI 리소스 사용이 완료되면 **Azure Portal**의 `https://portal.azure.com`에서 배포 또는 전체 리소스를 삭제해야 합니다.
