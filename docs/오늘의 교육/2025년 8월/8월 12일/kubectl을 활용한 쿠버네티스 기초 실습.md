---
layout: default
title: kubectl을 활용한 쿠버네티스 기초 실습
parent: 8월 12일
nav_order: 2
---

# 2025년 8월 12일 교육 내용

# 쿠버네티스

kubectl

run : 크리에이트와 실행을 동시해 한다

* 리눅스 우분투
kubectl run pod-1 --image=nginx:1.14
kubectl get pods
kubectl delete pods pod-1

* 윈도우 명령 프롬프트
kubectl cluster-info
Kubernetes control plane is running at https://10.10.8.103:6443
CoreDNS is running at https://10.10.8.103:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl kubectl을 활용한 쿠버네티스 기초 cluster-info dump'.

```
kubectl api-resources | more
```

kubectl get pods
kubectl get no
kubectl explain pods

D:\kube-menifests\nginx>

kubectl run podname1 --image=ubuntu:latest
kubectl describe pod podname1

kubectl get po -n default

kubectl delete po podname1

kubectl version

kubectl get pods --all-namespaces

----------------------------------------------------------------

## ✅ 1. 기본 연결 및 정보 확인  
가장 먼저 kubectl이 클러스터와 제대로 통신하는지 확인하는 단계입니다.

1) 버전 확인  
kubectl과 클러스터의 버전을 확인합니다. 이 명령어가 성공적으로 실행되면 기본적인 연결은 성공한 것입니다.

DOS
```
sudo -i
kubectl version
```
성공 예시: Client Version과 Server Version 정보가 모두 에러 없이 나타나면 됩니다.

2) 클러스터 정보 확인  
현재 연결된 클러스터의 핵심 구성요소(Control Plane) 주소를 보여줍니다.

DOS
```
kubectl cluster-info
```

3) 노드(Node) 목록 확인  
클러스터에 포함된 작업 서버(노드)들의 목록과 상태를 확인합니다. STATUS가 Ready로 표시되어야 합니다.

DOS
```
kubectl get nodes
```
이 명령어가 성공하면 클러스터를 제어할 준비가 완료된 것입니다.

## ℹ️ 2. 클러스터 내부 상태 조회
실제로 클러스터 내부의 자원들을 조회하는 단계입니다.

1) 전체 네임스페이스의 파드(Pod) 확인  
클러스터에서 실행 중인 모든 파드(애플리케이션 단위) 목록을 확인합니다. 쿠버네티스 시스템을 위한 파드들도 함께 보입니다.

DOS
```
kubectl get pods --all-namespaces
```
또는 단축 명령어인 -A를 사용해도 됩니다: kubectl get pods -A

2) 네임스페이스(Namespace) 목록 확인  
클러스터 내부의 논리적인 공간 분리 단위인 네임스페이스 목록을 봅니다.

DOS
```
kubectl get namespaces
```
default, kube-system 등이 보이면 정상입니다.

## 🚀 3. 간단한 테스트 배포 및 삭제 (실습)
가장 확실한 테스트는 직접 간단한 웹서버를 배포해보는 것입니다. Nginx 웹서버를 배포하고 삭제하는 과정입니다.

1) Nginx 배포 (Deployment 생성)
Nginx 이미지를 사용하는 디플로이먼트를 생성합니다. "Nginx 컨테이너 1개를 실행시켜줘"라는 명령입니다.

DOS
```
kubectl create deployment nginx --image=nginx
```

2) 배포 상태 확인
방금 생성한 nginx 파드가 Running 상태가 될 때까지 확인합니다. (처음에는 ContainerCreating 상태일 수 있습니다.)

DOS
```
kubectl get pods
```

3) 로그 확인
실행 중인 nginx 파드의 로그를 확인하여 정상적으로 작동하는지 봅니다.

DOS
```
kubectl logs deployment/nginx
```
4) 🧹 테스트 자원 삭제 (중요!)
테스트가 끝났으면 배포했던 자원을 깨끗하게 삭제합니다.

DOS
```
kubectl delete deployment nginx
```
이 명령어들이 모두 에러 없이 실행된다면, 쿠버네티스 클러스터를 사용하기 위한 모든 설정이 완벽하게 끝난 것입니다.

----------------------------------------------------------------

## 현재 네임스페이스에 있는 모든 파드(Pod)의 목록을 상세하게 보여줍니다.
```
kubectl get pods -o wide
```

----------------------------------------------------------------

## 📜 쿠버네티스 기본 명령어 실습 (논리적인 순서)
이 가이드는 가장 작은 단위인 Pod에서 시작하여, 애플리케이션 관리 단위인 Deployment, 그리고 가장 권장되는 방식인 YAML 파일을 사용한 관리 순서로 진행됩니다.

## 1부: 가장 작은 단위, 파드(Pod) 다루기
쿠버네티스에서 실행되는 컨테이너의 가장 작은 단위인 파드를 직접 만들고, 확인하고, 접속해 봅니다.

1) Nginx 파드 생성하기  
가장 먼저, nginx:1.14 이미지를 사용하는 nginx라는 이름의 파드 한 개를 생성합니다.

Bash
```
kubectl run nginx --image=nginx:1.14
```

2) 파드 상태 및 IP 확인하기  
파드가 잘 생성되었는지 확인합니다. -o wide 옵션을 추가하면 파드가 어떤 노드에 생성되었는지, 어떤 IP를 할당받았는지 등 더 자세한 정보를 볼 수 있습니다.

Bash
```
kubectl get pods -o wide
```

3) 파드 상세 정보 조회하기  
describe 명령어를 사용하면 파드의 이벤트 로그, 상태, 볼륨 정보 등 매우 상세한 내용을 확인할 수 있습니다. 문제가 생겼을 때 가장 먼저 사용하는 명령어 중 하나입니다.

Bash
```
kubectl describe pod nginx
```
4) 실행 중인 파드에 접속하기  
exec 명령어를 사용하면 실행 중인 컨테이너 내부에 직접 들어가서 명령을 내릴 수 있습니다. 마치 SSH로 접속하는 것과 비슷합니다.

Bash
```
# nginx 컨테이너 안으로 /bin/bash 셸을 실행하며 접속
kubectl exec -it nginx -- /bin/bash
```

5) 파드 IP 주소 확인하기
exec를 이용해 컨테이너 내부에서 명령을 실행하고 결과만 볼 수도 있습니다. 아래 명령어는 nginx 파드의 내부 IP를 출력합니다.

Bash
```
kubectl exec nginx -- hostname -I
```

6) 파드 정리하기
실습이 끝난 파드를 삭제합니다.

Bash
```
kubectl delete pod nginx
```

## 2부: 애플리케이션 관리, 디플로이먼트(Deployment)
파드를 하나씩 관리하는 것은 비효율적입니다. 디플로이먼트는 여러 개의 동일한 파드를 안정적으로 유지하고 관리(자동 복구, 확장 등)하는 역할을 합니다.

1) Nginx 디플로이먼트 생성하기
이번에는 동일한 Nginx 파드 4개를 유지하도록 하는 디플로이먼트를 생성합니다.

Bash
```
kubectl create deployment nginx --image=nginx:1.14 --replicas=4
```

2) 디플로이먼트 및 파드 상태 확인하기
디플로이먼트가 어떻게 4개의 파드를 생성하고 관리하는지 확인합니다.

Bash
```
# 축약형 사용: deployments -> deploy
kubectl get deploy,pods -o wide
```

3) 자동 복구 기능 테스트하기 (쿠버네티스의 핵심!)
디플로이먼트의 가장 강력한 기능 중 하나인 '자동 복구'를 직접 확인해 봅니다.

Bash
```
# 1. 현재 실행 중인 파드 중 하나의 전체 이름을 복사합니다.
kubectl get pods

# 2. 복사한 이름으로 파드 하나를 강제로 삭제합니다.
kubectl delete pods <복사한_파드_이름>

# 3. 다시 파드 목록을 확인하면, 잠시 후 새로운 파드가 자동으로 생성되는 것을 볼 수 있습니다!
kubectl get pods
```

4) 디플로이먼트 정리하기
파드를 개별적으로 지울 필요 없이, 디플로이먼트만 삭제하면 관련된 모든 파드가 함께 삭제됩니다.

Bash
```
kubectl delete deployment nginx
```

## 3부: 가장 권장되는 방식, YAML 파일로 관리하기
명령어로 리소스를 관리하는 것은 편리하지만, 설정 내용이 복잡해지면 관리하기 어렵습니다. YAML 파일을 사용하면 리소스 설정을 코드로 관리(Infrastructure as Code)할 수 있어 매우 효율적입니다.

1) YAML 파일 작성하기
deploy-nginx.yaml이라는 이름으로 아래 내용의 파일을 생성합니다.

YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
```

2) YAML 파일로 디플로이먼트 생성하기
작성한 파일을 kubectl에 전달하여 리소스를 생성합니다.

Bash
```
kubectl create -f deploy-nginx.yaml
```

3) 생성된 리소스 확인하기
디플로이먼트의 전체 YAML 설정을 확인하고 싶을 때 사용합니다.

Bash
```
kubectl get deploy nginx-deployment -o yaml
```

4) YAML 파일로 리소스 삭제하기
생성할 때 사용했던 파일로 삭제까지 깔끔하게 관리할 수 있습니다.

Bash
```
kubectl delete -f deploy-nginx.yaml
```

## 4부: 기타 유용한 조회 명령어
클러스터 전반의 상태를 확인할 때 사용하는 명령어들입니다.

Bash
```
# 클러스터에 참여한 모든 노드(서버) 목록 보기
kubectl get nodes

# 특정 노드의 상세 정보 보기 (CPU/메모리 사용량, 실행 중인 파드 등)
kubectl describe node <노드_이름>
```
* 파드 로그 확인 (CrashLoopBackOff 등 문제 있을 때)

Bash
```
kubectl logs <파드_이름>
```

* 네트워크 테스트 시 올바른 curl 명령어 예시

Bash
```
curl http://<파드ip>
```

----------------------------------------------------------------

## 📜 쿠버네티스 실전 명령어 워크플로우
## 0. 준비 및 환경 설정  

1) 모든 리소스 정리 (실습 시작 전)
깨끗한 환경에서 시작하기 위해 현재 네임스페이스의 모든 리소스를 삭제합니다.

Bash
```
# 현재 namespace의 모든 리소스를 삭제합니다.
kubectl delete all --all
```

2) Alias (단축키) 설정  
kubectl을 k로 줄여서 사용하면 훨씬 편리합니다.

Bash
```
alias k='kubectl'
```

## 1부: 파드(Pod) 생성 및 내부 조작
1) Nginx 파드 생성   
nginx:1.15 버전을 사용하는 nginx 파드를 생성합니다.

Bash
```
k run nginx --image=nginx:1.15
```

2) 파드 내부 접속 및 탐색   
exec 명령어로 실행 중인 컨테이너에 접속하여 리눅스 명령어들을 사용해 봅니다.

Bash
```
# nginx 파드에 bash 셸로 접속
k exec -it nginx -- bash

# --- (이제부터는 컨테이너 내부) ---

# 운영체제 정보 확인
uname -a

# nginx 서비스 상태 확인
service nginx status

# nginx 서비스 재시작
service nginx restart

# nginx 웹 루트 디렉토리로 이동 및 파일 목록 확인
cd /usr/share/nginx/html/
ls

# 새로운 index.html 파일 생성
echo "Welcome our world" > index.html
cat index.html

# 컨테이너에서 빠져나오기
exit
```

3) Pod IP로 직접 접속하여 확인
파드에 할당된 IP를 확인하고, curl로 접속하여 방금 수정한 index.html 내용이 보이는지 확인합니다.

Bash
```
# 1. nginx 파드의 IP 확인
k get pod nginx -o wide

# 2. 확인된 IP로 접속 테스트
curl http://[파드의-IP-주소]
```

## 2부: port-forward로 내 컴퓨터에 파드 연결하기
파드 IP는 클러스터 외부에서 직접 접속할 수 없습니다. port-forward는 내 컴퓨터의 포트와 파드의 포트를 임시로 연결하여 개발 및 테스트를 쉽게 해주는 기능입니다.

1) 포트 포워딩 실행
내 컴퓨터의 81번 포트로 들어오는 요청을 nginx 파드의 80번 포트로 전달합니다.

Bash
```
# foreground로 실행 (이 터미널은 이제 응답 대기 상태가 됨)
k port-forward nginx 81:80
```
이제 웹 브라우저에서 http://localhost:81 또는 http://127.0.0.1:81로 접속하면 Nginx 페이지가 보입니다.

2) 외부 접속 허용 및 백그라운드 실행
--address 0.0.0.0 옵션을 주면 내 컴퓨터의 IP로 다른 PC에서도 접속할 수 있습니다. &를 붙이면 백그라운드에서 실행됩니다.

Bash
```
# 모든 IP에서 접속 가능하도록 백그라운드로 실행
kubectl port-forward --address 0.0.0.0 nginx 81:80 &
```
## 3부: 디플로이먼트(Deployment)로 애플리케이션 관리

1) 디플로이먼트 생성
3개의 동일한 Nginx 파드를 유지하는 dep-nginx 디플로이먼트를 생성합니다.

Bash
```
kubectl create deployment dep-nginx --image=nginx:1.14 --replicas=3
```

2) 실행 중인 디플로이먼트 수정
edit 명령어로 실행 중인 리소스의 설정을 직접 수정할 수 있습니다. 예를 들어 replicas를 3에서 5로 변경하고 저장하면, 파드가 즉시 5개로 늘어납니다.

Bash
```
kubectl edit deploy dep-nginx
```

3) 디플로이먼트 설정을 YAML 파일로 저장
실행 중인 리소스의 설정을 YAML 파일로 저장하여 코드로 관리할 수 있습니다.

Bash
```
kubectl get deploy dep-nginx -o yaml > dep-nginx.yml
```

4) YAML 파일 수정 및 재배포
다운받은 YAML 파일을 직접 수정한 뒤, 기존 디플로이먼트를 삭제하고 파일로부터 다시 생성할 수 있습니다. 이것이 쿠버네티스를 관리하는 가장 표준적인 방법입니다.

Bash
```
# 1. 기존 디플로이먼트 삭제
kubectl delete deployment dep-nginx

# 2. (vi 편집기 등으로 dep-nginx.yml 파일 수정)
vi dep-nginx.yml

# 3. 수정된 YAML 파일로 다시 생성
kubectl create -f dep-nginx.yml
```

----------------------------------------------------------------

# 📜 쿠버네티스 파드(Pod) 생성 및 레이블 관리 워크플로우   
## 0. 준비: 실습 환경 정리 및 단축키 설정   

1) 모든 리소스 정리 (실습 시작 전)   
깨끗한 환경에서 시작하기 위해 현재 네임스페이스의 모든 파드와 디플로이먼트를 삭제합니다.

Bash
```
kubectl delete pods,deployments --all
```

2) Alias (단축키) 설정
kubectl을 k로 줄여서 사용하면 훨씬 편리합니다.

Bash
```
alias k='kubectl'
```

## 1부: YAML 파일로 파드 생성하기 (Best Practice)  
명령어로 파드를 만드는 것도 가능하지만, 실제 운영 환경에서는 설정을 파일로 관리하는 것이 표준입니다. run 명령어와 --dry-run 옵션을 사용하면 YAML 파일의 초안을 쉽게 만들 수 있습니다.

1) 파드 설정(YAML) 파일 생성하기  
--dry-run=client -o yaml 옵션은 파드를 실제로 생성하지 않고, 생성에 필요한 YAML 내용을 화면에 출력해 줍니다. 이 결과를 파일로 저장합니다.

Bash
```
k run web1 --image=nginx:1.14 --port=80 --dry-run=client -o yaml > web1.yml
```

2) YAML 파일 수정 및 적용  
저장된 web1.yml 파일을 열어 원하는 대로 수정한 뒤, apply 명령어로 클러스터에 적용합니다.

Bash
```
# 1. vi 편집기로 파일을 열어 metadata.name과 spec.containers[0].name을 'web2'로 수정합니다.
vi web1.yml

# 2. 수정된 파일로 web2 파드를 생성합니다.
k apply -f web1.yml
```

3) 생성된 파드 확인 및 삭제
파드가 잘 생성되었는지 확인하고, 다음 실습을 위해 삭제합니다.

Bash
```
k get pods
k delete pod web2
```

## 2부: Here Document으로 파드 즉시 생성하기
스크립트를 작성하거나 임시로 파드를 생성할 때, 파일을 만들지 않고 터미널에서 바로 YAML 내용을 전달하여 파드를 생성할 수 있습니다.

Bash
```
cat <<EOF | k apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: myweb
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - image: nginx:latest
    name: myweb
    ports:
    - containerPort: 80
      protocol: TCP
EOF
```
cat <<EOF | ...는 EOF라는 문자를 만나기 전까지의 모든 내용을 kubectl apply -f - 명령어의 입력으로 전달하라는 의미입니다.

## 3부: 레이블(Label)로 파드 관리하기 (쿠버네티스의 핵심)

레이블은 리소스를 식별하고 그룹화하는 데 사용되는 핵심 기능입니다. 수십, 수백 개의 파드를 효율적으로 관리할 수 있게 해줍니다.

1) 모든 파드와 레이블 확인하기
--show-labels 옵션으로 현재 실행 중인 모든 파드와 각 파드에 붙어있는 레이블을 확인합니다.

Bash
```
k get pods --show-labels
```
2) 레이블로 원하는 파드만 선택하여 조회하기
-l 옵션을 사용하여 특정 레이블이 붙은 파드만 골라서 볼 수 있습니다.

Bash
```
# environment=production 레이블이 붙은 파드만 조회
k get pod -l environment=production
```

3) 실행 중인 파드에 레이블 제거하기
label 명령어를 사용하여 실행 중인 파드의 레이블을 동적으로 추가하거나 제거할 수 있습니다.

Bash
```
# myweb 파드에서 'environment' 레이블을 제거합니다. (레이블 이름 뒤에 '-'를 붙입니다)
k label pods myweb environment-

# 레이블이 제거되었는지 다시 확인합니다.
k get pods --show-labels
```

4) 파드 내부 환경변수 확인하기
쿠버네티스는 파드에 대한 유용한 정보들을 컨테이너의 환경변수로 자동으로 주입합니다.

Bash
```
k exec myweb -- env
```

## 4. 최종 정리
실습이 끝났으면 아래 명령어로 생성했던 모든 리소스를 다시 한번 깨끗하게 삭제합니다.

Bash
```
kubectl delete pods,deployments --all
```