# Lab 14 - Attribuer un ServiceAccount à un workload

> 🇬🇧 [English version](README.md)

## Question

Le Deployment `application` du namespace `ckad-workload-sa` utilise actuellement le ServiceAccount `default` du namespace. Modifiez-le pour qu'il utilise le ServiceAccount existant `application-sa`.

## Résultat attendu

- Le modèle de Pod définit `serviceAccountName: application-sa`.
- Un nouveau Pod est créé par le Deployment.
- Le Pod en cours d'exécution indique `application-sa` comme ServiceAccount.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

```bash
kubectl -n ckad-workload-sa patch deployment application \
  --patch-file solution.yaml
kubectl -n ckad-workload-sa rollout status deployment/application
```

Variante par édition :

```bash
kubectl -n ckad-workload-sa edit deployment application
```

Ajoutez `serviceAccountName: application-sa` sous `spec.template.spec`.

## Vérification

```bash
kubectl -n ckad-workload-sa get deployment application \
  -o jsonpath='{.spec.template.spec.serviceAccountName}{"\n"}'
kubectl -n ckad-workload-sa get pods \
  -o custom-columns=NAME:.metadata.name,SERVICEACCOUNT:.spec.serviceAccountName,STATUS:.status.phase
```

## Nettoyage

```bash
kubectl delete namespace ckad-workload-sa
```

