# Plan de travail 

## Vue d'ensemble du projet

Attention des divergences peuvent exister que ce soit l'invite de command ou le nombre de classe. Le nombre de PR, de branche c'est personnel. Bien regarder les imports

## Flux de travail

- **Nombre total d'issues :** 5 --> **Pull Requests (PR) :** 5 --> **Branches créées :** 5
- 1: Model & class
- 2: Test
- 3: Script db
- 4: API
- 5: Docker
## Structure du dépôt

Voici les composants clés du dépôt visualisés dans les diagrammes de structure :

### Structure des fichiers du dépôt

![Structure des fichiers du dépôt](https://github.com/Sebb955/PlanTECHNOLOG/assets/79416415/fce5732d-55f1-41f5-9675-e0fdda7e6230)

## Étapes de développement

### Étape 1 : Code Python pour la validation des modèles

Développer le `Field_validator` pour les modèles afin de garantir l'intégrité des données.

### Étape 2 : Tests unitaires

Implémenter des tests unitaires utilisant `Unittest`, `Pytest`, et `Mock` pour s'assurer que les fonctionnalités fonctionnent comme prévu.

### Étape 3.1 : Déclaration des constantes `Constantes.py`

```python
connect = 'mongodb'
db = "MagasinBurger"
collection = "Burgers"
```

### Étape 3.2 : Script pour peupler la BD `seeder.py` ou `script.py`

```python
from pymongo import MongoClient

from constantes import *


client = MongoClient(connect)
Magasin = client[db]
burgers= Magasin[collection]

b1 = { "prix" : 10, "description" : 'Burger1' , "allergènes" : ["moutarde"] , "cuisson" : "saignant", "scoville": 5000}
b2 = { "prix" : 15, "description" : 'Burger2' , "allergènes" : ["citron","noix"] , "cuisson" : "cuit", "scoville": 50000}
b3 = { "prix" : 19, "description" : 'Burger3' , "allergènes" : ["cacao","noix"] , "cuisson" : "à point", "scoville": 15000}



peuple_burgers = [b1,b2,b3]

burgers.insert_many(peuple_burgers)
```

### Étape 4 : Implementation de l'`API`

```python
from fastapi import FastAPI, HTTPException
from pymongo import MongoClient
from constantes import *
from burger import Burger

app = FastAPI()

# Connexion à la base de données MongoDB
client = MongoClient(connect)
db = client[db]
burgers_collection = db[collection1]

@app.post("/burgers/")
async def create_burger(burger: Burger):
    result = burgers_collection.insert_one({
        "prix": burger.prix,
        "description": burger.description,
        "allergènes": burger.allergènes,
        "cuisson": burger.cuisson,
        "scoville": burger.scoville
    })
    if result.acknowledged:
        return {"message": "Burger ajouté avec succès"}
    else:
        raise HTTPException(status_code=500, detail="Erreur lors de l'ajout du burger")


@app.get("/burgers/")
async def get_burgers():
    all_burgers = list(burgers_collection.find({}, {"_id": 0}))
    return all_burgers

@app.get("/burgers/prix/{prix}")
async def get_burgers_by_price(prix: int):
    burgers = list(burgers_collection.find({"prix": prix}, {"_id": 0}))
    if not burgers:
        raise HTTPException(status_code=404, detail="Aucun burger trouvé pour ce prix")
    return burgers

@app.get("/burgers/exclude/{allergene}")
async def get_burgers_exclude_allergen(allergene: str):
    burgers = list(burgers_collection.find({"allergènes": {"$nin": [allergene]}}, {"_id": 0}))
    if not burgers:
        raise HTTPException(status_code=404, detail="Aucun burger trouvé sans cet allergène")
    return burgers

# @app.delete("/burgers/{burger_id}")
# async def delete_burger(burger_id: str):
#     result = burgers_collection.delete_one({"_id": ObjectId(burger_id)})
#     if result.deleted_count == 0:
#         raise HTTPException(status_code=404, detail="Burger non trouvé")
#     else:
#         return {"message": "Burger supprimé avec succès"}
```

### Étape 5 : Implementation du `dockerfile` et `docker-compose.yml` :

`docker-compose.yml`
```python
version: "3"
services:
  mongodb:
    image: mongo:latest
    ports:
      - 27017:27017
  app:
    build: .
    ports:
      - 8763:8763
    depends_on:
      - mongodb
```

`dockerfile`
```python
FROM python:3.10-alpine

RUN pip install fastapi uvicorn pymongo pydantic

WORKDIR /app

COPY /app/ .

EXPOSE 8763

CMD sh -c 'python3 seeder.py && uvicorn api:app --host 127.0.0.1 --port 8763'
```

### Test :

#On vérifie la base de donnée du `seeder.py`/`script.py` :

```python
use Basedonnée
db.collection.find()
```

#Invite de commande(`cmd`) : (peut changer)
```python
docker-compose up 
```


MANQUE DOCKER HUB 

### [AIDE] Commande docker :

```python
##Commandes Docker

#créer un réseau
docker network create mynet

#lancer le container mongodb en le connectant avec mynet
docker run --name mongodb --network mynet -d mongo

#lancer le python-mongo
docker run --name python-mongo --network mynet -it
python:alpine

#lancer le terminal interactif de python-mongo pour installer pymongo
docker exec -it python-mongo sh

#installer pymongo
pip install pymongo

#copier le fichier seeder.py dans le conteneur
docker cp seeder.py python-mongo:/seeder.py

#executer le fichier seeder.py
docker exec -it python-mongo python seeder.py

#executer le docker compose
docker-compose up --build
```
