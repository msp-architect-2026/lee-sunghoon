## CI/CD 파이프라인 구성 (GitOps Pipeline)

개발자의 코드 병합부터 K8s 클러스터 배포까지의 전 과정은 **GitLab CI**와 **ArgoCD**를 통한 GitOps 방식으로 자동화되어, 무중단 배포(Zero-Downtime Deployment) 실현



### Continuous Integration (GitLab CI)
코드 푸시 시 자동으로 품질을 검증하고 컨테이너 이미지 생성 과정

* **Trigger:** 개발자가 `main` 또는 `release` 브랜치에 코드 Merge
* **Lint & Test:** Python(FastAPI)은 `flake8` 및 `pytest`를, React(Next.js)는 `ESLint` 및 `Jest`를 실행하여 코드 컨벤션과 단위 테스트를 통과하는지 검증
* **Build Image:** Docker를 이용해 프론트엔드와 백엔드 이미지 각각 빌드
* **Push Registry:** 빌드된 이미지를 GitLab Container Registry (또는 하버, ECR)에 업로드
* **Update Manifest:** 인프라를 관리하는 별도의 Git 저장소(Helm Chart 저장소)의 `values.yaml` 파일 내 Image Tag를 새로 빌드된 버전으로 자동 업데이트(Commit & Push)

### Continuous Deployment (ArgoCD)
K8s 클러스터 내부에서 인프라 저장소의 상태를 감시하고 동기화하는 과정

* **Detect Changes:** ArgoCD가 Helm Chart 저장소의 `values.yaml` 변경(Image Tag 업데이트) 감지
* **Sync & Rollout:** K8s 클러스터의 상태를 Git 저장소의 선언적 상태와 일치시킵니다. 기존 파드(Pod)를 하나씩 내리고 새 파드를 올리는 **Rolling Update** 방식을 사용하여 사용자의 서비스 접속 끊김 방지
* **Health Check:** Liveness/Readiness Probe를 통해 새 파드가 정상적으로 트래픽을 받을 준비가 되었는지 확인 후 배포 완료
