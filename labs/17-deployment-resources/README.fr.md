# Lab 17 - Ajouter des requêtes et des limites CPU

> 🇬🇧 [English version](README.md)

## Question

Dans le namespace `ckad-resources`, modifiez le Deployment `web` pour que le conteneur `nginx` demande `100m` de CPU et soit limité à `300m` de CPU. Redéployez l'application.

## Résultat attendu

- La requête est `100m`.
- La limite est `300m`.
- Les deux réplicas deviennent disponibles après le déploiement.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

Appliquez un patch de fusion stratégique :

```bash
kubectl -n ckad-resources patch deployment web --patch-file solution.yaml
kubectl -n ckad-resources rollout status deployment/web
```

Une variante plus rapide :

```bash
kubectl -n ckad-resources set resources deployment web \
  --containers=nginx --requests=cpu=100m --limits=cpu=300m
```

## Vérification

```bash
kubectl -n ckad-resources get deployment web \
  -o jsonpath='{.spec.template.spec.containers[0].resources}{"\n"}'
kubectl -n ckad-resources get deployment web
```

## Nettoyage

```bash
kubectl delete namespace ckad-resources
```

