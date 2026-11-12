# Lab 15 - Configurer une sonde de vivacité (liveness probe)

> 🇬🇧 [English version](README.md)

## Question

Le Deployment `web` du namespace `ckad-probe` s'exécute sans liveness probe. Configurez une liveness probe HTTP GET pour le conteneur `web` sur le chemin `/healthz`, port `8080`, avec un délai initial de `10` secondes et une période de `10` secondes.

## Résultat attendu

- La sonde utilise HTTP GET `/healthz` sur le port `8080`.
- Les champs de temporisation correspondent à l'énoncé.
- La nouvelle liveness probe réussit et le Pod de remplacement reste en cours d'exécution sans redémarrages répétés.

## Préparation

```bash
kubectl apply -f setup.yaml
kubectl -n ckad-probe rollout status deployment/web
```

L'application utilise `registry.k8s.io/e2e-test-images/agnhost:2.53` avec
`netexec --http-port=8080`. Son endpoint intégré `/healthz` renvoie HTTP 200
tant qu'elle est saine ; aucune image personnalisée ni ConfigMap n'est nécessaire. Il s'agit d'une image de test
destinée aux environnements d'apprentissage isolés, pas d'une application de production. Utilisez `netexec`,
et non la commande distincte `liveness`, qui devient volontairement défaillante.

Inspectez le Deployment en cours d'exécution et vérifiez qu'aucune liveness probe n'est configurée :

```bash
kubectl -n ckad-probe get pods
kubectl -n ckad-probe describe pod -l app=web
kubectl -n ckad-probe get deployment web \
  -o jsonpath='{.spec.template.spec.containers[0].livenessProbe}{"\n"}'
```

La sortie JSONPath est vide tant que vous n'avez pas configuré la sonde.

## Étapes de réponse

```bash
kubectl -n ckad-probe patch deployment web --patch-file solution.yaml
kubectl -n ckad-probe rollout status deployment/web
```

## Vérification

```bash
kubectl -n ckad-probe get deployment web \
  -o jsonpath='{.spec.template.spec.containers[0].livenessProbe}{"\n"}'
kubectl -n ckad-probe get pods
```

## Nettoyage

```bash
kubectl delete namespace ckad-probe
```
