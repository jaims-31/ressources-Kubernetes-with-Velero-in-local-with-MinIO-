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




## 4. Bucket MinIO pour Velero

Création via la console MinIO (Buckets → Create Bucket) : bucket nommé `velero`.

## 5. Access key MinIO pour Velero

Création via la console (Access Keys → Create access key). Clés enregistrées localement dans un fichier `credentials-velero` (non commité, exclu par `.gitignore`) :

\`\`\`
[default]
aws_access_key_id = ********
aws_secret_access_key = ********
\`\`\`

## 6. Installation de Velero

\`\`\`bash
velero install \\
  --provider aws \\
  --plugins velero/velero-plugin-for-aws:v1.2.1 \\
  --bucket velero \\
  --secret-file ./credentials-velero \\
  --use-volume-snapshots=false \\
  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://minio-1787660754.minio.svc:9000
\`\`\`

Résultat : `Velero is installed! ⛵`. Connexion à MinIO validée avec `velero backup-location get` → `PHASE: Available`.





## 7. Déploiement de l'application de test

Fichier `demo-app.yaml` : namespace `demo-app` + Deployment nginx (1 replica) + Service ClusterIP.

\`\`\`bash
kubectl apply -f demo-app.yaml
\`\`\`

Résultat : Pod, Deployment et Service nginx créés et opérationnels (`1/1 Running`) dans le namespace `demo-app`.

## 8. Backup manuel avec la CLI Velero

\`\`\`bash
velero backup create nginx-backup --include-namespaces demo-app
velero backup describe nginx-backup
\`\`\`

Résultat : `Phase: Completed`, 16/16 items sauvegardés. Le bucket `velero` contient un nouveau dossier `backups/nginx-backup/` avec 9 fichiers (14,2 KiB au total, dont `nginx-backup.tar.gz`).

