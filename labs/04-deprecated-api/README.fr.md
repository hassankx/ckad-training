# Lab 04 - Remplacer une API Deployment obsolète

> 🇬🇧 [English version](README.md)

## Question

Le manifeste `deprecated-deployment.yaml` utilise une version d'API qui n'est plus servie par les versions actuelles de Kubernetes. Mettez-le à jour vers une version d'API prise en charge et déployez-le dans le namespace `ckad-api`.

## Résultat attendu

- La version d'API est `apps/v1`.
- Le sélecteur obligatoire est présent.
- Les labels du sélecteur correspondent à ceux du modèle de Pod.
- Deux Pods deviennent prêts dans `ckad-api`.

## Préparation

```bash
kubectl apply -f setup.yaml
kubectl apply -f deprecated-deployment.yaml
```

La seconde commande doit échouer sur un cluster récent. Cet échec est la situation de départ.

## Étapes de réponse

Remplacez :

```yaml
apiVersion: extensions/v1beta1
```

par :

```yaml
apiVersion: apps/v1
```

Ajoutez `spec.selector.matchLabels` correspondant au label du modèle de Pod. Le fichier corrigé est `solution.yaml` :

```bash
kubectl apply -f solution.yaml
kubectl -n ckad-api rollout status deployment/legacy-web
```

## Vérification

```bash
kubectl -n ckad-api get deployment legacy-web
kubectl -n ckad-api get pods -l app=legacy-web
```

## Nettoyage

```bash
kubectl delete namespace ckad-api
```

