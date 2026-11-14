# devops-with-kubernetes-config
赫爾辛基大學 DevOps with Kubernetes 課程練習的部署設定倉庫，與程式碼倉庫分開（https://github.com/LaomaiAI/devops-with-kubernetes）。
## 內容
- argocd/：ArgoCD Application 清單（log-output、project-staging、project-production）。
- log-output/manifests/：log-output 應用的 Kubernetes 清單（由 ArgoCD 套用）。
- greeter/manifests/：greeter 服務的 Kubernetes 清單（兩個版本、75/25 分流 HTTPRoute），以 kubectl 套用（未納入 ArgoCD）。
- project/：project 應用的 Kustomize 設定——base（四個元件清單目錄的聚合）、overlays/staging、overlays/production。
## 部署流程
- 安裝 ArgoCD 後，套用本倉庫的 Application 清單：kubectl apply -f argocd/
- project-staging Application（targetRevision HEAD）跟隨 main 提交部署到 staging；project-production Application（targetRevision 4.10）跟隨標籤部署到 production；log-output Application（targetRevision HEAD）部署到 exercises 命名空間。
- staging：broadcaster 只記錄不轉發（FORWARD_ENABLED=false，NATS 主題 todos-staging）；staging 不備份資料庫。
- production：主程式 3 個副本；每日 02:00 以 CronJob todo-db-backup 備份 todo-db 到 PVC todo-db-backup。
## 密鑰與映像
- todo-db-secret.yaml 不在 ArgoCD 管理的資源內，須手動套用：kubectl apply -f project/todo-backend/manifests/todo-db-secret.yaml；staging 須將該檔的 namespace 改為 project-staging 再套用。
- 映像標籤為 latest 時，程式碼改動後須重建映像、k3d image import 並重啟 Deployment，ArgoCD 不會因標籤不變而重新部署。
## 來源
程式碼見 https://github.com/LaomaiAI/devops-with-kubernetes。Exercise 4.10 起程式碼與設定分倉；本倉庫為設定倉庫。
