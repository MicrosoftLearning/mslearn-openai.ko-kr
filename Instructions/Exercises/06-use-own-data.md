---
lab:
  title: Azure OpenAI Service를 사용하여 RAG(검색 증강 생성) 구현
---

# Azure OpenAI Service를 사용하여 RAG(검색 증강 생성) 구현

Azure OpenAI Service를 사용하면 기본 LLM의 인텔리전스와 함께 자체 데이터를 사용할 수 있습니다. 관련 항목에 대한 데이터만 사용하도록 모델을 제한하거나 미리 학습된 모델의 결과와 혼합할 수 있습니다.

이 연습 시나리오에서 Margie's Travel Agency에서 근무하는 소프트웨어 개발자의 역할을 수행하게 됩니다. Azure AI 검색을 사용하여 사용자 고유의 데이터를 인덱싱하고 Azure OpenAI와 함께 사용하여 프롬프트를 보강하는 방법을 살펴봅니다.

이 연습은 약 **30**분 정도 소요됩니다.

## Azure 리소스 프로비전

이 연습을 완료하려면 다음이 필요합니다.

- Azure OpenAI 리소스
- Azure AI 검색 리소스입니다.
- Azure Storage 계정 리소스.

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

3. Azure OpenAI 리소스를 프로비저닝하는 동안 다음 설정을 사용하여 **Azure AI Search** 리소스를 만듭니다.
    - **구독**: *Azure OpenAI 리소스를 프로비저닝한 구독*
    - **리소스 그룹**: *Azure OpenAI 리소스를 프로비전한 리소스 그룹*
    - **서비스 이름**: *원하는 고유한 이름*
    - **위치**: *Azure OpenAI 리소스를 프로비저닝한 지역*
    - **가격 책정 계층**: 기본
4. Azure AI Search 리소스를 프로비저닝하는 동안 다음 설정을 사용하여 **Storage 계정** 리소스를 만듭니다.
    - **구독**: *Azure OpenAI 리소스를 프로비저닝한 구독*
    - **리소스 그룹**: *Azure OpenAI 리소스를 프로비전한 리소스 그룹*
    - **스토리지 계정 이름**: *원하는 고유한 이름*
    - **위치**: *Azure OpenAI 리소스를 프로비저닝한 지역*
    - **성능**: 표준
    - **중복도**: LRS(로컬 중복 스토리지)
5. 세 리소스가 모두 Azure 구독에 성공적으로 배포된 후 Azure Portal에서 검토하고 다음 정보를 수집합니다(연습의 뒷부분에서 필요함).
    - 만든 Azure OpenAI 리소스의 **엔드포인트** 및 **키**(Azure Portal의 Azure OpenAI 리소스에 대한 **키 및 엔드포인트** 페이지에서 사용 가능)
    - Azure AI 검색 서비스의 엔드포인트(Azure Portal의 Azure AI 검색 리소스 개요 페이지에 있는 **Url** 값).
    - Azure AI 검색 리소스의 **기본 관리 키**(Azure Portal의 Azure AI 검색 리소스에 대한 **키** 페이지에서 사용 가능).

## 데이터 업로드

사용자 고유의 데이터를 사용하여 생성형 AI 모델에서 사용하는 프롬프트를 그라운딩합니다. 이 연습에서, 데이터는 가상의 *Margies Travel* 회사의 여행 브로셔 컬렉션으로 구성됩니다.

1. 새 브라우저 탭에서 `https://aka.ms/own-data-brochures`의 브로셔 데이터 보관을 다운로드합니다. PC의 폴더에 브로셔를 추출합니다.
1. Azure Portal에서 스토리지 계정 및 **스토리지 브라우저** 페이지 보기로 이동합니다.
1. **Blob 컨테이너**를 선택한 다음 명명된 새 컨테이너를 추가합니다.`margies-travel`
1. **margies-travel** 컨테이너를 선택한 다음 이전에 추출한 .pdf 브로슈어를 Blob 컨테이너의 루트 폴더에 업로드합니다.

## AI 모델 배포

이 연습에서는 두 개의 AI 모델을 사용합니다.

- 그라운딩 프롬프트에서 사용하기 위해 효율적으로 인덱싱할 수 있도록 브로슈어의 텍스트를 *벡터화하는* 텍스트 포함 모델입니다.
- 애플리케이션에서 데이터에 그라운딩된 프롬프트에 대한 응답을 생성하는 데 사용할 수 있는 GPT 모델입니다.

이러한 모델을 배포하려면 AI Studio를 사용합니다.

1. Azure Portal에서 Azure OpenAI 리소스로 이동합니다. 그런 다음, 링크를 사용하여 **Azure AI 스튜디오**에서 리소스를 엽니다.
1. Azure AI 스튜디오의 **배포** 페이지에서 기존 모델 배포를 확인합니다. 그런 다음, 다음 설정을 사용하여 **text-embedding-ada-002** 모델의 새 기본 모델 배포를 만듭니다.
    - **배포 이름**: text-embedding-ada-002
    - **모델**: text-embedding-ada-002
    - **모델 버전**: *기본 버전*
    - **배포 유형**: 표준
    - **분당 토큰 속도 제한**: 5K\*
    - **콘텐츠 필터**: 기본값
    - **동적 할당량 사용**: 사용
1. 텍스트 포함 모델을 배포한 후 **배포 ** 페이지로 돌아가서 다음 설정을 사용하여 **gpt-35-turbo-16k** 모델의 새 배포를 만듭니다.
    - **배포 이름**: gpt-35-turbo-16k
    - **모델**: gpt-35-turbo-16k *(16k 모델을 사용할 수 없는 경우 gpt-35-turbo 선택)*
    - **모델 버전**: *기본 버전*
    - **배포 유형**: 표준
    - **분당 토큰 속도 제한**: 5K\*
    - **콘텐츠 필터**: 기본값
    - **동적 할당량 사용**: 사용

    > \* 분당 5,000개 토큰의 속도를 제한하더라도 동일한 구독을 사용하는 다른 사용자에게 용량을 남겨두면서 이 연습을 충분히 완료할 수 있습니다.

## 인덱스 만들기

프롬프트에서 사용자 고유의 데이터를 쉽게 사용할 수 있도록 Azure AI 검색을 사용하여 인덱싱합니다. 인덱싱 프로세스 중에 이전에 배포한 텍스트 포함 모델을 사용하여 텍스트 데이터를 *벡터화*합니다(인덱스의 각 텍스트 토큰이 숫자 벡터로 표현됨으로써 생성형 AI 모델이 텍스트를 나타내는 방식과 호환되도록 합니다).

1. Azure Portal에서 Azure AI 검색 리소스로 이동합니다.
1. **개요** 페이지에서 **데이터 가져오기 및 벡터화**를 선택합니다.
1. **데이터 연결 설정** 페이지에서 **Azure Blob Storage**를 선택하고 다음 설정을 사용하여 데이터 원본을 구성합니다.
    - **구독**: 스토리지 계정을 프로비저닝한 Azure 구독입니다.
    - **Blob Storage 계정**: 이전에 만든 스토리지 계정을 선택합니다.
    - **Blob 컨테이너**: margies-travel
    - **Blob 폴더**: *비워두기*
    - **삭제 추적 사용**: 선택되지 않음
    - **관리 ID를 사용하여 인증**: 선택되지 
1. **텍스트 벡터화** 페이지에서 다음 설정을 선택합니다.
    - **종류**: Azure OpenAI
    - **구독**: Azure OpenAI 서비스를 프로비저닝한 Azure 구독입니다.
    - **Azure OpenAI Service**: Azure OpenAI Service 리소스
    - **모델 배포**: text-embedding-ada-002
    - **인증 유형**: API 키
    - **Azure OpenAI Service에 연결하면 내 계정에 추가 비용이 발생한다는 내용을 확인했음**: 선택됨
1. 다음 페이지에서 이미지를 벡터화하거나 AI 기술을 사용하여 데이터를 추출할 toptions를 선택하지 <u>마세요</u>.
1. 다음 페이지에서 의미 체계 순위를 사용하도록 설정하고 인덱서가 한 번 실행되도록 예약합니다.
1. 마지막 페이지에서 **개체 이름 접두사**를`margies-index` 설정한 다음 인덱스를 만듭니다.

## Visual Studio Code에서 앱 개발 준비

이제 Azure OpenAI 서비스 SDK를 사용하는 앱에서 자체 데이터를 사용하는 방법을 살펴보겠습니다. 여기서는 Visual Studio Code를 사용하여 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-openai** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
2. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-openai` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **참고**: Visual Studio Code에서 열려는 코드를 신뢰하라는 팝업 메시지가 표시되면 팝업에서 **예, 작성자를 신뢰합니다.** 옵션을 클릭합니다.

4. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## 애플리케이션 사용

C# 및 Python용 애플리케이션이 모두 제공되었으며 두 앱 모두 동일한 기능을 제공합니다. 먼저, Azure OpenAI 리소스를 사용할 수 있도록 애플리케이션의 일부 주요 부분을 완료합니다.

1. Visual Studio Code의 **탐색기** 창에서 **Labfiles/06-use-own-data** 폴더를 찾아 언어 선택에 따라 **CSharp** 또는 **Python** 폴더를 확장합니다. 각 폴더에는 Azure OpenAI 기능을 통합할 앱에 대한 언어별 파일이 포함되어 있습니다.
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
    - 모델 배포에 대해 지정한 **배포 이름**(Azure AI 스튜디오의 **배포** 페이지에서 사용 가능).
    - 검색 서비스의 엔드포인트(Azure Portal의 검색 리소스 개요 페이지에 있는 **Url** 값)
    - 검색 리소스용 **키**(Azure Portal의 검색 리소스에 대한 **키** 페이지에서 사용 가능 - 관리자 키 중 하나를 사용할 수 있음)
    - 검색 인덱스의 이름(`margies-index`이어야 함).
5. 구성 파일을 저장합니다.

### Azure OpenAI 서비스를 사용하기 위한 코드 추가

이제 Azure OpenAI SDK를 사용하여 배포된 모델을 사용할 준비가 되었습니다.

1. **탐색기** 창의 **CSharp** 또는 **Python** 폴더에서 기본 설정 언어에 대한 코드 파일을 열고 ***데이터 원본 구성*** 주석을 Azure OpenAI SDK 라이브러리를 추가하는 코드로 바꿉니다.

    **C#**: ownData.cs

    ```csharp
    // Configure your data source
    AzureSearchChatExtensionConfiguration ownDataConfig = new()
    {
            SearchEndpoint = new Uri(azureSearchEndpoint),
            Authentication = new OnYourDataApiKeyAuthenticationOptions(azureSearchKey),
            IndexName = azureSearchIndex
    };
    ```

    **Python**: ownData.py

    ```python
    # Configure your data source
    extension_config = dict(dataSources = [  
            { 
                "type": "AzureCognitiveSearch", 
                "parameters": { 
                    "endpoint":azure_search_endpoint, 
                    "key": azure_search_key, 
                    "indexName": azure_search_index,
                }
            }]
        )
    ```

2. 나머지 코드를 검토하고 데이터 원본 설정에 대한 정보를 제공하는 데 사용되는 요청 본문의 *확장* 사용에 유의해야 합니다.

3. 변경 내용을 코드 파일에 저장합니다.

## 애플리케이션 실행

이제 앱이 구성되었으므로 앱을 실행하여 모델에 요청을 보내고 응답을 확인합니다. 서로 다른 옵션 간의 유일한 차이점은 프롬프트의 내용이며, 다른 모든 매개 변수(예: 토큰 수 및 온도)는 각 요청에서 동일하게 유지됩니다.

1. 대화형 터미널 창에서 폴더 컨텍스트가 기본 설정 언어의 폴더인지 확인합니다. 그런 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    - **C#**: `dotnet run`
    - **Python**: `python ownData.py`

    > **팁**: 터미널 도구 모음의 **패널 크기 최대화**(**^**) 아이콘을 사용하면 더 많은 콘솔 텍스트를 볼 수 있습니다.

2. 프롬프트 `Tell me about London`에 대한 응답을 검토합니다. 여기에는 검색 서비스에서 가져오는 프롬프트를 기반으로 하는 데 사용된 데이터의 일부 세부 정보와 답변이 포함되어야 합니다.

    > **팁**: 검색 인덱스에서 인용을 보려면 코드 파일 상단 근처의 ***인용 표시*** 변수를 **true**로 설정합니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 **Azure Portal**의 `https://portal.azure.com`에서 리소스를 삭제해야 합니다. 상대적으로 큰 비용이 발생할 수 있으므로 스토리지 계정 및 검색 리소스도 포함해야 합니다.
