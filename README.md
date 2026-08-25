# ressources-Kubernetes-with-Velero-in-local-with-MinIO-
Backing up and restoring Kubernetes resources with Velero (locally with MinIO)


## 1. Contexte Projet

Mettez en place une politique de sauvegarde des ressources Kubernetes (Services, Deployments, ConfigMaps...) avec l'outil "Velero". Ces sauvegardes pourront être restaurées en cas de problème sur le cluster, ou pour une migration vers un autre cluster.



- Cluster : Kind (`tp-velero`)
- Outils : Helm, Velero CLI, MinIO

## 2. Installation de MinIO (Helm)

\`\`\`bash
kubectl create ns minio
helm repo add minio https://charts.min.io/
helm install --set resources.requests.memory=512Mi --set replicas=1 --set persistence.enabled=false --set mode=standalone --set rootUser=rootuser,rootPassword=rootpass123 --generate-name minio/minio --namespace minio
\`\`\`

Résultat : MinIO installé dans le namespace `minio`, avec deux Services générés — `minio-1787660754` (port 9000, API S3) et `minio-1787660754-console` (port 9001, interface web).

## 3. Accès à la console MinIO

\`\`\`bash
kubectl port-forward -n minio svc/minio-1787660754-console 9001:9001
\`\`\`

Console accessible sur http://localhost:9001, connexion réussie avec l'utilisateur `rootuser`.
