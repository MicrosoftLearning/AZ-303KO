---
lab:
    title: '6: Azure Storage 파일 서비스 및 Blob Service 구현 및 구성'
    module: '모듈 6: 스토리지 계정 구현'
---

# 랩: Azure Storage 파일 서비스 및 Blob Service 구현 및 구성
# 학생 랩 매뉴얼

## 랩 시나리오
 
Adatum Corporation은 온-프레미스 스토리지에 많은 양의 비정형 및 반구조화 데이터를 호스팅합니다. 유지 보수는 점점 더 복잡해지고 관련 비용이 증가합니다. 일부 데이터는 데이터 보존 요구 사항을 해결하기 위해 아주 오랫동안 보존됩니다. Adatum 엔터프라이즈 아키텍처 팀은 계층 스토리지를 지원하는 동시에 데이터 반출 가능성이 최소화되는 보안 액세스를 허용하는 저렴한 대안을 찾고 있습니다. 팀은 Azure Storage에서 거의 무제한 용량을 제공한다는 것을 알고 있지만 해당 스토리지 계정의 전체 콘텐츠에 무제한 액세스 권한을 부여하는 계정 키의 사용이 우려스럽습니다. 키를 질서 정연하게 회전할 수 있지만 이러한 작업은 적절한 계획에 따라 수행해야 합니다. 또한 액세스 키는 사용량을 적절하게 감사하는 기능을 제한하는 권한 부여 메커니즘만 구성합니다.

이러한 단점을 해결하기 위해 아키텍처 팀은 공유 액세스 서명의 사용을 살펴보기로 결정했습니다. SAS(공유 액세스 서명)는 의도하지 않은 데이터 노출 가능성을 최소화하면서 스토리지 계정의 리소스에 대한 안전하고 위임된 액세스를 제공합니다. SAS는 Blob과 같은 개별 스토리지 개체에 대한 액세스를 제한하고, 사용자 지정 시간 창에 대한 액세스를 제한하며, 지정된 IP 주소 범위에 대한 네트워크 액세스를 필터링하는 등 데이터 액세스를 섬세하게 컨트롤할 수 있습니다. 또한 아키텍처 팀은 감사 요구 사항을 해결하기 위해 Azure Storage와 Azure Active Directory 간의 통합 수준을 평가하려고 합니다. 또한 아키텍처 팀은 일부 온-프레미스 파일 공유의 대안으로 Azure Files의 적합성을 결정하기로 결정했습니다.

이러한 목표를 완료하기 위해 Adatum Corporation은 다음과 같은 Azure Storage 리소스에 대한 인증 범위 및 권한 부여 메커니즘을 테스트합니다.

-  계정과 컨테이너, 개체 수준에서 공유 액세스 서명 사용

-  Blob에 대한 액세스 수준 구성 

-  Azure Active Directory 기반 권한 부여 구현

-  스토리지 계정 액세스 키 사용


## 목표
  
이 랩을 완료하면 다음과 같은 작업을 수행할 수 있습니다.

-  공유 액세스 서명을 활용하여 Azure Storage Blob의 권한 부여 구현

-  Azure Active Directory를 활용하여 Azure Storage Blob의 권한 부여 구현

-  액세스 키를 활용하여 Azure Storage 파일 공유 권한 부여 구현


## 랩 환경
  
Windows 서버 관리자 자격 증명

-  사용자 이름: **학생**

-  암호: **Pa55w.rd1234**

예상 소요 시간: 90분


## Lab Files

-  \\\\AZ303\\AllFiles\\Labs\\06\\azuredeploy30306suba.json

-  \\\\AZ303\\AllFiles\\Labs\\06\\azuredeploy30306rga.json

-  \\\\AZ303\\AllFiles\\Labs\\06\\azuredeploy30306rga.parameters.json


### 연습 0: 랩 환경 준비

이 연습의 주요 작업은 다음과 같습니다.

 1. Azure Resource Manager 템플릿을 사용하여 Azure VM 배포


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure VM 배포

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고, 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택한 다음, 드롭다운 메뉴에서 **업로드**를 선택하여 **\\\\AZ303\\AllFiles\Labs\\06\\azuredeploy30306suba.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 리소스 그룹을 만듭니다(`<Azure region>` 자리 표시자를 구독에 Azure VM을 배포할 수 있고 랩 컴퓨터 위치에 가장 가까운 Azure 지역의 이름으로 대체).

   ```powershell
   $location = '<Azure region>'
   ```
   
   ```powershell
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az30306subaDeployment `
     -TemplateFile $HOME/azuredeploy30306suba.json `
     -rgLocation $location `
     -rgName 'az30306a-labRG'
   ```

      > **참고**: Azure VM을 프로비전할 수 있는 Azure 지역을 확인하려면 [**https://azure.microsoft.com/ko-kr/regions/offers/**](https://azure.microsoft.com/ko-kr/regions/offers/)을 참조하세요.

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\\\AZ303\\AllFiles\Labs\\06\\azuredeploy30306rga.json**을 업로드합니다.

1. Cloud Shell 창에서 Azure Resource Manager 매개 변수 파일**\\\\AZ303\\AllFilesLabs\\06\\azuredeploy30306rga.parameters.json**을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 사용할 Windows 서버 2019를 실행하는 Azure VM을 배포합니다.

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az30306rgaDeployment `
     -ResourceGroupName 'az30306a-labRG' `
     -TemplateFile $HOME/azuredeploy30306rga.json `
     -TemplateParameterFile $HOME/azuredeploy30306rga.parameters.json `
     -AsJob
   ```

    > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 연습을 진행하세요. 배포에는 최대 5분이 소요됩니다.

1. Azure Portal에서 **Cloud Shell** 창을 닫습니다. 


### 연습 1: 공유 액세스 서명을 사용하여 Azure Storage 계정 권한 부여를 구성합니다.
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Storage 계정 만들기

1. Storage Explorer 설치

1. 계정 수준 공유 액세스 서명 생성

1. Azure Storage Explorer를 사용하여 Blob 컨테이너 만들기

1. AzCopy를 사용하여 Blob 컨테이너에 파일을 업로드

1. Blob 수준 공유 액세스 서명을 사용하여 Blob에 액세스


#### 작업 1: Azure Storage 계정 만들기

1. Azure Portal에서 **스토리지 계정**을 검색 및 선택하고 **스토리지 계정** 블레이드에서 **+ 추가**를 선택합니다.

1. **스토리지 계정 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹 **az30306a-labRG**의 이름 |
    | 스토리지 계정 이름 | 문자와 숫자로 구성된 3~24자 사이의 전역적으로 고유한 이름 |
    | 위치 | Azure Storage 계정을 만들 수 있는 Azure 지역의 이름  |
    | 성능 | **표준** |
    | 계정 종류 | **StorageV2(범용 v2)** |
    | 복제 | **LRS(로컬 중복 스토리지)** |

1. **다음: 네트워킹 >**을 선택하고, **스토리지 계정 만들기** 블레이드의 **네트워킹** 탭에서 사용 가능한 옵션을 검토하고 기본 옵션 **공용 엔드포인트(모든 네트워크)**를 수락하고 **다음:**을 선택합니다.** 데이터 보호 >**를 클릭합니다.

1. **스토리지 계정 만들기** 블레이드의 **데이터 보호** 탭에서 사용 가능한 옵션을 검토하고 기본값을 수락하고 **다음: 고급 >**.

1. **스토리지 계정 만들기** 블레이드의 **고급** 탭에서 사용 가능한 옵션을 검토하고 기본값을 수락하고 **검토 + 만들기**를 선택한 다음, 유효성 검사 프로세스가 완료될 때까지 기다린 후 **만들기**를 선택합니다.

    >**참고**: 스토리지 계정이 만들어질 때까지 기다립니다. 약 2분이 소요됩니다.


#### 작업 2: Storage Explorer 설치

   > **참고**: 계속하기 전에 이 랩의 시작 부분에서 시작한 Azure VM의 배포가 완료되었는지 확인합니다. 

1. Azure Portal에서 **가상 머신**을 검색 및 선택하고 **가상 머신** 블레이드의 가상 머신 목록에서 **az30306a-vm0**을 선택합니다.

1. **az30306a-vm0** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **RDP**를 선택한 다음 **RDP 파일 다운로드**를 선택합니다.

1. 메시지가 표시되면 다음 자격 증명으로 로그인합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 사용자 이름 | **학생** |
    | 암호 | **Pa55w.rd1234** |

1. **az30306a-vm0**에 대한 원격 데스크톱 내 서버 관리자 창에서 **로컬 서버**를 선택하고 **IE 강화된 보안 구성** 레이블 옆에 있는 **켜기** 링크를 선택한 다음, **IE 강화된 보안 구성** 대화 상자에서 둘 다 **끄기** 옵션을 선택합니다.

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내에서 Internet Explorer를 시작하여 [Microsoft Edge](https://www.microsoft.com/ko-kr/edge/business/download) 다운로드 페이지로 이동하고 Microsoft Edge 설치 프로그램을 다운로드하여 설치를 실행합니다. 

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내에서 Microsoft Edge를 실행하여 [Azure Storage Explorer](https://azure.microsoft.com/ko-kr/features/storage-explorer/) 다운로드 페이지로 이동합니다.

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내에서 Azure Storage Explorer를 다운로드하고 기본 설정을 사용하여 설치합니다. 


#### 작업 3: 계정 수준 공유 액세스 서명 생성

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내에서 Microsoft Edge를 시작하고 [Azure Portal](https://portal.azure.com)로 이동한 후 이 랩에서 사용 중인 구독의 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. 새로 만든 스토리지 계정의 블레이드로 이동한 후 **액세스 키**를 선택하고 대상 블레이드의 설정을 검토합니다.

    >**참고**: 각 스토리지 계정에는 독립적으로 다시 생성할 수 있는 키 두 개가 있습니다. 스토리지 계정 이름과 두 키 중 하나에 대한 정보는 전체 스토리지 계정에 대한 모든 권한을 제공합니다. 

1. 스토리지 계정 블레이드에서 **공유 액세스 서명**을 선택하고 대상 블레이드의 설정을 검토합니다.

1. 결과 블레이드에서 다음 설정을 지정합니다(다른 설정은 기본값을 그대로 사용).

    | 설정 | 값 | 
    | --- | --- |
    | 허용된 서비스 | **Blob** |
    | 허용되는 리소스 종류 | **서비스** 및 **컨테이너** |
    | 허용된 권한 | **읽기**, **나열** 및 **만들기** |
    | Blob 버전 관리 권한 | 사용 안 함 |
    | 시작 | 현재 표준 시간대의 현재 시간 이전 24시간 | 
    | 종료 | 현재 표준 시간대의 현재 시간 이후 24시간 |
    | 허용된 프로토콜: | **HTTPS만** |
    | 서명 키 | **key1** |

1. **SAS 및 연결 문자열 생성**을 선택합니다.

1. **Blob service SAS URL** 값을 클립보드에 복사합니다.


#### 작업 4: Azure Storage Explorer를 사용하여 Blob 컨테이너 만들기

1. **az30306a-vm0**에 대한 원격 데스크톱 세션에서 Azure Storage Explorer를 시작합니다. 

1. Azure Storage Explorer 창에 있는 **Azure Storage에 연결** 창의 **리소스 선택** 탭에서 **스토리지 계정**을 선택합니다.

1. Azure Storage Explorer 창에 있는 **Azure Storage에 연결** 인증 선택 창의 **인증 방법 선택** 탭에서 **SAS(공유 액세스 서명)**를 선택합니다.

1. Azure Storage Explorer 창에 있는 **Azure Storage에 연결** 창의 **연결 정보 입력** 탭에서 **표시 이름** 텍스트 상자에 **az30306a-blobs**를 입력하고 **SAS 연결 문자열 또는 서비스 URL** 텍스트 상자에서 클립보드에 복사한 값을 붙여넣고 **다음**을 선택합니다. 

1. Azure Storage Explorer 창에 있는 **Azure Storage에 연결** 창의 **요약** 탭에서 **연결**을 선택합니다. 

1. Azure Storage Explorer 창의 **EXPLORER** 창에서 **az30306a-blobs** 항목으로 이동하여 확장하면 **Blob 컨테이너** 엔드포인트에만 액세스할 수 있습니다. 

1. 오른쪽에서 **az30306a-blobs** 항목을 선택하고 오른쪽 클릭 메뉴에서 **Blob 컨테이너 만들기**를 선택한 다음, 빈 텍스트 상자를 사용하여 컨테이너 이름을 **container1**로 설정합니다.

1. **container1**을 선택하여 Storage Explorer 창의 주 창에서 새 탭을 열고 **container1** 탭에서 **업로드**를 선택하고 드롭다운 목록에서 **파일 업로드**를 선택합니다.

1. **파일 업로드** 창에서 **선택한 파일** 레이블 옆에 있는 타원 단추를 선택한 다음, **업로드할 파일 선택** 창에서 **C:\Windows\system.ini**를 선택하고 **열기**를 선택합니다.

1. **파일 업로드** 창으로 다시 돌아가서 **업로드**를 선택하고 **활동** 목록에 표시된 오류 메시지를 확인합니다. 

    >**참고**: 공유 액세스 서명이 개체 수준 권한을 제공하지 않으므로 이 항목이 예상됩니다. 

1. Azure Storage Explorer 창을 열어 둡니다.


#### 작업 5: AzCopy를 사용하여 Blob 컨테이너에 파일을 업로드

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내 브라우저 창의 **공유 액세스 서명** 블레이드에서 다음 설정을 지정합니다(다른 설정은 기본값으로 둡니다).

    | 설정 | 값 | 
    | --- | --- |
    | 허용된 서비스 | **Blob** |
    | 허용되는 리소스 종류 | **개체** |
    | 허용된 권한 | **읽기**, **만들기** |
    | Blob 버전 관리 권한 | 사용 안 함 |
    | 시작 | 현재 표준 시간대의 현재 시간 이전 24시간 | 
    | 종료 | 현재 표준 시간대의 현재 시간 이후 24시간 |
    | 허용된 프로토콜: | **HTTPS만** |
    | 서명 키 | **key1** |

1. **SAS 및 연결 문자열 생성**을 선택합니다.

1. **SAS 토큰** 값을 클립보드에 복사합니다.

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

1. Cloud Shell 창에서 다음을 실행하여 파일을 만들고 텍스트 줄을 추가합니다.

   ```powershell
   New-Item -Path './az30306ablob.html'

   Set-Content './az30306ablob.html' '<h3>Hello from az30306ablob via SAS</h3>'
   ```

1. Cloud Shell 창에서 이 연습에서 전에 만든 Azure Storage 계정의 container1에 새로 만든 파일을 Blob으로 업로드합니다(`<sas_token>` 자리 표시자를 이 작업에서 전에 클립보드에 복사한 공유 액세스 서명 값으로 대체).

   ```powershell
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName 'az30306a-labRG')[0].StorageAccountName

   azcopy cp './az30306ablob.html' "https://$storageAccountName.blob.core.windows.net/container1/az30306ablob.html<sas_token>"
   ```

1. AzCopy에서 생성된 출력을 검토하고 작업이 성공적으로 완료되었는지 확인합니다.

1. Cloud Shell 창을 닫습니다.

1. **az30306a-vm0**에서 원격 데스크톱 세션 내의 브라우저 창에서 스토리지 계정 블레이드의 **Blob service** 섹션에서 **컨테이너**를 선택합니다.

1. 컨테이너 목록에서 **container1**을 선택합니다.

1. **container1** 블레이드에서 **az30306ablob.html**이 Blob 목록에 나타나는지 확인합니다.


#### 작업 6: Blob 수준 공유 액세스 서명을 사용하여 Blob에 액세스

1. **az30306a-vm0**에서 원격 데스크톱 세션 내 브라우저 창의 **container1** 블레이드에서 **액세스 수준 변경**을 선택하여 **비공개(익명 액세스 안 됨)**로 설정됐는지 확인하고 **취소**를 선택합니다.

    >**참고**: 익명 액세스를 허용하려면 공용 액세스 수준을 **Blob(Blob에 대해서만 익명 읽기 액세스)** 또는 **컨테이너(컨테이너 및 Blob에 대한 익명 읽기 액세스)**로 설정할 수 있습니다.

1. **container1** 블레이드에서 **az30306ablob.html**를 선택합니다.

1. **az30306ablob.html** 블레이드에서 **SAS 생성**을 선택하여 수정하지 않고 사용 가능한 옵션을 검토한 다음, **SAS 토큰 및 URL 생성**을 선택합니다.

1. **Blob SAS URL** 값을 클립보드에 복사합니다.

1. 브라우저 창에서 새 탭을 열고 이전 단계에서 클립보드에 복사한 URL로 이동합니다.

1. **Hello from az30306ablob via SAS** 메시지가 브라우저 창에 나타나는지 확인합니다.


### 연습 2: Azure Active Directory를 사용하여 Azure Storage Blob Service 권한 부여를 구성
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure AD 사용자 만들기.

1. Azure Storage Blob Service에 대한 Azure Active Directory 권한 부여를 사용하도록 설정

1. AzCopy를 사용하여 Blob 컨테이너에 파일을 업로드


#### 작업 1: Azure AD 사용자 만들기.

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내 브라우저 창에서 **Cloud Shell** 창 내의 **PowerShell** 세션을 엽니다.

1. Cloud Shell 창에서 다음을 실행하여 새 Azure AD 테넌트에 대해 명시적으로 인증합니다.

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
   New-AzureADUser -AccountEnabled $true -DisplayName 'az30306auser1' -PasswordProfile $passwordProfile -MailNickName 'az30306auser1' -UserPrincipalName "az30306auser1@$domainName"
   ```

1. Cloud Shell 창에서 다음 명령을 실행하여 새로 만든 Azure AD 사용자의 사용자 계정 이름을 확인합니다.

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'az30306auser1'").UserPrincipalName
   ```

1. UPN(사용자 원 이름)을 참고합니다. 이 연습의 뒷부분에서 필요할 것입니다. 

1. Cloud Shell 창을 닫습니다.


#### 작업 2: Azure Storage Blob Service에 대한 Azure Active Directory 권한 부여를 사용하도록 설정

1. **az30306a-vm0**에 할당된 원격 데스크톱 세션 내의 Azure Portal을 표시하는 브라우저 창에서 **container1** 블레이드로 다시 이동합니다.

1. **container1** 블레이드에서 **Azure AD 사용자 계정으로 전환**을 선택합니다.

1. Blob 컨테이너에 데이터를 나열할 권한이 더 이상 없음을 나타내는 오류 메시지를 참고합니다. 정상적인 현상이므로 무시하세요.

    >**참고**: 구독에서 **Owner** 역할이 있음에도 불구하고 **Storage Blob 데이터 소유자**, **Storage Blob 데이터 기여자** 또는 **Storage Blob 데이터 판독기**와 같은 스토리지 계정의 Blob 콘텐츠에 대한 액세스 권한을 제공하는 기본 제공 역할이나 사용자 지정 역할도 할당해야 합니다.

1. Azure Portal에서 **container1**을 호스팅하는 스토리지 계정의 블레이드로 다시 이동하여 **액세스 제어(IAM)**에 이어 **+ 추가**를 선택하고 드롭다운 목록에서 **역할 할당 추가**를 선택합니다. 

    >**참고**: 스토리지 계정의 이름을 적어 둡니다. 다음 작업에 필요합니다.

1. **역할 할당 추가** 블레이드의 **역할** 드롭다운 목록에서 **Storage Blob 데이터 소유자**를 선택하고 드롭다운 목록 항목에 대한 **액세스 권한 할당 대상**이 **사용자, 그룹 또는 서비스 주체**로 설정되어 있는지 확인하고, **선택** 텍스트 상자 아래에 표시된 목록에서 이전 작업에서 만든 사용자 계정과 현재 사용자 계정을 모두 선택한 다음 **저장**을 선택합니다.

1. **container1** 블레이드로 다시 이동하여 컨테이너의 내용을 볼 수 있는지 확인합니다.


#### 작업 3: AzCopy를 사용하여 Blob 컨테이너에 파일을 업로드

1. **az30306a-vm0**에 할당된 원격 데스트톱 세션 내에서 Windows PowerShell을 시작합니다. 

1. Windows PowerShell 프롬프트에서 다음을 실행하여 **azcopy.zip** 아카이브를 다운로드하여 콘텐츠를 추출한 다음, **azcopy.exe**가 포함된 위치로 전환합니다.

   ```powershell
   $url = 'https://aka.ms/downloadazcopy-v10-windows'
   $zipFile = '.\azcopy.zip'

   Invoke-WebRequest -Uri $Url -OutFile $zipFile

   Expand-Archive -Path $zipFile -DestinationPath '.\'

   Set-Location -Path 'azcopy*'
   ```

1. Windows PowerShell 프롬프트에서 다음을 실행하여 이 연습의 첫 번째 작업에서 만든 Azure AD 사용자 계정을 사용하여 AzCopy를 인증합니다. 

   ```powershell
   .\azcopy.exe login
   ```

    >**참고**: 이 목적으로 Microsoft 계정을 사용할 수 없으므로 Azure AD 사용자 계정을 먼저 만들어야 합니다.

1. 이전 단계에서 실행한 명령으로 생성된 메시지에 제공된 지침에 따라 **az30306auser1**사용자 계정으로 인증합니다. 자격 증명에 대한 메시지가 표시되면 이 연습의 첫 번째 작업에서 기록한 계정과 암호 **Pa55w.rd1234**의 사용자 계정 이름을 제공합니다.

1. 성공적으로 인증되면 Windows PowerShell 프롬프트에서 다음을 실행하여 **container1**에 업로드할 파일을 만듭니다.

   ```powershell
   New-Item -Path './az30306bblob.html'

   Set-Content './az30306bblob.html' '<h3>Hello from az30306bblob via Azure AD</h3>'
   ```

1. Windows PowerShell 프롬프트에서 다음을 실행하여 새로 만든 파일을 이전 연습에서 만든 Azure Storage 계정의 **container1**에 Blob으로 업로드합니다(이전 작업에서 확인한 스토리지 계정의 값으로 `<storage_account_name>` 자리 표시자를 대체합니다).

   ```powershell
   .\azcopy cp './az30306bblob.html' 'https://<storage_account_name>.blob.core.windows.net/container1/az30306bblob.html'
   ```

1. AzCopy에서 생성된 출력을 검토하고 작업이 성공적으로 완료되었는지 확인합니다.

1. Windows PowerShell 프롬프트에서 다음을 실행하여 AzCopy 유틸리티에서 제공하는 보안 컨텍스트 외부에서 업로드된 Blob에 액세스할 수 없는지 확인합니다(이전 작업에서 확인한 스토리지 계정의 값으로 `<storage_account_name>` 자리 표시자를 대체합니다).

   ```powershell
   Invoke-WebRequest -Uri 'https://<storage_account_name>.blob.core.windows.net/container1/az30306bblob.html'
   ```

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내 브라우저 창에서 **container1**로 다시 이동합니다.

1. **container1** 블레이드에서 **az30306bblob.html**이 Blob 목록에 나타나는지 확인합니다.

1. **container1** 블레이드에서 **액세스 수준 변경**을 선택하고 공용 액세스 수준을 **Blob(Blob에 대해서만 익명 읽기 액세스)**으로 설정하고 **확인**을 선택합니다. 

1. Windows PowerShell 프롬프트로 다시 전환하고 다음을 다시 실행하여 업로드된 Blob에 익명으로 액세스할 수 있는지 확인합니다(이전 작업에서 확인한 스토리지 계정의 값으로 `<storage_account_name>` 자리 표시자를 대체합니다).

   ```powershell
   Invoke-WebRequest -Uri 'https://<storage_account_name>.blob.core.windows.net/container1/az30306bblob.html'
   ```


### 연습 3: Azure Files를 구현합니다.
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Storage 파일 공유 만들기

1. 드라이브를 Windows에서 Azure Storage 파일 공유로 매핑합니다.

1. 랩에 배포된 Azure 리소스 제거


#### 작업 1: Azure Storage 파일 공유 만들기

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내에서 Azure Portal을 표시하는 브라우저 창을 통해 이 랩의 첫 번째 연습에서 만든 스토리지 계정의 블레이드로 다시 이동하여 **파일 서비스** 섹션에서 **파일 공유**를 선택합니다.

1. **+ 파일 공유**를 선택하고 다음 설정에 따라 파일 공유를 만듭니다.

    | 설정 | 값 |
    | --- | --- |
    | 이름 | **az30306a-share** |
    | 할당량 | **1024** |


#### 작업 2: 드라이브를 Windows에서 Azure Storage 파일 공유로 매핑합니다.

1. 새로 만든 파일 공유를 선택하고 **연결**을 선택합니다.

1. **연결** 블레이드에서 **Windows** 탭이 선택되었는지 확인하고 **클립보드에 복사**를 선택합니다.

    >**참고**: Azure Storage 파일 공유 매핑은 대상 공유에 액세스하기 위해 스토리지 계정 이름과 두 개의 스토리지 계정 키 중 하나를 각각 사용자 이름과 암호에 상응하는 것으로 사용합니다.

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내에서 복사한 스크립트를 PowerShell 프롬프트에 붙여넣고 실행합니다.

1. 스크립트가 성공적으로 완료되었는지 확인합니다. 

1. 파일 탐색기를 시작하고 **Z:** 드라이브로 이동하여 매핑이 성공했는지 확인합니다. 

1. 파일 탐색기에서 **Folder1**이라는 폴더를 만들고 폴더 내부에 **File1.txt**라는 텍스트 파일을 만듭니다.

1. Azure Portal을 표시하는 브라우저 창으로 다시 전환하여 **az30306a-share** 블레이드에서 **새로 고침**을 선택하고 **Folder1**이 폴더 목록에 표시되는지 확인합니다. 

1. **Folder1**을 선택하고 **File1.txt**가 파일 목록에 표시되는지 확인합니다.


#### 작업 3: 랩에 배포된 Azure 리소스 제거

1. **az30306a-vm0**에 대한 원격 데스크톱 세션 내에서 Azure Portal을 표시하는 브라우저 창을 통해 Cloud Shell 창 내에서 PowerShell 세션을 시작합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30306*'
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30306*' | Remove-AzResourceGroup -Force -AsJob
   ```

1. Cloud Shell 창을 닫습니다.

1. Azure Portal에서 Azure 구독과 연결된 Azure Active Directory 테넌트의 **사용자** 블레이드로 이동합니다.

1. 사용자 계정 목록에서 **az30306auser1** 사용자 계정을 나타내는 항목을 선택하고 도구 모음에서 줄임표 아이콘을 선택한 다음 **사용자 삭제**를 선택하고 확인하라는 메시지가 표시되면 **예**를 선택합니다.  
