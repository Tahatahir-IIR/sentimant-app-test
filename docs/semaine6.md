# Semaine 6 : le paquet, notes du binôme

> Trois questions, quelques phrases chacune. Répondez avec ce que vous avez observé.

## 1. Pourquoi le modèle n'est-il pas dans la roue ? Comment le paquet le trouve-t-il ?

La roue construite (`python -m build --wheel`) ne contient que `api`, `sentiment_app`, `ui` et ses métadonnées : ni `tests`, ni `data`, ni `models`, ni `src`. Le modèle a son propre cycle de vie : on le réentraîne (`python src/train.py`) sans republier le code, et il n'est de toute façon jamais versionné dans Git. Le paquet le trouve par `chemin_modele()` dans `sentiment_app/predict.py` : la variable `SENTIMENT_MODELE` si elle est définie, sinon `models/sentiment.joblib` à côté du code. Installée hors du dépôt, la roue cherchait donc `...\roue-tp6\Lib\site-packages\models\sentiment.joblib`, qui n'existe pas, et l'a dit : « modèle introuvable », code de sortie 1.

## 2. Qu'est-ce qui a changé dans api/model.py quand predict() est entré dans le paquet, et pourquoi est-ce mieux ?

Avant, `api/model.py` calculait la racine du dépôt, ajoutait `src/` au chemin de Python avec `sys.path.insert`, puis importait `predict` avec deux `# noqa: E402`. Maintenant, deux lignes suffisent : `from sentiment_app.predict import charger_modele` et `from sentiment_app.predict import predict as _predict`, et `Select-String -Path api\*.py -Pattern "sys.path"` ne trouve plus rien. C'est mieux parce que l'import ne dépend plus de l'endroit où se trouvent les fichiers : une fois le paquet installé, l'API fonctionne aussi hors du dépôt, ruff n'a plus d'exception à tolérer, et mypy suit l'import sans réglage de chemin. Effet de bord observé : tant que l'API importait encore l'ancien `predict`, devenu le fichier de compatibilité, le crochet pre-commit a refusé le commit (mypy en échec), ce qui nous a obligés à faire les étapes B et C ensemble.

## 3. La roue installée hors du dépôt : qu'a-t-il fallu lui fournir, et pourquoi scikit-learn est-il épinglé dans pyproject.toml ?

Dans un environnement neuf (`py -3.11 -m venv roue-tp6`), `pip install` de la roue a installé seul ses dépendances, lues dans ses métadonnées, dont scikit-learn 1.5.2 et FastAPI 0.115.14. Il a fallu lui fournir une seule chose : le chemin du modèle, par `SENTIMENT_MODELE`, après quoi elle a répondu `{"label": "POSITIVE", "score": 0.777, ...}`. scikit-learn est épinglé (`>=1.5,<1.6`) parce qu'un modèle sauvegardé ne se relit qu'avec la version qui l'a produit : nous avons entraîné le même pipeline avec scikit-learn 1.9.1, la version installée sans épinglage, puis l'avons fait lire par la roue, en 1.5.2 ; elle s'est arrêtée sur `AttributeError: 'LogisticRegression' object has no attribute 'multi_class'`. C'est aussi pourquoi la CI entraîne désormais avec `scikit-learn==1.5.*`.
