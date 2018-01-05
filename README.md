# Azure Functions

Azure Functions란 마이크로소프트에서 제공하는 서버리스 서비스 입니다. 서버리스 서비스라는 말을 요즘 많이 들어보셨을텐데요, 문자그대로 해석하면 서버가 없다? 당연히 서버는 존재하죠! 하지만 클라우드 서비스에서 이 또한 관리해주고 추상화하여 제공하기 때문에 이를 사용하는 사용자 입장에서는 정말 나의 코드조각만 신경쓰면 됩니다. 이를 잘 표현하기 위해 이름에 Function이란 단어가 포함되었네요! 

유사한 다른 벤더사의 제품으로는 AWS Lambda를 들 수 있습니다. 서버리스 서비스가 주목을 받고 있는 이유는 이를 이용하여 개발시 여러가지 장점을 가저다 주기 때문인데요, 그 중 하나가 확장성입니다. 이벤트 기반으로 동작하는 서비스에 Azure Function이 적합해요. 또한 과금 정책도 매우 합리적이랍니다! 딱 내 Function이 수행되는 만큼만 돈을 지불하면 되거든요! 매우 합리적이죠?

Azure Functions의 장점을 소개하자면, 여러 Binding 옵션을 제공합니다. 이벤트 기반으로 동작하는 서비스에 사용하는 것이 적잘하다고 말씀드렸는데요, 여러 Azure 상의 서비스와 Binding 할 수 있도록 제공하고 있습니다. 예를들어 이미지 리사이징 서비스를 만든다고 가정했을때 Azure Blob Storage와 바인딩하여, 스토리지에 이미지가 업데이트 되면 동작하도록 손쉽게 구성할 수 있습니다. 
이 밖에도 Azure의 Logic App과 궁합이 좋습니다. Azure Logic Apps는 다양한 3rd Party 서비스와의 커넥터 서비스를 제공하는데 이를 적절히 활용한다면 매우 손이 많이 가는 서비스도 쉽게 만들어보실 수 있습니다. 
또한 Azure Functions이 수행되는 런타임을 오픈소스로 공개하였기 때문에 오프라인 디버깅 및 개발이 가능하고 원하는 경우 자신이 가지고 있는 온프레미스 장비에서도 Azure Functions를 수행하실 수 있습니다. 

Azure Functions를 사용하기 위해 별도로 Library나 Framework을 공부하실 필요가 전혀 없어요! Binding 하는 부분 및, Azure Functions가 수행되기 위한 조건 설정에 대한 약간의 지식만 있으시다면 아주 편리하게 사용하실 수 있습니다. 오늘 실습을 통해 이러한 부분들을 같이 연습해 볼 예정이에요 :)


## 실습내용

Blob Storage에 사용자가 이미지를 업로드 하면 Azure Function을 이용하여 이미지 파일을 Cognitive Vision API를 이용하여 분석하고 부적절한 이미지가 업로드 되면 reject 컨테이너에 이미지를 별도로 저장한다. reject 컨테이너에 이미지가 업로드되면 Logic Apps를 활용하여 관리자에게 안내 메일을 전송한다. 

## 실습 준비 

(Windows, Mac 공통)
* Azure 계정 활성화 ([portal.azure.com](portal.azure.com) 로그인 및 리소스 생성 가능 상태)
  - Azure Pass를 사용하시는 경우 [링크](https://www.microsoftazurepass.com/)를 참고하세요 
* [Node 8.5 이상 (Node Version 9 이하여야 함)](https://nodejs.org/ko/download/)
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)
* [.NET Core](https://www.microsoft.com/net/learn/get-started2) 

## 실습 과정
### 1. Azure Portal에서 Azure Functions 만들기

node package 추가하기

1. Kudu 접속
2. wwwroot/ 하위 디렉토리에서 **npm init**명령어 이용하여 **package.json** 생성
3. **npm install --save request-promise** 명령어 입력하여 필요로하는 패키지 추가

코드
```
var request = require('request-promise');

module.exports = function (context, myBlob) {
    context.log("JavaScript blob trigger function processed blob \n Name:", context.bindingData.name, "\n Blob Size:", myBlob.length, "Bytes");
    
    var options = getAnalysisOptions(myBlob, <key>, <endpoint>);
    analyzeAndProcessImage(context, options);
    
    function getAnalysisOptions(image, subscriptionKey, endpoint) {
        return  {
            uri: endpoint + "/analyze?visualFeatures=Description",
            method: 'POST',
            body: image,
            headers: {
                'Content-Type': 'application/octet-stream',
                'Ocp-Apim-Subscription-Key': subscriptionKey
            }
        }        
    };

    function analyzeAndProcessImage(context, options) {
        request(options)
        .then((response) => {
             context.log(response);            
        })
        .catch((error) => context.log(error))
        .finally(() => context.done());
    };
};
```

## 2. Azure Functions 로컬에서 개발하기 

참고링크: https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local

(Windows)


(Mac)
Mac이나 Linux와 같이 Windows 운영체제가 아닌 컴퓨터에서도 Azure Function 개발이 가능하고, Azure Portal에서 경험하는 것과 동일하게 개발하실 수 있습니다. 이는 Azure Function 런타임은 오픈소스이며 공개되어있기 때문입니다. 
Azure Function 런타임은 1.0 버전과 2.0 버전으로 나뉘는데, 1.0에서는 .NET Framework를 사용하기 때문에 크로스 플랫폼 지원이 불가하며 Mac / Linux는 .NET Core를 사용하기 때문에 이를 기반으로 윈도우가 아닌 다른 운영체제에서도 닷넷 응용 프로그램이 개발 가능한 것 입니다. 

Mac에서 Azure Function 로컬 개발을 위해서는 1. .NET Core를 설치하셔야 하고(6MB) 2. Azure Functions CLI를 설치하여 개발 하실 수 있습니다. 


