---
layout: default
title: kubectl을 활용한 쿠버네티스 핵심 자원 관리 실습
parent: 8월 13일
nav_order: 2
---

# 2025년 8월 13일 교육 내용

# 쿠버네티스 

쿠버네티스 자원 정리: 네임스페이스와 서비스
말씀하신 내용을 바탕으로 쿠버네티스의 핵심 개념들을 정리해 드리겠습니다. 이는 쿠버네티스 클러스터에서 애플리케이션을 배포하고 접근하는 기본 절차와 관련이 있습니다.

## 1) 네임스페이스 (Namespace)
네임스페이스는 쿠버네티스 클러스터의 자원들을 논리적으로 분리하여 관리하는 가상의 공간입니다.

역할: 여러 팀이나 환경(개발, 운영)을 분리하여 자원 간의 충돌을 막고, 효율적인 권한 관리를 가능하게 합니다.

확인 방법: kubectl get ns 명령어로 현재 클러스터에 존재하는 네임스페이스 목록을 확인할 수 있습니다.

## 2) 서비스 (Service)
서비스는 파드(Pod)에 안정적인 네트워크 연결을 제공하여 외부에서 애플리케이션에 접근할 수 있도록 해주는 핵심 자원입니다. 서비스의 종류에 따라 접근 방식이 달라집니다.

ClusterIP (클러스터 내부 접속)
역할: 클러스터 내부에만 유효한 고정 IP를 할당합니다.

특징: 클러스터 외부에서는 접근할 수 없으며, 주로 클러스터 내부의 다른 파드나 서비스 간 통신에 사용됩니다.

NodePort (클러스터 외부 접속)
역할: 각 **노드(Node)**의 특정 포트를 열어 외부에 서비스를 노출합니다.

특징: 클러스터 외부에서 노드_IP:노출된_포트 주소를 통해 서비스에 접근할 수 있게 됩니다.

```
        > 노드          |
클러스터                | > 파드 > 컨테이너
        > 네임스페이스  |
```

노드와 네임스페이스의 역할
노드(Node): 파드가 **어디(어떤 서버)**에 있는지 확인하는 용도입니다. 이는 파드의 물리적인 실행 위치를 알려줍니다. kubectl get pods -o wide 명령어를 사용하면 NODE 컬럼에서 파드가 실행 중인 노드를 확인할 수 있습니다.

네임스페이스(Namespace): 파드가 어떤 그룹에 속해 있는지 확인하는 용도입니다. 이는 파드의 논리적인 소속을 나타냅니다. kubectl get pods --namespace <네임스페이스_이름> 명령어를 통해 특정 그룹에 있는 파드를 조회할 수 있습니다.

----------------------------------------------------------------------------------------

접근 제어 방식 (Access Control Methods)
RBAC (Role-Based Access Control)
사용자에게 특정 권한을 부여하는 대신, **'역할(Role)'**에 권한을 정의하고 그 역할을 사용자에게 할당하는 방식입니다. 예를 들어, '개발자' 역할에는 파드 생성 권한을 주고, '운영자' 역할에는 전체 클러스터 관리 권한을 주는 식입니다. 쿠버네티스에서 표준으로 사용되는 접근 제어 방식입니다.

ACL (Access Control List)
특정 자원(파일, 폴더 등)에 접근할 수 있는 사용자나 그룹의 목록을 직접 정의하는 방식입니다. RBAC가 '역할'을 중심으로 한다면, ACL은 **'자원'**을 중심으로 누가 접근할 수 있는지 하나씩 지정합니다.

인증 및 신원 확인 (Authentication and Identity Verification)
인증서 (Certificate)
사용자, 서버, 또는 장치의 신원을 보증하는 디지털 파일입니다. 통신 암호화(HTTPS)나 신뢰할 수 있는 접속을 위해 사용됩니다.

DN (Distinguished Name)
디지털 인증서에서 사용되는 고유 식별 이름입니다. 인증서의 주체(Subject)를 명확하게 구분하기 위해 사용됩니다. DN은 여러 구성 요소로 이루어져 있습니다.

DC (Domain Component): 도메인 이름의 각 구성 요소 (예: com, example, co.kr)

OU (Organizational Unit): 조직 내의 부서나 그룹 (예: IT, HR, Engineering)

CN (Common Name): 인증서가 속한 대상의 이름 (예: www.example.com 또는 John Doe)

----------------------------------------------------------------------------------------

# 볼룸

## 1. 문제점: 컨테이너는 일회용입니다
가장 먼저 알아야 할 것은 **컨테이너는 기본적으로 일회용(Ephemeral)**이라는 사실입니다.

컨테이너는 마치 포스트잇과 같습니다. 무언가를 적어두었다가 떼어서 버리면 그 안의 내용은 영원히 사라집니다. 마찬가지로, 컨테이너 안에 파일을 저장해 두어도 컨테이너가 어떤 이유로든 재시작되거나 삭제되면 그 안의 데이터는 모두 사라집니다. 데이터베이스나 중요한 사용자 파일이 이렇게 사라진다면 큰일이겠죠.

## 2. 해결책: 볼륨(Volume) - 컨테이너의 외장하드
쿠버네티스 볼륨은 이 문제를 해결하기 위한 **'컨테이너의 외장하드'**와 같습니다.

볼륨은 파드(Pod)에 장착할 수 있는 저장 공간입니다. 컨테이너가 포스트잇이라면, 볼륨은 따로 들고 다니는 USB 메모리나 외장하드입니다. 포스트잇을 찢어 버려도 외장하드에 있는 데이터는 그대로 남아있는 것처럼, 컨테이너가 재시작되어도 볼륨에 저장된 데이터는 안전하게 보존됩니다.

핵심 특징:
볼륨의 생명주기는 컨테이너가 아닌 파드(Pod)에 연결됩니다.
파드가 살아있는 동안에는, 그 안의 컨테이너가 재시작되어도 볼륨 데이터는 유지됩니다.
하나의 파드 안에 여러 컨테이너가 있을 경우, 이 컨테이너들은 같은 볼륨을 공유하여 데이터를 주고받을 수 있습니다.

## 3. 볼륨의 종류와 용도
쿠버네티스는 다양한 종류의 볼륨을 지원하며, 용도에 따라 선택해서 사용합니다.

emptyDir:
개념: 파드가 생성될 때 함께 만들어지는 임시 폴더입니다. 파드가 사라지면 이 폴더도 함께 삭제됩니다.

비유: 회의실(파드) 안에 있는 화이트보드. 회의에 참여한 여러 사람(컨테이너)들이 함께 쓰고 지울 수 있지만, 회의가 끝나면(파드가 삭제되면) 내용도 모두 지워집니다.

용도: 하나의 파드 내 컨테이너 간의 임시 파일 공유, 캐시 데이터 저장 등

hostPath:

개념: 컨테이너가 실행 중인 노드(서버)의 실제 폴더를 파드에 연결합니다.

비유: 아파트(노드)의 특정 방(폴더) 열쇠를 컨테이너에게 주는 것과 같습니다.

용도: 노드의 로그 파일을 읽거나, 도커 소켓에 접근하는 등 특수한 경우에 사용됩니다.

⚠️ 주의: 파드가 특정 노드에 묶이게 되므로 꼭 필요한 경우에만 제한적으로 사용해야 합니다.

awsElasticBlockStore, gcePersistentDisk, nfs (네트워크 볼륨):

개념: AWS의 EBS, Google Cloud의 Persistent Disk, 사내 파일 서버(NFS) 같은 외부 네트워크 스토리지를 파드에 직접 연결합니다.

비유: 파드를 외부의 전문적인 물류 창고에 연결하는 것과 같습니다. 파드나 노드가 사라져도 데이터는 창고에 안전하게 보관됩니다.

용도: 데이터베이스 파일, 사용자 업로드 파일 등 영구적으로 보존해야 할 중요한 데이터 저장.

## 4. 가장 좋은 방법: PV와 PVC
위의 네트워크 볼륨을 직접 사용하는 것은, 개발자가 인프라(AWS 볼륨 ID 등) 정보를 알아야 하므로 좋은 방법이 아닙니다. 그래서 쿠버네티스는 **PV(PersistentVolume)**와 **PVC(PersistentVolumeClaim)**라는 더 발전된 방법을 사용합니다.

레스토랑의 외투 보관소에 비유할 수 있습니다.

관리자 (인프라 담당자):  
외투 보관소 직원처럼, 미리 다양한 종류의 저장 공간(PV)을 준비해 둡니다. "이건 5GB짜리 빠른 SSD", "저건 100GB짜리 느린 HDD"처럼 말이죠.

개발자 (애플리케이션 담당자):  
레스토랑 손님처럼, 보관소에 가서 "제 외투를 맡길 공간 하나 주세요"라고 **요청(PVC)합니다. "10GB 정도의 빠른 공간이 필요해요" 라고만 말하면 됩니다.

쿠버네티스:  
손님의 요청(PVC)에 맞는 저장 공간(PV)을 찾아서 자동으로 연결해주고 '보관증'을 줍니다. 개발자는 이 '보관증'만 파드 설정에 넣으면 됩니다.

PV (PersistentVolume): 관리자가 미리 준비해 둔 실제 저장 공간 (추상화)

PVC (PersistentVolumeClaim): 개발자가 "이만큼의 공간이 필요해요" 라고 하는 저장 공간 요청서


----------------------------------------------------------------------------------------

## 📜 kubectl 네임스페이스와 컨텍스트: 종합 실습 가이드
이 가이드는 네임스페이스로 리소스를 격리하고, YAML로 선언적으로 관리하며, 컨텍스트로 작업 효율을 높이는 전체 과정을 상세한 설명과 결과 예시를 포함하여 진행합니다.

## 0. 사전 준비 (Prerequisites & Setup)
0.1) 실습 환경 정리
깨끗한 환경에서 시작하기 위해 현재 선택된 네임스페이스의 모든 리소스를 삭제합니다.

⚠️ 주의: 이 명령어는 네임스페이스 자체를 삭제하지는 않습니다.

Bash

kubectl delete all --all
0.2) Alias (단축키) 설정 및 확인
kubectl을 k로 줄여서 사용하면 실습이 훨씬 편리해집니다.

Bash

# alias 설정
alias k='kubectl'

# alias가 잘 설정되었는지 확인
type k
💡 실행 결과 예시
k is aliased to 'kubectl'

## 1. 네임스페이스(Namespace) 생성 및 리소스 격리
1.1) 네임스페이스 3개 생성 및 확인
ns1, ns2, ns3 세 개의 독립된 작업 공간(폴더)을 만듭니다.

Bash

k create namespace ns1
k create namespace ns2
k create namespace ns3

# 현재 클러스터의 모든 네임스페이스 목록 확인
k get namespace
💡 실행 결과 예시

NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
ns1               Active   1s
ns2               Active   1s
ns3               Active   1s

1.2) 각 네임스페이스에 리소스 반복 배포
동일한 이름(ui)의 디플로이먼트를 각 네임스페이스에 생성하여 서로 완벽하게 격리되는 것을 확인합니다.

Bash

k create deployment ui --image=nginx -n ns1
k create deployment ui --image=nginx -n ns2
k create deployment ui --image=nginx -n ns3

1.3) 네임스페이스별 배포 상태 확인
각 "폴더" 안에 파드들이 잘 생성되었는지 -n 옵션을 사용하여 따로따로 확인합니다.

Bash

k get deployment,pods -n ns1
💡 실행 결과 예시 (ns1)

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ui   1/1     1            1           10s
NAME                        READY   STATUS    RESTARTS   AGE
pod/ui-6856f59f7d-abcde   1/1     Running   0          10s

## 2. YAML 파일을 활용한 선언적 관리
💡 Tip: create vs apply

create: "이 리소드를 생성해". 이미 존재하면 오류가 발생합니다.

apply: "이 파일과 상태를 똑같이 맞춰". 없으면 생성하고, 변경된 부분이 있으면 업데이트합니다. apply를 사용하는 것이 더 권장됩니다.

2.1) YAML 파일 초안 빠르게 만들기 (--dry-run)
리소스를 실제로 만들지 않고 YAML 내용만 파일로 저장하는 매우 유용한 방법입니다.

Bash

k run web1 --image=nginx:1.14 --dry-run=client -o yaml > web1.yml
⚠️ 주의: --dry-run=client는 클라이언트 측에서만 실행하라는 뜻입니다. 만약 --dry-run=server를 사용하면, API 서버가 실제로 리소스를 생성할 수 있는지 검증까지 하지만, 이 옵션은 곧 지원 중단(deprecated)될 예정입니다.

2.2) YAML 파일 수정 및 적용
YAML 파일 안에 namespace와 여러 개의 ports를 직접 지정하여 더 명확하게 리소스를 관리할 수 있습니다.

Bash
```
# 아래 내용으로 nginx-pod.yml 파일 생성
vi nginx-pod.yml
YAML

# nginx-pod.yml 파일 내용
apiVersion: v1
kind: Pod
metadata:
  name: web-nginx
  namespace: ns1      # <-- ns1 네임스페이스에 생성하도록 지정
spec:
  containers:
    - name: nginx
      image: nginx:1.14
      ports:
        - containerPort: 80   # <-- HTTP 포트
        - containerPort: 443  # <-- HTTPS 포트
```

Bash
```
# 위 파일 적용 후 ns1 네임스페이스 확인
k apply -f nginx-pod.yml
k get pods -n ns1
```

## 3. 컨텍스트(Context)로 작업 효율 극대화
3.1) 현재 설정 확인

Bash
```
k config view
k config current-context
```
3.2) 새로운 컨텍스트 생성 및 전환
ns1을 기본 작업 공간으로 사용하는 ns1-context를 만들고 전환합니다.

Bash
```
k config set-context ns1-context --cluster=kubernetes --user=kubernetes-admin --namespace=ns1
k config use-context ns1-context
```
3.3) 새 컨텍스트에서 작업하기
이제 -n 옵션 없이도 모든 k 명령어는 ns1을 대상으로 실행됩니다.

Bash
```
# 현재 컨텍스트가 'ns1-context'로 변경된 것을 확인
k config current-context

# --namespace 옵션 없이도 ns1의 파드가 보임
k get pods
```

3.4) 컨텍스트 이름 변경

Bash
```
k config rename-context ns1-context ns1-dev
## 4. 정리 및 특이 케이스 확인
```

4.1) 네임스페이스 삭제
네임스페이스를 삭제하면 그 안의 모든 리소스가 함께 삭제됩니다.

⚠️ 주의: 네임스페이스 삭제 명령 후, 내부 리소스가 정리될 때까지 Terminating 상태로 몇 분간 남아있을 수 있습니다.

Bash
```
k delete namespace ns1 ns2 ns3
```

4.2) "고아"가 된 컨텍스트 오류 확인
ns1 네임스페이스는 삭제했지만, 그곳을 가리키던 ns1-dev 컨텍스트는 여전히 남아있습니다. 이 컨텍스트를 사용하면 오류가 발생합니다.

Bash
```
# 1. ns1-dev 컨텍스트로 전환
k config use-context ns1-dev

# 2. 파드 조회를 시도하면, 네임스페이스가 존재하지 않는다는 오류가 발생
k get pods
```
  ❌ 실제 오류 예시:  
  Error from server (NotFound): namespaces "ns1" not found

4.3) 불필요한 컨텍스트 삭제 및 최종 확인
더 이상 사용하지 않는 컨텍스트는 삭제하여 설정을 깔끔하게 유지합니다.

Bash
```
# 1. 먼저 안전하게 원래 컨텍스트로 돌아갑니다.
k config use-context kubernetes-admin@kubernetes

# 2. 이제 ns1-dev 컨텍스트를 삭제합니다.
k config delete-context ns1-dev

# 3. 컨텍스트가 깨끗하게 정리되었는지 최종 확인합니다.
k config view
```

| 단계 | 주요 작업 | 핵심 명령어 |
| :--- | :--- | :--- |
| **0. 준비** | 환경 정리 및 단축키 설정 | `kubectl delete all --all`, `alias` |
| **1. 격리** | 네임스페이스 생성 및 리소스 분산 배포 | `kubectl create namespace`, `kubectl create deployment -n` |
| **2. 선언적 관리** | YAML 파일 생성, 수정 및 `apply`를 통한 관리 | `... --dry-run -o yaml`, `kubectl apply -f` |
| **3. 효율화** | 컨텍스트 생성, 전환, 변경으로 작업 간소화 | `kubectl config set/use/rename-context` |
| **4. 정리** | 사용한 리소스 및 설정(네임스페이스, 컨텍스트) 삭제 | `kubectl delete namespace`, `kubectl config delete-context` |


## 전체 실습 환경을 초기화하는 가이드
실습 환경을 처음 상태로 되돌리려면, 네임스페이스와 불필요한 컨텍스트까지 모두 삭제해야 합니다.

아래 명령어를 순서대로 실행하세요.

Bash
```
# 1. 모든 리소스가 삭제될 수 있도록 기본 컨텍스트로 전환 (안전 절차)
k config use-context kubernetes-admin@kubernetes

# 2. 모든 실습 네임스페이스 삭제
# 네임스페이스를 삭제하면 그 안에 있는 모든 리소스가 함께 삭제됩니다.
k delete namespace ns1 ns2 ns3

# 3. default 네임스페이스에 남아있는 리소스 삭제 (선택 사항)
k delete all --all

# 4. 실습 중 생성한 컨텍스트 삭제
k config delete-context ns1-context
k config delete-context ns1-dev

# 5. 모든 네임스페이스가 삭제되었는지 확인
k get namespace
```

----------------------------------------------------------------------------------------

## 🚀 쿠버네티스 볼륨(Volume) 완전 정복 실습 가이드
이 가이드는 쿠버네티스에서 데이터를 저장하고 공유하는 다양한 볼륨 타입의 특징과 사용법을 직접 실습하며 익힐 수 있도록 구성되었습니다.

#### 실습 1: emptyDir 볼륨 - Pod 안의 공유 메모장  
🎯 목표: Pod 내의 여러 컨테이너가 데이터를 공유하는 가장 간단한 방법인 emptyDir 볼륨의 동작을 이해합니다.

💡 핵심 개념: emptyDir 볼륨은 Pod가 생성될 때 함께 만들어지는 임시 저장 공간입니다. Pod가 살아있는 동안에는 유지되지만, Pod가 사라지면 데이터도 함께 삭제됩니다. 한 Pod 안의 컨테이너들이 파일을 주고받는 용도로 사용됩니다. 마치 한 회의실 안에서만 사용하는 공유 화이트보드와 같습니다.

단계별 실습:

1) Sidecar 패턴을 이용한 YAML 파일 작성  
하나의 컨테이너(app)는 1초마다 로그 파일에 현재 시간을 기록하고, 다른 컨테이너(sidecar)는 그 로그 파일을 계속해서 읽어 화면에 출력하는 Pod를 만듭니다. emptyDir 볼륨(varlog)이 이 두 컨테이너의 공유 통로 역할을 합니다.

vl-emptydir.yaml
YAML
```
# vl-emptydir.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  volumes:
  - name: varlog
    emptyDir: {} # 두 컨테이너가 공유할 빈 디렉토리 볼륨
  containers:
  - name: app # 1초마다 로그를 쓰는 컨테이너
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      while true; do
        echo "$(date) - Writing log" >> /var/log/example.log;
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: sidecar # 로그를 읽어서 출력하는 컨테이너
    image: busybox
    args:
    - /bin/sh
    - -c
    - "tail -f /var/log/example.log"
    volumeMounts:
    - name: varlog
      mountPath: /var/log
```

2) Pod 생성  
작성한 YAML 파일을 클러스터에 적용합니다.

Bash
```
kubectl apply -f vl-emptydir.yaml
```

3) 동작 확인
sidecar 컨테이너의 로그를 실시간으로 확인하면, app 컨테이너가 기록하는 로그가 그대로 출력되는 것을 볼 수 있습니다.

Bash
```
# -f 옵션으로 로그를 실시간으로 모니터링합니다.
kubectl logs -f sidecar-pod -c sidecar
```

  출력 예상:
  ```
  Thu Aug 14 17:10:01 KST 2025 - Writing log
  Thu Aug 14 17:10:02 KST 2025 - Writing log
  Thu Aug 14 17:10:03 KST 2025 - Writing log
  ```

4) 실습 마무리
Pod를 삭제하면 emptyDir 볼륨과 그 안의 로그 파일도 함께 사라집니다.

Bash
```
kubectl delete pod sidecar-pod
```

#### 실습 2: hostPath 볼륨 - Node와 파일 공유
🎯 목표: Pod가 실행되는 호스트 노드(Host Node)의 특정 파일 시스템 경로를 Pod 내부에 연결하는 hostPath 볼륨을 사용해 봅니다.

💡 핵심 개념: hostPath는 Pod가 노드의 파일 시스템에 직접 접근해야 할 때 사용합니다. Pod가 삭제되어도 노드에 파일이 그대로 남아 데이터가 유지됩니다. 하지만 Pod가 특정 노드에 종속되므로, 신중하게 사용해야 합니다.

  ⚠️ 주의: hostPath 사용의 위험성
  hostPath는 노드의 민감한 파일을 노출하거나 시스템을 손상시킬 수 있어 운영 환경(Production)에서는 사용을 적극적으로 피해야 합니다. 노드 레벨의 로그 수집 등 시스템에 대한 깊은 이해가 필요한 제한된 용도로만 사용됩니다.

단계별 실습:

1) hostPath 볼륨을 사용하는 Pod YAML 작성  
Nginx 컨테이너의 /data 경로를 호스트 노드의 /tmp/pod-data 경로와 연결하는 Pod를 생성합니다.

vl-hostpath.yaml
YAML
```
# vl-hostpath.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: web-server
    image: nginx:1.14
    volumeMounts:
    - name: host-storage
      mountPath: /data
  volumes:
  - name: host-storage
    hostPath:
      path: /tmp/pod-data # 호스트 노드의 이 경로를 사용
      type: DirectoryOrCreate # 경로가 없으면 자동으로 생성
```

2) Pod 생성 및 실행 노드 확인  
Pod를 생성하고, 어느 노드에서 실행되는지 확인합니다.

Bash
```
kubectl apply -f vl-hostpath.yaml
kubectl get pod hostpath-pod -o wide
```

3) Pod 내부에서 파일 생성
exec 명령으로 Pod에 접속하여, 공유된 /data 디렉터리에 파일을 생성합니다.

Bash
```
kubectl exec -it hostpath-pod -- /bin/bash
```

Bash
```
# --- Pod 내부 쉘에서 실행 ---
root@hostpath-pod:/# echo "Hello from the Pod!" > /data/pod-message.txt
root@hostpath-pod:/# exit
```

4) 호스트 노드에서 파일 확인
2단계에서 확인한 노드에 SSH로 접속하여, Pod에서 생성한 파일이 실제로 노드에 저장되었는지 확인합니다.

Bash
```
# 예: ssh root@node1.labs.local
ssh root@<POD가-실행된-노드-IP>
```
Bash
```
# --- 노드 내부 쉘에서 실행 ---
$ cat /tmp/pod-data/pod-message.txt
```
  출력 예상: Hello from the Pod!

5) 실습 마무리  
Pod를 삭제해도 노드에 생성된 파일은 그대로 남아있는 것을 확인할 수 있습니다.

Bash
```
kubectl delete pod hostpath-pod
```

#### 실습 3: PersistentVolume(PV)과 PVC - 영속적 데이터 관리의 표준
🎯 목표: 관리자가 제공하는 스토리지(PV)를 개발자가 요청(PVC)하여 사용하는, 쿠버네티스의 표준 영속 데이터 관리 방법을 익힙니다. 이번 실습은 NFS와 Local Volume 두 가지 시나리오로 진행됩니다.

💡 핵심 개념: PV와 PVC는 스토리지 관리와 사용을 분리하는 개념입니다.

  PersistentVolume (PV): 클러스터 관리자가 준비해 둔 스토리지 자원입니다. (NFS, 클라우드 디스크 등)

  PersistentVolumeClaim (PVC): **개발자(사용자)**가 "이 정도 크기의 스토리지가 필요해요" 라고 요청하는 문서입니다.

  쿠버네티스는 요청된 PVC에 가장 적합한 PV를 찾아 연결(Bind)해 줍니다.

3-A. NFS를 이용한 공유 볼륨 실습
사전 준비: NFS 서버 구성
먼저 클러스터의 모든 노드가 접근할 수 있는 NFS 서버를 준비합니다. (예: 192.168.108.3 IP를 가진 별도의 서버)

  💡 Note: 아래 권한 설정(777, no_root_squash)은 실습 편의를 위한 것입니다. 운영 환경에서는 보안을 고려하여 더 엄격한 권한을 적용해야 합니다.

Bash
```
# --- NFS 서버에서 실행 ---
sudo apt update
sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/shared-data
sudo chown nobody:nogroup /srv/nfs/shared-data
sudo chmod 777 /srv/nfs/shared-data
echo "/srv/nfs/shared-data *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee /etc/exports
sudo exportfs -a
sudo systemctl restart nfs-kernel-server

# --- 모든 쿠버네티스 노드에서 실행 ---
sudo apt update
sudo apt install -y nfs-common
```

단계별 실습:

1) PV, PVC, Pod YAML 파일 작성  
아래 세 리소스를 하나의 파일 nfs-all-in-one.yaml 로 만듭니다.

vi nfs-all-in-one.yaml
YAML
```
# nfs-all-in-one.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-nfs-pv
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany # 여러 Pod에서 동시 읽기/쓰기 가능
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: manual
  nfs:
    server: 192.168.108.3 # NFS 서버 IP
    path: "/srv/nfs/shared-data" # NFS 공유 경로
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
  storageClassName: manual
---
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: nfs-storage
  volumes:
  - name: nfs-storage
    persistentVolumeClaim:
      claimName: my-nfs-pvc
```

2) 리소스 생성 및 상태 확인
작성한 리소스들을 한 번에 생성하고, PV와 PVC가 Bound 상태로 잘 연결되었는지 확인합니다.

Bash
```
kubectl apply -f nfs-all-in-one.yaml
```

Bash
```
# PV와 PVC 상태 확인
kubectl get pv,pvc
```
  출력 예상:
  ```
  NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
  persistentvolume/my-nfs-pv   2Gi        RWX            Recycle          Bound    default/my-nfs-pvc   manual                  20s
  NAME                               STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  persistentvolumeclaim/my-nfs-pvc   Bound    my-nfs-pv    2Gi        RWX            manual         20s
  ```

3) 데이터 공유 확인
Pod에 접속하여 index.html 파일을 생성하고, NFS 서버에서 파일이 보이는지 확인합니다.

Bash
```
kubectl exec -it nfs-test-pod -- /bin/bash -c 'echo "Hello from NFS!" > /usr/share/nginx/html/index.html'
```

Bash
```
# NFS 서버에 접속해서 확인
ssh root@192.168.108.3
cat /srv/nfs/shared-data/index.html
```
  출력 예상: Hello from NFS!

4) 실습 마무리  
YAML 파일 하나로 생성한 모든 리소스를 삭제합니다.

Bash
```
kubectl delete -f nfs-all-in-one.yaml
```

3-B. Local Persistent Volume 실습
시나리오: 특정 노드(node1.labs.local)에만 장착된 고속 SSD를 데이터베이스 Pod가 사용하도록 배포합니다.

  💡 핵심: 이 실습은 StorageClass의 volumeBindingMode: WaitForFirstConsumer와 PV의 nodeAffinity를 조합하여, Pod가 스케줄링될 때 비로소 스토리지의 위치(노드)를 고려하게 만드는 지능적인 스케줄링 방법을 배웁니다.

단계별 실습:

1. [node1] Local Volume 디렉토리 생성
master 노드에서 node1.labs.local로 SSH 접속하여 디렉토리를 생성합니다.

Bash
```
ssh node1.labs.local
# --- node1에서 실행 ---
sudo mkdir -p /mnt/fast-ssd/postgresql
```

2) [master] StorageClass 생성  
다시 master 노드로 돌아와, Pod 생성을 기다리는 StorageClass를 정의합니다.

YAML
```
# local-storage.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-fast-ssd
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
Bash
```
kubectl apply -f local-storage.yaml
```

3) [master] Local PersistentVolume(PV) 생성  
node1의 디렉토리를 가리키고, node1에만 존재함을 알리는 PV를 정의합니다.

vi local-pv.yaml
YAML
```
# local-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fast-ssd-pv-for-node1
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-fast-ssd
  local:
    path: /mnt/fast-ssd/postgresql
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1.labs.local
```
Bash
```
kubectl apply -f local-pv.yaml
```

4) [master] PersistentVolumeClaim(PVC) 생성  
local-fast-ssd 스토리지를 요청하는 PVC를 정의하고, 상태가 Pending인지 확인합니다.

vi db-pvc.yaml
YAML
```
# db-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-fast-ssd
  resources:
    requests:
      storage: 3Gi
```
Bash
```
kubectl apply -f db-pvc.yaml
kubectl get pvc postgres-pvc
```

  출력 예상: STATUS가 Pending으로 표시됩니다.

5) [master] Pod 배포 및 최종 결과 확인  
생성한 PVC를 사용하는 PostgreSQL Pod를 배포하여 스케줄러가 어떻게 동작하는지 확인합니다.

vi postgres-pod.yaml
YAML
```
# postgres-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-perf-db
spec:
  containers:
  - name: postgres
    image: postgres:13
    env:
    - name: POSTGRES_PASSWORD
      value: "password"
    ports:
    - containerPort: 5432
    volumeMounts:
    - mountPath: /var/lib/postgresql/data
      name: db-data
  volumes:
  - name: db-data
    persistentVolumeClaim:
      claimName: postgres-pvc
```
Bash
```
kubectl apply -f postgres-pod.yaml
```

잠시 후, Pod의 상태와 어느 노드에 배포되었는지 확인합니다.

Bash
```
kubectl get pod high-perf-db -o wide
```
  결과: high-perf-db Pod가 PV가 존재하는 node1.labs.local에 정확히 배포된 것을 확인할 수 있습니다.

이제 PV와 PVC의 상태를 다시 확인하면 Bound로 변경된 것을 볼 수 있습니다.

Bash
```
kubectl get pv,pvc
```

6) 실습 마무리
모든 리소스를 역순으로 삭제합니다.

Bash
```
kubectl delete pod high-perf-db
kubectl delete pvc postgres-pvc
kubectl delete pv fast-ssd-pv-for-node1
kubectl delete sc local-fast-ssd
```