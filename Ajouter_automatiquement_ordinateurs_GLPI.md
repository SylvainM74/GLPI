

### Guide Récupération du Session Token depuis l'API GLPI

Ce guide vous montrera comment créer un script en Python pour récupérer le Session Token à partir de l'API GLPI. Le Session Token sera utilisé pour effectuer des opérations ultérieures, comme l'importation d'ordinateurs à partir d'un fichier CSV.

## Prérequis

- Python 3.x installé sur votre ordinateur
- Un compte utilisateur avec les droits d'accès nécessaires à l'API GLPI
- L'URL de votre instance GLPI (par exemple : http://glpi.olympus.gr)
- Votre App Token (un token d'application généré à partir de GLPI)

## Étape 1 : Obtenir l'API Token pour l'authentification

Avant de pouvoir accéder à l'API GLPI, vous avez besoin d'un API Token pour l'authentification. Cet API Token permettra à votre script Python de s'identifier auprès de l'API et d'effectuer des opérations.

### Prérequis

- Vous avez un compte utilisateur avec les droits d'accès nécessaires à l'API GLPI.
- Vous savez comment accéder à l'interface web de votre instance GLPI.

### Marche à suivre

1. Connectez-vous à l'interface web de votre instance GLPI en utilisant vos identifiants d'utilisateur.

2. Une fois connecté, accédez aux paramètres de votre compte ou de l'application.

3. Recherchez la section dédiée à l'API ou aux tokens d'application. Les emplacements varient en fonction de la version de GLPI que vous utilisez.

4. Créez un nouvel API Token. Vous pourriez devoir fournir un nom ou une description pour le token. Assurez-vous de comprendre les droits accordés à ce token, car cela déterminera les actions qu'il peut effectuer.

5. Une fois que le token est créé, copiez-le et stockez-le en lieu sûr. Ce sera votre clé d'authentification pour accéder à l'API.

6. Gardez à l'esprit que cet API Token est une information sensible. Ne le partagez avec personne et ne le stockez pas dans des fichiers publics ou accessibles en ligne.

Une fois que vous avez obtenu l'API Token, vous êtes prêt à le mettre en œuvre dans votre script Python pour obtenir le Session Token et effectuer d'autres opérations.


## Étape 2

1. **Importation des bibliothèques nécessaires**

    Avant de commencer, assurez-vous que les bibliothèques `requests` et `json` sont installées. Si ce n'est pas le cas, vous pouvez les installer à l'aide de la commande suivante :

    ```
    pip install requests
    ```

2. **Écriture du script**

    Créez un nouveau fichier Python (par exemple `obtenir_token.py`) et ajoutez le code suivant :

    ```python
    import requests

    URL_GLPI = "http://votre_url_glpi/apirest.php"
    APP_TOKEN = "Votre_App_Token"

    # Entêtes pour les requêtes
    headers = {
        "App-Token": APP_TOKEN,
        "Content-Type": "application/json"
    }

    # Étape 1: Authentification pour obtenir le Session-Token
    auth_data = {
        "login": "Votre_Nom_d'utilisateur",
        "password": "Votre_Mot_de_passe"
    }
    response = requests.post(f"{URL_GLPI}/initSession", json=auth_data, headers=headers)
    data = response.json()

    if 'session_token' in data:
        SESSION_TOKEN = data['session_token']
        print("Session Token obtenu avec succès:", SESSION_TOKEN)
    else:
        print("Erreur lors de l'obtention du Session Token:", data)
    ```

3. **Exécution du script**

    Ouvrez votre terminal/commande, accédez au répertoire contenant le script (`obtenir_token.py`) et exécutez-le :

    ```
    python obtenir_token.py
    ```

    Vous devriez voir un message indiquant si le Session Token a été obtenu avec succès ou s'il y a eu une erreur.

Cela complète la première étape de récupération du Session Token depuis l'API GLPI. Dans la prochaine étape, nous verrons comment importer des ordinateurs à partir d'un fichier CSV en utilisant le Session Token.

______________________________________________________________________________

### Guide Importation d'Ordinateurs depuis un Fichier CSV avec le Session Token

Dans cette étape, nous allons utiliser le Session Token obtenu précédemment pour importer des données d'ordinateurs depuis un fichier CSV vers l'API GLPI.

## Prérequis

- Vous avez suivi la première étape du guide et avez obtenu avec succès le Session Token depuis l'API GLPI.
- Vous avez un fichier CSV (`ordinateurs.csv`) contenant les données des ordinateurs à importer.
- Vous avez les droits d'accès nécessaires pour ajouter des ordinateurs à l'API GLPI.

## Étape 1

1. **Écriture du script**

    Créez un nouveau fichier Python (par exemple `importer_csv.py`) et ajoutez le code suivant :

    ```python
    import requests
    import csv

    URL_GLPI = "http://Votre_Url_Glpi/apirest.php"
    APP_TOKEN = "Votre_App_Token"
    SESSION_TOKEN = "Votre_Session_Token"  # Remplacez par le Session Token obtenu

    # Entêtes pour les requêtes
    headers = {
        "App-Token": APP_TOKEN,
        "Content-Type": "application/json",
        "Session-Token": SESSION_TOKEN
    }

    def get_type_id(type_name):
        response = requests.get(f"{URL_GLPI}/ComputerType", headers=headers)
        data = response.json()
        for type_data in data:
            if "name" in type_data and type_data["name"] == type_name:
                return type_data["id"]
        return None

	# Vous pouvez faire Shift + Clique droit sur votre fichier csv pour obtenir son            chemin d'accès
    with open(r'Votre_Fichier_Csv', mode='r', encoding='utf-8') as file: 
        reader = csv.DictReader(file)
        next(reader)  # Passez l'en-tête

        for row in reader:
            type_id = get_type_id(row["type"])
            if type_id is not None:
                computer_data = {
                    "input": [{
                        "name": row["nom"],
                        "computertypes_id": type_id
                    }]
                }

                response = requests.post(f"{URL_GLPI}/Computer/", headers=headers, json=computer_data, verify=False)
                if response.status_code == 201:
                    print(f"Ordinateur {row['nom']} ajouté avec succès.")
                else:
                    print(f"Erreur lors de l'ajout de l'ordinateur {row['nom']}. Code d'erreur : {response.status_code}")
    ```

2. **Exécution du script**

    Ouvrez votre terminal/commande, accédez au répertoire contenant le script (`importer_csv.py`) et exécutez-le :

    ```
    python importer_csv.py
    ```

    Le script lira le fichier CSV et tentera d'ajouter chaque ordinateur à l'API GLPI. Vous verrez les messages de succès ou d'erreur dans le terminal.

Cela termine la deuxième étape d'importation des ordinateurs depuis un fichier CSV en utilisant le Session Token. Vous avez maintenant une solution pour automatiser l'ajout d'ordinateurs à votre base de données GLPI.