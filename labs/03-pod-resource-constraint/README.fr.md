# Lab 03 - Créer un Pod avec des contraintes CPU

> 🇬🇧 [English version](README.md)

## Question

Un manifeste de Pod est fourni dans `pod.yaml`. Inspectez le namespace `ckad-pod-cpu` et fixez la limite CPU du conteneur du Pod à la moitié de la valeur CPU maximale autorisée. Ne modifiez pas la requête CPU existante. Créez le Pod `worker` dans le namespace `ckad-pod-cpu` à partir du fichier mis à jour.

## Résultat attendu

- Vous inspectez la contrainte du namespace au lieu de supposer sa valeur.
- Vous ajoutez uniquement la limite CPU à `pod.yaml`, conservez la requête CPU existante et créez le Pod à partir de ce fichier.
- La requête CPU est `200m` ; la limite CPU vaut la moitié du maximum du LimitRange (`500m`).

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

Inspectez la contrainte :

```bash
kubectl -n ckad-pod-cpu get limitrange cpu-constraints -o yaml
```

Le maximum du LimitRange est de `1` CPU, la moitié vaut donc `500m`. Ajoutez `limits.cpu: 500m` dans la section `resources` existante du conteneur `worker`, sans toucher à `requests.cpu: 200m`. La section obtenue doit être la suivante (voir `solution.yaml` pour la réponse complète) :

```yaml
      resources:
        requests:
          cpu: 200m
        limits:
          cpu: 500m
```

Créez le Pod à partir du fichier mis à jour :

```bash
kubectl create -f pod.yaml
kubectl -n ckad-pod-cpu wait --for=condition=Ready pod/worker --timeout=90s
```

## Vérification

```bash
kubectl -n ckad-pod-cpu get pod worker \
  -o jsonpath='{.spec.containers[0].resources}{"\n"}'
```

## Nettoyage

```bash
kubectl delete namespace ckad-pod-cpu
```
