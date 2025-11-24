
# MINI TUTORIEL PYTHON - RECHERCHE COMMUNE → CODE POSTAL

## Niveau : Débutant / Intermédiaire
## Durée : 1-2 heures
## Plateforme : CodeSandbox Python

---

## ÉNONCÉ

**Créer une application Python qui :**
1. Demande à l'utilisateur d'entrer le nom d'une commune française
2. Retourne le code postal correspondant
3. Gère les erreurs (commune inexistante)
4. Affiche les résultats de façon élégante

**Exemple d'usage :**
```
Entrez le nom d'une commune : Paris
→ Code postal : 75001

Entrez le nom d'une commune : Lyon
→ Code postal : 69000

Entrez le nom d'une commune : Marseille
→ Code postal : 13000
```

---

## OBJECTIFS PÉDAGOGIQUES

À la fin du tutoriel, l'étudiant pourra :
- Utiliser une API REST externe
- Faire des requêtes HTTP en Python
- Parser du JSON
- Gérer les erreurs (try/except)
- Créer une boucle interactive
- Utiliser CodeSandbox pour le prototypage rapide

---

## ÉTAPE 1 : SETUP CODESANDBOX

### Sur CodeSandbox

1. Aller sur https://codesandbox.io
2. Créer un nouveau sandbox
3. Choisir Python comme template
4. Nommer le sandbox : commune-to-zipcode

Fichier initial : main.py

---

## ÉTAPE 2 : VERSION 1 - SIMPLE AVEC DICTIONNAIRE

Commencer simple : un dictionnaire de communes

```python
# main.py

# Dictionnaire communes → codes postaux
communes = {
    "paris": "75001",
    "lyon": "69000",
    "marseille": "13000",
    "toulouse": "31000",
    "nice": "06000",
    "nantes": "44000",
    "strasbourg": "67000",
    "montpellier": "34000",
    "bordeaux": "33000",
    "lille": "59000",
}

def rechercher_code_postal(commune):
    """Recherche le code postal d'une commune"""
    commune = commune.lower().strip()

    if commune in communes:
        return communes[commune]
    else:
        return None

# Programme principal
print("=== RECHERCHE CODE POSTAL ===\n")

while True:
    entree = input("Entrez le nom d'une commune (ou 'quit' pour quitter) : ")

    if entree.lower() == "quit":
        print("Au revoir!")
        break

    code_postal = rechercher_code_postal(entree)

    if code_postal:
        print(f"OK {entree.capitalize()} → Code postal : {code_postal}\n")
    else:
        print(f"ERREUR Commune '{entree}' non trouvée dans la base de données\n")
```

**Concepts enseignés :**
- Dictionnaire Python
- Fonction simple
- Boucle while
- Conditions if/else
- Gestion casse (.lower(), .strip())

**À tester :**
```
Entrez commune : paris
OK Paris → Code postal : 75001

Entrez commune : toulouse  
OK Toulouse → Code postal : 31000

Entrez commune : Unknown
ERREUR Commune 'Unknown' non trouvée
```

---

## ÉTAPE 3 : VERSION 2 - AVEC API EXTERNE - CORRIGÉE

Utiliser une véritable API pour des données réelles

### API à utiliser : Adresse.data.gouv.fr

C'est l'API officielle française des adresses !

```python
import requests
import json

def rechercher_commune_api(commune):
    """
    Recherche une commune via l'API Adresse.data.gouv.fr
    Retourne le code postal si trouvé
    """

    # URL de l'API
    url = "https://api-adresse.data.gouv.fr/search/"

    # Paramètres
    params = {
        "q": commune,
        "type": "municipality",  # Chercher seulement les communes
        "limit": 1  # Limiter à 1 résultat
    }

    try:
        # Faire la requête
        response = requests.get(url, params=params)
        response.raise_for_status()  # Vérifier erreurs HTTP

        # Parser JSON
        data = response.json()

        # CORRECTION : Vérifier que la liste "features" n'est pas vide AVANT d'accéder à l'index [0]
        if data.get("features") and len(data["features"]) > 0:
            # Premier résultat - Maintenant on est sûr qu'il existe
            commune_data = data["features"][0]
            properties = commune_data.get("properties", {})

            # Extraire info
            nom = properties.get("name")
            code_postal = properties.get("postcode")

            # Vérifier que les données existent
            if nom and code_postal:
                return {
                    "nom": nom,
                    "code_postal": code_postal,
                    "trouvee": True
                }
            else:
                return {"trouvee": False}
        else:
            return {"trouvee": False}

    except requests.exceptions.RequestException as e:
        print(f"Erreur API : {e}")
        return {"trouvee": False}
    except (KeyError, IndexError, TypeError) as e:
        print(f"Erreur parsing JSON : {e}")
        return {"trouvee": False}

# Programme principal
print("=== RECHERCHE CODE POSTAL ===\n")
print("(Utilise l'API officielle Adresse.data.gouv.fr)\n")

while True:
    entree = input("Entrez le nom d'une commune (ou 'quit') : ")

    if entree.lower() == "quit":
        print("Au revoir!")
        break

    if not entree.strip():
        print("ERREUR : Veuillez entrer une commune valide\n")
        continue

    print("Recherche en cours...")
    resultat = rechercher_commune_api(entree)

    if resultat["trouvee"]:
        nom = resultat["nom"]
        code_postal = resultat["code_postal"]
        print(f"OK {nom} → Code postal : {code_postal}\n")
    else:
        print(f"ERREUR Commune '{entree}' non trouvée\n")
```

**CORRECTIONS APPORTÉES :**
- ✓ Vérifier que `data["features"]` existe avec `.get()`
- ✓ Vérifier que la liste n'est pas vide avec `len() > 0`
- ✓ Accéder à l'index [0] SEULEMENT après vérification
- ✓ Gestion des KeyError, IndexError, TypeError
- ✓ Valider les données avant de les retourner

**Concepts enseignés :**
- Bibliothèque requests
- Requêtes HTTP GET
- Paramètres URL
- Parsing JSON sécurisé
- Gestion erreurs réseau
- Accès structures imbriquées
- Validation données

**À tester :**
```
Entrez commune : Bordeaux
Recherche en cours...
OK Bordeaux → Code postal : 33000

Entrez commune : Strasbourg
OK Strasbourg → Code postal : 67000

Entrez commune : UnknownCity123
ERREUR Commune 'UnknownCity123' non trouvée
```

---

## ÉTAPE 4 : VERSION 3 - AMÉLIORATIONS UX - CORRIGÉE

Rendre l'app plus agréable avec historique

```python
import requests
from datetime import datetime

def rechercher_commune_api(commune):
    """Recherche une commune via API"""

    url = "https://api-adresse.data.gouv.fr/search/"
    params = {
        "q": commune,
        "type": "municipality",
        "limit": 5  # Retourner jusqu'à 5 résultats
    }

    try:
        response = requests.get(url, params=params)
        response.raise_for_status()
        data = response.json()

        # CORRECTION : Vérification sécurisée
        if data.get("features") and len(data["features"]) > 0:
            resultats = []
            for feature in data.get("features", []):
                properties = feature.get("properties", {})

                nom = properties.get("name")
                code_postal = properties.get("postcode")

                # Seulement ajouter si données valides
                if nom and code_postal:
                    resultats.append({
                        "nom": nom,
                        "code_postal": code_postal,
                    })

            if resultats:
                return {"trouvees": True, "resultats": resultats}
            else:
                return {"trouvees": False}
        else:
            return {"trouvees": False}

    except requests.exceptions.RequestException as e:
        print(f"Erreur API : {e}")
        return {"trouvees": False}
    except (KeyError, IndexError, TypeError) as e:
        print(f"Erreur parsing : {e}")
        return {"trouvees": False}

def afficher_resultats(resultats):
    """Affiche les résultats de manière élégante"""

    if len(resultats) == 1:
        r = resultats[0]
        print("\n--- RESULTAT ---")
        print(f"Commune : {r['nom']}")
        print(f"Code postal : {r['code_postal']}")
        print("---\n")
    else:
        print(f"\n{len(resultats)} resultats trouves:\n")
        for i, r in enumerate(resultats, 1):
            print(f"  {i}. {r['nom']:<25} → {r['code_postal']}")
        print()

# Programme principal
print("=== RECHERCHE CODE POSTAL COMMUNES ===")
print("(Powered by data.gouv.fr)\n")

historique = []

while True:
    try:
        entree = input("Entrez commune (ou 'quit'/'history') : ").strip()

        if entree.lower() == "quit":
            print("\nAu revoir!")
            break

        if entree.lower() == "history":
            if historique:
                print("\nHISTORIQUE :")
                for h in historique:
                    print(f"  - {h}")
                print()
            else:
                print("\n(Aucune recherche dans l'historique)\n")
            continue

        if not entree:
            print("ATTENTION : Veuillez entrer une commune\n")
            continue

        print("Recherche en cours...")
        resultat = rechercher_commune_api(entree)

        if resultat["trouvees"]:
            afficher_resultats(resultat["resultats"])
            historique.append(f"{entree} → {resultat['resultats'][0]['code_postal']}")
        else:
            print(f"\nERREUR Commune '{entree}' non trouvée\n")

    except KeyboardInterrupt:
        print("\n\nInterruption utilisateur. Au revoir!")
        break
    except Exception as e:
        print(f"ERREUR : {e}\n")
```

**Améliorations :**
- Affichage formaté
- Historique des recherches
- Gestion multiple résultats
- Gestion Ctrl+C
- Validation entrée
- Gestion complète erreurs

---

## ÉTAPE 5 : VERSION 4 - AVEC TESTS UNITAIRES

Introduire les tests automatisés

```python
import requests
import unittest

def rechercher_commune(commune):
    """Fonction principale"""
    url = "https://api-adresse.data.gouv.fr/search/"
    params = {"q": commune, "type": "municipality", "limit": 1}

    try:
        response = requests.get(url, params=params)
        data = response.json()

        if data.get("features") and len(data["features"]) > 0:
            properties = data["features"][0].get("properties", {})
            nom = properties.get("name")
            code_postal = properties.get("postcode")

            if nom and code_postal:
                return {"trouvees": True, "resultats": [{"nom": nom, "code_postal": code_postal}]}

        return {"trouvees": False}
    except:
        return {"trouvees": False}

class TestRechercheCommune(unittest.TestCase):
    """Tests unitaires"""

    def test_commune_existante_paris(self):
        """Test : Paris doit retourner 75"""
        resultat = rechercher_commune("Paris")
        self.assertTrue(resultat["trouvees"])
        self.assertEqual(resultat["resultats"][0]["code_postal"][:2], "75")

    def test_commune_existante_lyon(self):
        """Test : Lyon doit retourner 69"""
        resultat = rechercher_commune("Lyon")
        self.assertTrue(resultat["trouvees"])
        self.assertEqual(resultat["resultats"][0]["code_postal"][:2], "69")

    def test_commune_inexistante(self):
        """Test : Commune fake doit retourner False"""
        resultat = rechercher_commune("CommuneFantome123")
        self.assertFalse(resultat["trouvees"])

if __name__ == "__main__":
    unittest.main()
```

**Concepts enseignés :**
- Test-driven development (TDD)
- Assertions
- Classe TestCase
- Couverture de tests

---

## ÉTAPE 6 : DÉPLOIEMENT

Transformer en application web simple avec Flask

```python
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

def rechercher_commune_api(commune):
    """Logique de recherche"""
    url = "https://api-adresse.data.gouv.fr/search/"
    params = {"q": commune, "type": "municipality", "limit": 1}

    try:
        response = requests.get(url, params=params)
        data = response.json()

        # CORRECTION : Vérification sécurisée
        if data.get("features") and len(data["features"]) > 0:
            properties = data["features"][0].get("properties", {})
            name = properties.get("name")
            postcode = properties.get("postcode")

            if name and postcode:
                return {
                    "found": True,
                    "name": name,
                    "postcode": postcode
                }

        return {"found": False}
    except:
        return {"found": False, "error": "API error"}

@app.route('/rechercher', methods=['GET'])
def rechercher():
    """API endpoint"""
    commune = request.args.get('commune')

    if not commune or not commune.strip():
        return jsonify({"error": "Commune requise"}), 400

    resultat = rechercher_commune_api(commune)
    return jsonify(resultat)

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=3000)
```

**URL d'appel :**
```
http://localhost:3000/rechercher?commune=Paris
```

---

## CHECKLIST D'ÉVALUATION

- [ ] Code exécute sans erreur
- [ ] Fonction rechercher_commune() implémentée
- [ ] Gestion erreurs (try/except) complète
- [ ] Pas d'IndexError sur liste vide
- [ ] Boucle interactive fonctionnelle
- [ ] API externe utilisée (Version 2+)
- [ ] Affichage résultats clair
- [ ] Tests unitaires écrits (Version 4+)
- [ ] Documentation dans le code
- [ ] Git commits significatifs

---

## ERREURS COMMUNES À ÉVITER

### ERREUR 1 : IndexError sur liste vide
```python
# MAUVAIS - Provoque IndexError si pas de résultats
if data["features"]:
    commune_data = data["features"][0]  # DANGER!

# BON - Vérifier la longueur
if data.get("features") and len(data["features"]) > 0:
    commune_data = data["features"][0]  # OK
```

### ERREUR 2 : KeyError sur clés manquantes
```python
# MAUVAIS
name = properties["name"]  # Peut crasher!

# BON - Utiliser .get()
name = properties.get("name")  # Retourne None si absent
```

### ERREUR 3 : Pas valider les données
```python
# MAUVAIS
if data["features"]:
    return data["features"][0]  # Peut avoir des données vides

# BON
if nom and code_postal:
    return {"nom": nom, "code_postal": code_postal}
```

---

## RESSOURCES UTILES

- Requests library : https://requests.readthedocs.io/
- API Adresse : https://adresse.data.gouv.fr/api
- Python docs : https://docs.python.org/3/
- CodeSandbox Python : https://codesandbox.io/docs/projects/learn/setting-up-your-project

---

## BONUS CHALLENGES

Pour les étudiants avancés :

1. **Géolocalisation** : Ajouter latitude/longitude
2. **Interface GUI** : Utiliser Tkinter pour interface graphique
3. **Cache** : Stocker résultats en fichier local
4. **Statistiques** : Compter requêtes par commune
5. **Export** : Générer fichier CSV des résultats
6. **Multi-langues** : Supporter anglais + français
7. **Autocomplete** : Suggestions pendant la saisie

---

## RÉSUMÉ

Ce tutoriel progresse de simple (dictionnaire) à professionnel (tests + API + web).

DURÉE TOTALE : 1-2 heures pour version complète.

TECHNOLOGIES : Python, Requêtes HTTP, APIs, JSON, Tests, Git.

CORRECTIONS APPORTÉES : Gestion robuste des erreurs, validation données, pas d'IndexError.

UTILITÉ RÉELLE : Les étudiants créent quelque chose d'utile et déployable.

