# Lab 05 - Modifier le nom et l'image d'un conteneur

> 🇬🇧 [English version](README.md)

## Question

Modifiez le Deployment `api` du namespace `ckad-container`. Renommez le conteneur `old-container` en `web-container` et définissez son image à `nginx:1.25-alpine`. Assurez-vous que le Pod obtenu atteint l'état `Running`.

## Résultat attendu

- Le modèle de Pod contient un seul conteneur nommé `web-container`.
- Son image est `nginx:1.25-alpine`.
- Le déploiement se termine et le Pod est prêt.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

Comme la fusion stratégique identifie les conteneurs par leur nom, un patch avec un nouveau nom peut ajouter un second conteneur. Utilisez un patch JSON explicite pour remplacer les deux champs :

```bash
kubectl -n ckad-container patch deployment api --type=json -p='[
  {"op":"replace","path":"/spec/template/spec/containers/0/name","value":"web-container"},
  {"op":"replace","path":"/spec/template/spec/containers/0/image","value":"nginx:1.25-alpine"}
]'
kubectl -n ckad-container rollout status deployment/api
```

Sous Windows PowerShell, placez le JSON dans un fichier ou utilisez `kubectl edit deployment api -n ckad-container` pour éviter les problèmes de guillemets. `solution.yaml` montre le fragment de modèle de Pod attendu.

## Vérification

```bash
kubectl -n ckad-container get deployment api \
  -o jsonpath='{.spec.template.spec.containers[0].name}{" "}{.spec.template.spec.containers[0].image}{"\n"}'
kubectl -n ckad-container get pods
```

## Nettoyage

```bash
kubectl delete namespace ckad-container
```

