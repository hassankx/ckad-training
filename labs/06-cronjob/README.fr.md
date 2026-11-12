# Lab 06 - Créer un CronJob

> 🇬🇧 [English version](README.md)

## Question

Dans le namespace `ckad-cron`, créez le CronJob `half-hour-report` qui s'exécute toutes les 30 minutes. Il doit utiliser le nom de conteneur `sp`, l'image `busybox:1.28` et la politique de redémarrage `Never`. Exécutez `/bin/sh -c 'date; echo Hello from the Kubernetes cluster'`.

Fixez le champ `activeDeadlineSeconds` de chaque Job à `3`. Conservez jusqu'à `3` Jobs réussis et `1` Job échoué à l'aide de `successfulJobsHistoryLimit` et `failedJobsHistoryLimit`.

Créez un Job nommé `manual-test` à partir du CronJob `half-hour-report` pour l'exécuter immédiatement.

## Résultat attendu

- La planification est `*/30 * * * *`.
- Le CronJob utilise `batch/v1`.
- Le modèle de Job utilise le conteneur `sp`, l'image `busybox:1.28`, la commande `["/bin/sh", "-c", "date; echo Hello from the Kubernetes cluster"]` et la politique de redémarrage `Never`.
- Le délai du Job est défini dans `spec.jobTemplate.spec.activeDeadlineSeconds: 3`.
- Les limites d'historique sont définies dans `spec.successfulJobsHistoryLimit: 3` et `spec.failedJobsHistoryLimit: 1`.
- Le Job `manual-test` est créé à partir du modèle de Job du CronJob.

## Préparation

```bash
kubectl create namespace ckad-cron
```

## Étapes de réponse

```bash
kubectl apply -f solution.yaml
```

Point de départ impératif :

```bash
kubectl -n ckad-cron create cronjob half-hour-report \
  --image=busybox:1.28 --schedule='*/30 * * * *' \
  -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
kubectl -n ckad-cron patch cronjob half-hour-report --type=merge \
  -p '{"spec":{"successfulJobsHistoryLimit":3,"failedJobsHistoryLimit":1,"jobTemplate":{"spec":{"activeDeadlineSeconds":3,"template":{"spec":{"restartPolicy":"Never","containers":[{"name":"sp","image":"busybox:1.28","command":["/bin/sh","-c","date; echo Hello from the Kubernetes cluster"]}]}}}}}}'
```

Après avoir créé le CronJob par l'une ou l'autre méthode, créez le Job immédiat :

```bash
kubectl -n ckad-cron create job --from=cronjob/half-hour-report manual-test
```

## Vérification

```bash
kubectl -n ckad-cron get cronjob half-hour-report
kubectl -n ckad-cron get cronjob half-hour-report \
  -o jsonpath='{.spec.schedule}{"\n"}'
kubectl -n ckad-cron get cronjob half-hour-report \
  -o jsonpath='{.spec.jobTemplate.spec.activeDeadlineSeconds}{"\n"}{.spec.successfulJobsHistoryLimit}{"\n"}{.spec.failedJobsHistoryLimit}{"\n"}'
```

Vérifiez le Job immédiat :

```bash
kubectl -n ckad-cron get job manual-test
kubectl -n ckad-cron wait --for=condition=complete job/manual-test --timeout=90s
kubectl -n ckad-cron logs job/manual-test
```

Le Job hérite du délai de 3 secondes ; un démarrage lent du Pod peut donc le faire échouer avec `DeadlineExceeded`.

## Nettoyage

```bash
kubectl delete namespace ckad-cron
```
