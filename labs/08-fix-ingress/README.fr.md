# Lab 08 - Dépanner un Ingress inaccessible

> 🇬🇧 [English version](README.md)

## Question

L'Ingress, le Service et le Deployment `catalog` existent dans le namespace `ckad-ingress-fix`, mais `http://catalog.ckad.local/` est inaccessible. Identifiez et corrigez le problème de configuration.

## Résultat attendu

- Inspectez le trajet de la requête, de l'Ingress au Service puis au Pod.
- Conservez les ressources qui fonctionnent et ne modifiez que la configuration incorrecte.
- Le Service `catalog` possède au moins un endpoint après la correction.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

Inspectez chaque couche :

```bash
kubectl -n ckad-ingress-fix describe ingress catalog
kubectl -n ckad-ingress-fix get service catalog -o yaml
kubectl -n ckad-ingress-fix get endpointslice -l kubernetes.io/service-name=catalog
kubectl -n ckad-ingress-fix get pods --show-labels
```

Le sélecteur du Service utilise `app: catalogue`, alors que le label du Pod est `app: catalog`. Corrigez le Service :

```bash
kubectl -n ckad-ingress-fix patch service catalog --patch-file solution.yaml
```

## Vérification

```bash
kubectl -n ckad-ingress-fix get endpointslice \
  -l kubernetes.io/service-name=catalog
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
curl -H 'Host: catalog.ckad.local' http://localhost:8080
```

## Nettoyage

```bash
kubectl delete namespace ckad-ingress-fix
```

