# 1. AWX Operator 설치 가이드

## 1.1 AWX Operator

`awx-operator` 리소스를 배포하기 위한 `kustomization.yaml` 파일 작성

`kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - github.com/ansible/awx-operator/config/default?ref=2.19.1
  #- awx.yaml
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.19.1

namespace: awx
```

> AWX Operator 버전 릴리즈: https://github.com/ansible/awx-operator/releases

### 1.1.1 배포 및 확인

```bash
kubectl apply -k .
```

```bash
kubectl get po -n awx

NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-666ddcf9c5-6zcd2   2/2     Running   0          4m9s
```

`awx-operator-controller-manager` 실행이 확인 된 후 다음 단계 진행

## 1.2 AWX 인스턴스

`awx` 리소스를 배포하기 위한 `awx.yaml` 파일 작성

`awx.yaml`

```yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  # AWX Web UI
  admin_user: admin
  # admin_email: admin@example.com
  # admin_password_secret: awx-admin-password

  # Ingress
  # ingress_type: Ingress #Default: none
  # ingress_class_name: nginx
  # ingress_hosts:
  #  - hostname: awx.example.com
  # ingress_path: /

  # Service
  service_type: LoadBalancer

  # Replicas
  replicas: 1
  #web_replicas: 2
  #task_replicas: 2
  
  # Database Storage
  #postgres_storage_class: standard
```

### 1.2.1 kustomization.yaml 수정

`awx.yaml` 파일을 `kustomization.yaml`에 추가

`kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - github.com/ansible/awx-operator/config/default?ref=2.19.1
  - awx.yaml

images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.19.1

namespace: awx
```

### 1.2.2 배포

`awx` 리소스 배포

```bash
kubectl apply -k .
```

## 1.3 AWX 구성 요소

- awx_operator: AWX 설치 및 관리 자동화를 위한 Operator
- awx_web: 웹 인터페이스
- awx_task: 작업 실행
- awx_ee: 실행 환경(Execution Environment)
- awx_postgres: 데이터베이스
- (옵션) awx_redis: 캐싱 및 메시지 브로커 역할
- (옵션) awx_rsyslog: 로그 수집 및 관리
