
# 1. AWX 웹 리소스

![](images/awx_component-ko.png)

## 1.2 AWX 웹 접속

- URL: `http://<EXTERNAL-IP>:<PORT>`
- ID: admin
- Password: 아래 명령어로 확인

```bash
kubectl get secret -n awx awx-admin-password -o jsonpath="{.data.password}" | base64 --decode ; echo
```

## 1.3 리소스 설정

### 서비스 계정 권한 및 토큰 생성

- AWX 서비스 계정에 클러스터 관리자 권한 부여:

```bash
kubectl create clusterrolebinding awx-admin --clusterrole=cluster-admin --serviceaccount=awx:awx
```

- AWX 서비스 계정 토큰 생성:

```bash
kubectl create token awx -n awx --duration=2400h
```

### 1.3.1 인증 정보

- 이름: nginx-test
- 조직: Default
- 인증 정보 유형: OpenShift 또는 Kubernetes API 전달자 토큰
- OpenShift 또는 Kubernetes API 끝점: `https://kubernetes.default`
- API 인증 전달자 토큰: `AWX 서비스 계정 토큰 생성` 명령어로 생성된 토큰 입력
- 인증 기관 데이터:
    - Minikube: `~/.minikube/ca.crt`
    - Kubernetes: `/etc/kubernetes/pki/ca.crt`

### 1.3.2 프로젝트 설정

- 이름: nginx-test-project
- 조직: Default
- 소스 제어 유형: git
- 소스 제어 URL: 플레이북이 있는 깃허브 저장소

`nginx-test-playbook.yaml`

```yaml
---
- name: Simple Nginx Pod Test
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Create nginx pod
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Pod
          metadata:
            name: nginx-test-pod
            namespace: default
          spec:
            containers:
            - name: nginx
              image: nginx:latest
              ports:
              - containerPort: 80

    - name: Check pod status
      kubernetes.core.k8s_info:
        kind: Pod
        name: nginx-test-pod
        namespace: default
      register: pod_info
```

### 1.3.3 인벤토리 설정

- 이름: nginx-test-inventory
- 조직: Default

### 1.3.4 작업 템플릿 설정

- 이름: nginx-test-template
- 작업 유형: 실행
- 인벤토리: nginx-test-inventory
- 프로젝트: nginx-test-project
- 실행 환경: AWX EE(24.6.1)
- Playbook: nginx-test-playbook.yaml
- 인증 정보:
	- 선택한 카테고리: OpenShift 또는 Kubernetes API 전달자 토큰
	- nginx-test
## 1.4 실행 및 확인

### 1.4.1 템플릿 실행

`nginx-test-template`

### 1.4.2 결과 확인

```bash
kubectl get po
```

```bash
NAME              READY   STATUS    RESTARTS   AGE
nginx-test-pod    1/1     Running   0          10s
```