# Lab 07 - Créer un Ingress

> 🇬🇧 [English version](README.md)

## Question

Le Deployment et le Service `web` existent déjà dans le namespace `ckad-ingress` sur un cluster kind. Créez l'Ingress `web` avec la classe `nginx`. Routez l'hôte `ckad.local` et le préfixe de chemin `/` vers le Service `web` sur le port `80`. Utilisez curl pour interroger `http://ckad.local/` à travers le contrôleur Ingress et affichez le statut HTTP ainsi que le corps de la réponse.

## Résultat attendu

- La version d'API est `networking.k8s.io/v1`.
- `ingressClassName` vaut `nginx`.
- L'hôte, le chemin, le type de chemin, le Service backend et le port correspondent à l'énoncé.
- Le chemin est `/` avec `pathType: Prefix` ; aucune annotation de réécriture n'est nécessaire.
- Curl renvoie `HTTP/1.1 200 OK` et une page HTML contenant `Welcome to nginx!`.

## Préparation

Un contrôleur ingress-nginx avec l'IngressClass `nginx` doit déjà être installé dans le cluster kind. Vérifiez-le avant de commencer :

```bash
kubectl get ingressclass nginx
kubectl -n ingress-nginx get pods
```

```bash
kubectl apply -f setup.yaml
kubectl -n ckad-ingress rollout status deployment/web --timeout=120s
```

## Étapes de réponse

```bash
kubectl apply -f solution.yaml
```

Point de départ impératif possible :

```bash
kubectl -n ckad-ingress create ingress web \
  --class=nginx --rule='ckad.local/*=web:80'
```

## Vérification

```bash
kubectl -n ckad-ingress get ingress web
kubectl -n ckad-ingress describe ingress web
```

Sur kind, une adresse d'Ingress `localhost` ne garantit pas que le port du contrôleur soit exposé sur votre machine. Un Service LoadBalancer peut aussi afficher `<pending>`. Utilisez un port-forward vers le **Service du contrôleur**, et non vers le Service backend `web`, afin que la requête teste toujours la règle Ingress.

Dans un terminal, laissez tourner cette commande :

```bash
kubectl -n ingress-nginx port-forward --address 127.0.0.1 service/ingress-nginx-controller 8080:80
```

Dans un autre terminal, exécutez :

```bash
curl --noproxy "*" --fail-with-body -i --resolve ckad.local:8080:127.0.0.1 http://ckad.local:8080/
```

Sous Windows PowerShell, utilisez `curl.exe` au lieu de `curl` pour éviter l'alias `Invoke-WebRequest` :

```powershell
curl.exe --noproxy "*" --fail-with-body -i --resolve ckad.local:8080:127.0.0.1 http://ckad.local:8080/
```

`--resolve` se connecte au port local redirigé tout en conservant l'en-tête Host `ckad.local` ; aucune modification du fichier hosts n'est nécessaire. Le port `8080` n'est que le port d'accès local ; le backend de l'Ingress reste le Service `web` sur le port `80`.

Sortie attendue (en-têtes et HTML abrégés) :

```text
HTTP/1.1 200 OK
...
<h1>Welcome to nginx!</h1>
...
```

Le backend sert sa page d'accueil sur `/`, aucune réécriture de chemin n'est donc nécessaire. Si le contrôleur vient de se recharger, relancez la commande curl.

## Nettoyage

Arrêtez le port-forward avec `Ctrl+C`, puis supprimez les ressources du lab :

```bash
kubectl delete namespace ckad-ingress
```
