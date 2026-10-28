# Journal des versions

Le numéro de version suit les TP : 0.N.0 à la fin du TP N. Chaque version publiée porte une étiquette Git `v0.N.0`.

## 0.6.0 (TP6) : le paquet

- **Pour celui qui installe** : une roue, `sentiment_app-0.6.0-py3-none-any.whl`, qui s'installe avec `pip` et installe seule ses dépendances (FastAPI, uvicorn, Pydantic, scikit-learn 1.5, joblib) ; l'interface est un extra, `sentiment-app[ui]`. Elle fournit la commande `sentiment` : `--version`, `predict`, `serve`, `ui`. Le modèle n'est pas dans la roue : la variable `SENTIMENT_MODELE` indique où il est, sinon la commande s'arrête sur « modèle introuvable », code de sortie 1.
- **Pour celui qui développe** : `predict.py` est déplacé dans le paquet `sentiment_app` (par `git mv`, son historique suit) ; `src/predict.py` reste comme fichier de compatibilité, si bien que `python src/predict.py` et `import predict` marchent toujours. `api/model.py` importe `sentiment_app.predict` et ne modifie plus `sys.path`. `pip install -e .` installe le paquet en mode développement. La version est écrite une seule fois, dans `sentiment_app/__init__.py`.
- **Pour la CI** : scikit-learn est épinglé en 1.5, comme dans le paquet ; une nouvelle étape construit la roue, l'installe dans un environnement neuf hors du dépôt et y fait une prédiction. 104 passed, 1 xfailed.

## 0.5.0 (TP5) : l'interface

- Page Streamlit : un avis, ou un fichier d'avis ; le client HTTP traduit chaque réponse de l'API.

## 0.4.0 (TP4) : tests et qualité

- Contrat d'entrée testé, abstention `INDETERMINE`, jeu d'or, tests d'intégration, contrat typé, ruff, mypy, pre-commit.

## 0.3.0 (TP3) : l'API

- FastAPI : `/health`, `/predict`, contrat Pydantic ; la CI entraîne le modèle avant les tests.

## 0.2.0 (TP2) : le modèle

- TF-IDF et régression logistique ; les métriques sont versionnées, le modèle ne l'est pas.

## 0.1.0 (TP1) : le cadrage

- Environnement Python 3.11, structure du dépôt, trois fiches de cadrage, dépôt GitHub à deux auteurs.
