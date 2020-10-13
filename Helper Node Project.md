# 목차

1. [OCP란 무엇인가?](#ocp란-무엇인가?)
    1. [Red Hat OpenShift의 특징](#red-hat-openshift의-특징)
2. [OCP 4.5 클러스터 아키텍처](#ocp-4.5-클러스터-아키텍처)
    1. [Bootstrap](#bootstrap) 
    2. [Master](#master)
    3. [Worker](#worker)
    4. [Bastion](#bastion)
    5. [HAProxy](#haproxy)
    6. [PXE](#pxe)
        1. [DNS](#dns)
        2. [DHCP](#dhcp)
        3. [TFTP](#tftp)
        4. [FTP, HTTP, NFS](#ftp,-http,-nfs)
3. [Helper Node를 이용한 Bare-metal에 클러스터 구축](#helper-node를-이용한-bare-metal에-클러스터-구축)
4. [서비스 소개](#서비스-소개)
5. [서비스 구축과정](#서비스-구축과정)

# OCP란 무엇인가?
> Red Hat OpensShift Container Platform<br>하이브리드 클라우드로, 더 좋은 애플리케이션을 더 빠르게 빌드하고 제공하기 위한 기업형 쿠버네티스 기반의 플랫폼

## Red Hat OpenShift의 특징
- 컨테이너 호스트와 런타임
    - 컨테이너 호스트 운영체제로 Kubernetes의 Master용 RHCOS와 Worker용 RHEL를 포함
    - 컨테이너 엔진으로는 표준 Docker와 CRI-O 컨테이너 런타임을 지원(OCP 4.5는 CRi-O 컨테이너 엔진 사용)
- 통합된 컨테이너 레지스트리
    - 통합된 프라이빗 컨테이너 저장소를 제공(Kubernetes의 일부분으로 설치되거나 유연성을 높이기 위해 독단적으로 설치)
- 기업형 Kubernetes
    - Kubernetes의 모든 릴리즈에 대한 업스트립의 결함, 보안, 성능 문제에 대한 수백개의 수정을 포함
    - 수십개의 기술들로 테스트되고 9년의 생명주기에 걸쳐 지원되는 통합된 플랫폼
- 개발자 워크플로우
    - 젠킨스 파이프라인과  애플리케이션의 코드에서 컨테이너로 직행하는 S2I 기술을 기본으로 포함하여 더 빠른 개발을 돕는 간소화된 워크플로우를 제공
    - Istio와 Knative와 같은 새 프레임워크로 확장 가능
- 인증된 통합
    - 소프트웨어적으로 정의된 네트워크를 포함하고 추가적인 일반적 네트워크 솔루션을 인증
    - 모든 릴리즈에서 수많은 스토리지와 third-party 플러그인을 인증
- 쉬운 서비스 접근
    - 서비스 중개자(AWS 서비스의 직접적 접근 등), 검증된 third-party  솔루션, 내장된 OperatorHub를 통한 Kubernetes 오퍼레이터로 관리자를 돕고 애플리케이션 팀을 지원
- 신뢰할 수 있는 플랫폼
    - 애플리케이션의 라이프사이클 전반에 지속적인 보안검사를 컨테이너 스택에 구축
- 통합 생태계
    - 수백개의 파트너 사와 협력하여 오픈시프트의 기술 통합을 검증
- 내장된 모니터링
    - 네이티브 클라우드 클러스터와 애플리케이션 모니터링의 표준 Prometheus를 포함
    - 시각화 도구로 Grafana 대시보드 사용
- 중앙집중식 정책관리
    - 관리자에게 모든 오픈시프트 클러스터에 걸쳐 통합된 콘솔을 이용해 여러 팀에서 정책을 구현하고 수행 할 수 있는 하나의 공간을 제공
- 주문형 환경
    - 중앙집중식 관리와 제어로 승인된 서비스와 인프라구조를 제공, 사용자가 원하는 환경을 직접 구성하는 셀프 서비스


# OCP 4.5 클러스터 아키텍처

## Bootstrap
OCP는 Master 노드에 필요한 정보를 제공하기 위한 초기 설정동안 일시적인 Bootstrap 노드를 사용한다. 클러스터를 어떻게 생성할지 명세한 Ignition config 파일을 이용해 부팅을 시작한다. Bootstrap 노드가 ignition config 파일을 바탕으로 Master 노드를 구성하고, Master 노드가 Worker 노드를 생성한다. Bootstrap 머신은 OS로 Red Hat CoreOS(RHCOS)를 사용한다.

## Master
 Control plane 역할을 하는 모든 머신들이 Master 머신들이기 때문에, 용어를 설명하는데 'Master'와 'Control plane'은 같은 의미의 용어로 사용된다.
 Master 노드는 전체 클러스터 환경의 제어기능을 수행하는 서버이다. OpenShift와 관련된 모든 오브젝트의 생성 및 관리, 스케줄링을 담당한다. 클러스터를 관리하기 위한 Kubernetes 서비스들(API 서버, etcd, Controller Manager 서버 등)을 하나의 OpenShift 바이너리에 포함한다. OCP 4.5의 모든 Control plane 머신에는 운영체제로 반드시 RHCOS를 사용해야한다.

3대의 Master 노드를 사용한다. 비록 이론적으로 어떤 수의 master 노드를 사용 가능하더라도, master의 static 파드와 etcd static 파드가 같은 호스트에서 작동하기에 etcd 쿼럼에 의해 그 수가 제한된다. 

## Worker
Worker 노드는 사용자에 의해 요청된 실제 워크로드가 동작하고 관리되는 곳이다. CRI-O(컨테이너 엔진), kubelet(컨테이너의 워크로드의 동작과 정지 요청을 수용하고 만족시키는 서비스), 그리고 kube-proxy(파드와 Worker 간의 의사소통을 관리)와 같은 중요한 서비스들이 각 Worker 노드에서 작동한다. Worker 역할의 머신들은 autoscale되는 명세된 머신 풀에 의해 관리되는 컴퓨팅 워크로드를 구동한다. OCP가 여러 머신 종류를 지원하기 때문에 Worker 머신들은 Compute 머신으로 분류된다. 4.5 릴리스에서 Compute 머신의 유일한 기본 유형이 Worker 머신이기 때문에 'Worker  머신'과 'Compute 머신' 은 같은 의미로 사용된다. Compute 머신에는 운영체제로 RHCOS 뿐만 아니라 Red Hat Enterprise Linux(RHEL)도 사용 할 수 있다.

## Bastion

## HAProxy

## PXE

### DNS

### DHCP

### TFTP

### FTP, HTTP, NFS

# Helper Node를 이용한 Bare-metal에 클러스터 구축
    물리머신 환경
    - Ubuntu 18.04
    - 500 GB disk
    - 8 vCPUs    
    - 20 GB of RAM    
    - 192.168.20.0/24 (wifi)


## Helper Node

## 구축 과정 오류

## 최종 구축 과정

가상머신 환경
	
<최소 요구사항>
|    Node   |     Operating SYS    | vCPUs |   RAM  | Disk Storage |
|:---------:|:--------------------:|:-----:|:------:|:------------:|
| Helper    | CentOS/RHEL 7 or 8   | 4     |   8 GB | 50 GB        |
| Bootstrap | RHCOS                | 4     | 16 GB  | 120 GB       |
| Master    | RHCOS                | 4     | 16 GB  | 120 GB       |
| Worker    | RHCOS or RHEL 7 or 8 | 2     |   8 GB | 120 GB       |

<IP 주소 설정>
|   Node   |    IP address  |
|:--------|:----------------|
|Helper   |192.168.20.111   |
|Bootstrap|192.168.20.150   |
|Master0  |192.168.20.151   |
|Master1  |192.168.20.152   |
|Master2  |192.168.20.153   |
|Worker0  |192.168.20.161   |
|Worker1  |192.168.20.162   |



## 클러스터 구성 후 작업

# 서비스 소개

# 서비스 구축과정



Helper Node를 이용한 Bare-Metal에 클러스터 구축	7
물리머신 환경	7
Helper Node	7
최소 요구사항	7
구축 절차 오류 과정	7
최종 구축 과정&결과	11
사전 설정	11
클러스터 아키텍처	12
Helper Node 구성	12
Playbook 설정&실행	12
Ignition config 파일 생성	13
Bootstrap/Master/Worker Node 구성	15
물리머신에 각 Node의 가상머신 준비	15
가상머신 설치	15
클러스터 구성 완료 후 작업	16
oc command bash completion	17
Web console에 Login	17
기본적인 setting	17
4. Service Introduction	19
아키텍처	19
설명	19
서비스 구축 과정	20
Grafana	20
jupyter notebook	21
Openshift용 Jupyter notebook	21
Importing Minimal Notebook	21
Making MInimal Notebook	21
Deploying Minimal Notebook	21
automation	22
Create the metric	22
Prometheus	22
5.6.1 Prometheus metric library	22
library : prometheus-api-client	22
서비스를 모니터링하려면 먼저 Prometheus 클라이언트 라이브러리 중 하나를 통해 코드에 계측을 추가해야한다. 이들은 Prometheus 측정을 구현한다 .	23
Python Prometheus 클라이언트 라이브러리를 사용하여 애플리케이션 인스턴스의 HTTP 엔드 포인트를 통해 내부 측정 항목을 정의하고 노출 할 수 있습니다.	23
5.6.2 Prometheus metric extraction	23
metric  pdf	24
merge(pdf + image watermark)	24
gui 환경 셋업	24
서비스 시연 결과물	
