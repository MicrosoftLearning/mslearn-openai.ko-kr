---
lab:
  title: Azure OpenAI 시작
---

# Azure OpenAI Service 시작

Azure OpenAI Service는 OpenAI에서 개발한 생성 AI 모델을 Azure 플랫폼에 제공하여 Azure 클라우드 플랫폼에서 제공하는 서비스의 보안, 확장성 및 통합의 이점을 활용하는 강력한 AI 솔루션을 개발할 수 있도록 합니다. 이 연습에서는 서비스를 Azure 리소스로 프로비전하고 Azure OpenAI Studio를 사용하여 OpenAI 모델을 배포 및 탐색하여 Azure OpenAI를 시작하는 방법을 알아봅니다.

이 연습에는 약 **30**분이 소요됩니다.

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

## 모델 배포

Azure OpenAI는 모델을 배포, 관리 및 탐색하는 데 사용할 수 있는 **Azure OpenAI Studio**라는 웹 기반 포털을 제공합니다. Azure OpenAI Studio를 사용하여 모델을 배포함으로써 Azure OpenAI 탐색을 시작합니다.

1. Azure OpenAI 리소스의 **개요** 페이지에서 **Azure OpenAI Studio로 이동** 단추를 사용하여 새 브라우저 탭에서 Azure OpenAI Studio를 엽니다.
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

## 채팅 플레이그라운드 사용

*채팅* 플레이그라운드는 GPT 3.5 이상 모델에 대한 챗봇 인터페이스를 제공합니다. 이전 *Completions* API 대신 *ChatCompletions* API를 사용합니다.

1. **플레이그라운드** 섹션에서 **채팅** 페이지를 선택하고 구성 창에서 모델이 선택되었는지 확인합니다.
2. **도우미 설정** 섹션의 **시스템 메시지** 상자에서 현재 텍스트를 다음 문으로 바꿉니다. `The system is an AI teacher that helps people learn about AI`.

3. **시스템 메시지** 상자 아래에서 **퓨샷 예 추가**를 클릭하고 지정된 상자에 다음 메시지와 응답을 입력합니다.

    - **사용자**: `What are different types of artificial intelligence?`
    - **도우미**: `There are three main types of artificial intelligence: Narrow or Weak AI (such as virtual assistants like Siri or Alexa, image recognition software, and spam filters), General or Strong AI (AI designed to be as intelligent as a human being. This type of AI does not currently exist and is purely theoretical), and Artificial Superintelligence (AI that is more intelligent than any human being and can perform tasks that are beyond human comprehension. This type of AI is also purely theoretical and has not yet been developed).`

    > **참고**: 예상되는 응답 형식의 예를 모델에 제공하는 데 퓨샷 예를 사용합니다. 이 모델은 예의 어조와 스타일을 자체 응답에 반영하려고 시도합니다.

4. 변경 내용을 저장하여 새 세션을 시작하고 채팅 시스템의 동작 컨텍스트를 설정합니다.
5. 페이지 하단의 쿼리 상자에 `What is artificial intelligence?` 텍스트를 입력합니다.
6. 메시지를 제출하고 응답을 보려면 **보내기** 단추를 사용합니다.

    > **참고**: API 배포가 아직 준비되지 않았다는 응답을 받을 수 있습니다. 그렇다면 몇 분 정도 기다렸다가 다시 시도합니다.

7. 응답을 검토한 후 다음 메시지를 제출하여 대화를 계속합니다. `How is it related to machine learning?` 
8. 이전 상호 작용의 컨텍스트가 유지된다는 점에 주목하여 응답을 검토합니다. 따라서 모델은 "it"이 인공 지능을 의미한다는 것을 이해합니다.
9. **코드 보기** 단추를 사용하여 상호 작용 코드를 봅니다. 프롬프트는 *시스템* 메시지, *사용자*, *보조* 메시지 및 지금까지 채팅 세션의 *사용자* 및 *도우미* 메시지의 시퀀스의 퓨샷 예로 구성됩니다.

## 프롬프트 및 매개 변수 탐색

프롬프트와 매개 변수를 사용하여 필요한 응답을 생성할 가능성을 최대화할 수 있습니다.

1. **매개 변수** 창에서 다음 매개 변수 값을 설정합니다.
    - **Temperature**: 0
    - **최대 응답**: 500

2. 다음 메시지를 제출합니다.

    ```
    Write three multiple choice questions based on the following text.

    Most computer vision solutions are based on machine learning models that can be applied to visual input from cameras, videos, or images.*

    - Image classification involves training a machine learning model to classify images based on their contents. For example, in a traffic monitoring solution you might use an image classification model to classify images based on the type of vehicle they contain, such as taxis, buses, cyclists, and so on.*

    - Object detection machine learning models are trained to classify individual objects within an image, and identify their location with a bounding box. For example, a traffic monitoring solution might use object detection to identify the location of different classes of vehicle.*

    - Semantic segmentation is an advanced machine learning technique in which individual pixels in the image are classified according to the object to which they belong. For example, a traffic monitoring solution might overlay traffic images with "mask" layers to highlight different vehicles using specific colors.
    ```

3. 교사가 프롬프트의 Computer Vision 항목에 대해 학생들을 테스트하는 데 사용할 수 있는 객관식 질문으로 구성되어야 하는 결과를 검토합니다. 전체 응답은 매개 변수로 지정한 최대 길이보다 작아야 합니다.

    사용한 프롬프트와 매개 변수에 대해 다음 사항을 확인합니다.

    - 프롬프트에는 원하는 출력이 세 개의 객관식 질문이어야 한다고 구체적으로 명시되어 있습니다.
    - 매개 변수에는 응답 생성에 임의 요소가 포함되는 정도를 제어하는 *온도*가 포함됩니다. 제출 시 사용된 **0** 값은 임의성을 최소화하여 안정적이고 예측 가능한 응답을 제공합니다.

## 코드 생성 살펴보기

자연어 응답을 생성하는 것 외에도 GPT 모델을 사용하여 코드를 생성할 수 있습니다.

1. **도우미 설정** 창에서 **빈 예제** 템플릿을 선택하여 시스템 메시지를 다시 설정합니다.
2. 시스템 메시지: `You are a Python developer.`를 입력하고 변경 내용을 저장합니다.
3. **채팅 세션** 창에서 **채팅 지우기**를 선택하여 채팅 기록을 지우고 새 세션을 시작합니다.
4. 다음 사용자 메시지를 제출합니다.

    ```
    Write a Python function named Multiply that multiplies two numeric parameters.
    ```

5. 프롬프트의 요구 사항을 충족하는 샘플 Python 코드가 포함되어야 하는 응답을 검토합니다.

## 정리

Azure OpenAI 리소스 사용이 완료되면 [Azure Portal](https://portal.azure.com)에서 배포 또는 전체 리소스를 삭제해야 합니다.
