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
    1. [Helper Node](#helper-node)
    2. [구축 과정 오류](#구축-과정-오류)
    3. [최종 구축 과정](#최종-구축-과정)
        1. [사전설정](#사전설정)
	    2. [클러스터 아키텍처](#클러스터-아키텍처)
	    3. [Helper Node 구성](#helper-node-구성)
	        1. [Playbook 설정&실행](#playbook-설정&실행)
	        2. [Ignition config 파일 생성](#ignition-config-파일-생성)
	    4. [Bootstrap/Master/Worker Node 구성](#bootstrap/master/worker-node-구성)
	        1. [물리머신에 각 Node의 가상머신 준비](#물리머신에-각-node의-가상머신-준비)
	        2. [가상머신 설치](#가상머신-설치)
    4. [클러스터 구성 완료 후 작업](#클러스터-구성-완료-후-작업)
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

### 사전설정
1. 물리 머신 환경
- Ubuntu Desktop 18.04
- virt-manager 설치
```
sudo apt update 
sudo apt install -y qemu-kvm libvirt-bin bridge-utils virtinst virt-manager 
sudo systemctl enable libvirtd 
reboot
```
- 무선 네트워크 드라이버 설치
```
sudo apt install -y bc module-assistant build-essential dkms git
sudo m-a prepare 
git clone https://github.com/tomaspinho/rtl8821ce 
cd rtl8821ce 
sudo ./dkms-install.sh 
lsmod | grep 8821
```
- 가상머신 원격접속을 위한 openssh 설치
```
sudo apt install -y openssh-server
```

2. 가상머신 환경	
<최소 요구사항>
|    Node   |     Operating SYS    | vCPUs |   RAM  | Disk Storage |
|:---------:|:--------------------:|:-----:|:------:|:------------:|
| Helper    | CentOS/RHEL 7 or 8   | 4     |   8 GB | 50 GB        |
| Bootstrap | RHCOS                | 4     | 16 GB  | 120 GB       |
| Master    | RHCOS                | 4     | 16 GB  | 120 GB       |
| Worker    | RHCOS or RHEL 7 or 8 | 2     |   8 GB | 120 GB       |

<IP 주소 설정>
|   Node   |   IP address    |
|:---------|:----------------|
|Helper    |192.168.20.111   |
|Bootstrap |192.168.20.150   |
|Master0   |192.168.20.151   |
|Master1   |192.168.20.152   |
|Master2   |192.168.20.153   |
|Worker0   |192.168.20.161   |
|Worker1   |192.168.20.162   |

### 클러스터 아키텍처
사진

### Helper Node 구성
> Helper에 dns를 패키지를 설치하는 등 외부접속이 필요한 때는 공인 IP를 넣어주고 이후에는 자신의 IP 주소를 넣어준다. Node가 부팅 시 dns를 따라가 파일을 불러오기 때문

#### Playbook 설정&실행

1. 필요 패키지 설치
```
# EPEL 설치
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %rhel).noarch.rpm

# ansible과 git 설치
yum -y install ansible git

# 플레이북 git에서 가져오기
git clone https://github.com/RedHatOfficial/ocp4-helpernode
cd ocp4-helpernode
```

2. vars.yaml 파일을 환경에 맞게 수정
```
cp docs/examples/vars-static.yaml .
vars-static.yaml
---
staticips: true
helper:
  name: "helpnode"
  ipaddr: "192.168.20.111"
dns:
  domain: "example.com"
  clusterid: "ocp4"
  forwarder1: "8.8.8.8"
  forwarder2: "8.8.4.4"
bootstrap:
  name: "bootstrap"
  ipaddr: "192.168.20.150"
masters:
  - name: "master0"
    ipaddr: "192.168.20.151"
  - name: "master1"
    ipaddr: "192.168.20.152"
  - name: "master2"
    ipaddr: "192.168.20.153"
workers: 
  - name: "worker0"
    ipaddr: "192.168.20.161"
  - name: "worker1"
    ipaddr: "192.168.20.162"
...
```

3. playbook 실행
```
ansible-playbook -e @vars-static.yaml -e staticips=true tasks/main.yml
```

> 플레이북 실행 후 Helper Node의 리소스가 제대로 생성되었는지 확인하는 helpernodecheck 명령어
/usr/local/bin/helpernodecheck 로 확인

#### Ignition config 파일 생성

1. 작업 디렉토리 생성&이동
```
mkdir ~/ocp4
cd ~/ocp4
```

2. 시크릿 파일 생성
```
mkdir -p ~/.openshift
cat <<EOF > ~/.openshift/pull-secret
# https://cloud.redhat.com/openshift/install/metal에서 secret 복사&붙여넣기
EOF
```
> 이 플레이북은 ~/.ssh/helper_rsa 에 sshkey를 생성한다. 다른 키를 사용하고 싶으면 ~/.ssh/config 를 수정한다.

3. install-config.yaml 파일 생성
```
cat <<EOF > install-config.yaml
apiVersion: v1
baseDomain: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp4
networking:
  clusterNetworks:
  - cidr: 10.254.0.0/16
    hostPrefix: 24
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
pullSecret: '$(< ~/.openshift/pull-secret)'
sshKey: '$(< ~/.ssh/helper_rsa.pub)'
EOF
```

4. installation manifest 생성&수정
```
openshift-install create manifests
```

> master 노드에 파드 스케줄링을 막기 위해 mastersSchedulable 의 값 수정
master에 파드를 배치하려면 파일 수정은 건너뛴다.

```
# 파일 수정
sed -i 's/mastersSchedulable: true/mastersSchedulable: false/g' manifests/cluster-scheduler-02-config.yml

cat manifests/cluster-scheduler-02-config.yml
apiVersion: config.openshift.io/v1
kind: Scheduler
metadata:
  creationTimestamp: null
  name: cluster
spec:
  mastersSchedulable: false    # 이 부분 확인
  policy:
    name: ""
status: {}
```

5. ignition config 생성
```
openshift-install create ignition-configs

#가상머신의 네트워크 부팅에 쓰이는 8080포트의 경로로 ignition 파일을 복사
cp ~/ocp4/.ign /var/www/html/ignition/

#selinux context 복구 & 권한 추가
restorecon -vR /var/www/html/
chmod o+r /var/www/html/ignition/.ign
```

### Bootstrap/Master/Worker Node 구성
#### 물리머신에 각 Node의 가상머신 준비
- 모든 물리&가상머신은 동일한 무선 네트워크 대역(192.168.20.0) 사용
- 물리머신의 무선 네트워크를 가상머신의 bridge로 연결 (NIC : virtio)
- IP주소를 정적으로 설정
- Bootstrap -> Master -> Worker 순으로 가상머신 설정

#### 가상머신 설치
RHCOS ISO Installer을 사용하여 인스턴스를 부팅한다.
1. booting이 시작되면 boot menu에서 tab을 누른다.
사진
2. 각 Node에 맞는 정적ip와 coreOS 설정을 한 줄로 입력한다. (각 필드는 space로 구분)

Bootstrap 입력 예시
```
ip=192.168.20.150::192.168.20.1:255.255.255.0:bootstrap.ocp4.example.com:ens3:none
nameserver=192.168.20.111
coreos.inst.install_dev=sda
coreos.inst.image_url=http://192.168.20.111:8080/install/bios.raw.gz
coreos.inst.ignition_url=http://192.168.20.111:8080/ignition/bootstrap.ign
```
> 영구적용되는 정적 IP 설정
[ip=<ipaddr>::<defaultgw>:<netmask>:<hostname>:<iface>:none] → 인터페이스 ens3
DNS 서버 설정: 여러번 입력 가능
[nameserver=<dnsserver>] → Helper의 IP주소

Bootstrap -> Masters -> Worker 순으로 입력 후 가동

3. helper node 에서 다음 명령어로 설치 시작
```
openshift-install wait-for bootstrap-complete --log-level debug
```
> 웹 브라우저에서 Helper의 HAProxy(9000번 포트)를 통해 Node의 상태를 볼 수 있다 http://192.168.20.111:9000
Master Node가 모두 올라오면 Bootstrap Node를 삭제해도 된다.

## 클러스터 구성 후 작업

# 서비스 소개

# 서비스 구축과정



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
