---
lab:
    title: '10: Azure 역할 기반 액세스 제어 관리'
    module: '모듈 10: Azure 거버넌스 구현 및 관리'
---

# 랩: Azure 역할 기반 액세스 제어 관리
# 학생 랩 매뉴얼

## 랩 시나리오

Azure Active Directory(Azure AD)가 ID 관리 환경에서 필수적인 부분이 됨에 따라 Adatum 엔터프라이즈 아키텍처 팀은 최적의 권한 부여 접근 방식도 결정해야 합니다. Azure 리소스에 대한 액세스를 제어하는 상황에서 이러한 접근 방식에는 Azure RBAC(역할 기반 액세스 제어)를 사용해야 합니다. Azure Resource Manager를 기반으로 구축된 권한 부여 시스템인 Azure RBAC를 사용하면 Azure 리소스 액세스를 섬세하게 관리할 수 있습니다.

Azure RBAC의 주요 개념은 역할 할당입니다. 역할 할당은 보안 주체, 역할 정의 및 범위의 세 가지 요소로 구성됩니다. 보안 주체는 Azure 리소스 액세스 권한을 요청하는 사용자, 그룹, 서비스 주체 또는 관리 ID를 나타내는 개체입니다. 역할 정의는 읽기나 쓰기, 삭제와 같이 역할 할당이 부여할 작업의 모음입니다. 역할은 일반적일 수도 있지만 리소스에 따라 다를 수 있습니다. Azure에는 4개의 기본 제공 일반 역할(Owner, Contributor, Reader 및 User Access Administrator)과 상당히 많은 수의 기본 제공 리소스별 역할(예: Azure 가상 머신을 만들고 관리할 수 있는 권한이 포함된 가상 머신 참가자)이 포함됩니다. 또한 사용자 지정 역할을 정의할 수도 있습니다. 범위는 액세스 권한이 적용되는 리소스 집합입니다. 여러 수준(관리 그룹, 구독, 리소스 그룹 또는 리소스)에서 범위를 설정할 수 있습니다. 범위는 부모-자식 관계로 구성됩니다.

Adatum 엔터프라이즈 아키텍처 팀은 사용자 지정 역할 기반 액세스 제어 역할을 사용하여 Azure 관리 위임을 테스트하려고 합니다. 평가를 시작하기 위해 Azure 가상 머신에 대한 제한된 액세스를 제공하는 사용자 지정 역할을 만들려고 합니다. 
  

## 목표
  
이 랩을 완료하면 다음과 같은 작업을 수행할 수 있습니다.

-  사용자 지정 RBAC 역할 정의 

-  사용자 지정 RBAC 역할 할당


## 랩 환경
  
Windows 서버 관리자 자격 증명

-  사용자 이름: **Student**

-  암호: **Pa55w.rd1234**

예상 소요 시간: 60분


## Lab Files

-  \\\\AZ303\\AllFiles\\Labs\\10\\azuredeploy30310suba.json

-  \\\\AZ303\\AllFiles\\Labs\\10\\azuredeploy30310rga.json

-  \\\\AZ303\\AllFiles\\Labs\\10\\azuredeploy30310rga.parameters.json

-  \\\\AZ303\\AllFiles\\Labs\\10\\roledefinition30310.json


## 지침

### 연습 0: 랩 환경 준비

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 Azure VM 배포

1. Azure Active Directory 사용자 만들기


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure VM 배포

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고, 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택한 다음, 드롭다운 메뉴에서 **업로드**를 선택하여 **\\\\AZ303\\AllFiles\Labs\\10\\azuredeploy30310suba.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 리소스 그룹을 만듭니다(`<Azure region>` 자리 표시자를 구독에 Azure VM을 배포할 수 있고 랩 컴퓨터 위치에 가장 가까운 Azure 지역의 이름으로 대체).

   ```powershell
   $location = '<Azure region>'
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az30310subaDeployment `
     -TemplateFile $HOME/azuredeploy30310suba.json `
     -rgLocation $location `
     -rgName 'az30310a-labRG'
   ```

      > **참고**: Azure VM을 프로비전할 수 있는 Azure 지역을 확인하려면 [**https://azure.microsoft.com/ko-kr/regions/offers/**](https://azure.microsoft.com/ko-kr/regions/offers/)을 참조하세요.

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\\\AZ303\\AllFiles\Labs\\10\\azuredeploy30310rga.json**을 업로드합니다.

1. Cloud Shell 창에서 Azure Resource Manager 매개 변수 파일 **\\\\AZ303\\AllFilesLabs\\10\\azuredeploy30310rga.parameters.json** 을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 사용할 Windows 서버 2019를 실행하는 Azure VM을 배포합니다.

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az30310rgaDeployment `
     -ResourceGroupName 'az30310a-labRG' `
     -TemplateFile $HOME/azuredeploy30310rga.json `
     -TemplateParameterFile $HOME/azuredeploy30310rga.parameters.json `
     -AsJob
   ```

    > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 작업으로 진행하세요. 배포에는 최대 5분이 소요됩니다.


#### 작업 2: Azure Active Directory 사용자 만들기

1. Azure Portal 내 Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 Azure 구독과 연결된 Azure AD 테넌트에 대해 인증합니다.

   ```powershell
   Connect-AzureAD
   ```
   
1. Cloud Shell 창에서 다음 명령을 실행하여 Azure AD DNS 도메인 이름을 확인합니다.

   ```powershell
   $domainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 새 Azure AD 사용자를 만듭니다.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = 'Pa55w.rd1234'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'az30310aaduser1' -PasswordProfile $passwordProfile -MailNickName 'az30310aaduser1' -UserPrincipalName "az30310aaduser1@$domainName"
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 새로 만든 Azure AD 사용자의 사용자 계정 이름을 확인합니다.

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'az30310aaduser1'").UserPrincipalName
   ```

      > **참고**: 새로 만든 Azure AD 사용자의 사용자 원이름을 기록합니다. 이 랩 뒷부분에서 해당 이름이 필요합니다.

1. Cloud Shell 창을 닫습니다.


### 연습 1: 사용자 지정 RBAC 역할 정의
  
이 연습의 주요 작업은 다음과 같습니다.

1. RBAC를 통해 위임할 작업 식별

1. Azure AD 테넌트에서 사용자 지정 RBAC 역할 만들기


#### 작업 1: RBAC를 통해 위임할 작업 식별

1. Azure Portal에서 **az30310a-labRG** 블레이드로 이동합니다.

1. **az30310a-labRG** 블레이드에서 **IAM(액세스 제어)** 을 선택합니다.

1. **az30310a-labRG - IAM(액세스 제어)** 블레이드에서 **역할**을 클릭합니다.

1. **역할** 블레이드에서 **Owner**를 선택합니다.

1. **Owner** 블레이드에서 **권한**을 선택합니다.

1. **권한(미리 보기)** 블레이드에서 **Microsoft Compute**를 선택합니다.

1. **Microsoft Compute** 블레이드에서 **가상 머신**을 선택합니다.

1. **가상 머신** 블레이드에서 RBAC를 통해 위임할 수 있는 관리 작업 목록을 검토합니다. 여기에는 **가상 머신 할당 취소** 및 **가상 머신 시작** 작업이 포함됩니다.


#### 작업 2: Azure AD 테넌트에서 사용자 지정 RBAC 역할 만들기

1. 랩 컴퓨터에서 **\\\\AZ303\\AllFiles\\Labs\\10\\roledefinition30310.json** 파일을 열어 콘텐츠를 검토합니다.

   ```json
   {
      "Name": "Virtual Machine Operator (Custom)",
      "Id": null,
      "IsCustom": true,
      "Description": "Allows to start/restart Azure VMs",
      "Actions": [
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/restart/action",
          "Microsoft.Compute/virtualMachines/start/action"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```

1. 랩 컴퓨터의 Azure Portal을 표시하는 브라우저 창에서 **Cloud Shell** 내에서 **PowerShell** 세션을 시작합니다. 

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\\\AZ303\\AllFiles\\Labs\\10\\roledefinition30310.json**을 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 `SUBSCRIPTION_ID` 자리 표시자를 Azure 구독의 ID 값으로 바꿉니다.

   ```powershell
   $subscription_id = (Get-AzContext).Subscription.id
   (Get-Content -Path $HOME/roledefinition30310.json) -Replace 'SUBSCRIPTION_ID', "$subscription_id" | Set-Content -Path $HOME/roledefinition30310.json
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 `SUBSCRIPTION_ID` 자리 표시자를 Azure 구독의 ID 값으로 바꿉니다.

   ```powershell
   Get-Content -Path $HOME/roledefinition30310.json
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 사용자 지정 역할 정의를 만듭니다.

   ```powershell
   New-AzRoleDefinition -InputFile $HOME/roledefinition30310.json
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 역할이 제대로 생성되었는지 확인합니다.

   ```powershell
   Get-AzRoleDefinition -Name 'Virtual Machine Operator (Custom)'
   ```

1. Cloud Shell 창을 닫습니다.


### 연습 2: 사용자 지정 RBAC 역할 할당 및 테스트
  
이 연습의 주요 작업은 다음과 같습니다.

1. RBAC 역할 할당 만들기

1. RBAC 역할 할당 테스트


#### 작업 1: RBAC 역할 할당 만들기
 
1. Azure Portal에서 **az30310a-labRG** 블레이드로 이동합니다.

1. **az30310a-labRG** 블레이드에서 **IAM(액세스 제어)** 을 선택합니다.

1. **az30310a-labRG - 액세스 제어(IAM)** 블레이드에서 **+ 추가**를 선택하고 **역할 할당 추가** 옵션을 선택합니다.

1. **역할 할당 추가** 블레이드에서 다음 설정을 지정하고(다른 설정은 기존값으로 유지) **저장**을 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 역할 | **가상 머신 운영자(사용자 지정)** |
    | 액세스 권한 할당 대상 | **사용자, 그룹 또는 서비스 주체** |
    | 선택 | **az30310aaduser1** |


#### 작업 2: RBAC 역할 할당 테스트

1. 랩 컴퓨터에서 새 비공개 웹 브라우저 세션을 시작하고 [Azure Portal](https://portal.azure.com)로 이동한 다음 **az30310aaduser1** 사용자 계정과 **Pa55w.rd1234** 암호를 사용하여 로그인합니다.

    > **참고**: 이 랩에서 이전에 기록한 **az30310aaduser1** 사용자 계정의 사용자 원이름을 사용해야 합니다.

1. Azure Portal에서 **리소스 그룹** 블레이드로 이동합니다. 아무런 리소스 그룹도 볼 수 없다는 것을 확인하세요. 

1. Azure Portal에서 **모든 리소스** 블레이드로 이동합니다. **az30310a-vm0** 및 해당 관리 디스크만 볼 수 있습니다.

1. Azure Portal에서 **az30310a-vm0** 블레이드로 이동합니다. 가상 머신을 중지해봅니다. 알림 영역에서 오류 메시지를 검토하고, 현재 사용자에게 이 작업을 수행할 권한이 부여되지 않아 해당 작업이 실패했음을 확인합니다.

1. 가상 머신을 재시작하고 작업이 정상적으로 완료되었는지 확인합니다.

1. 비공개 웹 브라우저 세션을 닫습니다.


#### 작업 3: 랩에 배포된 Azure 리소스 제거

1. 랩 컴퓨터에서, Azure Portal을 표시하는 기존 브라우저 창의 Cloud Shell 창에서 PowerShell 세션을 시작합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30310*'
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30310*' | Remove-AzResourceGroup -Force -AsJob
   ```

1. Cloud Shell 창에서 다음을 실행하여 이 랩의 앞부분에서 업로드한 랩 파일을 삭제합니다.

   ```powershell
   Get-ChildItem -Path . -Filter 'az30310*.json' | Remove-Item -Force
   Get-ChildItem -Path . -Filter 'roledefinition30310.json' | Remove-Item -Force
   ```

1. Cloud Shell 창을 닫습니다.

1. Azure Portal에서 Azure 구독과 연결된 Azure Active Directory 테넌트의 **사용자** 블레이드로 이동합니다.

1. 사용자 계정 목록에서 **az30310aaduser1** 사용자 계정을 나타내는 항목을 선택하고 도구 모음에서 줄임표 아이콘을 선택한 다음 **사용자 삭제**를 선택하고 확인하라는 메시지가 표시되면 **예**를 선택합니다.  

1. Azure Portal에서 Azure 구독의 속성을 표시하는 블레이드로 이동하여 **액세스 제어(IAM)** 항목을 선택한 다음, **역할**을 선택합니다.

1. 역할 목록에서 **가상 머신 운영자(사용자 지정)** 항목을 선택하고 **제거**를 선택한 다음, 확인하라는 메시지가 표시되면 **예**를 선택합니다.
