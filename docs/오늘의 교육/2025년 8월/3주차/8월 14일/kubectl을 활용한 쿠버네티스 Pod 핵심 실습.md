---
layout: default
title: kubectl을 활용한 쿠버네티스 Pod 핵심 실습
parent: 8월 14일
nav_order: 2
---

# 2025년 8월 14일 교육 내용

# 쿠버네티스 

## 파드 개념 및 사용하기

## 🚀 쿠버네티스 Pod 핵심 개념 실습 가이드
이 가이드는 쿠버네티스에서 가장 기본적이면서도 중요한 단위인 Pod에 대한 핵심 개념들을 직접 실습하며 익힐 수 있도록 구성되었습니다.

#### 실습 1: Pod 생성과 관리의 기본
🎯 목표: kubectl 명령과 YAML 파일을 이용해 Pod를 생성하고, 상태를 확인하며, 삭제하는 기본 흐름을 익힙니다.

1) 명령어로 Pod 바로 생성하기  
가장 간단하게 Nginx Pod를 하나 실행해 봅시다.

Bash
```
kubectl run my-nginx-pod --image=nginx:1.14 --port=80
```

2) Pod 상태 확인하기  
Pod가 잘 생성되었는지 확인합니다. -o wide 옵션은 IP와 실행된 노드 정보까지 보여줍니다.

Bash
```
kubectl get pods -o wide
```

3) YAML 파일로 Pod 생성하기  
실제 운영 환경에서는 대부분 YAML 파일을 사용합니다. 아래 내용으로 pod-web.yaml 파일을 생성하세요.

vi pod-web.yaml
YAML
```
# pod-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-web-server
  labels:
    app: web
spec:
  containers:
  - name: web-container
    image: nginx:1.14
    ports:
    - containerPort: 80
```

4) YAML 파일 적용하여 Pod 생성  
작성한 YAML 파일을 클러스터에 적용합니다.

Bash
```
kubectl apply -f pod-web.yaml
```

5) Pod 상세 정보 확인 (가장 중요!)  
describe 명령어는 Pod의 이벤트 로그를 포함한 모든 상세 정보를 보여주므로, 문제가 생겼을 때 가장 먼저 사용해야 할 명령어입니다.

Bash
```
kubectl describe pod my-web-server
```

11) Pod 로그 확인  
컨테이너 내부에서 출력되는 로그를 확인합니다.

Bash
```
kubectl logs my-web-server
```

7) Pod 삭제 (실습 마무리)   
실습했던 Pod들을 이름으로 지정하여 삭제합니다.

Bash
```
kubectl delete pod my-nginx-pod my-web-server
```

--------------------------------------------------------------------------

#### 실습 2: Liveness Probe를 이용한 자가 치유(Self-Healing) Pod
🎯 목표: Pod 내부의 애플리케이션에 문제가 생겼을 때, 쿠버네티스가 이를 자동으로 감지하고 컨테이너를 재시작하여 서비스를 복구하는 과정을 확인합니다.

1) Liveness Probe가 포함된 YAML 작성  
liveness-pod.yaml 파일을 생성합니다. 3초마다 Nginx의 기본 페이지(/)가 정상 응답(HTTP 200)하는지 검사하고, 3번 실패하면 컨테이너를 재시작하도록 설정합니다.

vi liveness-pod.yaml
YAML
```
# liveness-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-test-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5  # Pod 시작 후 5초 뒤에 검사 시작
      periodSeconds: 3      # 3초 간격으로 검사
      failureThreshold: 3   # 3번 연속 실패하면 재시작
```

2) Pod 생성 및 모니터링  
터미널 2개를 준비합니다.

터미널 1 (모니터링용): Pod의 상태를 실시간으로 모니터링합니다. RESTARTS 횟수를 주목하세요.

Bash
```
kubectl get pod liveness-test-pod --watch
```

터미널 2 (실행용): Pod를 생성합니다.

Bash
```
kubectl apply -f liveness-pod.yaml
```

3) 고의로 문제 발생시키기  
터미널 2에서 exec 명령으로 Pod에 접속하여, Liveness Probe가 검사하는 index.html 파일을 삭제하여 문제를 일으킵니다.

Bash
```
# Pod 내부로 들어가서 index.html 파일 이름 변경
kubectl exec -it liveness-test-pod -- /bin/bash -c "mv /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.bak"
```

4) 자동 복구 확인  
터미널 1을 보면, 잠시 후 Pod의 RESTARTS 횟수가 0에서 1로 증가하며 컨테이너가 재시작되는 것을 볼 수 있습니다. 재시작되면서 정상적인 index.html을 가진 새 컨테이너가 생성되어 서비스가 자동으로 복구됩니다.

5) 실습 마무리
생성했던 Pod를 삭제합니다.

Bash
```
kubectl delete pod liveness-test-pod
```

--------------------------------------------------------------------------

#### 실습 3: Init Container를 이용한 사전 작업 
🎯 목표: 메인 컨테이너가 시작되기 전에, 필요한 설정이나 파일을 준비하는 Init Container의 동작을 이해합니다.

1) Init Container가 포함된 YAML 작성  
init-pod.yaml 파일을 생성합니다. 메인 컨테이너(main-app)가 실행되기 전, init-container가 먼저 실행되어 /work-dir 디렉터리에 hello.txt 파일을 생성합니다.

vi init-pod.yaml
YAML
```
# init-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo-pod
spec:
  containers:
  - name: main-app
    image: busybox
    command: ['sh', '-c', 'echo "Main App Starting..."; ls -l /work-dir; sleep 3600']
    volumeMounts:
    - name: workdir
      mountPath: /work-dir
  initContainers:
  - name: init-container
    image: busybox
    command: ['sh', '-c', 'echo "Init Container: Preparing work directory..."; touch /work-dir/hello.txt']
    volumeMounts:
    - name: workdir
      mountPath: /work-dir
  volumes:
  - name: workdir
    emptyDir: {}
```

2) Pod 생성 및 상태 확인  
watch로 Pod의 생성 과정을 지켜보면 STATUS가 Pending -> Init:0/1 -> PodInitializing -> Running 순서로 변하는 것을 볼 수 있습니다.

Bash
```
kubectl apply -f init-pod.yaml
kubectl get pod init-demo-pod --watch
```

3) 결과 확인  
메인 컨테이너의 로그를 확인하면, Init Container가 미리 생성해 둔 hello.txt 파일이 존재하는 것을 확인할 수 있습니다.

Bash
```
kubectl logs init-demo-pod
```

    출력 예상:  
    ```
    Main App Starting...
    total 0
    -rw-r--r-- 1 root root 0 Aug 14 05:30 hello.txt
    ```

4) 실습 마무리
생성했던 Pod를 삭제합니다.

```
kubectl delete pod init-demo-pod
```

--------------------------------------------------------------------------

#### 실습 4: Static Pod 만들기  
🎯 목표: API 서버를 통하지 않고, 특정 노드의 kubelet이 직접 관리하는 Static Pod를 생성해 봅니다.

1) Worker Node 접속 및 경로 확인  
클러스터의 Worker Node 중 하나에 SSH로 접속합니다.

Bash
```
# 예시: ssh root@node1.labs.local
# 위 노드 이름은 실제 실습 환경에 맞게 변경해야 합니다.
ssh root@<YOUR-WORKER-NODE-IP-OR-HOSTNAME>
```

kubelet 설정 파일에서 Static Pod를 감지할 경로를 확인합니다. (보통 /etc/kubernetes/manifests)

Bash
```
# 접속한 노드에서 실행
cat /var/lib/kubelet/config.yaml | grep staticPodPath
```

2) Static Pod YAML 파일 생성  
확인한 경로로 이동하여, 아래 내용으로 static-nginx.yaml 파일을 생성합니다.

Bash
```
# 접속한 노드에서 실행
# 경로는 환경에 따라 다를 수 있습니다.
cd /etc/kubernetes/manifests
vim static-nginx.yaml

YAML

# static-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-static-nginx
  labels:
    role: static
spec:
  containers:
  - name: nginx
    image: nginx:1.14
```

3) Master Node에서 확인  
다시 Master Node 터미널로 돌아와 get pods를 실행하면, kubelet이 YAML 파일을 감지하고 자동으로 생성한 Pod를 볼 수 있습니다. Pod 이름 뒤에 노드 이름이 접미사로 붙는 것이 특징입니다.

Bash
```
# Master Node에서 실행
kubectl get pods -o wide
```

    출력 예상:  
    ```
    NAME                               READY   STATUS    ...   NODE
    my-static-nginx-node1.labs.local   1/1     Running   ...   node1.labs.local
    ```

4) 자동 복구 확인  
Master Node에서 이 Pod를 삭제해 보세요.

Bash
```
# Master Node에서 실행
kubectl delete pod my-static-nginx-node1.labs.local
```
  
    ✨ 핵심 포인트: API 서버를 통해 Pod를 삭제해도, Worker Node의 kubelet은 지정된 경로(/etc/kubernetes/manifests)에 YAML 파일이 여전히 존재하는 것을 보고 Pod를 즉시 다시 생성합니다. 이것이 Static Pod의 자가 치유 방식입니다.
  
5) 완전히 삭제하기  
Static Pod를 완전히 삭제하려면, Worker Node로 다시 돌아가 생성했던 YAML 파일을 삭제해야 합니다.

Bash
```
# 접속한 노드에서 실행
rm /etc/kubernetes/manifests/static-nginx.yaml
```

파일이 삭제되면 Master Node에서 Pod가 자동으로 사라지는 것을 확인할 수 있습니다.

--------------------------------------------------------------------------

## 종합 실습: 실전 시나리오 Pod 구성하기
🎯 목표: 앞에서 배운 개념(네임스페이스, 리소스, 환경변수)을 종합하여 요구 조건에 맞는 Pod를 배포합니다.

요구 조건:

Pod 이름: myweb, 이미지: nginx:1.14
리소스: CPU 200m, 메모리 500Mi를 **요청(request)**하고, CPU 1개, 메모리 1Gi로 **제한(limit)**합니다.
환경변수: DB=mydb를 포함해야 합니다.
네임스페이스: product 네임스페이스에서 동작해야 합니다.

실행 단계:

1) 네임스페이스 생성  
product 네임스페이스가 없다면 먼저 생성합니다.

Bash
```
kubectl create namespace product
```

2) 요구사항을 반영한 YAML 파일 작성  
product-web-pod.yaml 파일을 생성합니다.

vi product-web-pod.yaml
YAML
```
# product-web-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myweb
  namespace: product # 4. 네임스페이스 지정
spec:
  containers:
  - name: myweb-container
    image: nginx:1.14 # 1. 이미지 지정
    resources: # 2. 리소스 할당
      requests:
        cpu: 200m
        memory: 500Mi
      limits:
        cpu: "1"
        memory: 1Gi
    env: # 3. 환경변수 설정
    - name: "DB"
      value: "mydb"
```

3) Pod 생성  
작성한 YAML을 product 네임스페이스에 적용합니다.

Bash
```
kubectl apply -f product-web-pod.yaml
```

4) 최종 확인  
product 네임스페이스에 Pod가 잘 생성되었는지 확인하고, describe 명령으로 리소스와 환경변수가 올바르게 설정되었는지 검증합니다.

Bash
```
# Pod 상태 확인
kubectl get pods -n product

# Pod 상세 정보로 설정 검증
kubectl describe pod myweb -n product
```

5) 실습 마무리
YAML 파일로 생성한 Pod 삭제
kubectl apply와 마찬가지로, kubectl delete -f 명령어를 사용해 YAML 파일에 정의된 리소스를 한 번에 삭제할 수 있습니다.

Bash
```
kubectl delete -f product-web-pod.yaml
```
이 명령어를 실행하면 product 네임스페이스에 있는 myweb 파드가 삭제됩니다.

네임스페이스 삭제
파드를 삭제한 후에는 실습을 위해 만들었던 product 네임스페이스도 삭제하는 것이 좋습니다. 네임스페이스를 삭제하면 그 안에 있던 모든 리소스가 함께 사라집니다.

Bash
```
kubectl delete namespace product
```
위 두 명령어를 실행하면 실습 환경이 완전히 초기화됩니다.

