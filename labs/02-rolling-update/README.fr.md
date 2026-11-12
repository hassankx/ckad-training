# Lab 02 - Configurer, effectuer et annuler une mise à jour

> 🇬🇧 [English version](README.md)

## Question

Pour le Deployment `frontend` du namespace `ckad-rollout`, configurez les mises à jour progressives avec `maxUnavailable: 1` et `maxSurge: 2`. Faites passer l'image `nginx` de `nginx:1.24-alpine` à `nginx:1.25-alpine`, vérifiez le déploiement, puis revenez à la révision précédente.

## Résultat attendu

- Les valeurs RollingUpdate correspondent à l'énoncé.
- La nouvelle image est constatée avant le retour arrière.
- Le retour arrière rétablit `nginx:1.24-alpine`.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

```bash
kubectl -n ckad-rollout patch deployment frontend --patch-file solution.yaml
kubectl -n ckad-rollout set image deployment/frontend nginx=nginx:1.25-alpine
kubectl -n ckad-rollout rollout status deployment/frontend
kubectl -n ckad-rollout rollout history deployment/frontend
kubectl -n ckad-rollout rollout undo deployment/frontend
kubectl -n ckad-rollout rollout status deployment/frontend
```

## Vérification

```bash
kubectl -n ckad-rollout get deployment frontend \
  -o jsonpath='{.spec.strategy.rollingUpdate}{"\n"}{.spec.template.spec.containers[0].image}{"\n"}'
```

Image finale attendue : `nginx:1.24-alpine`.

## Nettoyage

```bash
kubectl delete namespace ckad-rollout
```

