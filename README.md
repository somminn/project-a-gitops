# GitOps Configuration Repository

이 저장소는 UniBooker 서비스의 Kubernetes 배포 설정을 관리하는 GitOps 전용 레포입니다.

<br>

## Related Repositories

역할에 따라 총 4개의 레포지토리로 분리했습니다.

| Repository | Description | Tech Stack |
| --- | --- | --- |
| **[Frontend](https://github.com/somminn/project-a-frontend)** | Frontend CI packaging repo | Jenkins / Gihub Action  |
| **[Backend](https://github.com/somminn/project-a-backend)** | Backend CI packaging repo | Jenkins / Gihub Action |
| **[Automation](https://github.com/somminn/infra-automation)** | 자동화 스크립트 | Ansible |
| **[GitOps](https://github.com/somminn/project-a-gitops)** | **(현재)** K8s 매니페스트 및 배포 설정 관리 | Kustomize |

<br>

## Directory Structure (GitOps)

이 저장소의 주요 디렉토리 구조는 다음과 같습니다.

```text

├── app
│   ├── backend
│   │   ├── kustomization.yml
│   │   ├── rollout-bluegreen.yml
│   │   ├── service-active.yml
│   │   └── service-preview.yml
│   │
│   └── frontend
│       ├── deployment.yml
│       ├── kustomization.yml
│       └── service.yml
│
└── infra
    ├── argocd
    │   ├── apps
    │   │   ├── backend.yml
    │   │   ├── frontend.yml
    │   │   └── infra.yml
    │   │  
    │   └── application.yml
    │
    └── gateway
    │    ├── gateway.yml
    │    └── httproute.yml
    │
    ├── kustomization.yml
    └── namespace.yml
```

<br>

## Deployment Process

### 1. 이미지 태그 업데이트

CI 파이프라인이 성공하면 `yq` 도구를 사용하여 `app/*/kustomization.yml` 파일의 `newTag`를 자동으로 업데이트합니다.

### 2. 수동 배포 (필요 시)

특정 버전을 수동으로 배포하려면 아래 파일을 수정합니다.

* **Path:** `app/*/kustomization.yml`
* **Field:** `images[name: cowmin/project-a-frontend].newTag`

<br>

## CI/CD Flow

<img width="1200" alt="final" src="https://github.com/user-attachments/assets/1c12c2a8-fd48-4cf7-aa57-2f724c1f4e46" />

### Description
1. 애플리케이션 레포에 코드 Push
2. CI 파이프라인이 Docker 이미지를 빌드하고 Registry에 Push
3. GitOps 레포의 kustomization.yml 이미지 태그를 자동 업데이트
4. ArgoCD가 변경 사항을 감지하고 클러스터에 동기화

> 클러스터에 직접 접근하지 않고 Pull 기반으로 배포합니다.

<br>
 
## Deployment Strategy
### Backend – Blue/Green (Argo Rollouts)
- Active / Preview 서비스 분리
- 트래픽 전환 기반 무중단 배포
- 롤백 시 서비스 스위칭으로 즉시 복구 가능

### Why Blue/Green?
- 안정성 우선 전략
- Canary 대비 운영 단순성
- 롤백 속도 확보

### Frontend – Rolling Update
- 정적 리소스 기반 서비스
- 상태 비저장 구조
- 기본 Kubernetes Deployment 전략 사용

