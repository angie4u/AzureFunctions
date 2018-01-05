# Azure Functions
![functions](./images/functions.jpg)
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
* Azure 계정 활성화 ([portal.azure.com](https://portal.azure.com) 로그인 및 리소스 생성 가능 상태)
  - Azure Pass를 사용하시는 경우 [링크](https://www.microsoftazurepass.com/)를 참고하세요 
* [Node 8.5 이상 (Node Version 9 이하여야 함)](https://nodejs.org/ko/download/)
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)
* [.NET Core](https://www.microsoft.com/net/learn/get-started2) 

## 실습 과정
### Part 1. Azure Portal에서 Azure Functions 만들기
1. [Azure Portal](https://portal.azure.com)에 접속한 후 로그인 합니다. 

2. **+ 새로만들기** -> **계산(Compute)** -> **기능 앱(Function App)**을 차례로 클릭합니다.
 ![001](./images/001.PNG)

3. 다음과 같이 값을 입력하신 후 **만들기** 버튼을 눌러서 Function을 생성하시기 바랍니다. 
    * 앱 이름: **이니셜-func-날짜(ex. eunji-func-0106)** 
    * 리소스 그룹: 새로만들기 -> **FunctionLabRG**
    * 호스팅 계획: **사용 계획**
    * 위치: **동아시아**
    * 저장소: 새로만들기 -> **이니셜func날짜(ex. eunjifunc0106)**
    
        ![002](./images/002.PNG)

4. 대쉬보드로 돌아와서 좌측 메뉴중에 **리소스 그룹**을 선택하시고, 방금 생성했던 **FunctionLabRG** 리소스 그룹을 선택하여 접속합니다. 
    ![003](./images/003.PNG)

5. 생성된 Azure Functions를 확인하고 선택하여 접속합니다.
    ![004](./images/004.PNG)

6. 함수 앱 목록에서 함수 옆 **+** 버튼을 클릭하여 새로운 함수를 생성합니다. 

    ![005](./images/005.png)

7. **Webhook + API** -> **JavaScript** -> **이 함수 만들기**를 차례로 클릭하여 첫 번재 Azure Functions를 생성합니다. 

    ![006](./images/006.PNG)

8. 함수 생성이 완료되었습니다. 방금 만든 함수는 HTTP POST 요청으로 값을 보내면 이에 대해 응답을 해주는 아주 간단한 함수입니다. 오른쪽 **테스트** 탭을 확장한 후 아래 부분의 **실행** 버튼을 눌러서 테스트 해보실 수 있습니다. 

    ![007](./images/007.PNG)
    ![008](./images/008.PNG)

### Part 2. 실습에 필요한 리소스 생성 및 BlobTrigger 기능을 하는 Azure Functions 만들기
1. 대쉬보드에서 **리소스 그룹**을 선택하시고, 위에서 진행했던 것과 같이 **FunctionLabRG** 리소스 그룹을 선택하여 접속한다. 
    ![003](./images/003.PNG)

2. **추가** 버튼을 클릭한 후, 검색창에 **app service plan**으로 검색하여 **App Service 계획**을 찾고 선택한 후 **만들기** 버튼을 누른다.

    ![009](./images/009.PNG)    
    ![010](./images/010.PNG)

3. 다음과 같이 입력한 후 **만들기** 버튼을 눌러서 App Service 계획을 생성한다. 

    ![011](./images/011.PNG)

4. 리소스 그룹으로 돌아와서 앞에서 생성핬던 **eunji-func-0106**를 다시 선택한다. 

    ![012](./images/012.PNG)

5. **함수** -> **새 함수** 버튼을 차례로 선택한다. 

   ![013](./images/013.PNG)

6. **Blob trigger** -> **JavaScript**를 선택한다. 

   ![014](./images/014.PNG)

7. 다음과 같이 값을 변경하고 **만들기** 버튼을 눌러서 새 함수를 추가한다. 
    * 이름: **BlobImageAnalysis**
    * Path: **uploaded/{name}**

        ![015](./images/015.PNG)

8. 왼쪽 메뉴에서 **Azure Functions 이름(eunji-func-0106)** 선택 후, **개요** 탭 아래의 구성된 기능 중 **응용 프로그램 설정**을 클릭한다. 

   ![016](./images/016.PNG)

9. **응용 프로그램 설정** 부분에서 **AzureWebJobsDashboard** 값을 확인한다. 

   ![017](./images/017.PNG)

10. Azure Functions와 관련된 기본 코드 및 설정은 완료되었고 Storage와 관련된 작업이 남았다. **Azure Storage Explorer**를 실행하고 Azure 아이디를 이용하여 로그인을 진행한다. 

11. Storage Accounts 목록에서 위에서 생성했던 스토리지(ex.eunjifunc0106)을 선택한 후 **Blob Container**를 확장한다음 마우스 오른쪽 버튼을 클릭하여 **uploaded**란 이름의 컨테이너를 생성한다. 

   ![018](./images/018.PNG)

12. uploaded 컨테이너에 가지고 있는 이미지 파일을 업로드 해본다. 

   ![019](./images/019.PNG)

13. 생성했던 BlobImageAnalysis 함수의 로그창에서 이미지 업로드시, Function이 수행되어 로그값이 찍히는 것을 확인할 수 있다. 

   ![020](./images/020.PNG)

### Part 3. Local에서 Azure Functions 만들고 디버깅 해보기
Reference Link: [https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local)

1. 터미널을 열고 다음의 명령어를 통해 .NET Core에서 동작하는 Azure Functios runtime 2.x 툴을 설치한다. 
```
[Windows]
npm install -g azure-functions-core-tools@core
```

```
[Mac]
sudo npm install -g azure-functions-core-tools@core --unsafe-perm true
```

2. 원하는 디렉토리로 이동한 후 **func init functionslab** 명령어를 입력하여 로컬 Functions 프로젝트를 생성한다. 그리고나서 생성된 하위 디렉토리로 이동한다. 

```
[Windows, Mac 동일]
func init functionslab
cd functionslab
```
   ![022](./images/022.PNG)

3. **Part 2 - 6**에서 했던 것 처럼 **func new** 명령어를 이용하여 Functions 프로젝트에 새 함수를 추가하고, 템플릿 선택시 **JavaScript** -> **BlobTrigger**를 차례로 선택한다. 함수 이름은 기본값인 BlobTriggerJS 그대로 둔다.  
[Windows, Mac 동일]
```
func new
```
   ![023](./images/023.PNG)

4. **func host start**를 입력하여 Azure Functions를 일단 실행해본다. 다음과 같은 에러를 만날 수 있을 것이다. 이는 AzureWebJobStorage의 값이 입력되지 않아서 발생한다. 
[Windows, Mac 동일]
```
func host start
```
   ![024](./images/024.PNG)

5. Visual Studio Code를 실행하고 functionslab 디렉토리 위치로 이동한다. 

6. 탐색기 탭에서 **local.settings.json**을 오픈한 후, **AzureWebJobStorage** 값이 비어있는 것을 확인한다. 

   ![025](./images/025.PNG)

7. **Part 2 - 9**에서 확인했던 값을 복사하여 비어있는 곳에 붙여넣는다.

   ![026](./images/026.PNG)
   ![027](./images/027.PNG)

8. 터미널에서 **func host start** 명령어를 입력하여 Azure Functions를 다시 실행해본다. Azure Functions이 문제없이 동작하는지 확인한다. 
[Windows, Mac 동일]
```
func host start
```
   ![028](./images/028.PNG)

9. **Part 2**에서 만든 함수와 동일하게 동작하도록 하기 위해 몇 가지 값을 바꾸어 주어야한다. Visual Studio Code에서 **function.json** 파일을 열고 **path**와 **connection**에 다음의 값을 입력한다. 
* path: **"uploaded/{name}"**
* connection: **"AzureWebJobsStorage"**

   ![030](./images/030.PNG)

10. Azure에서 동일한 기능을 하는 함수가 똑같이 실행되고 있으므로 테스트를 위해서 그 함수를 중지해야 한다. 다시 Azure Portal로 돌아가서 **함수**탭을 클릭하고 **BlobImageAnalysis**함수의 상태를 **사용안함**으로 변경한다. 

   ![029](./images/029.PNG)

11. Azure Storage Explorer의 uploaded 컨테이너에 다른 이미지를 업로드하여 로컬에서 만든 함수가 잘 작동하는지 확인한다. 

   ![031](./images/031.PNG)
   ![032](./images/032.PNG)


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


