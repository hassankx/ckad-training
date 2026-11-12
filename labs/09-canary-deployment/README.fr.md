# Lab 09 - Créer un déploiement canary

> 🇬🇧 [English version](README.md)

## Question

Le namespace `ckad-canary` ne peut pas contenir plus de dix Pods applicatifs. Le Deployment `storefront` exécute cinq Pods avec l'image `nginx:1.24-alpine`. Passez-le à huit Pods et créez un Deployment canary nommé `storefront-canary` avec deux Pods utilisant l'image `nginx:1.25-alpine`. Les deux versions doivent recevoir du trafic via le Service existant `storefront`, avec une cible d'environ 80 % de trafic stable et 20 % de trafic canary.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: storefront
  namespace: ckad-canary
spec:
  replicas: 5
  selector:
    matchLabels:
      app: storefront
  template:
    metadata:
      labels:
        app: storefront
    spec:
      containers:
        - name: storefront
          image: nginx:1.24-alpine
          ports:
            - containerPort: 80
```

## Résultat attendu

- Le nombre de réplicas stables passe de cinq à huit.
- Le nombre de réplicas canary est de deux.
- Le total des Pods applicatifs est de dix.
- Les deux Deployments partagent le label `app: storefront`.
- Un second label distingue la version stable de la version canary.
- Le sélecteur du Service correspond aux deux versions.
- Lorsque les dix Pods sont prêts, huit endpoints stables et deux endpoints canary donnent une répartition du trafic d'environ 80/20.

## Préparation

```bash
kubectl apply -f setup.yaml
```

## Étapes de réponse

```bash
kubectl apply -f solution.yaml
kubectl -n ckad-canary rollout status deployment/storefront
kubectl -n ckad-canary rollout status deployment/storefront-canary
```

## Vérification

```bash
kubectl -n ckad-canary get deployments
kubectl -n ckad-canary get pods -l app=storefront --show-labels
kubectl -n ckad-canary get pods -l app=storefront --no-headers | wc -l
kubectl -n ckad-canary get endpointslice \
  -l kubernetes.io/service-name=storefront
```

La commande de comptage des Pods doit renvoyer `10`. Les Deployments doivent afficher `8/8` réplicas prêts pour la version stable et `2/2` pour la version canary, et les EndpointSlices du Service doivent contenir dix endpoints de Pods prêts.

### Port-forward et curl : test de connectivité

Dans un terminal, exécutez :

```bash
kubectl -n ckad-canary port-forward service/storefront 8080:80
```

Dans un autre terminal, vérifiez la réponse et la version de nginx :

```bash
curl -sS -I http://localhost:8080
```

Vous devez obtenir `HTTP/1.1 200 OK` et soit `Server: nginx/1.24.0` (stable), soit `Server: nginx/1.25.5` (canary). La version de correctif peut varier selon le tag de l'image.

**Le port-forward ne permet pas de valider la répartition du trafic.** Même lorsqu'il cible un Service, `kubectl port-forward` sélectionne un seul Pod et lui transmet toutes les connexions. Répéter curl sur `localhost:8080` ne répartit pas la charge entre les dix Pods. Arrêtez le port-forward avec `Ctrl+C` une fois terminé.

### Curl à travers le Service : échantillonner la répartition 80/20

Utilisez la commande Bash suivante pour lancer un client curl temporaire dans le namespace `default`. Il appelle le nom DNS interne du Service, ce qui sollicite la répartition de charge du Service plutôt que le port-forward. Garder le client hors de `ckad-canary` maintient ce namespace à dix Pods.

Assurez-vous d'abord que les deux Deployments sont prêts :

```bash
kubectl -n ckad-canary rollout status deployment/storefront
kubectl -n ckad-canary rollout status deployment/storefront-canary
```

Envoyez ensuite 1 000 requêtes, en ouvrant une nouvelle connexion pour chacune et en identifiant le backend grâce à son en-tête nginx `Server` :

```bash
kubectl -n default run storefront-traffic-check \
  --image=curlimages/curl:8.10.1 \
  --restart=Never --rm -i --command -- sh -c '
    set -eu
    stable=0
    canary=0
    i=0
    while [ "$i" -lt 1000 ]; do
      headers=$(curl --fail --silent --show-error --head \
        --connect-timeout 5 --max-time 10 \
        -H "Connection: close" \
        http://storefront.ckad-canary.svc.cluster.local) || exit 1
      case "$headers" in
        *nginx/1.24.*) stable=$((stable + 1)) ;;
        *nginx/1.25.*) canary=$((canary + 1)) ;;
        *) printf "Unexpected response headers:\n%s\n" "$headers" >&2; exit 1 ;;
      esac
      i=$((i + 1))
    done
    printf "Stable: %d/1000 (%d.%d%%)\n" "$stable" "$((stable / 10))" "$((stable % 10))"
    printf "Canary: %d/1000 (%d.%d%%)\n" "$canary" "$((canary / 10))" "$((canary % 10))"
  '
```

Exemple de sortie (vos chiffres varieront) :

```text
Stable: 803/1000 (80.3%)
Canary: 197/1000 (19.7%)
```


```bash
kubectl -n default delete pod storefront-traffic-check --ignore-not-found
```

## Nettoyage

```bash
kubectl delete namespace ckad-canary
```
