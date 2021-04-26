---
lab:
    title: '05: 항상 사용 가능한 Azure Iaas 컴퓨팅 아키텍처 구현'
    module: '모듈 05: 부하 분산 및 네트워크 보안 구현'
---

# 랩: 항상 사용 가능한 Azure Iaas 컴퓨팅 아키텍처 구현
# 학생 랩 매뉴얼

## 랩 시나리오
  
Adatum Corporation은 실제 서버와 가상 머신의 혼합 형태에서 여러 온-프레미스 워크로드를 실행하고 있습니다. 대부분의 워크로드에는 다양한 고가용성 SLA를 포함한 일정 수준의 복원력이 필요합니다. 대부분의 워크로드 클러스터 노드 간의 동기 복제와 함께 Windows 서버 장애 조치 클러스터링 또는 Linux Corosync 클러스터 및 Pacemaker 리소스 관리자를 활용합니다. Adatum은 Azure에서 동등한 기능을 구현 방법을 결정하려고 합니다. 특히 Adatum 엔터프라이즈 아키텍처 팀은 동일 데이터 센터 내에서 그리고 동일 지역의 데이터 센터 간에 고가용성 요구 사항을 지원하는 Azure 플랫폼 기능을 탐색하고 있습니다.

또한 Adatum 엔터프라이즈 아키텍처 팀은 복원력만으로는 비즈니스 운영에서 예상되는 가용성 수준을 제공하기에 충분하지 않을 수 있음을 깨닫습니다. 현재 일부 워크로드는 지속적인 모니터링 및 사용자 지정 스크립팅 솔루션, 추가 클러스터 노드 자동 프로비전 및 프로비전 해제를 기반으로 하여 처리되는 높은 동적 사용 패턴이 나타납니다. 그 외의 사용 패턴은 예측하기가 쉬운 편이지만, 경우에 따라 디스크 공간, 메모리 또는 처리 리소스에 대한 수요 증가를 고려하여 조정해야 합니다.

이러한 목표를 달성하기 위해 아키텍처 팀은 다음과 같은 다양한 고가용성 IaaS 컴퓨팅 배포를 테스트하려고 합니다.

-  Azure Load Balancer Basic 뒤에서 Azure VM의 가용성 집합 기반 배포

-  Azure Load Balancer Standard 뒤에서 Azure VM의 영역 중복 배포

-  Azure 애플리케이션 게이트웨이 뒤에서 Azure VM Scale Sets의 영역 중복 배포

-  Azure VM Scale Sets의 자동 가로 크기 조정(자동 크기 조정) 

-  Azure VM Scale Sets의 수동 세로 크기 조정(컴퓨팅 및 저장소)

가용성 집합은 동일한 Azure 데이터 센터 내에서 실제 위치를 제어하는 Azure VM의 논리적 그룹화를 나타냅니다. Azure에서는 동일한 가용성 집합 안에 있는 VM을 여러 실제 서버, 컴퓨팅 랙, 스토리지 단위, 네트워크 스위치 간에 실행할 수 있습니다. 하드웨어 또는 소프트웨어 오류가 발생하더라도 VM의 하위 집합만 영향을 받으며 전체 솔루션은 작동 가능한 상태로 유지됩니다. 가용성 집합은 신뢰할 수 있는 클라우드 솔루션을 구축하는 데 필수적입니다. Azure는 가용성 집합으로 99.95%의 VM 작동 시간 SLA를 제공합니다.

가용성 영역은 단일 Azure 지역 내에서 고유한 실제 위치를 나타냅니다. 각 영역은 독립적인 전력, 냉각 및 네트워킹이 장착된 하나 이상의 데이터 센터로 구성됩니다. 지역 내 가용성 영역을 물리적으로 분리하여 애플리케이션과 데이터를 데이터 센터 오류로부터 보호합니다. 영역 중복 서비스는 가용성 영역의 애플리케이션과 데이터를 복제하여 단일 지점의 실패로부터 보호합니다. Azure는 가용성 영역으로 99.99%의 VM 작동 시간 SLA를 제공합니다.

Azure Virtual Machine Scale Sets를 사용하면 부하 분산된 동일한 VM의 그룹을 만들고 관리할 수 있습니다. VM 인스턴스 수는 수요 또는 정의된 일정에 따라 자동으로 증가하거나 감소할 수 있습니다. 확장 집합을 사용하면 애플리케이션에 고가용성을 제공하고 많은 수의 VM을 중앙에서 관리하고 구성하고 업데이트할 수 있습니다. Virtual Machine Scale Sets를 사용하면 컴퓨팅, 빅 데이터 및 컨테이너 워크로드와 같은 영역에 대한 대규모 서비스를 빌드할 수 있습니다.

## 목표
  
이 랩을 완료하면 다음과 같은 작업을 수행할 수 있습니다.

-  Azure Load Balancer Basic 뒤에 있는 동일 가용성 집합에 상주하는 고가용성 Azure VM의 특성을 설명합니다.

-  Azure Load Balancer 표준 뒤에 있는 다양한 가용성 영역에 상주하는 고가용성 Azure VM의 특성을 설명합니다.

-  Azure VM Scale Sets의 자동 수평 크기 조정의 특성을 설명합니다.

-  Azure VM Scale Sets의 수동 수직 크기 조정의 특성을 설명합니다.


## 랩 환경
  
Windows 서버 관리자 자격 증명

-  사용자 이름: **Student**

-  암호: **Pa55w.rd1234**

예상 소요 시간: 120분


## Lab Files

-  \\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305suba.json

-  \\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rga.json

-  \\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rga.parameters.json

-  \\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgb.json

-  \\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgb.parameters.json

-  \\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgc.json

-  \\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgc.parameters.json

-  \\AZ303\\AllFiles\\Labs\\05\\az30305e-configure_VMSS_with_data_disk.ps1


## 지침

### 연습 1: 가용성 집합과 Azure Load Balancer Basic을 사용한 고가용성 Azure VM 배포를 구현 및 분석합니다.
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 Azure Load Balancer Basic 뒤에 있는 가용성 집합에 고가용성 Azure VM을 배포

1. Azure Load Balancer Basic 뒤에 있는 가용성 집합에 배포된 고가용성 Azure VM을 분석

1. 연습에서 배포된 Azure 리소스를 제거합니다.


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure Load Balancer Basic 뒤에 있는 가용성 집합에 고가용성 Azure VM을 배포

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고, 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. Azure Portal에서 입력란의 오른쪽에 있는 도구 모음 아이콘에서 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

    > **참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 
    
1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 향후 다룰 연습을 준비하기 위해 Microsoft.Insights 리소스 공급자를 등록합니다.

   ```Bash
   az provider register --namespace 'Microsoft.Insights'
   ```

1. Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택한 다음 Cloud Shell 홈 디렉터리에 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305suba.json** 파일을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 사용할 Azure 지역을 지정합니다(`<Azure region>` 자리 표시자는 구독에서 Azure VM 배포에 사용할 수 있으며 랩 컴퓨터 위치에 가장 가까운 Azure 지역의 이름으로 바꿉니다).

   ```Bash
   LOCATION='<Azure region>'
   ```
   
      > **참고**: Azure VM을 프로비전할 수 있는 Azure 지역을 확인하려면 [**https://azure.microsoft.com/ko-kr/regions/offers/**](https://azure.microsoft.com/ko-kr/regions/offers/)을 참조하세요.

      > **참고**: **LOCATION** 변수를 설정할 때 사용할 Azure 지역의 이름을 식별하려면 `az account list-locations --query "[].{name:name}" -o table`을 실행합니다. 공백을 포함하지 않는 표기법을 사용해야 합니다(예: **US East**가 아닌 **eastus**).

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 향후 다룰 연습을 준비하기 위해 Network Watcher 인스턴스를 만듭니다.

   ```Bash
   az network watcher configure --resource-group NetworkWatcherRG --locations $LOCATION --enabled -o table
   ```

1. Cloud Shell 창에서 다음을 실행하여 지정한 Azure 지역에 리소스 그룹을 만듭니다.
 
   ```Bash
   az deployment sub create \
   --location $LOCATION \
   --template-file azuredeploy30305suba.json \
   --parameters rgName=az30305a-labRG rgLocation=$LOCATION
   ```
      
1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rga.json**을 업로드합니다.

1. Cloud Shell 창에서 Azure Resource Manager 매개 변수 파일 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rga.parameters.json**을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 Windows Server 2019 Datacenter Core를 호스팅하는 Azure VM 쌍으로 구성된 백엔드 풀을 사용하여 Azure Load Balancer Basic을 동일한 가용성 집합에 배포합니다.

   ```Bash
   az deployment group create \
   --resource-group az30305a-labRG \
   --template-file azuredeploy30305rga.json \
   --parameters @azuredeploy30305rga.parameters.json
   ```

    > **참고**: 다음 작업으로 진행하기 전에 배포가 완료될 때까지 기다립니다. 약 10분이 소요됩니다.

1. Azure Portal에서 **Cloud Shell** 창을 닫습니다. 


#### 작업 2: Azure Load Balancer Basic 뒤에 있는 가용성 집합에 배포된 고가용성 Azure VM을 분석

1. Azure Portal에서 **Network Watcher**를 검색 및 선택하고 **Network Watcher** 블레이드에서 **토폴로지**를 선택합니다.

1. **Network Watcher** 에서 **| 토폴로지** 블레이드는 다음 설정을 지정합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30305a-LabRG** |
    | 가상 네트워크 | **az30305a-vnet** |

1. 결과 토폴로지 다이어그램을 검토하여 백 엔드 풀에서 공용 IP 주소, 부하 분산 장치 및 Azure VM의 네트워크 어댑터 간의 연결을 확인합니다.

1. **Network Watcher** 블레이드에서 **효과적인 보안 규칙**을 선택합니다.

1. **Network Watcher** 에서 **| 효과적인 보안 규칙** 블레이드는 다음 설정을 지정합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30305a-LabRG** |
    | 가상 머신 | **az30305a-vm0** |
    | 네트워크 인터페이스 | **az30305a-nic0** |

1. RDP 및 HTTP를 통한 인바운드 연결을 허용하는 두 가지 사용자 지정 규칙을 포함하여 관련 네트워크 보안 그룹과 효과적인 보안 규칙을 검토합니다.  

    > **참고**: 또는 다음에서 **효과적인 보안 규칙**을 확인할 수 있습니다.
    - **az30305a-nic0** 네트워크 인터페이스 블레이드
    - **az30305a-web-nsg** 네트워크 보안 그룹 블레이드 
    
1. **Network Watcher** 블레이드에서 **연결 문제 해결**을 선택합니다.

    > **참고**: 의도는 동일한 가용성 집합에서 두 Azure VM의 근접(네트워킹 용어)을 확인하는 것입니다.

1. **Network Watcher** 에서 **| 연결 문제 해결** 블레이드에서 다음 설정을 지정하고 **확인**을 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30305a-LabRG** |
    | 원본 유형 | **가상 머신** |
    | 가상 머신 | **az30305a-vm0** |
    | 대상 | **가상 머신 선택** |
    | 리소스 그룹 | **az30305a-LabRG** |
    | 가상 머신 | **az30305a-vm1** |
    | 프로토콜 | **TCP** |
    | 대상 포트| **80** |

    > **참고**: **Azure Network Watcher Agent** VM 확장이 Azure VM에 설치되려면 결과를 잠시 기다려야 합니다.

1. 결과를 검토하고 Azure VM 간의 네트워크 연결 대기 시간을 기록합니다.

    > **참고**: 두 VM이 동일한 가용성 집합(동일한 Azure 데이터 센터 내에서)에 있으므로 대기 시간은 약 1밀리초여야 합니다.

1. Azure Portal에서 **az30305a-labRG** 리소스 그룹 블레이드로 이동하여 리소스 목록에서 **az30305a-avset** 가용성 집합 항목을 선택하고 **az30305a-avset** 블레이드에서 두 Azure VM에 할당된 장애 도메인 및 업데이트 도메인을 기록합니다.

1. Azure Portal에서 **az30305a-labRG** 리소스 그룹 블레이드로 다시 이동하여 리소스 목록에서 **az30305a-lb** 부하 분산 장치 항목을 선택하고 **az30305a-lb** 블레이드에서 공용 IP 주소 항목을 기록합니다.

1. Azure Portal의 Cloud Shell 창에서 **Bash** 세션을 시작합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Azure Load Balancer의 백 엔드 풀에서 Azure VM에 대한 HTTP 트래픽의 부하 분산을 테스트합니다(여기서, `<lb_IP_address>` 자리 표시자를 이전에 식별한 부하 분산 장치 프런트 엔드의 IP 주소로 대체함).

   ```Bash
   for i in {1..4}; do curl <lb_IP_address>; done
   ```

    > **참고**: 반환된 메시지가 백 엔드 Azure VM에 라운드 로빈 방식으로 요청이 전달되고 있음을 나타내는지 확인합니다.

1. **az30305a-lb** 블레이드에서 **부하 분산 규칙** 항목을 선택하고 **az30305a-lb** 에서 **| 부하 분산 규칙** 블레이드에서 HTTP 트래픽을 처리하는 부하 분산 규칙을 나타내는 **az303005a-lbruletcp80** 항목을 선택합니다. 

1. **az303005a-lbruletcp80** 블레이드의 **세션 지속성** 드롭다운 목록에서 **클라이언트 IP**를 선택한 다음 **저장**을 선택합니다.

1. 업데이트가 완료될 때까지 기다렸다가 Cloud Shell 창에서 다음을 다시 실행하여 세션 지속성 없이 Azure Load Balancer의 백 엔드 풀에서 Azure VM에 대한 HTTP 트래픽의 부하 분산을 테스트합니다(`<lb_IP_address>` 자리 표시자를 이전에 식별한 부하 분산 장치의 프런트 엔드 IP 주소로 대체함).

   ```Bash
   for i in {1..4}; do curl <lb_IP_address>; done
   ```

    > **참고**: 반환된 메시지가 동일한 백 엔드 Azure VM으로 요청이 전달되고 있음을 나타내는지 확인합니다.

1. Azure Portal에서 **az30305a-lb** 블레이드로 다시 이동하여 **인바운드 NAT 규칙** 항목을 선택하고 각각 TCP 포트 33890 및 33891의 원격 데스크톱을 통해 백 엔드 풀 VM의 첫 번째 및 두 번째에 연결할 수 있는 두 가지 규칙을 기억합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Azure Load Balancer의 백 엔드 풀에서 첫 번째 Azure VM에 NAT를 통한 원격 데스크톱 연결을 테스트합니다(`<lb_IP_address>` 자리 표시자는 이전에 식별한 부하 분산 장치의 프런트 엔드 IP 주소로 대체함).

   ```Bash
   curl -v telnet://<lb_IP_address>:33890
   ```

    > **참고**: 반환된 메시지가 성공적으로 연결되어 있음을 나타내는지 확인합니다. 

1. **Ctrl+C** 키 조합을 눌러 Bash 셸 프롬프트로 돌아간 후 다음을 실행하여 Azure Load Balancer의 백엔드 풀에서 두 번째 Azure VM에 NAT를 통한 원격 데스크톱 연결을 테스트합니다(`<lb_IP_address>` 자리 표시자는 이전에 식별한 부하 분산 장치의 프런트 엔드 IP 주소로 대체함).

   ```Bash
   curl -v telnet://<lb_IP_address>:33891
   ```

    > **참고**: 반환된 메시지가 성공적으로 연결되어 있음을 나타내는지 확인합니다. 

1. **Ctrl+C** 키 조합을 눌러 Bash 셸 프롬프트로 돌아갑니다.


#### 작업 3: 연습에서 배포된 Azure 리소스를 제거합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```Bash
   az group list --query "[?starts_with(name,'az30305a-')]".name --output tsv
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```sh
   az group list --query "[?starts_with(name,'az30305a-')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Cloud Shell 창을 닫습니다.


### 연습 2: 가용성 영역 및 Azure Load Balancer Standard를 사용하여 고가용성 Azure VM 배포를 구현 및 분석
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 Azure Load Balancer Standard 뒤에 있는 가용성 영역에 고가용성 Azure VM을 배포

1. Azure Load Balancer Standard 뒤에 있는 가용성 영역에 배포된 고가용성 Azure VM을 분석

1. 연습에서 배포된 Azure 리소스를 제거합니다.


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure Load Balancer Standard 뒤에 있는 가용성 영역에 고가용성 Azure VM을 배포

1. 필요한 경우 Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

1. Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택한 다음 Cloud Shell 홈 디렉터리에 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305subb.json** 파일을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 리소스 그룹을 만듭니다(`<Azure region>` 자리 표시자는 구독에서 사용할 수 있으며 랩 컴퓨터 위치와 가장 가까운 Azure 지역의 이름으로 바꿉니다).

   ```Bash
   LOCATION='<Azure region>'
   ```
   
   ```Bash
   az deployment sub create \
   --location $LOCATION \
   --template-file azuredeploy30305subb.json \
   --parameters rgName=az30305b-labRG rgLocation=$LOCATION
   ```

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgb.json**을 업로드합니다.

1. Cloud Shell 창에서 Azure Resource Manager 매개 변수 파일 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgb.parameters.json**을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 두 개의 가용성 영역에서 Windows Server 2019 데이터 센터 코어를 호스팅하는 Azure VM 쌍으로 구성된 백 엔드 풀로 Azure Load Balancer 표준을 배포합니다.

   ```Bash
   az deployment group create \
   --resource-group az30305b-labRG \
   --template-file azuredeploy30305rgb.json \
   --parameters @azuredeploy30305rgb.parameters.json
   ```

    > **참고**: 다음 작업으로 진행하기 전에 배포가 완료될 때까지 기다립니다. 약 10분이 소요됩니다.

1. Azure Portal에서 Cloud Shell 창을 닫습니다. 


#### 작업 2: Azure Load Balancer Standard 뒤에 있는 가용성 영역에 배포된 고가용성 Azure VM을 분석

1. Azure Portal에서 **Network Watcher**를 검색 및 선택하고 **Network Watcher** 블레이드에서 **토폴로지**를 선택합니다.

1. **Network Watcher** \| 에서 **토폴로지** 블레이드는 다음 설정을 지정합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30305b-LabRG** |
    | 가상 네트워크 | **az30305b-vnet** |

1. 결과 토폴로지 다이어그램을 검토하여 백 엔드 풀에서 공용 IP 주소, 부하 분산 장치 및 Azure VM의 네트워크 어댑터 간의 연결을 확인합니다.

    > **참고**: 이 다이어그램은 다른 영역(사실상 Azure 데이터 센터)에 있음에도 불구하고 Azure VM이 동일한 서브넷에 있기 때문에 이전 연습에서 본 다이어그램과 거의 동일합니다.
    
1. **Network Watcher** 블레이드에서 **효과적인 보안 규칙**을 선택합니다.

1. **Network Watcher** \| 에서 **효과적인 보안 규칙** 블레이드는 다음 설정을 지정합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30305b-LabRG** |
    | 가상 머신 | **az30305b-vm0** |
    | 네트워크 인터페이스 | **az30305b-nic0** |

1. RDP 및 HTTP를 통한 인바운드 연결을 허용하는 두 가지 사용자 지정 규칙을 포함하여 관련 네트워크 보안 그룹과 효과적인 보안 규칙을 검토합니다. 

    > **참고**: 이 목록은 또한 이전 연습에서 본 목록과 거의 동일하며 두 Azure VM이 연결된 서브넷과 연결된 네트워크 보안 그룹을 사용하여 네트워크 수준 보호가 구현됩니다. 그러나 이 경우 네트워크 보안 그룹은 HTTP 및 RDP 트래픽이 Azure Load Balancer 표준 SKU의 사용으로 인해 백 엔드 풀 Azure VM에 도달하는 데 필요한 것입니다(기본 SKU를 사용할 때 NSG는 선택 사항임).  
    
    > **참고**: 또는 다음에서 **효과적인 보안 규칙**을 확인할 수 있습니다.
    - **az30305a-nic0** 네트워크 인터페이스 블레이드
    - **az30305a-web-nsg** 네트워크 보안 그룹 블레이드 

1. **Network Watcher** 블레이드에서 **연결 문제 해결**을 선택합니다.

    > **참고**: 의도는 동일한 가용성 집합에서 두 Azure VM의 근접(네트워킹 용어)을 확인하는 것입니다.

1. **Network Watcher** \| 에서 **연결 문제 해결** 블레이드에서 다음 설정을 지정하고 **확인**을 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30305b-LabRG** |
    | 원본 유형 | **가상 머신** |
    | 가상 머신 | **az30305b-vm0** |
    | 대상 | **가상 머신 선택** |
    | 리소스 그룹 | **az30305b-LabRG** |
    | 가상 머신 | **az30305b-vm1** |
    | 프로토콜 | **TCP** |
    | 대상 포트| **80** |

    > **참고**: **Azure Network Watcher Agent** VM 확장이 Azure VM에 설치되려면 결과를 잠시 기다려야 합니다.

1. 결과를 검토하고 Azure VM 간의 네트워크 연결 대기 시간을 기록합니다.

    > **참고**: 두 VM이 서로 다른 영역(다른 Azure 데이터 센터 내)에 있기 때문에 대기 시간은 이전 실습에서 관찰한 대기 시간보다 약간 높을 수 있습니다.

1. Azure Portal에서 **az30305b-labRG** 리소스 그룹 블레이드로 이동하고, 리소스 목록에서 **az30305b-vm0** 가상 머신 항목을 선택한 뒤 **az30305b-vm0** 블레이드에서 **위치** 및 **가용성 영역** 항목을 확인합니다. 

1. Azure Portal에서 **az30305b-labRG** 리소스 그룹 블레이드로 이동하고, 리소스 목록에서 **az30305b-vm1** 가상 머신 항목을 선택한 뒤 **az30305b-vm1** 블레이드에서 **위치** 및 **가용성 영역** 항목을 확인합니다. 

    > **참고**: 검토한 항목은 각 Azure VM이 다른 가용성 영역에 있는지 확인합니다.

1. Azure Portal에서 **az30305b-labRG** 리소스 그룹 블레이드로 이동하고, 리소스 목록에서 **az30305b-lb** 부하 분산 장치 항목을 선택한 뒤 **az30305b-lb**블레이드에서 공용 IP 주소 항목을 확인합니다.

1. Azure Portal의 Cloud Shell 창에서 새 **Bash** 세션을 시작합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Azure Load Balancer의 백 엔드 풀에서 Azure VM에 대한 HTTP 트래픽의 부하 분산을 테스트합니다(여기서, `<lb_IP_address>` 자리 표시자를 이전에 식별한 부하 분산 장치 프런트 엔드의 IP 주소로 대체함).

   ```Bash
   for i in {1..4}; do curl <lb_IP_address>; done
   ```

    > **참고**: 반환된 메시지가 백 엔드 Azure VM에 라운드 로빈 방식으로 요청이 전달되고 있음을 나타내는지 확인합니다.

1. **az30305b-lb** 블레이드에서 **부하 분산 규칙** 항목을 선택하고 **az30305b-lb** | 에서 **부하 분산 규칙** 블레이드에서 HTTP 트래픽을 처리하는 부하 분산 규칙을 나타내는 **az303005b-lbruletcp80** 항목을 선택합니다. 

1. **az303005b-lbruletcp80** 블레이드의 **세션 지속성** 드롭다운 목록에서 **클라이언트 IP**를 선택한 다음 **저장**을 선택합니다.

1. 업데이트가 완료될 때까지 기다렸다가 Cloud Shell 창에서 다음을 다시 실행하여 세션 지속성 없이 Azure Load Balancer의 백 엔드 풀에서 Azure VM에 대한 HTTP 트래픽의 부하 분산을 테스트합니다(`<lb_IP_address>` 자리 표시자를 이전에 식별한 부하 분산 장치의 프런트 엔드 IP 주소로 대체함).

   ```Bash
   for i in {1..4}; do curl <lb_IP_address>; done
   ```

    > **참고**: 반환된 메시지가 동일한 백 엔드 Azure VM으로 요청이 전달되고 있음을 나타내는지 확인합니다.

1. Azure Portal에서 **az30305b-lb** 블레이드로 다시 이동하여 **인바운드 NAT 규칙** 항목을 선택하고 각각 TCP 포트 33890 및 33891의 원격 데스크톱을 통해 백 엔드 풀 VM의 첫 번째 및 두 번째에 연결할 수 있는 두 가지 규칙을 기억합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Azure Load Balancer의 백 엔드 풀에서 첫 번째 Azure VM에 NAT를 통한 원격 데스크톱 연결을 테스트합니다(`<lb_IP_address>` 자리 표시자는 이전에 식별한 부하 분산 장치의 프런트 엔드 IP 주소로 대체함).

   ```Bash
   curl -v telnet://<lb_IP_address>:33890
   ```

    > **참고**: 반환된 메시지가 성공적으로 연결되어 있음을 나타내는지 확인합니다. 

1. **Ctrl+C** 키 조합을 눌러 Bash 셸 프롬프트로 돌아간 후 다음을 실행하여 Azure Load Balancer의 백엔드 풀에서 두 번째 Azure VM에 NAT를 통한 원격 데스크톱 연결을 테스트합니다(`<lb_IP_address>` 자리 표시자는 이전에 식별한 부하 분산 장치의 프런트 엔드 IP 주소로 대체함).

   ```Bash
   curl -v telnet://<lb_IP_address>:33891
   ```

    > **참고**: 반환된 메시지가 성공적으로 연결되어 있음을 나타내는지 확인합니다. 

1. **Ctrl+C** 키 조합을 눌러 Bash 셸 프롬프트로 돌아가 Cloud Shell 창을 닫습니다.

1. **az30305b-lb** 블레이드에서 **부하 분산 규칙** 항목을 선택하고 **az30305b-lb** | 에서| **부하 분산 규칙** 블레이드에서 HTTP 트래픽을 처리하는 부하 분산 규칙을 나타내는 **az303005b-lbruletcp80** 항목을 선택합니다. 

1. **az303005b-lbruletcp80** 블레이드의 **아웃바운드 SNAT(Source Network Address Translation)** 섹션에서 **아웃바운드 규칙을 사용하여 백 엔드 풀 구성원에게 인터넷 액세스 권한 제공(권장)** 을 선택한 다음 **저장**을 선택합니다.

1. **az30305b-lb** 블레이드로 다시 이동하여 **아웃바운드 규칙** 항목을 선택하고 **az30305b-lb** | 에서 **아웃바운드 규칙** 블레이드에서 **+ 추가**를 선택합니다.

1. **아웃바운드 규칙 추가** 블레이드에서 다음 설정을 지정하고 **추가**를 선택합니다(다른 모든 설정을 기본값으로 남겨둡니다).

    | 설정 | 값 | 
    | --- | --- |
    | 이름 | **az303005b-obrule** | 
    | 프론트 엔드 IP 주소 | **az30305b-lb** 부하 분산 장치의 기존 프런트 엔드 IP 주소의 이름 |
    | 백 엔드 풀 | **az30305b-bepool** |
    | 포트 할당 | **아웃바운드 포트 수를 수동으로 선택** |
    | 선택 기준 | **최대 백 엔드 인스턴스 수** |
    | 최대 백 엔드 인스턴스 수 | **3** |

    > **참고**: Azure Load Balancer 표준을 사용하면 아웃바운드 트래픽에 대한 전용 프런트 엔드 IP 주소를 지정할 수 있습니다(여러 프런트 엔드 IP 주소가 할당된 경우).

1. Azure Portal에서 **az30305b-labRG** 리소스 그룹 블레이드로 이동하여 리소스 목록에서 **az30305b-vm0** 가상 머신 항목을 선택하고 **az30305b-vm0** 블레이드의 **작업** 블레이드에서 **실행 명령**을 선택합니다.

1. **az30305b-vm0** \|에서 **실행 명령** 블레이드에서 **RunPowerShellScript**를 선택합니다. 

1. **실행 명령 스크립트** 블레이드의 **PowerShell 스크립트** 텍스트 상자에서 다음을 입력하고 **실행**을 선택합니다.

   ```powershell
   (Invoke-RestMethod -Uri "http://ipinfo.io").IP
   ```

    > **참고**: 이 명령은 웹 요청이 시작된 공용 IP 주소를 반환합니다.

1. 출력을 검토하고 아웃바운드 부하 분산 규칙에 할당된 Azure Load Balancer 표준의 프런트 엔드에 할당된 공용 IP 주소와 일치하는지 확인합니다.


#### 작업 3: 연습에서 배포된 Azure 리소스를 제거합니다.

1. Azure Portal의 Cloud Shell 창에서 새 **Bash** 세션을 시작합니다. 

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```Bash
   az group list --query "[?starts_with(name,'az30305b-')]".name --output tsv
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```Bash
   az group list --query "[?starts_with(name,'az30305b-')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Cloud Shell 창을 닫습니다.


### 연습 3: 가용성 영역 및 Azure Application Gateway를 사용하여 고가용성 Azure VM 확장 집합 배포를 구현 및 분석합니다.
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 Azure Application Gateway 뒤에 있는 가용성 영역에 고가용성 Azure VM 확장 집합을 배포

1. Azure Application Gateway 뒤에 있는 가용성 영역에 배포된 가용성이 높은 Azure VM Scale Set 분석

1. 연습에서 배포된 Azure 리소스를 제거합니다.


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure Application Gateway 뒤에 있는 가용성 영역에 고가용성 Azure VM 확장 집합을 배포

1. 필요한 경우 Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **Bash**를 선택합니다. 

1. Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택한 다음 Cloud Shell 홈 디렉터리에 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305subc.json** 파일을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 리소스 그룹을 만듭니다(`<Azure region>`자리 표시자는 구독에서 사용할 수 있으며 랩 컴퓨터 위치와 가장 가까운 Azure 지역의 이름으로 바꿉니다).

   ```Bash
   az deployment sub create --location '<Azure region>' --template-file azuredeploy30305subc.json --parameters rgName=az30305c-labRG rgLocation='<Azure region>'
   ```

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgc.json**을 업로드합니다.

1. Cloud Shell 창에서 Azure Resource Manager 매개 변수 파일 **\\\\AZ303\\AllFiles\\Labs\\05\\azuredeploy30305rgc.parameters.json**을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 Windows 서버 2019 데이터 센터 코어를 호스팅하는 Azure VM 쌍으로 구성된 백 엔드 풀이 있는 Azure Application Gateway를 다양한 가용성 영역에 배포합니다.

   ```
   az deployment group create --resource-group az30305c-labRG --template-file azuredeploy30305rgc.json --parameters @azuredeploy30305rgc.parameters.json
   ```

    > **참고**: 다음 작업으로 진행하기 전에 배포가 완료될 때까지 기다립니다. 약 10분이 소요됩니다.

1. Azure Portal에서 Cloud Shell 창을 닫습니다. 


#### 작업 2: Azure Application Gateway 뒤에 있는 가용성 영역에 배포된 가용성이 높은 Azure VM Scale Set 분석

1. Azure Portal에서 **Network Watcher**를 검색 및 선택하고 **Network Watcher** 블레이드에서 **토폴로지**를 선택합니다.

1. **Network Watcher** \| **에서 토폴로지** 블레이드는 다음 설정을 지정합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30305c-LabRG** |
    | 가상 네트워크 | **az30305c-vnet** |

1. 결과 토폴로지 다이어그램을 검토하여 백 엔드 풀에서 Azure 가상 머신 확장 집합의 공용 IP 주소, 부하 분산 장치 및 Azure VM 인스턴스의 네트워크 어댑터 간 연결을 확인합니다. 

    > **참고**: 또한 Azure Application Gateway를 배포하려면 다이어그램에 포함된 전용 서브넷이 필요합니다(게이트웨이가 표시되지 않는 경우에도).

    > **참고**: 이 구성에서는 Network Watcher를 사용하여 효과적인 네트워크 보안 규칙(Azure VM과 Azure VM Scale Set 인스턴스 간의 차이점 중 하나)을 확인할 수 없습니다. 마찬가지로 Azure VM Scale Set 인스턴스에서 네트워크 연결을 테스트하기 위해 **연결 문제 해결** 사용에 의존할 수는 없지만 Azure Application Gateway에서 연결을 테스트하는 데 사용할 수는 있습니다.

1. Azure Portal에서 **az30305c-labRG** 리소스 그룹 블레이드로 이동하여 리소스 목록에서 **az30305c-vmss** 가상 머신 확장 집합 항목을 선택합니다. 

1. **az30305c-vmss** 블레이드에서 **위치** 및 **장애 도메인** 항목을 확인합니다. 

    > **참고**: Azure VM과 달리 Azure VM Scale Sets의 개별 인스턴스는 동일한 영역에 배포된 인스턴스를 포함하여 별도의 장애 도메인에 배포합니다. 또한 최대 3개의 장애 도메인을 사용할 수 있는 Azure VM과 달리 5개의 장애 도메인을 지원합니다. 

1. **az30305c-vmss** 블레이드에서 **인스턴스**를 선택하고 **az30305c-vmss** | 에서 **인스턴스** 블레이드에서 첫 번째 인스턴스를 선택하고 **위치** 속성 값을 검토하여 가용성 영역을 식별합니다. 

1. **az30305c-vmss** \| 로 다시 이동하여 **인스턴스** 블레이드에서 두 번째 인스턴스를 선택하고 **위치** 속성 값을 검토하여 가용성 영역을 식별합니다. 

    > **참고**: 각 인스턴스가 다른 가용성 영역에 있는지 확인합니다.

1. Azure Portal에서 **az30305c-labRG** 리소스 그룹 블레이드로 이동하여 리소스 목록에서 **az30305c-appgw** 부하 분산 장치 항목을 선택하고 **az30305c-appgw** 블레이드에서 공용 IP 주소 항목을 확인합니다.

1. Azure Portal의 Cloud Shell 창에서 새 **Bash** 세션을 시작합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Azure Application Gateway의 백 엔드 풀에서 Azure VM Scale Set 인스턴스에 대한 HTTP 트래픽의 부하 분산을 테스트합니다(`<lb_IP_address>` 자리 표시자는 이전에 식별한 게이트웨이의 프런트 엔드 IP 주소로 대체함).

   ```Bash
   for i in {1..4}; do curl <appgw_IPaddress>; done
   ```

    > **참고**: 반환된 메시지가 백 엔드 Azure VM에 라운드 로빈 방식으로 요청이 전달되고 있음을 나타내는지 확인합니다.

1. **az30305c-appgw** 블레이드에서 **HTTP 설정** 항목을 선택하고 **az30305c-appgw** | 에서 **HTTP 설정** 블레이드에서 HTTP 트래픽을 처리하는 부하 분산 규칙을 나타내는 **appGwBackentHttpSettings** 항목을 선택합니다. 

1. **appGwBackentHttpSettings** 블레이드에서 변경 없이 기존 설정을 검토하고 **쿠키 기반 선호도**를 활성화할 수 있음을 확인합니다.

    > **참고**: 이 기능을 사용하려면 클라이언트가 쿠키 사용을 지원해야 합니다.

    > **참고**: Azure Application Gateway를 사용하여 Azure VM Scale Set 인스턴스에 대한 RDP 연결을 위해 NAT를 구현할 수 없습니다. Azure Application Gateway는 HTTP/HTTPS 트래픽만 지원합니다.


### 연습 4: 가용성 영역 및 Azure Applicatoin Gateway를 사용하여 Azure VM Scale Sets의 자동 크기 조정을 구현합니다.
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure VM Scale Set의 자동 크기 조정 구성

1. Azure VM Scale Set의 자동 크기 조정 테스트

#### 작업 1: Azure VM Scale Set의 자동 크기 조정 구성

1. Azure Portal에서 **az30305c-labRG** 리소스그룹 블레이드로 이동하여 리소스 목록에서 **az30305c-vmss** 가상 머신 확장 집합 항목을 선택하고, **az30305c-vmss** 블레이드에서 **크기 조정**을 선택합니다. 

1. **az30305c-vmss** | 에서 **크기 조정** 블레이드에서 **사용자 지정 자동 크기 조정** 옵션을 선택합니다.

1. **사용자 지정 크기 조정** 섹션에서 다음 설정을 지정합니다(나머지는 기본값을 그대로 유지).

    | 설정 | 값 | 
    | --- | --- |
    | 크기 조정 모드 | **메트릭을 기반으로 스케일링** |
    | 인스턴스 한도 최소 | **1** |
    | 인스턴스 한도 최대 | **3** |
    | 인스턴스 한도 기본 | **1** |

1. **규칙 추가**를 선택합니다.

1. **크기 조정 규칙** 블레이드에서 다음 설정을 지정하고 **추가**를 선택합니다(나머지는 기본값을 그대로 유지).

    | 설정 | 값 | 
    | --- | --- |
    | 시간 집계 | **최대** |
    | 메트릭 네임스페이스 | **가상 머신 호스트** |
    | 메트릭 이름 | **CPU 비율** |
    | VMName 연산자 | **=** |
    | 차원 값 | **az30305c-vmss_0** |
    | 인스턴스 수별 메트릭 분할 사용 | **사용** |
    | 연산자 | **보다 큼** |
    | 스케일링 작업을 트리거하는 메트릭 임계값 | **1** |
    | 지속 시간(분) | **1** |
    | 시간 조직 통계 | **최대** |
    | 작업 | **개수 증가 기준** |
    | 인스턴스 개수 | **1** |
    | 휴지 기간(분) | **5** |

    > **참고**: 이러한 값은 가능한 한 빨리 크기 조정을 트리거하려는 랩 목적을 위해 엄격하게 선택됩니다. Azure VM Scale Set 크기 조정에 대한 지침은 [Microsoft Docs](https://docs.microsoft.com/ko-kr/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)를 참조하세요. 

1. **az30305c-vmss** | 로 돌아갑니다 **크기 조정** 블레이드, **+ 규칙 추가**를 선택합니다.

1. **크기 조정 규칙** 블레이드에서 다음 설정을 지정하고 **추가**를 선택합니다(나머지는 기본값을 그대로 유지).

    | 설정 | 값 | 
    | --- | --- |
    | 시간 집계 | **평균** |
    | 메트릭 네임스페이스 | **가상 머신 호스트** |
    | 메트릭 이름 | **CPU 비율** |
    | VMName 연산자 | **=** |
    | 차원 값 | **2개 선택됨** |
    | 인스턴스 수별 메트릭 분할 사용 | **사용** |
    | 연산자 | **미만** |
    | 스케일링 작업을 트리거하는 메트릭 임계값 | **1** |
    | 지속 시간(분) | **1** |
    | 시간 조직 통계 | **최소** |
    | 작업 | **개수 감소 기준** |
    | 인스턴스 개수 | **1** |
    | 휴지 기간(분) | **5** |

1. **az30305c-vmss** 로 돌아갑니다. **| 크기 조정**블레이드에서 **저장**을 선택합니다.


#### 작업 2: Azure VM Scale Set의 자동 크기 조정 테스트

1. Azure Portal의 Cloud Shell 창에서 새 **Bash** 세션을 시작합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Azure Application Gateway의 백 엔드 풀에서 Azure VM Scale Set 인스턴스의 자동 크기 조정을 트리거합니다(`<lb_IP_address>` 자리 표시자를 이전에 식별한 게이트웨이의 프런트 엔드 IP 주소로 대체).

   ```Bash
   for (( ; ; )); do curl -s <lb_IP_address>?[1-10]; done
   ```
1. Azure Portal의 **az30305c-vmss** 블레이드에서 **CPU(평균)** 차트를 검토하고, Application Gateway의 CPU 사용률이 트리거를 확장할 만큼 충분히 증가했는지 확인하세요.

    > **참고**: 몇 분 기다려야 할 수도 있습니다.

1. **az30305c-vmss** 블레이드에서 **인스턴스** 항목을 선택하고 인스턴스 수가 증가했는지 확인합니다.

    > **참고**: **az30305c-vmss** | 를 새로 고침해야 할 수 있습니다 **인스턴스** 블레이드.

    > **참고**: 인스턴스 수가 1이 아닌 2씩 증가하는 것을 볼 수 있습니다. 최종 실행 인스턴스 수가 3개인 한 이 항목이 예상됩니다. 

1. Azure Portal에서 **Cloud Shell** 창을 닫습니다. 

1. Azure Portal의 **az30305c-vmss** 블레이드에서 **CPU(평균)** 차트를 검토하고, Application Gateway의 CPU 사용률이 트리거를 축소하기에 충분히 감소했는지 확인하세요. 

    > **참고**: 몇 분 기다려야 할 수도 있습니다.

1. **az30305c-vmss** 블레이드에서 **인스턴스** 항목을 선택하고 인스턴스 수가 2로 감소했는지 확인하세요.

    > **참고**: **az30305c-vmss** | 를 새로 고침해야 할 수 있습니다 **인스턴스** 블레이드.

1. **az30305c-vmss** 블레이드에서 **크기 조정**을 선택합니다. 

1. **az30305c-vmss** | 에서 **크기 조정** 블레이드에서 **수동 크기 조정** 옵션을 선택하고 **저장**을 선택합니다.

    > **참고**: 이를 통해 다음 연습 중에 원하지 않는 크기 조정을 방지합니다. 


### 연습 5: Azure VM Scale Sets의 수직 크기 조정 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Azure 가상 머신 확장 집합 인스턴스의 컴퓨팅 리소스 크기를 조정합니다.

1. Azure 가상 머신 확장 집합 인스턴스의 스토리지 리소스의 크기를 조정합니다.


#### 작업 1: Azure 가상 머신 확장 집합 인스턴스의 리소스 크기를 조정합니다.

1. Azure Portal의 **az30305c-vmss** 블레이드에서 **크기**를 선택합니다.

1. 사용 가능한 크기 리스트에서 현재 구성된 것 이외의 사용 가능한 크기를 선택하고 **크기 조정**을 선택합니다.

1. **az30305c-vmss** 블레이드에서 **인스턴스** 항목을 선택하고, **az30305c-vmss** | 에서 **인스턴스** 블레이드에서 기존 인스턴스를 원하는 크기의 새 인스턴스로 대체하는 프로세스를 관찰합니다.

    > **참고**: **az30305c-vmss** | 를 새로 고침해야 할 수 있습니다 **인스턴스** 블레이드.

1. 인스턴스가 업데이트되고 실행될 때까지 기다립니다.


#### 작업 2: Azure 가상 머신 확장 집합 인스턴스의 스토리지 리소스 크기를 조정합니다.

1. **az30305c-vmss** 블레이드에서 **디스크**를 선택하고, **+ 데이터 디스크 추가**를 선택한 후, 다음 설정으로 새 관리 디스크를 연결하고 나서(다른 설정은 기본값을 그대로 사용) **저장**을 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | LUN | **0** |
    | 크기 | **32** |
    | 스토리지 계정 유형 | **표준 HDD** |

1. **az30305c-vmss** 블레이드에서 **인스턴스** 항목을 선택하고, **az30305c-vmss** | 에서 **인스턴스** 블레이드에서 기존 인스턴스를 업데이트하는 프로세스를 관찰합니다.

    > **참고**: 이전 단계에서 연결된 디스크는 원시 디스크입니다. 사용하려면 먼저 파티션을 만들고 포맷한 후 탑재해야 합니다. 이를 위해 사용자 지정 스크립트 확장을 통해 Azure VM 확장 집합 인스턴스에 PowerShell 스크립트를 배포합니다. 하지만 먼저 제거해야 합니다.

1. **az30305c-vmss** 블레이드에서 **확장**을 선택하고, **az30305c-vmss** | 에서 **확장** 블레이드, **customScriptExtension** 항목을 선택한 다음 **확장** 블레이드에서 **제거**를 선택합니다.

    > **참고**: 제거가 완료될 때까지 기다립니다.

1. Azure Portal에서 **az30305c-labRG** 리소스 그룹 블레이드로 이동한 다음 리소스 리스트에서 스토리지 계정 리소스를 선택합니다. 

1. 스토리지 계정 블레이드에서 **컨테이너**를 선택한 다음 **+ 컨테이너**를 선택합니다. 

1. **새 컨테이너** 블레이드에서 다음 설정을 지정하고(다른 설정은 기본값을 그대로 사용) **만들기**를 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 이름 | **스크립트** |
    | 공용 액세스 수준 | **프라이빗(익명 액세스 불가**) |
    
1. 컨테이너 목록을 표시하는 스토리지 계정 블레이드로 돌아가서 **스크립트**를 선택합니다.

1. **스크립트** 블레이드에서 **업로드**를 선택합니다.

1. **Blob 업로드** 블레이드에서 폴더 아이콘을 선택하고, **열기**대화 상자에서 **\\\AZ303\\AllFiles\Labs\05** 폴더로 이동하여**az30305e-configure_VMSS_with_data_disk.ps1**, **열기**를 차례로 선택하고 **Blob 업로드** 블레이드로 돌아와 **업로드**를 선택합니다. 

1. Azure Portal에서 **az30305c-vmss** 가상 머신 확장 집합 블레이드로 다시 이동합니다. 

1. **az30305c-vmss** 블레이드에서 **확장**을 선택하고, **az30305c-vmss** | 에서 **확장** 블레이드에서 **+ 추가**를 선택한 다음, **확장** 블레이드의 **customScriptExtension** 항목을 선택합니다.

1. **새 리소스** 블레이드에서 **사용자 지정 스크립트 확장**을 선택한 다음 **만들기**를 선택합니다.

1. **확장 설치** 블레이드에서 **찾아보기**를 선택합니다. 

1. **스토리지 계정** 블레이드에서 **az30305e-configure_VMSS_with_data_disk.ps1** 스크립트를 업로드한 스토리지 계정의 이름을 선택하고, **컨테이너** 블레이드에서 **스크립트**를 선택하고, **스크립트**블레이드에서 **az30305e-configure_VMSS_with_data_disk.ps1**을 선택한 다음 **선택**을 선택합니다. 

1. **확장 설치** 블레이드로 돌아가서 **확인**을 선택합니다.

1. **az30305c-vmss** 블레이드에서 **인스턴스** 항목을 선택하고, **az30305c-vmss** 에서 **| 인스턴스** 블레이드, 기존 인스턴스를 업데이트하는 프로세스를 관찰합니다.

    > **참고**: **az30305c-vmss** | 를 새로 고침해야 할 수 있습니다 **인스턴스** 블레이드.


#### 작업 3: 연습에서 배포된 Azure 리소스를 제거합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```Bash
   az group list --query "[?starts_with(name,'az30305c-')]".name --output tsv
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```Bash
   az group list --query "[?starts_with(name,'az30305c-')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```
   
1. Cloud Shell 창을 닫습니다.
