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

## Master

## Worker

## Bastion

## HAProxy

## PXE

### DNS

### DHCP

### TFTP

### FTP, HTTP, NFS

# Helper Node를 이용한 Bare-metal에 클러스터 구축

## Helper Node

## 구축 과정 오류

## 최종 구축 과정

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
