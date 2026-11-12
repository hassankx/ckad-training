# Lab 01 - Déplacer des valeurs de base de données codées en dur vers un Secret

> 🇬🇧 [English version](README.md)

## Question

Le Deployment `postgres-client` du namespace `ckad-secret` contient le nom et le mot de passe de la base de données PostgreSQL sous forme de valeurs littérales de variables d'environnement. Créez le Secret `postgres-credentials` avec les clés `database` et `password`, modifiez le Deployment pour qu'il utilise ces clés, puis redéployez-le.

## Résultat attendu

- Aucune valeur de base de données ni aucun mot de passe ne reste directement sous `env[].value`.
- `POSTGRES_DB` référence la clé `database`.
- `POSTGRES_PASSWORD` référence la clé `password`.
- Le Pod de remplacement atteint l'état `Running`.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

```bash
kubectl apply -f solution.yaml
```

`kubectl set env --from=secret/...` utiliserait les noms des clés du Secret comme noms de variables d'environnement. Comme cette tâche exige des noms différents, modifiez le Deployment :

```bash
kubectl -n ckad-secret edit deployment postgres-client
```

Remplacez les deux entrées d'environnement par :

```yaml
env:
  - name: POSTGRES_DB
    valueFrom:
      secretKeyRef:
        name: postgres-credentials
        key: database
  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: postgres-credentials
        key: password
```

Attendez ensuite le déploiement automatique :

```bash
kubectl -n ckad-secret rollout status deployment/postgres-client
```

## Vérification

```bash
kubectl -n ckad-secret get deployment postgres-client -o yaml
kubectl -n ckad-secret exec deploy/postgres-client -- sh -c \
  'test "$POSTGRES_DB" = inventory && test "$POSTGRES_PASSWORD" = ckad-password'
```

## Nettoyage

```bash
kubectl delete namespace ckad-secret
```
