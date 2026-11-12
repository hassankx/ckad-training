# Lab 11 - Network Policy

> 🇬🇧 [English version](README.md)

## Question

Le namespace `ckad-network` contient un Deployment `app`, un Deployment `database` et un Pod `client`. Les Services `app` et `database` exposent les Deployments correspondants sur le port TCP 80. Des NetworkPolicies sont déjà configurées, mais le Pod `client` ne peut se connecter à aucun des deux Services.

Faites en sorte que le Pod `client` puisse se connecter uniquement à `app` et `database` sur le port TCP 80, sans accès aux autres workloads. Ne modifiez, ne supprimez et ne remplacez aucune NetworkPolicy existante, et n'en créez aucune nouvelle.

## Résultat attendu

- Les politiques existantes restent inchangées et aucune nouvelle politique n'est créée.
- Le Pod `client` peut se connecter à `app` et `database` sur le port TCP 80, mais pas aux autres workloads.

## Préparation

```bash
kubectl apply -f setup.yaml
kubectl -n ckad-network wait --for=condition=Ready pod/client --timeout=90s
kubectl -n ckad-network rollout status deployment/app
kubectl -n ckad-network rollout status deployment/database
```

Ce lab nécessite un plugin CNI qui applique les NetworkPolicies.

Vérifiez la connectivité initiale :

```bash
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://app
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://database
```

Les workloads utilisent nginx comme point de terminaison HTTP de test ; `database` n'est pas un véritable serveur de base de données.

## Étapes de réponse

**Limite :** la connectivité demandée ne peut pas être obtenue avec les contraintes actuelles. Avec ces trois politiques seulement, les labels ne suffisent pas à rétablir l'accès, car le trafic sortant du client, DNS compris, est refusé. Les étapes ci-dessous démontrent cette limite plutôt qu'une solution réussie.

Inspectez les trois politiques et les labels des Pods :

```bash
kubectl -n ckad-network get networkpolicy default-deny-all app-ingress database-ingress -o yaml
kubectl -n ckad-network get pods --show-labels
```

La politique `default-deny-all` sélectionne tous les Pods avec `podSelector: {}` et refuse le trafic entrant comme sortant. Les règles d'autorisation des NetworkPolicies sont additives, mais `app-ingress` et `database-ingress` ne fournissent que des exceptions pour le trafic entrant. Aucune des trois politiques n'autorise le trafic sortant du client.

Les deux politiques d'entrée exigent des labels source différents : `app-ingress` exige `app-access: granted`, tandis que `database-ingress` exige `database-access: granted`. Ajoutez les deux labels au client sans modifier son label existant `app: client`, pour démontrer que satisfaire les règles d'entrée ne suffit pas :

```bash
kubectl -n ckad-network label pod client app-access=granted database-access=granted
```

## Vérification

```bash
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://app
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://database
kubectl -n ckad-network get networkpolicy default-deny-all app-ingress database-ingress -o yaml
```

Les deux requêtes doivent encore échouer. Chaque label ajouté fait correspondre `client` au sélecteur de source autorisé de la politique concernée, mais son trafic sortant reste bloqué. Les trois politiques restent inchangées.

Pour confirmer que le DNS n'est pas le seul obstacle, récupérez les IP des Services et remplacez `<APP_CLUSTER_IP>` et `<DATABASE_CLUSTER_IP>` ci-dessous par ces valeurs :

```bash
kubectl -n ckad-network get service app database -o wide
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://<APP_CLUSTER_IP>
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://<DATABASE_CLUSTER_IP>
```

Ces requêtes doivent également échouer, car le trafic sortant du client est refusé, même sans résolution DNS.

Rétablir la connectivité exigerait de modifier les politiques ou d'ajouter des politiques autorisant le trafic sortant du client vers les deux backends sur le port TCP 80 et vers le DNS sur le port 53 en UDP/TCP. Ces changements sont volontairement exclus des contraintes de ce lab. Le trafic de réponse des connexions autorisées est implicitement permis ; des règles de sortie distinctes côté backend ne sont pas nécessaires pour les réponses.

## Nettoyage

```bash
kubectl delete namespace ckad-network
```
