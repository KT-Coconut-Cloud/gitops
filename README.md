# gitops

k3s 배포 매니페스트 저장소
- ArgoCD가 이 저장소의 main 브랜치를 기준으로 클러스터를 동기화

## 구성

| 폴더 | 내용 |
| --- | --- |
| argocd/ | ArgoCD Application 정의 |
| argo/ | Argo Workflows 설치 값 |
| pii-job/ | 파이프라인 WorkflowTemplate, ServiceAccount, NetworkPolicy |
| app/ | 결과 조회 앱 Deployment, Service, NetworkPolicy |

## 작업 규칙

- 이미지 태그는 Jenkins가 자동으로 수정
- 매니페스트 추가, 변경은 PR과 승인 1명 필요
- 시크릿 값은 Vault에서 주입
