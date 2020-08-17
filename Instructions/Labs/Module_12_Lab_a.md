---
lab:
    title: '12A: 스테이징 슬롯을 사용하여 Azure App Service 웹앱 구현'
    module: '모듈 12: 애플리케이션 인프라 구현'
---

# 랩: 스테이징 슬롯을 사용하여 Azure App Service 웹앱 구현
# 학생 랩 매뉴얼

## 랩 시나리오

Adatum Corporation에는 비교적 자주 업데이트되는 여러 웹앱이 있습니다. Adatum은 아직 DevOps 원칙을 완전히 받아들이지 않았지만 Git의 버전 제어 기능에 의존하며 앱 업데이트를 간소화하는 옵션을 살펴보고 있습니다. Adatum이 일부 워크로드를 Azure로 전환함에 따라 Adatum 엔터프라이즈 아키텍처 팀은 이 목표를 달성하기 위해 Azure App Service 및 배포 슬롯 사용을 평가하기로 결정했습니다. 

배포 슬롯은 자체 호스트 이름이 있는 라이브 앱입니다. 두 배포 슬롯(프로덕션 슬롯 등) 간에 앱 콘텐츠 및 구성 요소를 교환할 수 있습니다. 애플리케이션을 프로덕션 이외의 슬롯에 배포할 경우 다음과 같은 이점이 있습니다.

- 프로덕션 슬롯으로 전환하기 전에 스테이징 배포 슬롯의 앱 변경 내용에 대한 유효성을 검사할 수 있습니다.

- 먼저 슬롯에 앱을 배포한 다음 프로덕션으로 교환하면 프로덕션으로 교환되기 전에 해당 슬롯의 모든 인스턴스가 준비됩니다. 이렇게 하면 앱 배포 중에 가동 중지 시간이 발생하지 않습니다. 트래픽도 원활하게 리디렉션되며 전환 작업으로 인해 요청이 삭제되지도 않습니다. 전환 전 유효성 검사를 수행할 필요가 없는 경우 자동 전환을 구성하여 이 워크플로 전체를 자동화할 수 있습니다.

- 전환 후에는 이전에 준비된 앱이 있던 슬롯에 이전 프로덕션 앱이 포함됩니다. 프로덕션 슬롯으로 전환된 변경 사항을 되돌려야 하는 경우 마지막으로 알려진 상태로 돌아가기 위해 또 다른 전환 동작이 즉시 수행됩니다.

배포 슬롯은 파란색/녹색 및 A/B 테스트라는 두 가지 일반적인 배포 패턴을 지원합니다. 파란색-녹색 배포에서는 실시간 애플리케이션과 분리된 별도의 프로덕션 환경으로 업데이트를 배포합니다. 배포의 유효성을 검사한 후에는 트래픽 라우팅이 업데이트된 버전으로 전환됩니다. A/B 테스트에서는 새 버전의 앱을 테스트하기 위해 일부 트래픽을 스테이징 사이트로 점진적으로 라우팅합니다.

Adatum 아키텍처 팀은 다음 두 배포 패턴을 테스트하기 위해 배포 슬롯과 함께 Azure App Service 웹앱을 사용하려고 합니다.

-  블루/그린 배포 

-  A/B 테스트 


## 목표
  
이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

-  Azure App Service 웹 앱의 배포 슬롯을 사용하여 블루/그린 배포 패턴 구현

-  Azure App Service 웹 앱의 배포 슬롯을 사용하여 A/B 테스트 수행


## 랩 환경
  
예상 시간: 60분


## Lab Files

없음

## 지침

### 연습 1: Azure App Service 웹앱 구현

1. Azure App Service 웹앱 배포

1. App Service 웹앱 배포 슬롯 만들기

#### 작업 1: Azure App Service 웹앱 배포

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고, 이 랩에서 사용할 구독에서 소유자 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창에서 다음을 실행하여 **az30305a1**이라는 새 디렉터리를 만들고 현재 디렉터리로 설정합니다.

   ```sh
   mkdir az30305a1
   cd ~/az30305a1/
   ```

1. Cloud Shell 창에서 다음을 실행하여 샘플 앱 리포지토리를 **az30305a1** 디렉터리로 복제합니다.

   ```sh
   REPO=https://github.com/Azure-Samples/html-docs-hello-world.git
   git clone $REPO
   cd html-docs-hello-world
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 배포 사용자를 구성합니다.

   ```sh
   USERNAME=az30305user$RANDOM
   PASSWORD=az30305pass$RANDOM
   az webapp deployment user set --user-name $USERNAME --password $PASSWORD 
   echo $USERNAME
   echo $PASSWORD
   ```
1. 배포 사용자가 성공적으로 만들어졌는지 확인합니다. 충돌을 나타내는 오류 메시지가 표시되면 이전 단계를 반복합니다.

    >**참고**: 사용자 이름과 해당 암호의 값을 기록해야 합니다.

1. Cloud Shell 창에서 다음을 실행하여 App Service 웹앱을 호스팅할 리소스 그룹을 만듭니다(`<location>` 자리 표시자는 구독에서 사용할 수 있으며 랩 컴퓨터 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다).

   ```sh
   LOCATION='<location>'
   RGNAME='az30305a-labRG'
   az group create --location $LOCATION --resource-group $RGNAME
   ```

1. Cloud Shell 창에서 다음을 실행하여 새 App Service 계획을 만듭니다.

   ```sh
   SPNAME=az30305asp$LOCATION$RANDOM
   az appservice plan create --name $SPNAME --resource-group $RGNAME --location $LOCATION --sku S1
   ```

1. Cloud Shell 창에서 다음을 실행하여 새 Git 사용 App Service 웹앱을 만듭니다.

   ```sh
   WEBAPPNAME=az30305$RANDOM$RANDOM
   az webapp create --name $WEBAPPNAME --resource-group $RGNAME --plan $SPNAME --deployment-local-git
   ```

    >**참고**: 배포가 완료될 때까지 기다립니다. 

1. Cloud Shell 창에서 다음 명령을 실행하여 새로 만든 App Service 웹앱의 게시 URL을 확인합니다.

   ```sh
   URL=$(az webapp deployment list-publishing-credentials --name $WEBAPPNAME --resource-group $RGNAME --query scmUri --output tsv)
   ```

1. Cloud Shell 창에서 다음을 실행하여 Git 사용 Azure App Service 웹앱을 나타내는 git 원격 별칭을 설정합니다.

   ```sh
   git remote add azure $URL
   ```

1. Cloud Shell 창에서 다음을 실행하여 git push azure master를 사용하여 Azure 원격으로 푸시합니다.

   ```sh
   git push azure master
   ```

    >**참고**: 배포가 완료될 때까지 기다립니다. 

1. Cloud Shell 창에서 다음을 실행하여 새로 배포한 App Service 웹앱의 FQDN을 식별합니다. 

   ```sh
   az webapp show --name $WEBAPPNAME --resource-group $RGNAME --query defaultHostName --output tsv
   ```

1. Cloud Shell 창을 닫습니다.


#### 작업 2: App Service 웹앱 배포 슬롯 만들기

1. Azure Portal에서 **App Services**를 검색 및 선택하고 **App Services**블레이드에서 새로 만든 App Service 웹앱을 선택합니다.

1. Azure Portal에서 새로 배포된 App Service 웹앱을 표시하는 블레이드로 이동하여 **URL** 링크를 선택하고 **Azure App Service - 샘플 정적 HTML 사이트**를 표시하는지 확인합니다. 브라우저 탭을 열어 둡니다.

1. App Service 웹앱 블레이드의 **배포** 섹션에서 **배포 슬롯**을 선택한 다음 **+ 슬롯 추가**를 선택합니다.

1. **슬롯 추가** 블레이드에서 다음 설정을 지정하고 **추가**를 선택한 다음 **닫기**를 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 이름 | **준비** |
    | 설정 복제 위치 | 웹앱의 이름 |


### 연습 2: App Service 웹앱 배포 슬롯 관리
  
이 연습의 주요 작업은 다음과 같습니다:

1. App Service 웹앱 스테이징 슬롯에 웹 콘텐츠 배포

1. App Service 웹앱 스테이징 슬롯 전환

1. A/B 테스트 구성

1. 랩에 배포된 Azure 리소스 제거


#### 작업 1: App Service 웹앱 스테이징 슬롯에 웹 콘텐츠 배포

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. Cloud Shell 창에서 다음을 실행하여 현재 설정된 **az30305a1/html-docs-hello-world**를 현재 디렉터리로 설정합니다.

   ```sh
   cd ~/az30305a1/html-docs-hello-world
   ```

1. Cloud Shell 창에서 다음을 실행하여 기본 제공 편집기를 시작합니다:

   ```sh
   code index.html
   ```
1. Cloud Shell 창의 코드 편집기에서 다음 줄을 바꿉니다:

   ```html
   <h1>Azure App Service - Sample Static HTML Site</h1>
   ```

   다음 줄로 바꿉니다.

   ```html
   <h1>Azure App Service - Sample Static HTML Site v1.0.1</h1>
   ```

1. 변경 내용을 저장하고 편집기 창을 닫습니다. 

1. Cloud Shell 창에서 다음 명령을 실행하여 필수 글로벌 git 구성 설정이 지정되었는지 확인합니다.

   ```sh
   git config --global user.email "user@az30305.com"
   git config --global user.name "user az30305"
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 마스터 브랜치에 로컬 적용한 변경을 커밋합니다.

   ```sh
   git add index.html
   git commit -m 'v1.0.1'
   ```

1. Cloud Shell 창에서 다음을 실행하여 App Service 웹앱의 새로 만든 스테이징 슬롯에 게시된 URL을 검색합니다.

   ```sh
   RGNAME='az30305a-labRG'
   WEBAPPNAME=$(az webapp list --resource-group $RGNAME --query "[?starts_with(name,'az30305')]".name --output tsv)
   SLOTNAME='staging'
   URLSTAGING=$(az webapp deployment list-publishing-credentials --name $WEBAPPNAME --slot $SLOTNAME --resource-group $RGNAME --query scmUri --output tsv)
   ```

1. Cloud Shell 창에서 다음을 실행하여 Git 지원 Azure App Service 웹 앱의 스테이징 슬롯을 나타내는 git 원격 별칭을 설정합니다.

   ```sh
   git remote add azure-staging $URLSTAGING
   ```

1. Cloud Shell 창에서 다음을 실행하여 git push azure master를 사용하여 Azure 원격으로 푸시합니다.

   ```sh
   git push azure-staging master
   ```

    >**참고**: 배포가 완료될 때까지 기다립니다. 

1. Cloud Shell 창을 닫습니다.

1. Azure Portal에서 App Service 웹앱의 배포 슬롯을 표시하는 블레이드로 이동하여 스테이징 슬롯을 선택합니다.

1. 스테이징 슬롯 개요를 표시하는 블레이드에서 **URL** 링크를 선택합니다.


#### 작업 2: App Service 웹앱 스테이징 슬롯 전환

1. Azure Portal에서 App Service 웹앱이 표시되는 블레이드로 다시 이동하여 **배포 슬롯**을 선택합니다.

1. 배포 슬롯 블레이드에서 **전환**을 선택합니다.

1. **전환** 블레이드에서 **전환**을 선택하고 **닫기**를 선택합니다.

1. App Service 웹앱을 보여주는 브라우저 탭으로 전환하고 브라우저 창을 새로 고침합니다. 스테이징 슬롯에 배포한 변경 내용이 표시되는지 확인합니다.

1. App Service 웹앱의 스테이징 슬롯을 보여주는 브라우저 탭으로 전환하고 브라우저 창을 새로 고칩니다. 원래 배포에 포함된 원래 웹 페이지가 표시되는지 확인합니다. 


#### 작업 3: A/B 테스트 구성

1. Azure Portal에서 App Service 웹앱의 배포 슬롯을 표시하는 블레이드로 다시 이동합니다.

1. Azure Portal에서 App Service 웹앱 배포 슬롯을 표시하는 블레이드의 프로덕션 슬롯을 표시하는 행에서 **TRAFFIC%** 열의 값을 50으로 설정합니다. 이렇게 하면 프로덕션 슬롯을 나타내는 행의 **TRAFFIC %**의 값이 50으로 자동 설정됩니다.

1. App Service 웹앱 배포 슬롯이 표시되는 블레이드에서 **저장**을 선택합니다. 

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. Cloud Shell 창에서 다음을 실행하여 대상 웹앱 및 관련 배포 그룹의 이름을 나타내는 변수를 설정했는지 확인합니다.

   ```sh
   RGNAME='az30305a-labRG'
   WEBAPPNAME=$(az webapp list --resource-group $RGNAME --query "[?starts_with(name,'az30305')]".name --output tsv)
   ```

1. Cloud Shell 창에서 다음을 여러 번 실행하여 두 슬롯 간의 트래픽 분포를 식별합니다.

   ```sh
   curl -H 'Cache-Control: no-cache' https://$WEBAPPNAME.azurewebsites.net --stderr - | grep '<h1>Azure App Service - Sample Static HTML Site'
   ```

    >**참고**: 트래픽 분포가 완전히 결정적인 것은 아니지만 각 대상 사이트에서 여러 응답을 볼 수 있습니다.

#### 작업 4: 랩에 배포된 Azure 리소스 제거

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30305')]".name --output tsv
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30305')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Cloud Shell 창을 닫습니다.
