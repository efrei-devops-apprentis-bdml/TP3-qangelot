## Objectifs

* Deployer sur Azure Container Instance (ACI) using Github Actions
* Mettre à disposition son image (format API) sur Azure Container Registry (ACR) using
Github Actions
* Mettre à disposition son code dans un repository Github

## Ressources 

https://docs.microsoft.com/en-us/azure/container-instances/container-instances-
github-action

## Push le code applicatif

- Rejoindre l'organisation Github : https://github.com/efrei-devops-apprentis-bdml 
- Créer un repository :  TP3-qangelot
- Pousser le code applicatif

## Setup les github secrets 

- tous les secrets liés à Microsoft Azure sont déjà present lors de la création d'un repository dans l'organisation
- il est nécessaire d'y ajouter le token OpenAPI  

## Créer le worklfow Github Action

- allez dans Actions et créer un nouveau workflow
- editer le fichier yaml :

```
name: Publish Docker image

on: push

jobs:

  push_to_registry-linux:
    name: Push Docker image to Azure Container Registry
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v3
      
      - name: 'Login via Azure CLI'
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: 'Build and push image'
        uses: azure/docker-login@v1
        with:
          login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      - run: |
          docker build . -t ${{ secrets.REGISTRY_LOGIN_SERVER }}/20210262:v1
          docker push ${{ secrets.REGISTRY_LOGIN_SERVER }}/20210262:v1
    
      - name: 'Deploy to Azure Container Instances'
        uses: 'azure/aci-deploy@v1'
        with:
          resource-group: ${{ secrets.RESOURCE_GROUP }}
          dns-name-label: devops-20210262
          image: ${{ secrets.REGISTRY_LOGIN_SERVER }}/20210262:v1
          registry-login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
          registry-username: ${{ secrets.REGISTRY_USERNAME }}
          registry-password: ${{ secrets.REGISTRY_PASSWORD }}
          environment-variables: API_KEY=${{ secrets.API_KEY }}
          name: 20210262
          location: 'france central'
```

- Ici on se log via le Azure CLI
- On construit et publie l'image sur le DockerHub
- On déploie cette image sur Azure en spécifiant le resouce groupe, l'image, les credentials et les variables d'environnement requises.

## Points d'intérêt 

- dans le code applicatif, on specifie l'host et le port: host = "0.0.0.0" et port=80
- enfin, dans le fichier Dockerfile, on expose le port 80
- par défaut, chez Azure le port utilisé est le port 80.

## Hadolint

Ce linter valide les meilleures pratiques décrites par Docker et adopte une approche soignée pour analyser le fichier Docker que vous devez vérifier.

On ajoute cette commande dans le fichier yaml original : 
```
uses: hadolint/hadolint-action@v2.0.0
with:
  dockerfile: Dockerfile
```

## Usage 

![image](https://user-images.githubusercontent.com/57401552/174299823-3418eb9f-1219-4f47-816a-b513392c125a.png)


