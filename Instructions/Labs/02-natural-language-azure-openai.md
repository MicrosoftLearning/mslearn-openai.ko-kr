---
lab:
  title: 앱에 Azure OpenAI 통합
---

# 앱에 Azure OpenAI 통합

Azure OpenAI Service를 사용하면 개발자는 자연어를 이해하는 데 탁월한 챗봇, 언어 모델 및 기타 애플리케이션을 만들 수 있습니다. Azure OpenAI는 미리 학습된 AI 모델에 대한 액세스를 제공할 뿐만 아니라 애플리케이션의 특정 요구 사항을 충족하도록 이러한 모델을 사용자 지정하고 미세 조정하기 위한 API 및 도구 모음도 제공합니다. 이 연습에서는 Azure OpenAI에서 모델을 배포하고 이를 자체 애플리케이션에서 사용하여 텍스트를 요약하는 방법을 알아봅니다.

이 연습은 약 **30**분 정도 소요됩니다.

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

1. Azure OpenAI 리소스의 **개요** 페이지에서 **탐색** 단추를 사용하여 새 브라우저 탭에서 Azure OpenAI Studio를 엽니다.
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

## Cloud Shell에서 애플리케이션 설정

Azure OpenAI 모델과 통합하는 방법을 보여 주기 위해 Azure의 Cloud Shell에서 실행되는 간단한 명령줄 애플리케이션을 사용할 예정입니다. Cloud Shell을 사용하려면 새 브라우저 탭을 엽니다.

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

5. 터미널이 시작되면 다음 명령을 입력하여 샘플 애플리케이션을 다운로드하고 `azure-openai`라는 폴더에 저장합니다.

    ```bash
   rm -r azure-openai -f
   git clone https://github.com/MicrosoftLearning/mslearn-openai azure-openai
    ```
  
6. 파일은 **azure-openai**라는 폴더에 다운로드됩니다. 다음 명령을 사용하여 이 연습의 랩 파일로 이동합니다.

    ```bash
   cd azure-openai/Labfiles/02-nlp-azure-openai
    ```

7. 다음 명령을 실행하여 기본 제공 코드 편집기를 엽니다.

    ```bash
    code .
    ```

8. 코드 편집기에서 **text-files** 폴더를 확장하고 **sample-text.txt**를 선택하여 모델을 사용하여 요약할 텍스트를 확인합니다.

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

6. 해당 언어에 대한 애플리케이션 코드를 열고 요청을 빌드하는 데 필요한 코드를 추가합니다. 이는 `prompt` 및 `temperature`와 같은 모델에 대한 다양한 매개 변수를 지정합니다.

    **C#**

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    
    // Build completion options object
    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatMessage(ChatRole.System, "You are a helpful assistant."),
            new ChatMessage(ChatRole.User, "Summarize the following text in 20 words or less:\n" + text),
        },
        MaxTokens = 120,
        Temperature = 0.7f,
        DeploymentName = oaiModelName
    };
    
    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);
    string completion = response.Choices[0].Message.Content;
    
    Console.WriteLine("Summary: " + completion + "\n");
    ```

    **Python**

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2023-05-15"
            )
    
    # Send request to Azure OpenAI model
    response = client.chat.completions.create(
        model=azure_oai_model,
        temperature=0.7,
        max_tokens=120,
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Summarize the following text in 20 words or less:\n" + text}
        ]
    )
    
    print("Summary: " + response.choices[0].message.content + "\n")
    ```

## 애플리케이션 실행

이제 앱이 구성되었으므로 앱을 실행하여 모델에 요청을 보내고 응답을 확인합니다.

1. Cloud Shell bash 터미널에서 기본 설정 언어의 폴더로 이동합니다.
1. 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

1. 샘플 텍스트 파일의 요약을 확인합니다.
1. 기본 설정 언어의 코드 파일로 이동하여 *온도* 값을 `1`로 변경합니다. 파일을 저장합니다.
1. 애플리케이션을 다시 실행하고 출력을 확인합니다.

온도를 높이면 임의성 증가로 인해 동일한 텍스트가 제공되더라도 요약이 달라지는 경우가 많습니다. 여러 번 실행하여 출력이 어떻게 변경되는지 확인할 수 있습니다. 동일한 입력으로 온도에 다른 값을 사용해 보세요.

## 정리

Azure OpenAI 리소스 사용이 완료되면 [Azure Portal](https://portal.azure.com?azure-portal=true)에서 배포 또는 전체 리소스를 삭제해야 합니다.
