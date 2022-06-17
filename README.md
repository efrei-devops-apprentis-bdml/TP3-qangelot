## Objectifs

* Deployer sur Azure Container Instance (ACI) using Github Actions
* Mettre à disposition son image (format API) sur Azure Container Registry (ACR) using
Github Actions
* Mettre à disposition son code dans un repository Github

## Intérêts

* Bénéficier de la puissance de Github et notamment du versioning. 
* Les GitHub Actions permettent d'automatiser facilement tout le workflow : on peut créer, tester et déployer le code directement depuis GitHub.
* Cela permet aussi le travail collaboratif : faire fonctionner les revues de code, la gestion des branches et le tri des problèmes.

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

## Add hadolint au workflow

Ce linter valide les meilleures pratiques décrites par Docker et adopte une approche soignée pour analyser le fichier Docker que vous devez vérifier.

On ajoute cette commande dans le fichier yaml original : 
```
uses: hadolint/hadolint-action@v2.0.0
with:
  dockerfile: Dockerfile
```

## Configuration d'une probe liveness HTTP

On ajoute un second fichier yaml à la racine du repository et l'on y place les éléments nécessaires à la mise en place d'une probe liveness. On rajoute les lignes nécessaires à l'execution de celle-ci dans la Github Action : 

```
azcliversion: 2.30.0
inlineScript: |
  az container create --resource-group ${{ secrets.RESOURCE_GROUP }} --image ${{ secrets.REGISTRY_LOGIN_SERVER }}/20210262:v1 --dns-name-label devops-20210262 --registry-login-server ${{ secrets.REGISTRY_LOGIN_SERVER }} --registry-username ${{ secrets.REGISTRY_USERNAME }} --registry-password ${{ secrets.REGISTRY_PASSWORD }} --secure-environment-variables API_KEY=${{ secrets.API_KEY }} --location 'france central' --ports 80 -f liveness-probe.yaml
          
```

On la configure pour s'executer toute les 5sec. 

Un problème se pose néanmoins car il semblerait que Azure Cli ne permettent pas de passer les secrets stocker dans le repository Github. On préfère en rester là afin d'éviter que des données sensibles ne soient stockées dans l'image ou le code source.

## Usage 

![image](https://user-images.githubusercontent.com/57401552/174299823-3418eb9f-1219-4f47-816a-b513392c125a.png)


