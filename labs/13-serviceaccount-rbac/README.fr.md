# Lab 13 - Diagnostiquer et corriger un problème de ServiceAccount/RBAC

> 🇬🇧 [English version](README.md)

## Question

Le Deployment `inspector` du namespace `ckad-rbac` est en cours d'exécution, mais ses logs contiennent des erreurs. Inspectez les logs pour identifier le problème, puis examinez les ServiceAccounts disponibles et leurs permissions RBAC existantes afin de déterminer lequel peut le résoudre. Modifiez uniquement le Deployment pour qu'il utilise ce ServiceAccount et vérifiez que les erreurs ont disparu. Ne modifiez ni les ServiceAccounts, ni les Roles, ni les RoleBindings.

## Résultat attendu

- Les logs révèlent un échec RBAC `Forbidden`.
- Le Deployment initial utilise `default`, qui n'a aucune permission de lister les Pods dans ce lab.
- Vous comparez les permissions liées à chaque ServiceAccount.
- Le Deployment reste en cours d'exécution malgré l'erreur d'autorisation ; un Pod en cours d'exécution ne signifie pas que ses requêtes API sont autorisées.
- Le Deployment utilise le ServiceAccount existant autorisé à lister les Pods dans `ckad-rbac`.
- Seul le champ `serviceAccountName` du Deployment est modifié ; les permissions RBAC existantes restent inchangées.
- Les logs du nouveau Pod montrent une liste des Pods obtenue avec succès.

## Préparation

```bash
kubectl apply -f setup.yaml
kubectl -n ckad-rbac rollout status deployment/inspector
```

Le conteneur utilise `alpine/kubectl:1.34.1`, qui inclut `/bin/sh`, `kubectl`
et `sleep` pour la boucle de diagnostic. Utilisez une image kubectl dont la version mineure s'écarte d'au plus une
de la version de Kubernetes de votre cluster.

Si une ancienne copie de cette préparation produit `ErrImagePull` pour
`bitnami/kubectl:1.30`, réappliquez le fichier `setup.yaml` mis à jour ; cet ancien tag d'image
n'est plus disponible. Pour les autres échecs de téléchargement d'image, inspectez les événements du Pod :

```bash
kubectl -n ckad-rbac describe pods -l app=inspector
```

L'échec voulu par ce lab est un `Forbidden` dans les logs d'un conteneur en cours d'exécution,
et non un échec de téléchargement d'image.

## Étapes de réponse

Inspectez les symptômes et le RBAC :

```bash
kubectl -n ckad-rbac logs deployment/inspector
kubectl -n ckad-rbac get deployment inspector \
  -o jsonpath='{.spec.template.spec.serviceAccountName}{"\n"}'
kubectl -n ckad-rbac get serviceaccounts
kubectl -n ckad-rbac get roles
kubectl -n ckad-rbac get rolebindings
kubectl -n ckad-rbac describe role read-pod-logs
kubectl -n ckad-rbac describe role read-pods
kubectl -n ckad-rbac describe rolebinding log-reader-binding
kubectl -n ckad-rbac describe rolebinding pod-reader-binding
kubectl auth can-i list pods \
  --as=system:serviceaccount:ckad-rbac:default -n ckad-rbac
kubectl auth can-i list pods \
  --as=system:serviceaccount:ckad-rbac:log-reader -n ckad-rbac
kubectl auth can-i list pods \
  --as=system:serviceaccount:ckad-rbac:pod-reader -n ckad-rbac
```

Résultats attendus : `no` pour `default` et `log-reader`, `yes` pour `pod-reader`. Les logs initiaux
indiquent que `system:serviceaccount:ckad-rbac:default` ne peut pas lister la ressource
`pods` dans le namespace `ckad-rbac`. La boucle continue après chaque requête échouée,
le conteneur reste donc en cours d'exécution.

> Ces vérifications par emprunt d'identité exigent que votre compte opérateur dispose des permissions
> d'impersonation, comme c'est normalement le cas pour un administrateur de cluster local.

Mettez à jour le Deployment :

```bash
kubectl -n ckad-rbac patch deployment inspector --patch-file solution.yaml
kubectl -n ckad-rbac rollout status deployment/inspector
```

## Vérification

```bash
kubectl -n ckad-rbac get deployment inspector \
  -o jsonpath='{.spec.template.spec.serviceAccountName}{"\n"}'
kubectl -n ckad-rbac logs deployment/inspector
```

ServiceAccount attendu : `pod-reader`. La modification du modèle de Pod crée un nouveau Pod ;
ses logs doivent lister les Pods sans erreur `Forbidden`.

## Nettoyage

```bash
kubectl delete namespace ckad-rbac
```
