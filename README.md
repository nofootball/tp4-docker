\# TP4 - Docker et Docker Compose (application MERN)

Amel Nechi 

groupe 5B



\## Prérequis

\- Docker Desktop installé et démarré



\## Structure du projet

\- `Client/` : application React (Vite) + Dockerfile

\- `Server/` : API Express + Dockerfile

\- `docker-compose.yml` : orchestration des 3 services



\## Étapes réalisées



\### 1. Cloner le projet

&#x20;   git clone https://github.com/wiem-kb/MyAppReact.git



\### 2. Dockerfiles

\- Serveur (`Server/Dockerfile`) : image de base node:lts-alpine, répertoire de travail /app, copie de package\*.json, npm install, copie du code, port 9000, démarrage avec npm start.

FROM node:lts-alpine

WORKDIR /app

COPY package\*.json ./

RUN npm install

COPY . .

EXPOSE 9000

CMD \["npm", "start"]







\- Client (`Client/Dockerfile`) : image de base node:lts-alpine, répertoire de travail /app, copie de package\*.json, npm install, copie du code, npm run build, installation de serve, port 3000, démarrage avec serve sur le dossier dist.

FROM node:lts-alpine

WORKDIR /app

COPY package\*.json ./

RUN npm install

COPY . .

RUN npm run build

RUN npm install -g serve

EXPOSE 3000

CMD \["serve", "-s", "dist", "-l", "3000"]





Construction des images :



&#x20;   docker build -t mern-server ./Server

&#x20;   docker build -t mern-client ./Client



\### 3. Réseau Docker

&#x20;   docker network create mern-network



\### 4. MongoDB

&#x20;   docker run -d --name mongodb --network mern-network mongo



\### 5. Serveur et client

&#x20;   docker run -d --name server --network mern-network -p 9000:9000 -e MONGO\_URI=mongodb://mongodb:27017/mern mern-server

&#x20;   docker run -d --name client --network mern-network -p 3000:3000 mern-client



\### 6. Docker Compose

Le fichier `docker-compose.yml` définit les services mongodb, server et client, connectés au réseau mern-network (driver bridge). Le serveur dépend de mongodb et définit la variable MONGO\_URI. Le client dépend du serveur.



Lancement :



&#x20;   docker compose up --build



\## Accès

\- Client : http://localhost:3000

\- API : http://localhost:9000/api/health



\## Arrêt

&#x20;   docker compose down

