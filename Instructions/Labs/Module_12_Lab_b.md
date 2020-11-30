---
lab:
    title: '12B: 메시지 기반 통합 아키텍처 구성'
    module: '모듈 12: 애플리케이션 인프라 구현'
---

# 랩: 메시지 기반 통합 아키텍처 구성

# 학생 랩 매뉴얼

## 랩 시나리오

Adatum은 온-프레미스 파일 서버에 정기적으로 업로드되는 파일을 처리하는 웹 애플리케이션을 여러 개 보유하고 있습니다. 파일 크기는 다양하지만 최대 크기는 100MB입니다. Adatum은 애플리케이션을 Azure App Service 또는 Azure Functions 기반 앱으로 마이그레이션하고 Azure Storage를 사용하여 업로드된 파일을 호스팅하는 것을 고려하고 있습니다. 다음 두 가지 시나리오를 테스트할 계획입니다.

- Azure Functions를 사용하여 Azure Storage 컨테이너에 업로드된 새 Blob을 자동으로 처리합니다.

  - Event Grid를 사용하여 Azure Storage 컨테이너에 업로드된 새 Blob을 참조하는 Azure Storage 큐 메시지를 생성합니다.

이러한 시나리오는 큰 메시지를 보내고, 받고, 조작할 때 메시지 기반 아키텍처에서 공통적으로 발생하는 어려움을 해결하기 위한 것입니다. 메시지 큐에 대규모 메시지를 직접 보내는 것은 좋지 않습니다. 메시지 플랫폼은 일반적으로 많은 양의 소규모 메시지를 처리하도록 미세 조정되어 있으므로 사용할 리소스가 더 많이 필요하고, 더 많은 대역폭을 사용하고, 처리 속도에 악영향을 미치기 때문입니다. 또한 대부분의 경우 메시지 플랫폼은 처리할 수 있는 최대 메시지 크기를 제한합니다.

한 가지 잠재적인 해결책은 Azure Blob Store, Azure SQL 또는 Azure Cosmos DB와 같은 외부 서비스에 메시지 페이로드를 저장하고, 저장된 페이로드에 대한 참조를 가져온 다음 해당 참조만 메시지 버스로 보내는 것입니다. 이 아키텍처 패턴을 "클레임 검사"라고 합니다. 특정 메시지의 처리에 관심이 있는 클라이언트는 필요한 경우 가져온 참조를 사용하여 페이로드를 검색할 수 있습니다.

Azure에서 이 패턴은 다양한 방법과 기술로 구현할 수 있지만, 일반적으로 이벤트를 사용하여 클레임 검사 생성을 자동화하고 클라이언트가 사용할 메시지 버스에 푸시하거나 페이로드 처리를 직접 트리거합니다. 이러한 구현에 포함되는 일반적인 구성 요소 중 하나는 Event Grid이며, 구성 가능한 기간(최대 24시간) 내에 이벤트 배달을 담당하는 이벤트 라우팅 서비스입니다. 그 후 이벤트는 삭제되거나 배달 못한 편지로 처리됩니다. 이벤트 콘텐츠를 보관하거나 이벤트 스트림을 다시 재생해야 하는 경우, 이벤트 허브에 대한 Event Grid 구독을 설정하거나 Azure Storage에서 메시지를 장기간 보존할 수 있으며 메세지 보관이 지원되는 큐를 설정합니다.

이 랩에서는 Azure Storage Blob 서비스를 사용하여 처리할 파일을 저장합니다. 클라이언트는 지정된 Azure Blob 컨테이너에 공유할 파일을 놓기만 하면 됩니다. 첫 번째 연습에서는 서버리스 특성을 활용하여 Azure 함수에서 파일을 직접 이용하겠습니다. 또한 Application Insights를 활용하여 계측을 제공하고, 모니터링을 지원하고, 파일 처리를 분석할 수 있습니다. 두 번째 연습에서는 Event Grid를 사용하여 클레임 검사 메시지를 자동으로 생성하고 Azure Storage 큐로 보냅니다. 이렇게 하면 클라이언트 애플리케이션이 큐를 폴링하고 메시지를 가져온 다음 저장된 참조 데이터를 사용하여 Azure Blob Storage에서 직접 페이로드를 다운로드할 수 있습니다.

첫 번째 연습의 Azure 함수가 Blob Storage 트리거에 의존한다는 점에 주목하세요. 다음 요구 사항을 처리할 때는 Blob Storage 트리거 대신 Event Grid 트리거를 선택해야 합니다.

- Blob 전용 스토리지 계정: Blob Storage 계정은 Blob 입력 및 출력 바인딩에 대해 지원되지만 Blob 트리거에는 지원되지 않습니다. Blob Storage 트리거에는 범용 스토리지 계정이 필요합니다.

  - 대규모: 대규모란 대략 100,000개 이상의 Blob이 있는 컨테이너 또는 초당 100개 이상의 Blob 업데이트가 이루어지는 스토리지 계정으로 정의할 수 있습니다.

  - 대기 시간 축소: 함수 앱이 소비 계획에 있다면, 함수 앱이 유휴 상태인 경우 새 Blob을 처리하는 데 최대 10분의 지연이 있을 수 있습니다. Event Grid 트리거를 사용하거나 항상 사용으로 설정한 App Service 계획으로 전환하면 이러한 대기 시간을 방지할 수 있습니다.

  - Blob 삭제 이벤트 처리: Blob 삭제 이벤트는 Blob Storage 트리거에서 지원되지 않습니다.

## 목적
  
이 랩을 완료하면 다음과 같은 작업을 수행할 수 있습니다.

- Azure 함수 앱 스토리지 Blob 트리거 구성 및 유효성 검사

- Azure Event Grid 구독 기반 큐 메시지 구성 및 유효성 검사

## 랩 환경
  
예상 시간: 60분

## Lab Files

없음

## 지침

### 연습 1: Azure 함수 앱 스토리지 Blob 트리거 구성 및 유효성 검사
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure 함수 앱 스토리지 Blob 트리거 구성

1. Azure 함수 앱 스토리지 Blob 트리거 유효성 검사

#### 작업 1: Azure 함수 앱 스토리지 Blob 트리거 구성

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고, 이 랩에서 사용할 구독에서 소유자 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다.

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 프로비전할 리소스 이름의 접두사로 사용할 의사 난수 문자열을 생성합니다.

   ```sh
   export PREFIX=$(echo `openssl rand -base64 5 | cut -c1-7 | tr '[:upper:]' '[:lower:]' | tr -cd '[[:alnum:]]._-'`)
   ```

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 리소스를 프로비전할 Azure 지역을 지정합니다 (`<Azure region>` 자리 표시자는 구독에서 사용할 수 있으며 랩 컴퓨터 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다).

   ```sh
   export LOCATION='<Azure region>'
   ```

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 프로비전할 모든 리소스를 호스팅하는 리소스 그룹을 만듭니다.

   ```sh
   export RESOURCE_GROUP_NAME='az30309a-labRG'

   az group create --name "${RESOURCE_GROUP_NAME}" --location "$LOCATION"
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure 함수에서 처리할 Blob이 있는 컨테이너를 호스팅할 Azure Storage 계정을 만듭니다.

   ```sh
   export STORAGE_ACCOUNT_NAME="az30309a${PREFIX}"

   export CONTAINER_NAME="workitems"

   export STORAGE_ACCOUNT=$(az storage account create --name "${STORAGE_ACCOUNT_NAME}" --kind "StorageV2" --location "${LOCATION}" --resource-group "${RESOURCE_GROUP_NAME}" --sku "Standard_LRS")
   ```

    > **참고**: Azure 함수에서도 동일한 스토리지 계정을 사용하여 자체 처리 요구 사항을 지원합니다. 실제 시나리오에서는 이를 위해 별도의 스토리지 계정을 만드는 것이 좋습니다.

1. Cloud Shell 창에서 다음을 실행하여 Azure Storage 계정의 연결 문자열 속성 값을 저장하는 변수를 만듭니다.

   ```sh
   export STORAGE_CONNECTION_STRING=$(az storage account show-connection-string --name "${STORAGE_ACCOUNT_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" -o tsv)
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure 함수에서 처리할 Blob을 호스트하는 컨테이너를 만듭니다.

   ```sh
   az storage container create --name "${CONTAINER_NAME}" --account-name "${STORAGE_ACCOUNT_NAME}" --connection-string "${STORAGE_CONNECTION_STRING}"
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure 함수 처리 Blob에 대한 모니터링을 제공하고 해당 키를 변수에 저장하는 Application Insights 리소스를 만듭니다.

   ```sh
   export APPLICATION_INSIGHTS_NAME="az30309ai${PREFIX}"

   az resource create --name "${APPLICATION_INSIGHTS_NAME}" --location "${LOCATION}" --properties '{"Application_Type": "other", "ApplicationId": "function", "Flow_Type": "Redfield"}' --resource-group "${RESOURCE_GROUP_NAME}" --resource-type "Microsoft.Insights/components"

   export APPINSIGHTS_KEY=$(az resource show --name "${APPLICATION_INSIGHTS_NAME}" --query "properties.InstrumentationKey" --resource-group "${RESOURCE_GROUP_NAME}" --resource-type "Microsoft.Insights/components" -o tsv)
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure Storage Blob 만들기에 해당하는 이벤트를 처리하는 Azure 함수를 만듭니다.

   ```sh
   export FUNCTION_NAME="az30309f${PREFIX}"

   az functionapp create --name "${FUNCTION_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --app-insights "$APPLICATION_INSIGHTS_NAME" --app-insights-key "$APPINSIGHTS_KEY" --storage-account "${STORAGE_ACCOUNT_NAME}" --consumption-plan-location "${LOCATION}" --runtime "dotnet" --functions-version 2
   ```

1. Cloud Shell 창에서 다음을 실행하여 새로 만든 함수의 애플리케이션 설정을 구성하고 Application Insights 및 Azure Storage 계정에 연결합니다.

   ```sh
   az functionapp config appsettings set --name "${FUNCTION_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --settings "APPINSIGHTS_INSTRUMENTATIONKEY=$APPINSIGHTS_KEY" FUNCTIONS_EXTENSION_VERSION=~2

   az functionapp config appsettings set --name "${FUNCTION_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --settings "STORAGE_CONNECTION_STRING=$STORAGE_CONNECTION_STRING" FUNCTIONS_EXTENSION_VERSION=~2
   ```

1. Azure Portal로 전환하고 이 작업의 앞에서 만든 Azure 함수 앱의 블레이드로 이동합니다.

1. Azure 함수 앱 블레이드에서 **함수**를 선택하고 **+ 추가**를 클릭합니다.

1. **새 함수** 블레이드에서 **Azure Blob Storage 트리거**템플릿을 선택합니다.

1. **새 함수** 블레이드에서 다음을 지정하고 **함수 만들기**를 선택하여 Azure 함수 내에서 새 함수를 만듭니다.

    | 설정 | 값 |
    | --- | --- |
    | 이름 | **BlobTrigger** |
    | 경로 | **workitems/{name}** |
    | 스토리지 계정 연결 | **STORAGE_CONNECTION_STRING** |

1. Azure 함수 앱 **BlobTrigger** 함수 블레이드에서 **코드+ 테스트**를 선택하고 run.csx 파일의 내용을 검토합니다.

   ```csharp
   public static void Run(Stream myBlob, string name, ILogger log)
   {
       log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
   }
   ```

    > **참고**: 기본적으로 이 함수는 단순히 새 Blob 생성에 해당하는 이벤트의 로그를 목적으로 구성하였습니다. Blob 처리 작업을 수행하기 위해서는 이 파일의 내용을 수정합니다.

#### 작업 2: Azure 함수 앱 스토리지 Blob 트리거 유효성 검사

1. 필요한 경우 Cloud Shell에서 Bash 세션을 다시 시작합니다.

1. Cloud Shell 창에서 다음을 실행하여 이전 작업에서 사용한 변수를 다시 채웁니다.

   ```sh
   export RESOURCE_GROUP_NAME='az30309a-labRG'

   export STORAGE_ACCOUNT_NAME="$(az storage account list --resource-group "${RESOURCE_GROUP_NAME}" --query "[0].name" --output tsv)"

   export CONTAINER_NAME="workitems"
   ```

1. Cloud Shell 창에서 다음을 실행하여 이 작업의 앞 부분에서 만든 Azure Storage 계정에 테스트 Blob을 업로드합니다.

   ```sh
   export STORAGE_ACCESS_KEY="$(az storage account keys list --account-name "${STORAGE_ACCOUNT_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --query "[0].value" --output tsv)"

   export WORKITEM='workitem1.txt'

   touch "${WORKITEM}"

   az storage blob upload --file "${WORKITEM}" --container-name "${CONTAINER_NAME}" --name "${WORKITEM}" --auth-mode key --account-key "${STORAGE_ACCESS_KEY}" --account-name "${STORAGE_ACCOUNT_NAME}"
   ```

1. Azure Portal에서, 이전 작업에서 만든 Azure 함수 앱을 표시하는 블레이드로 돌아갑니다.

1. 함수 앱에서 **Functions**를 클릭합니다.

1. 함수 목록에서 함수 이름을 클릭합니다.

1. Azure 함수 앱 블레이드에서 **모니터** 항목을 선택합니다.

1. Blob 업로드를 나타내는 단일 이벤트 항목을 확인합니다. 항목을 선택하여 **호출 추적** 세부 정보를 확인합니다.

    > **참고**: 이 연습의 Azure 함수 앱은 소비 계획에서 실행되므로 Blob 업로드와 함수 트리거 사이에 최대 몇 분의 지연이 발생할 수 있습니다. 소비 계획 대신 App Service 계획을 사용하여 함수 앱을 구현하면 대기 시간을 최소화할 수 있습니다.

1. **호출 세부 정보**에서 **Application Insights에서 쿼리 실행**을 클릭합니다.

1. Application Insights 포털에서 자동 생성된 Kusto 쿼리 및 그 결과를 검토합니다.

### 연습 2: Azure Event Grid 구독 기반 큐 메시지 구성 및 유효성 검사
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Event Grid 구독 기반 큐 메시지 구성

1. Azure Event Grid 구독 기반 큐 메시지 유효성 검사

1. 랩에 배포된 Azure 리소스 제거

#### 작업 1: Azure Event Grid 구독 기반 큐 메시지 구성

1. 필요한 경우 Cloud Shell에서 Bash 세션을 다시 시작합니다.

1. Cloud Shell 창에서 다음을 실행하여 구독에 eventgrid 리소스 공급자를 등록합니다.

   ```sh
   az provider register --namespace microsoft.eventgrid
   ```
  
1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 프로비전할 리소스 이름의 접두사로 사용할 의사 난수 문자열을 생성합니다.

   ```sh
   export PREFIX=$(echo `openssl rand -base64 5 | cut -c1-7 | tr '[:upper:]' '[:lower:]' | tr -cd '[[:alnum:]]._-'`)
   ```

1. Cloud Shell 창에서 다음을 실행하여 대상 리소스 그룹과 기존 리소스를 호스팅하는 Azure 지역을 식별합니다.

   ```sh
   export RESOURCE_GROUP_NAME_EXISTING='az30309a-labRG'

   export LOCATION=$(az group list --query "[?name == '${RESOURCE_GROUP_NAME_EXISTING}'].location" --output tsv)

   export RESOURCE_GROUP_NAME='az30309b-labRG'

   az group create --name "${RESOURCE_GROUP_NAME}" --location $LOCATION
   ```

1. Cloud Shell 창에서 다음을 실행하여 이 작업에서 구성할 Event Grid 구독에서 사용되는 컨테이너를 호스트할 Azure Storage 계정을 만듭니다.

   ```sh
   export STORAGE_ACCOUNT_NAME="az30309bst${PREFIX}"

   export CONTAINER_NAME="workitems"

   export STORAGE_ACCOUNT=$(az storage account create --name "${STORAGE_ACCOUNT_NAME}" --kind "StorageV2" --location "${LOCATION}" --resource-group "${RESOURCE_GROUP_NAME}" --sku "Standard_LRS")
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure Storage 계정의 연결 문자열 속성 값을 저장하는 변수를 만듭니다.

   ```sh
   export STORAGE_CONNECTION_STRING=$(az storage account show-connection-string --name "${STORAGE_ACCOUNT_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" -o tsv)
   ```

1. Cloud Shell 창에서 다음을 실행하여 Event Grid 구독에 사용할 컨테이너를 만듭니다.

   ```sh
   az storage container create --name "${CONTAINER_NAME}" --account-name "${STORAGE_ACCOUNT_NAME}" --connection-string "${STORAGE_CONNECTION_STRING}"
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure Storage 계정의 리소스 ID 속성 값을 저장하는 변수를 만듭니다.

   ```sh
   export STORAGE_ACCOUNT_ID=$(az storage account show --name "${STORAGE_ACCOUNT_NAME}" --query "id" --resource-group "${RESOURCE_GROUP_NAME}" -o tsv)
   ```

1. Cloud Shell 창에서 다음을 실행하여 이 작업에서 구성할 Event Grid 구독에서 생성된 메시지를 저장할 스토리지 계정 큐를 만듭니다.

   ```sh
   export QUEUE_NAME="az30309bq${PREFIX}"

   az storage queue create --name "${QUEUE_NAME}" --account-name "${STORAGE_ACCOUNT_NAME}" --connection-string "${STORAGE_CONNECTION_STRING}"
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure Storage 계정의 지정된 컨테이너에 대한 Blob 업로드의 응답으로 Azure Storage 큐의 메시지 생성을 지원하는 Event Grid 구독을 만듭니다.

   ```sh
   export QUEUE_SUBSCRIPTION_NAME="az30309bqsub${PREFIX}"

   az eventgrid event-subscription create --name "${QUEUE_SUBSCRIPTION_NAME}" --included-event-types 'Microsoft.Storage.BlobCreated' --endpoint "${STORAGE_ACCOUNT_ID}/queueservices/default/queues/${QUEUE_NAME}" --endpoint-type "storagequeue" --source-resource-id "${STORAGE_ACCOUNT_ID}"
   ```

#### 작업 2: Azure Event Grid 구독 기반 큐 메시지 유효성 검사

1. Cloud Shell 창에서 다음을 실행하여 이 작업의 앞에서 만든 Azure Storage 계정에 테스트 Blob을 업로드합니다.

   ```sh
   export AZURE_STORAGE_ACCESS_KEY="$(az storage account keys list --account-name "${STORAGE_ACCOUNT_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --query "[0].value" --output tsv)"

   export WORKITEM='workitem2.txt'

   touch "${WORKITEM}"

   az storage blob upload --file "${WORKITEM}" --container-name "${CONTAINER_NAME}" --name "${WORKITEM}" --auth-mode key --account-key "${AZURE_STORAGE_ACCESS_KEY}" --account-name "${STORAGE_ACCOUNT_NAME}"
   ```

1. Azure Portal에서 이 연습의 이전 작업에서 만든 Azure Storage 계정을 표시하는 블레이드로 이동합니다.

1. Azure Storage 계정의 블레이드에서 **큐**를 선택하여 큐 목록을 표시합니다.

1. 이 연습의 이전 작업에서 만든 큐를 나타내는 항목을 클릭합니다.

1. 큐에는 단일 메시지가 포함되어 있습니다. 해당 항목을 클릭하여 **메시지 속성** 블레이드를 표시합니다.

1. **메시지 본문**에서 이전 작업에 업로드한 Azure Storage Blob의 URL을 나타내는 **url** 속성의 값을 기록합니다.

#### 작업 3: 랩에 배포된 Azure 리소스 제거

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30309')]".name --output tsv
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30309')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Cloud Shell 창을 닫습니다.
