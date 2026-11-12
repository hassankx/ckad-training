# Lab 12 - Construire et exporter une image Docker

> 🇬🇧 [English version](README.md)

## Question

Construisez l'image Docker définie dans ce répertoire. Attribuez-lui le tag `ckad-web:v1` et exportez-la vers `ckad-web-v1.tar`.

## Résultat attendu

- Le tag de l'image locale est `ckad-web:v1`.
- L'image sert le fichier `index.html` fourni.
- `ckad-web-v1.tar` est une archive d'image Docker qui peut être chargée avec `docker load`.

## Préparation

Vérifiez que Docker est disponible :

```bash
docker version
```

## Étapes de réponse

```bash
docker build -t ckad-web:v1 .
docker save -o ckad-web-v1.tar ckad-web:v1
```

## Vérification

```bash
docker image inspect ckad-web:v1
docker run --rm -d --name ckad-web-test -p 8080:80 ckad-web:v1
curl http://localhost:8080
docker stop ckad-web-test
docker load -i ckad-web-v1.tar
```

## Nettoyage

```bash
docker image rm ckad-web:v1
rm -f ckad-web-v1.tar
```

Équivalent PowerShell pour le nettoyage :

```powershell
Remove-Item .\ckad-web-v1.tar
```

