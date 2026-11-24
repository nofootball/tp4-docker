**# TP4: Docker et Docker Compose (application MERN)**



etudiante : Amel Nechi, groupe 5B



**## Objectif**

Conteneuriser une application (client React + serveur Express), la faire communiquer avec MongoDB via un réseau Docker, puis orchestrer le tout avec Docker Compose.



**## Prérequis**

\- Docker Desktop installé et démarré (`docker version` doit afficher Client et Server)



**## Structure du projet**

\- `Client/` : application React (Vite) + Dockerfile

\- `Server/` : API Express + Dockerfile

\- `docker-compose.yml` : orchestration des 3 services

\- `Jenkinsfile` : pipeline CI/CD (build, test, push, deploy)



**## Étapes réalisées**



**### 1. Cloner le projet**

git clone https://github.com/wiem-kb/MyAppReact.git

cd MyAppReact





**### 2. Dockerfile du serveur (`Server/Dockerfile`)**

dockerfile

FROM node:lts-alpine

WORKDIR /app

COPY package\*.json ./

RUN npm install

COPY . .

EXPOSE 9000

CMD \["npm", "start"]





Construction :



docker build -t mern-server ./Server





**### 3. Dockerfile du client (`Client/Dockerfile`)**

dockerfile

FROM node:lts-alpine

WORKDIR /app

COPY package\*.json ./

RUN npm install

COPY . .

RUN npm run build

RUN npm install -g serve

EXPOSE 3000

CMD \["serve", "-s", "dist", "-l", "3000"]





Construction :



docker build -t mern-client ./Client





**### 4. Réseau Docker**



docker network create mern-network



Un réseau personnalisé permet aux conteneurs de communiquer entre eux en utilisant le nom du conteneur comme nom d'hôte.



**### 5. MongoDB**



docker run -d --name mongodb --network mern-network mongo





**### 6. Serveur et client**



docker run -d --name server --network mern-network -p 9000:9000 -e MONGO\_URI=mongodb://mongodb:27017/mern mern-server

docker run -d --name client --network mern-network -p 3000:3000 mern-client





Vérification : `docker ps` affiche les 3 conteneurs avec le statut `Up`.



\### 7. Docker Compose (`docker-compose.yml`)

yaml

services:

&#x20; mongodb:

&#x20;   image: mongo

&#x20;   container\_name: mongodb

&#x20;   networks:

&#x20;     - mern-network



&#x20; server:

&#x20;   build: ./Server

&#x20;   container\_name: server

&#x20;   ports:

&#x20;     - "9000:9000"

&#x20;   environment:

&#x20;     - MONGO\_URI=mongodb://mongodb:27017/mern

&#x20;   depends\_on:

&#x20;     - mongodb

&#x20;   networks:

&#x20;     - mern-network



&#x20; client:

&#x20;   build: ./Client

&#x20;   container\_name: client

&#x20;   ports:

&#x20;     - "3000:3000"

&#x20;   depends\_on:

&#x20;     - server

&#x20;   networks:

&#x20;     - mern-network



networks:

&#x20; mern-network:

&#x20;   driver: bridge





\- Les conteneurs anciens (étapes 4 à 6) ont été supprimés avant, pour libérer les ports et le réseau :



docker rm -f client server mongodb

docker network rm mern-network



Lancement :



docker compose up --build



**## Résultats**

\- Client : http://localhost:3000

\- API : http://localhost:9000/api/health (réponse : `{"message":"API is running"}`)



**## Pipeline Jenkins (CI/CD)**



Le fichier `Jenkinsfile` à la racine du projet automatise le cycle complet, comme dans le TP3 :



| Étape | Rôle |

|---|---|

| Checkout | Récupère le code depuis GitHub |

| Docker Build | Construit les images `mern-server` et `mern-client` |

| Smoke Test | Lance les deux conteneurs de test (ports 9001 et 3001) et vérifie `/api/health` et la réponse HTTP 200 du client |

| Push (Docker Hub) | Pousse les images avec les tags `build-N` et `latest` |

| Deploy | Lance l'application avec `docker compose up -d --build` |



Configuration Jenkins :

\- Job de type Pipeline, « Pipeline script from SCM », dépôt Git du projet, branche `main`, Script Path `Jenkinsfile`

\- Credential `dockerhub-creds` (Username with password) pour se connecter à Docker Hub



**## Images Docker Hub**

\- Serveur : `amelnechi/mern-server`

\- Client : `amelnechi/mern-client`



Récupération :



docker pull amelnechi/mern-server:latest

docker pull amelnechi/mern-client:latest





**## Arrêt**

docker compose down



