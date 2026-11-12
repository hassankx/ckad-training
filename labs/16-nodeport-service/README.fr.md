# Lab 16 - Exposer un Deployment avec NodePort

> 🇬🇧 [English version](README.md)

## Question

Exposez le Deployment `web` du namespace `ckad-nodeport` sous la forme du Service `web-nodeport`. Le Service doit écouter sur le port `80`, cibler le port `80` du conteneur et sélectionner les Pods du Deployment.

## Résultat attendu

- Le type du Service est `NodePort`.
- `port`, `targetPort` et `nodePort` sont corrects.
- Le Service possède des endpoints prêts.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

```bash
kubectl apply -f solution.yaml
```

Variante impérative :

```bash
kubectl -n ckad-nodeport expose deployment web \
  --name=web-nodeport --type=NodePort --port=80 --target-port=80
```

## Vérification

```bash
kubectl -n ckad-nodeport get service web-nodeport
```

## Nettoyage

```bash
kubectl delete namespace ckad-nodeport
```

