# Labs de formation CKAD 2026

> 🇬🇧 [English version](README.md)

Dix-sept labs pratiques couvrant les tâches CKAD courantes. Chaque lab est autonome et conçu pour s'entraîner en ligne de commande en temps limité.
🎥 Playlist vidéo complète avec explications (en anglais) : https://youtube.com/playlist?list=PLGPD0L7wWIvc&si=HuubJkB_8oq6_Sx2
## Prérequis

- Un cluster Kubernetes (kind, minikube, Docker Desktop ou un cluster d'entraînement distant)
- `kubectl` configuré pour ce cluster
- Docker pour le lab 12
- Un contrôleur Ingress pour les labs 07 et 08

## Méthode de travail recommandée

1. Ouvrez le répertoire d'un lab et lisez uniquement ses sections **Question** et **Résultat attendu**.
2. Exécutez la commande de préparation depuis ce répertoire.
3. Résolvez la tâche sans lire les **Étapes de réponse**.
4. Exécutez les commandes de vérification.
5. Comparez votre travail avec `solution.yaml` ou la réponse documentée.
6. Exécutez la commande de nettoyage avant de passer au lab suivant.

Utilisez un namespace neuf pour chaque lab. Les manifestes de préparation créent le namespace requis et les ressources de départ.

## Index des labs

| # | Sujet | Répertoire |
|---|---|---|
| 01 | Variables d'environnement issues d'un Secret | [`labs/01-secret-env`](labs/01-secret-env/README.fr.md) |
| 02 | Mise à jour progressive et retour arrière | [`labs/02-rolling-update`](labs/02-rolling-update/README.fr.md) |
| 03 | Requête CPU d'un Pod et limite du namespace | [`labs/03-pod-resource-constraint`](labs/03-pod-resource-constraint/README.fr.md) |
| 04 | Remplacer une version d'API obsolète | [`labs/04-deprecated-api`](labs/04-deprecated-api/README.fr.md) |
| 05 | Modifier le nom et l'image d'un conteneur | [`labs/05-container-update`](labs/05-container-update/README.fr.md) |
| 06 | CronJob toutes les 30 minutes | [`labs/06-cronjob`](labs/06-cronjob/README.fr.md) |
| 07 | Créer un Ingress | [`labs/07-create-ingress`](labs/07-create-ingress/README.fr.md) |
| 08 | Dépanner un Ingress inaccessible | [`labs/08-fix-ingress`](labs/08-fix-ingress/README.fr.md) |
| 09 | Créer un déploiement canary à 20 % | [`labs/09-canary-deployment`](labs/09-canary-deployment/README.fr.md) |
| 10 | Contexte de sécurité et capabilities | [`labs/10-security-context`](labs/10-security-context/README.fr.md) |
| 11 | NetworkPolicy | [`labs/11-network-policy`](labs/11-network-policy/README.fr.md) |
| 12 | Construire et exporter une image Docker | [`labs/12-docker-image-export`](labs/12-docker-image-export/README.fr.md) |
| 13 | Diagnostiquer et corriger un problème de ServiceAccount/RBAC | [`labs/13-serviceaccount-rbac`](labs/13-serviceaccount-rbac/README.fr.md) |
| 14 | Attribuer un ServiceAccount à un workload | [`labs/14-workload-serviceaccount`](labs/14-workload-serviceaccount/README.fr.md) |
| 15 | Configurer une liveness probe | [`labs/15-liveness-probe`](labs/15-liveness-probe/README.fr.md) |
| 16 | Exposer un Deployment avec NodePort | [`labs/16-nodeport-service`](labs/16-nodeport-service/README.fr.md) |
| 17 | Requêtes et limites CPU d'un Deployment | [`labs/17-deployment-resources`](labs/17-deployment-resources/README.fr.md) |

## Bonnes habitudes de formation

- Définissez le namespace dès le début : `kubectl config set-context --current --namespace=<namespace>`.
- Préférez les commandes impératives lorsqu'elles sont plus rapides, puis utilisez `--dry-run=client -o yaml` lorsqu'un manifeste est nécessaire.
- Utilisez `kubectl explain`, `kubectl get -o yaml`, `kubectl describe`, les logs et les événements au lieu de deviner.
- Vérifiez chaque modification auprès du serveur d'API, pas seulement dans un fichier local.
- Utilisez `kubectl rollout status`, l'historique des rollouts et les commandes de retour arrière pour les Deployments.
- Ne modifiez pas les ressources dont la question précise explicitement qu'elles doivent rester inchangées.
