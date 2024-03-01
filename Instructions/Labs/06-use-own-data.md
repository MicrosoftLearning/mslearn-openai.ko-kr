---
lab:
  title: Azure OpenAI로 자체 데이터 사용
---

# Azure OpenAI로 자체 데이터 사용

Azure OpenAI Service를 사용하면 기본 LLM의 인텔리전스와 함께 자체 데이터를 사용할 수 있습니다. 관련 항목에 대한 데이터만 사용하도록 모델을 제한하거나 미리 학습된 모델의 결과와 혼합할 수 있습니다.

이 연습은 약 **20**분 정도 소요됩니다.

## 시작하기 전에

Azure OpenAI 서비스에 액세스하려면 승인된 Azure 구독이 필요합니다. 

- 무료 Azure 구독에 등록하려면 [https://azure.microsoft.com/free](https://azure.microsoft.com/free)를 참조하세요.
- Azure OpenAI 서비스에 대한 액세스를 요청하려면 [https://aka.ms/oaiapply](https://aka.ms/oaiapply)를 참조하세요.

## Azure OpenAI 리소스 프로비전

Azure OpenAI 모델을 사용하려면 먼저 Azure 구독에서 Azure OpenAI 리소스를 프로비전해야 합니다.

1. [Azure Portal](https://portal.azure.com?azure-portal=true)에 로그인합니다.
2. 다음 설정을 사용하여 **Azure OpenAI** 리소스를 만듭니다.
    - **구독**: Azure OpenAI 서비스에 대한 액세스가 승인된 Azure 구독입니다.
    - **리소스 그룹**: 기존 리소스 그룹을 선택하거나 원하는 이름으로 새 리소스 그룹을 만듭니다.
    - **지역**: 사용 가능한 지역을 선택합니다.
    - **이름**: 원하는 고유한 이름.
    - **가격 책정 계층**: 표준 S0
3. 배포가 완료될 때까지 기다립니다. 그런 다음, Azure Portal에서 배포된 Azure OpenAI 리소스로 이동합니다.

## 모델 배포

Azure OpenAI와 채팅하려면 먼저 **Azure OpenAI Studio**를 통해 사용할 모델을 배포해야 합니다. 배포되면 플레이그라운드에서 모델을 사용하고 데이터를 사용하여 응답을 기반으로 합니다.

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

## 고유의 데이터를 추가하지 않고 일반적인 채팅 동작을 확인합니다.

Azure OpenAI를 데이터에 연결하기 전에 먼저 기본 모델이 기반 데이터 없이 쿼리에 어떻게 응답하는지 확인합니다.

1. **채팅** 플레이그라운드로 이동하여 배포한 모델이 **구성** 창에서 선택되어 있는지 확인합니다(배포된 모델이 하나만 있는 경우 이것이 기본값이어야 함).
1. 다음 프롬프트를 입력하고 출력을 확인합니다.

    ```code
    I'd like to take a trip to New York. Where should I stay?
    ```

    ```code
    What are some facts about New York?
    ```

1. 런던이나 샌프란시스코 등 기반 데이터에 포함될 다른 위치의 관광 및 숙박 장소에 대해 유사한 질문을 시도해 보세요. 지역이나 인근 지역에 대한 완전한 답변과 도시에 대한 몇 가지 일반적인 팩트를 가져올 수 있습니다.

## 채팅 플레이그라운드에서 데이터 연결

다음으로 채팅 플레이그라운드에 데이터를 추가하여 기반 데이터로 어떻게 반응하는지 확인합니다.

1. GitHub에서 사용할 [데이터를 다운로드](https://aka.ms/own-data-brochures)합니다. 제공된 `.zip`에서 PDF를 추출합니다.
1. **채팅** 플레이그라운드로 이동하여 도우미 설정 창에서 *데이터 추가*를 선택합니다.
1. **데이터 원본 추가**를 선택하고 드롭다운에서 *파일 업로드*를 선택합니다.
1. 스토리지 계정과 Azure AI 검색 리소스를 만들어야 합니다. 스토리지 리소스 드롭다운에서 **새 Azure Blob Storage 리소스 만들기**를 선택하고 다음 설정으로 스토리지 계정을 만듭니다. 지정하지 않은 항목은 기본값으로 그대로 둡니다.

    - **구독**: *Azure OpenAI 리소스와 동일한 구독*
    - **리소스 그룹**: *Azure OpenAI 리소스와 동일한 리소스 그룹*
    - **스토리지 계정 이름**: *전역적으로 고유한 이름을 입력합니다.*
    - **지역**: *Azure OpenAI 리소스와 동일한 지역*
    - **중복도**: LRS(로컬 중복 스토리지)

1. 리소스가 만들어지면 Azure OpenAI Studio로 돌아와 다음 설정을 사용하여 **새 Azure AI 검색 리소스 만들기**를 선택합니다. 지정하지 않은 항목은 기본값으로 그대로 둡니다.

    - **구독**: *Azure OpenAI 리소스와 동일한 구독*
    - **리소스 그룹**: *Azure OpenAI 리소스와 동일한 리소스 그룹*
    - **서비스 이름**: *전역적으로 고유한 이름을 입력합니다.*
    - **위치**: *Azure OpenAI 리소스와 동일한 위치*
    - **가격 책정 계층**: 기본

1. 검색 리소스가 배포될 때까지 기다린 다음, Azure AI Studio로 다시 전환하고 페이지를 새로 고칩니다.
1. **데이터 추가**에서 데이터 원본에 대해 다음 값을 입력하고 **다음**을 선택합니다.

    - **데이터 원본 선택**: 파일 업로드
    - **Azure Blob Storage 리소스 선택**: *만든 스토리지 리소스를 선택합니다.*
        - 메시지가 표시되면 CORS를 켭니다.
    - **Azure AI 검색 리소스 선택**: *만든 검색 리소스를 선택합니다.*
    - **인덱스 이름 입력**: margiestravel
    - **이 검색 리소스에 벡터 검색 추가**: 선택 취소됨
    - **Azure AI 검색 계정에 연결하면 내 계정이 사용된다는 점을 인정합니다.**: 선택됨

1. **파일 업로드** 페이지에서 다운로드한 PDF를 업로드하고 **다음**을 선택합니다.
1. **데이터 관리** 페이지의 드롭다운 메뉴에서 **키워드** 검색 유형을 선택한 후 **다음**을 선택합니다.
1. **검토 및 완료** 페이지에서 **저장 후 닫기**를 선택하면 데이터가 추가됩니다. 이 작업은 몇 분 정도 걸릴 수 있으므로 창을 열어 두어야 합니다. 완료되면 **도우미 설정** 창에 지정된 데이터 원본, 검색 리소스 및 인덱스가 표시됩니다.

## 데이터를 기반으로 한 모델과 채팅

이제 데이터를 추가했으므로 이전과 동일한 질문을 하고 응답이 어떻게 다른지 확인합니다.

```
I'd like to take a trip to New York. Where should I stay?
```

```
What are some facts about New York?
```

이번에는 특정 호텔에 대한 구체적인 내용, Margie's Travel에 대한 언급, 제공된 정보의 출처에 대한 언급 등 매우 다른 반응을 보게 될 것입니다. 응답에 나열된 PDF 참조를 열면 제공된 모델과 동일한 호텔이 표시됩니다.

두바이, 라스베거스, 런던, 샌프란시스코 등 기반 데이터에 포함된 다른 도시에 대해 요청합니다.

> **참고**: **데이터 추가**는 아직 미리 보기 상태이며 기반 데이터에 포함되지 않은 도시에 대한 잘못된 참조를 제공하는 것과 같이 이 기능이 항상 예상대로 작동하는 것은 아닙니다.

## 앱을 고유의 데이터에 연결

다음으로 앱을 연결하여 자체 데이터를 사용하는 방법을 살펴봅니다.

### Cloud Shell에서 애플리케이션 설정

Azure OpenAI 앱을 사용자 데이터에 연결하는 방법을 보여 주기 위해 Azure의 Cloud Shell에서 실행되는 간단한 명령줄 애플리케이션을 사용할 예정입니다. Cloud Shell을 사용하려면 새 브라우저 탭을 엽니다.

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
    cd azure-openai/Labfiles/06-use-own-data
    ```

7. 다음 명령을 실행하여 기본 제공 코드 편집기를 엽니다.

    ```bash
    code .
    ```

    > **팁**: Azure Cloud Shell 환경에서 파일 작업에 사용하는 방법에 대한 자세한 내용은 [Azure Cloud Shell 코드 편집기 설명서](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor)를 참조하세요.

## 애플리케이션 사용

이 연습에서는 Azure OpenAI 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다. C# 및 Python용 애플리케이션이 모두 제공되었습니다. 두 앱 모두 동일한 기능을 제공합니다.

1. 코드 편집기에서 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장합니다.

2. 해당 언어에 대한 구성 파일을 엽니다.

    - C#: `appsettings.json`
    - Python: `.env`
    
3. 다음을 포함하도록 구성 값을 업데이트합니다.
    - 만든 Azure OpenAI 리소스의 **엔드포인트** 및 **키**(Azure Portal의 Azure OpenAI 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능)
    - 모델 배포에 대해 지정한 이름(Azure OpenAI Studio의 **배포** 페이지에서 사용 가능)
    - 검색 서비스의 엔드포인트(Azure Portal의 검색 리소스 개요 페이지에 있는 **Url** 값)
    - 검색 리소스용 **키**(Azure Portal의 검색 리소스에 대한 **키** 페이지에서 사용 가능 - 관리자 키 중 하나를 사용할 수 있음)
    - 검색 인덱스의 이름(`margiestravel`이어야 함).
    
4. 업데이트된 구성 파일을 저장합니다.

5. 콘솔 창에서 다음 명령을 입력하여 기본 설정 언어의 폴더로 이동하고 필요한 패키지를 설치합니다.

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

6. 코드 편집기에서 기본 설정 언어 폴더로 이동하여 코드 파일을 선택하고 필요한 라이브러리를 추가합니다.

    **C#**: OwnData.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**: ownData.py

    ```python
    # Add OpenAI import
    from openai import AzureOpenAI
    ```

7. 특히 API 호출에 대한 매개 변수를 완료할 때 검색 값이 사용되는 코드 파일을 검토합니다.

## 애플리케이션 실행

이제 앱이 구성되었으므로 앱을 실행하여 모델에 요청을 보내고 응답을 확인합니다. 이제 스튜디오 환경과 유사하게 응답이 데이터에 기반을 두고 있음을 알 수 있습니다.

1. Cloud Shell bash 터미널에서 기본 설정 언어의 폴더로 이동합니다.
1. 브라우저 창의 대부분을 차지하도록 터미널을 확장하고 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python ownData.py`

1. `Tell me about London` 프롬프트를 제출하면 데이터를 참조하는 응답이 표시됩니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 [Azure Portal](https://portal.azure.com/?azure-portal=true)에서 리소스를 삭제해야 합니다. 상대적으로 큰 비용이 발생할 수 있으므로 스토리지 계정 및 검색 리소스도 포함해야 합니다.
