# Helper Node

플레이북 하나로 OCP 4 Cluster 구축을 위한 pxe서버 구축

### 최소 요구사항

- CentOS/RHEL 7 or 8
- 50 GB disk
- 4 CPUs
- 8 GB of RAM

### 실제 환경

- CentOS-8.2.2004-x86_64-minimal.iso
- 500 GB HD
- 8 CPUs
- 20 GB of RAM
- IP - 192.168.7.111
- NetMask - 255.255.255.0
- Gateway - 192.168.20.1
- DNS - 8.8.8.8

원격 접속용 vnc 설치 방법

[vnc 설치](https://www.notion.so/vnc-73a17e80f5e1429c95e065ec3218e6ee)

# Helper Node 설치

> 설치 참고
[https://github.com/RedHatOfficial/ocp4-helpernode/blob/master/docs/bmquickstart.md](https://github.com/RedHatOfficial/ocp4-helpernode/blob/master/docs/bmquickstart-static.md)

## 플레이북 구성

### 1. 필요 패키지 설치

```bash
# EPEL 설치
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %rhel).noarch.rpm

# ansible과 git 설치
yum -y install ansible git

# 플레이북 git에서 가져오기
git clone https://github.com/RedHatOfficial/ocp4-helpernode
cd ocp4-helpernode
```

### 2. vars.yaml 파일을 가져와 환경에 맞게 수정

> bootstrap, master node의 정보는 나중에 기술

```yaml
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

### 3. 플레이북 실행

```bash
ansible-playbook -e @vars-static.yaml -e staticips=true tasks/main.yml
```

플레이북 실행 후 Helper Node의 리소스가 제대로 생성되었는지 확인하는
helpernodecheck 명령어
*/usr/local/bin/helpernodecheck* 로 확인

```bash
# 예시
helpernodecheck dns-masters
helpernodecheck dns-etcd
helpernodecheck install-info
helpernodecheck services
```

## ignition config 파일 생성

### 1. 작업디렉토리 ocp4 생성

```bash
mkdir ~/ocp4
cd ~/ocp4
```

### 2. 시크릿을 저장할 곳 생성

```bash
mkdir -p ~/.openshift
cat <<EOF > ~/.openshift/pull-secret
# [https://cloud.redhat.com/openshift/install/metal](https://cloud.redhat.com/openshift/install/metal)에서 secret 복사 붙여넣기
EOF
```

이 플레이북은 ~/.ssh/helper_rsa 에 sshkey를 생성한다. 다른 키를 사용하고 싶으면 ~/.ssh/config 를 수정한다.

### 3. install-config.yaml 파일 생성

```yaml
cat <<EOF > install-config.yaml
apiVersion: v1
baseDomain: [example.com](http://example.com/)
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

### 4. installation manifests 생성

```bash
openshift-install create manifests
```

master 노드에 파드 스케줄링을  막기 위해 mastersSchedulable 의 값 수정
master에 파드를 배치하려면 이 단계 건너뜀

```bash
sed -i 's/mastersSchedulable: true/mastersSchedulable: false/g' manifests/cluster-scheduler-02-config.yml
```

```yaml
cat manifests/cluster-scheduler-02-config.yml
apiVersion: [config.openshift.io/v1](http://config.openshift.io/v1)
kind: Scheduler
metadata:
	creationTimestamp: null
	name: cluster
spec:
	mastersSchedulable: false
	policy:
		name: ""
status: {}
```

### 5. ignition config 생성

```bash
openshift-install create ignition-configs
# 가상머신의 네트워크 부팅에 쓰이는 8080포트의 경로로 ignition 파일을 복사
cp ~/ocp4/.ign /var/www/html/ignition/
# selinux context 복구 & 권한 추가
restorecon -vR /var/www/html/
chmod o+r /var/www/html/ignition/.ign
```

웹 브라우저에서 헬퍼노드 주소의 9000번 포트를 통해 상태를 볼 수 있다
[http://192.168.20.111:9000](http://192.168.0.14:9000/)

# Bootstrap/Master/Worker Node 설치

## 노드 가상머신 준비

> 반드시 가상머신의 네트워크 대역을 실제 물리적인 네트워크 대역과 동일하게 설정해야 한다. (192.168.20.0/24)

- mac주소와 IP주소 정적으로 설정 & 리소스 할당
- bootstrap → Masters → Worker 순으로 vm을 가동하고 설치 (지금은 Worker가 없는 상태)

### 아키텍쳐 구성도

![Helper](Helper%20Node%203d74b2c2128a46ed8ebd3367ff8476cc/__.png)

### 0. 물리적 머신 준비

### 물리적 머신 환경

- CentOS-7-x86_64-DVD-2003.iso
- 500 GB HD
- 8 CPUs
- 20 GB of RAM
- IP - 각 vm과 동일하게 설정

### 물리적 머신 구성

부팅 시 Server with GUI 선택

```bash
# 그룹 패키지 리스트
yum groups list -v
# 가상화 도구 그룹 설치
yum -y groups install Virtualization\ Host
# kvm virtual manager 설치
yum -y install virt-manager
```

### 가상 브릿지 생성

```bash
# 네트워크 기기 리스트
nmcli deveice show
# 가상 브릿지 생성
nmcli c add con-name br0 type bridge ifname br0
```

호스트에 접근 하려면 브릿지에도 IP 할당 필요

```bash
# 가상 브릿지에 물리 인터페이스 연결
nmcli c add con-name br0-port1 type bridge-slave ifname enp3s0fui master br0
nmcli c u br0
# 브릿지 생성 확인
brctl show
```

하나의 물리적 머신에서 여러 물리적 머신의 virtual-manager 통합 관리

```bash
# 피관리 물리머신에서
vi /etc/libvirt/libvirtd.conf
#listen_tcp = 1    ->주석 제거
```

```bash
# 서비스 데몬 재시작
systemctl restart libvirtd
```

전체를 관리할 물리머신에서 virtual-manager GUI 실행
flie - add connection - 호스트의 IP 넣고 연결 추가(ssh접속이 오류나면 openssh-askpasswd 설치)

### 1. Bootstrap Node 설치

```bash
# Helper Node에서 입력
openshift-install wait-for bootstrap-complete --log-level debug
```

> 이후 부트스트랩 vm PXE 부팅 실행

### Bootstrap Node

- RhCOS
- 120 GB HD
- 4 vCPUs
- 16 GB of RAM
- IP - 192.168.20.150
- NetMask - 255.255.255.0

### 2. Masters Node 설치

### Control Planes (0~2)

- RhCOS
- 120 GB HD
- 4 vCPUs
- 16 GB of RAM
- IP - 192.168.20.151~153
- NetMask - 255.255.255.0

### 3. Worker 설치

### Worker

- RhCOS
- 120 GB HD
- 2 vCPUs
- 8 GB of RAM
- IP - 192.168.20.161, 162
- NetMask - 255.255.255.0
