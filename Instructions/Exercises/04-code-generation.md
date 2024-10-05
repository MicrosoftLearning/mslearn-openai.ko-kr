---
lab:
  title: Azure OpenAI Service를 사용하여 코드 생성 및 개선
---

# Azure OpenAI Service를 사용하여 코드 생성 및 개선

Azure OpenAI Service 모델은 자연어 프롬프트를 사용하여 코드를 생성하고, 작성한 코드의 버그를 수정하고, 코드 주석을 제공할 수 있습니다. 이러한 모델은 기존 코드에 대해서도 그 기능과 개선 방법을 이해할 수 있도록 설명하고 간소화할 수 있습닌다.

이 연습의 시나리오에서 생성형 AI를 사용하여 코딩 작업을 보다 쉽고 효율적으로 만드는 방법을 탐구하는 소프트웨어 개발자의 역할을 수행하게 됩니다. 연습에 사용된 기술은 다른 코드 파일, 프로그래밍 언어 및 사용 사례에 적용될 수 있습니다.

이 연습은 약 **25**분 정도 소요됩니다.

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

Azure는 모델을 배포, 관리 및 탐색하는 데 사용할 수 있는 **Azure AI 스튜디오**라는 웹 기반 포털을 제공합니다. Azure AI 스튜디오를 사용하여 모델을 배포함으로써 Azure OpenAI 탐색을 시작합니다.

> **참고**: Azure AI 스튜디오를 사용하면 수행할 작업을 제안하는 메시지 상자가 표시될 수 있습니다. 이를 닫고 이 연습의 단계를 따를 수 있습니다.

1. Azure Portal의 Azure OpenAI 리소스에 대한 **개요** 페이지에서 **시작** 섹션까지 아래로 스크롤하여 **AI 스튜디오**로 이동하는 단추를 선택합니다.
1. Azure OpenAI 스튜디오의 왼쪽 창에서 **배포** 페이지를 선택하고 기존 모델 배포를 확인합니다. 아직 없는 경우 다음 설정을 사용하여 **gpt-35-turbo-16k** 모델의 새 배포를 만듭니다.
    - **배포 이름**: ‘원하는 고유한 이름’**
    - **모델**: gpt-35-turbo-16k *(16k 모델을 사용할 수 없는 경우 gpt-35-turbo 선택)*
    - **모델 버전**: *기본 버전 사용*
    - **배포 유형**: 표준
    - **분당 토큰 속도 제한**: 5K\*
    - **콘텐츠 필터**: 기본값
    - **동적 할당량 사용**: 사용할 수 없음

    > \* 분당 5,000개 토큰의 속도를 제한하더라도 동일한 구독을 사용하는 다른 사용자에게 용량을 남겨두면서 이 연습을 충분히 완료할 수 있습니다.

## 채팅 플레이그라운드에서 코드 생성

앱에서 사용하기 전에 Azure OpenAI가 채팅 플레이그라운드에서 어떻게 코드를 생성하고 설명하는지 검토합니다.

1. **플레이그라운드** 섹션에서 **채팅** 페이지를 선택합니다. **채팅** 플레이그라운드 페이지는 단추 행과 두 가지 기본 패널로 구성됩니다(화면 해상도에 따라 오른쪽에서 왼쪽으로 수평으로 배열되거나 위에서 아래로 수직으로 배열될 수 있음).
    - **구성** - 배포를 선택하고, 시스템 메시지를 정의하고, 배포와 상호 작용하기 위한 매개 변수를 설정하는 데 사용됩니다.
    - **채팅 세션** - 채팅 메시지를 제출하고 응답을 보는 데 사용됩니다.
1. **배포**에서 모델 배포가 선택되어 있는지 확인합니다.
1. **시스템 메시지** 영역에서 시스템 메시지를 `You are a programming assistant helping write code`(으)로 설정하고 변경 내용을 적용합니다.
1. **채팅 세션**에서 다음 쿼리를 제출합니다.

    ```
    Write a function in python that takes a character and a string as input, and returns how many times the character appears in the string
    ```

    모델은 함수가 수행하는 작업과 함수를 호출하는 방법에 대한 설명과 함께 함수로 응답할 가능성이 높습니다.

1. 다음으로 프롬프트 `Do the same thing, but this time write it in C#`을(를) 보냅니다.

    모델은 처음과 매우 유사하게 응답했겠지만 이번에는 C#으로 코딩했습니다. 다른 언어로 다시 요청하거나, 입력 문자열을 되돌리는 것과 같은 다른 작업을 완료하는 함수로 다시 요청해 볼 수 있습니다.

1. 다음으로 AI를 사용하여 코드를 이해하는 방법을 살펴보겠습니다. 사용자 메시지로 다음 프롬프트를 제출합니다.

    ```
    What does the following function do?  
    ---  
    def multiply(a, b):  
        result = 0  
        negative = False  
        if a < 0 and b > 0:  
            a = -a  
            negative = True  
        elif a > 0 and b < 0:  
            b = -b  
            negative = True  
        elif a < 0 and b < 0:  
            a = -a  
            b = -b  
        while b > 0:  
            result += a  
            b -= 1      
        if negative:  
            return -result  
        else:  
            return result  
    ```

    모델은 함수가 수행하는 작업, 즉 루프를 사용하여 두 숫자를 곱하는 작업을 설명해야 합니다.

7. 프롬프트 `Can you simplify the function?`을 제출합니다.

    모델은 함수의 더 간단한 버전을 작성해야 합니다.

8. 프롬프트 제출: `Add some comments to the function.`

    모델은 코드에 주석을 추가합니다.

## Visual Studio Code에서 앱 개발 준비

이제 Azure OpenAI 서비스를 사용하여 코드를 생성하는 사용자 지정 앱을 빌드하는 방법을 살펴보겠습니다. 여기서는 Visual Studio Code를 사용하여 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-openai** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-openai` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **참고**: Visual Studio Code에서 열려는 코드를 신뢰하라는 팝업 메시지가 표시되면 팝업에서 **예, 작성자를 신뢰합니다.** 옵션을 클릭합니다.

4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션과 요약을 테스트하는 데 사용할 샘플 텍스트 파일이 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure OpenAI 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/04-code-generation** 폴더를 찾아 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장합니다. 각 폴더에는 Azure OpenAI 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
2. 코드 파일이 포함된 **CSharp** 또는 **Python** 폴더를 마우스 오른쪽 단추로 클릭하고 통합 터미널을 엽니다. 그런 다음, 언어 선택에 적절한 명령을 실행하여 Azure OpenAI SDK 패키지를 설치합니다.

    **C#:**

    ```
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.14
    ```

    **Python**:

    ```
    pip install openai==1.13.3
    ```

3. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 구성 파일을 엽니다.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. 다음을 포함하도록 구성 값을 업데이트합니다.
    - 만든 Azure OpenAI 리소스의 **엔드포인트** 및 **키**(Azure Portal의 Azure OpenAI 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능)
    - 모델 배포에 대해 지정한 **배포 이름**(Azure AI 스튜디오의 **배포** 페이지에서 사용 가능)
5. 구성 파일을 저장합니다.

## Azure OpenAI 서비스 모델을 사용하기 위한 코드 추가

이제 Azure OpenAI SDK를 사용하여 배포된 모델을 사용할 준비가 되었습니다.

1. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 코드 파일을 엽니다. Azure OpenAI 모델을 호출하는 함수의 ***형식을 지정하고 모델에 요청 보내기*** 주석 아래에 형식을 지정하고 모델에 요청을 보내는 코드를 추가합니다.

    **C#**: Program.cs

    ```csharp
    // Format and send the request to the model
    var chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(systemPrompt),
            new ChatRequestUserMessage(userPrompt)
        },
        Temperature = 0.7f,
        MaxTokens = 1000,
        DeploymentName = oaiDeploymentName
    };

    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);

    ChatCompletions completions = response.Value;
    string completion = completions.Choices[0].Message.Content;
    ```

    **Python**: code-generation.py

    ```python
    # Format and send the request to the model
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

4. 변경 내용을 코드 파일에 저장합니다.

## 애플리케이션 실행

이제 앱이 구성되었으므로 이를 실행하여 각 사용 사례에 대한 코드를 생성해 봅니다. 사용 사례는 앱에서 번호가 매겨지며, 어떤 순서로든 실행할 수 있습니다.

> **참고**: 모델을 너무 자주 호출하는 경우 속도 제한이 발생할 수도 있습니다. 토큰 속도 제한에 대한 오류가 발생하면 1분 동안 기다린 다음 다시 시도합니다.

1. **탐색기** 창에서 **Labfiles/04-code-generation/sample-code** 폴더를 확장하고 해당 언어에 대한 함수와 *go-fish* 앱을 검토합니다. 이러한 파일은 앱의 작업에 사용됩니다.
2. 대화형 터미널 창에서 폴더 컨텍스트가 기본 설정 언어의 폴더인지 확인합니다. 그런 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python code-generation.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

3. 코드에 주석을 추가하려면 옵션 **1**을 선택하고 다음 프롬프트를 입력합니다. 이러한 각 작업에 응답이 몇 초 정도 걸릴 수 있습니다.

    ```prompt
    Add comments to the following function. Return only the commented code.\n---\n
    ```

    결과는 **result/app.txt**에 저장됩니다. 해당 파일을 열고 **샘플 코드**의 함수 파일과 비교합니다.

4. 그런 다음 옵션 **2**를 선택하여 동일한 함수에 대한 단위 테스트를 작성하고 다음 프롬프트를 입력합니다.

    ```prompt
    Write four unit tests for the following function.\n---\n
    ```

    결과는 **result/app.txt**에 있던 내용을 바꾸고 해당 함수에 대한 4가지 단위 테스트를 자세히 설명합니다.

5. 다음으로, **3**을 선택하여 Go Fish를 재생하는 앱의 버그를 수정합니다. 다음 프롬프트를 입력합니다.

    ```prompt
    Fix the code below for an app to play Go Fish with the user. Return only the corrected code.\n---\n
    ```

    결과는 **result/app.txt**에 있던 내용을 바꾸며 몇 가지 사항이 수정된 매우 유사한 코드를 갖게 됩니다.

    - **C#**: 30줄과 59줄에서 수정됩니다
    - **Python**: 18줄과 31줄에서 수정됩니다

    버그가 포함된 줄을 Azure OpenAI의 응답으로 바꾸면 **샘플 코드**의 Go Fish용 앱을 실행할 수 있습니다. 수정 없이 실행하는 경우 앱이 제대로 작동하지 않습니다.
    
    > **참고**: 이 Go Fish 앱의 코드가 일부 구문에서 수정되었지만 이것이 게임을 정확한 나타내지는 않습니다. 자세히 살펴보면 카드를 뽑을 때 데크가 비어 있는지 확인하지 않거나, 페어를 얻을 때 플레이어의 패에서 페어를 제거하지 않는 등의 문제와, 실현할 카드 게임에 대한 이해가 필요한 몇 가지 버그가 있음을 알 수 있습니다. 이 예는 생성형 AI 모델이 코드 생성을 지원하는 데 유용하지만 정확한 것으로 신뢰할 수 없고 개발자의 검증이 필요함을 잘 보여줍니다.

    Azure OpenAI의 전체 응답을 보려면 **printFullResponse** 변수를 `True`로 설정하고 앱을 다시 실행합니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 **Azure Portal**의 `https://portal.azure.com`에서 배포 또는 전체 리소스를 삭제해야 합니다.
