# Lab 10 - Configurer le contexte de sécurité et les capabilities

> 🇬🇧 [English version](README.md)

## Question

Modifiez le Deployment `toolbox` du namespace `ckad-security`. Configurez le conteneur `toolbox` pour qu'il s'exécute avec l'ID utilisateur `1000` et ajoutez les capabilities Linux `NET_ADMIN` et `SYS_TIME`.

## Résultat attendu

- `runAsUser` est configuré au niveau du conteneur.
- Les deux capabilities figurent sous `securityContext.capabilities.add`.
- Le Deployment se déploie correctement, sous réserve de la politique d'admission du cluster.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

```bash
kubectl -n ckad-security patch deployment toolbox --patch-file solution.yaml
kubectl -n ckad-security rollout status deployment/toolbox
```

## Vérification

```bash
kubectl -n ckad-security get deployment toolbox \
  -o jsonpath='{.spec.template.spec.containers[0].securityContext}{"\n"}'
kubectl -n ckad-security exec deploy/toolbox -- id
```

Les clusters restreints peuvent refuser les capabilities ajoutées. Dans ce cas, le manifeste peut tout de même être examiné, mais la politique d'admission du cluster ne doit pas être contournée.

## Nettoyage

```bash
kubectl delete namespace ckad-security
```

