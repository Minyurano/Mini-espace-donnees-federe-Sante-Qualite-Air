# 🏥🌫️ Mini-espace de données fédéré : Santé et qualité de l'air

Travaux pratiques **Data Spaces & Data Integration** (Master 2 IASD, FST Tanger, Université Abdelmalek Essaâdi, 2026-2027).

![Python](https://img.shields.io/badge/Python-3.14-blue) ![FastAPI](https://img.shields.io/badge/FastAPI-API%20REST-009688) ![SQLite](https://img.shields.io/badge/SQLite-local-lightgrey) ![Licence](https://img.shields.io/badge/Licence-MIT-green)

**Question centrale :** peut-on étudier le lien entre la pollution de l'air à Tanger (PM2.5) et les admissions hospitalières respiratoires **sans transférer les données individuelles de l'hôpital** au laboratoire ?

Trois acteurs :

| Acteur | Rôle | Données |
|---|---|---|
| **A** : Environnement | Classe, publie le catalogue, autorise et signe | `environnement.db` (PM2.5) |
| **B** : Hôpital | Calcule localement des agrégats | `hopital.db` (admissions) |
| **C** : Laboratoire | Découvre, négocie, interroge, compare | aucune base |

> Les données sont **fictives**. Le seuil « pollution élevée » (PM2.5 > 30 µg/m³) est un seuil pédagogique, pas une recommandation sanitaire.

## Progression pédagogique

| TP | Dossier | Contenu | Constat |
|---|---|---|---|
| 1 | `tp1_stockage_local/` | Deux bases SQLite indépendantes | Données séparées, mais aucune collaboration possible |
| 2 | `tp2_api_rest/` | API REST (FastAPI) et client | Interopérabilité, mais **données brutes transférées** au labo |
| 3 | `tp3_controle_acces/` | Clés API, catégories, calcul local chez B | **Attaque par différence** : un groupe d'un seul jour révèle la valeur exacte (8 admissions) |
| 4 | `tp4_gouvernance/` | Catalogue (DCAT), contrat et finalité, HMAC, seuil de 3 jours, bruit de Laplace | Falsification rejetée (403), seuls des agrégats circulent |

## Résultats clés (TP 4, 10 journées)

- Pollution au-dessus du seuil : n = 5, moyenne = **16,2** admissions/jour
- Pollution sous le seuil : n = 5, moyenne = **7,8** admissions/jour
- Écart : **8,4** admissions/jour (sans bruit)
- Avec bruit de Laplace : résultats variables d'une exécution à l'autre (l'écart peut même changer de signe sur de petits groupes)

Contrôles testés : finalité `commercial` → 403, jeu inexistant → 404, clé erronée → 401, classifications falsifiées → 403.

## Installation (Windows PowerShell)

```powershell
git clone https://github.com/Minyurano/Mini-espace-donnees-federe-Sante-Qualite-Air.git
cd Mini-espace-donnees-federe-Sante-Qualite-Air
python -m venv .venv
Set-ExecutionPolicy -Scope Process RemoteSigned
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

## Exécution

### TP 1 : bases locales
```powershell
cd tp1_stockage_local
python tp_federation.py
python consulter_bases.py
```
Copiez ensuite `environnement.db` et `hopital.db` dans le dossier du TP 2 ou 3 avant de lancer les API.

### TP 2 et TP 3 : API (deux terminaux) + client
```powershell
python -m uvicorn api_air:app --reload --port 8001      # TP 2 (api_air_v3 pour le TP 3)
python -m uvicorn api_hopital:app --reload --port 8002  # TP 2 (api_hopital_v3 pour le TP 3)
python client_analyse.py                                # TP 2 (laboratoire_v3.py puis attaque_difference.py pour le TP 3)
```

### TP 4 : gouvernance
```powershell
cd tp4_gouvernance
python setup_data.py
python -m uvicorn api_air:app --port 8001        # terminal 1
python -m uvicorn api_hopital:app --port 8002    # terminal 2
python client_laboratoire.py                     # terminal 3
python client_laboratoire.py --bruit             # avec bruit de Laplace
```
Pas de `--reload` au TP 4 : les contrats sont conservés en mémoire.
Documentation interactive : <http://127.0.0.1:8001/docs> et <http://127.0.0.1:8002/docs>.

## Limites

- Le seuil de 3 jours n'est **pas** du k-anonymat ; HMAC n'est **pas** du chiffrement ; le bruit de Laplace du prototype n'offre **pas** de garantie formelle de confidentialité différentielle.
- Secrets et clés de démonstration en clair : à ne jamais utiliser en production.
- Dix journées fictives : aucune conclusion causale ou médicale possible.
- Production : HTTPS, identité organisationnelle robuste, secrets protégés, contrats persistants, audit.

## Rapport

Le rapport complet (captures, essais, questions de soutenance) est dans [`docs/RAPPORT_DE_REFERENCE_.pdf`](docs/RAPPORT_DE_REFERENCE_.pdf).

## Auteurs

- **Aloys MINYURANO** : [@Minyurano](https://github.com/Minyurano)
- **Yahfdhou Mohamed LEHOUEIMEL**

Distribué sous licence MIT.