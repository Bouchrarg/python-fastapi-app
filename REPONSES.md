# TP5 – Construire un workflow CI avec GitHub Actions

**Dépôt :** https://github.com/Bouchrarg/python-fastapi-app (branche de travail `feature/tp-ci`, fusionnée dans `main`)
**Run complet et vert (lint + build → test) :** https://github.com/Bouchrarg/python-fastapi-app/actions/runs/42133274458

Workflows réalisés (`.github/workflows/`) : `01-filtres.yml`, `tag.yml`, `02-ci.yml`, `03-contextes.yml`.

## Étape 0 – Préparation

**Q0.**
- a. Le code est dans `app/` (`main.py`, `config.py`, `logging.py`) et les tests dans `tests/` (`test_endpoints.py`, `test_config.py`, `test_logging.py`).
- b. Les dépendances de développement sont dans `requirements-dev.txt` (pytest, pytest-cov, ruff, mypy, httpx) ; celles de l'application (fastapi, uvicorn, pydantic-settings) dans `requirements.txt`.
- c. Oui : `.github/workflows/ci.yml` existait déjà (« CI - Contextes GitHub Actions », déclenché uniquement sur push vers `main`).
- En local, après `pip install -r requirements.txt -r requirements-dev.txt`, `ruff check .` affiche « All checks passed! » et `pytest` passe.

## Étape 1 – Événements et filtres

**Q1.** `01-filtres.yml` : `push.branches: [main, "feature/**"]`, `push.paths: ["app/**", "tests/**"]`, `pull_request.types: [opened, synchronize]`.
Remarque : une première version avait été créée à la racine du dépôt ; GitHub ne lit que `.github/workflows/`, le fichier a donc été déplacé et corrigé (commit « ci: fix 01 filtres »).

**Q2.** Modifier uniquement `README.md` ne déclenche pas Filtres : aucun fichier modifié ne correspond à `paths`. Modifier un fichier de `app/` (`app/__init__.py`) le déclenche.

**Q3.** Entre `branches` et `paths`, c'est un ET : la branche ET au moins un fichier modifié doivent correspondre. Entre les valeurs d'une même liste et entre deux `types` d'activité (`opened`, `synchronize`), c'est un OU.

**Q4.** `tag.yml` : `on: push: tags: ["v*"]`. Le push du tag (`git tag v1.0.0 && git push --tags`) lance ce workflow, qui affiche `Tag v1.0.0` ; un push de branche ne le déclenche pas.

## Étape 2 – Jobs, steps et needs

**Q5.** `02-ci.yml` : `uses: actions/checkout@v4`, `uses: actions/setup-python@v5` avec `python-version: "3.13"`, `run: pip install -r requirements.txt -r requirements-dev.txt`, et pour test `needs: [lint, build]`.

**Q6.** Sans `needs` (commit « ci: q6 sans needs »), les trois jobs démarrent en parallèle et la durée du run diminue (elle correspond à celle du job le plus long). Avec `needs` rétabli, test attend la fin de lint et de build. Avec un `import os` inutilisé dans `app/main.py` (commit « test: q6 erreur lint »), lint échoue (F401), build réussit et test est ignoré (*skipped*).

**Q7.** Chaque job s'exécute sur un runner neuf et isolé (une nouvelle VM) : rien n'est partagé entre jobs, il faut donc refaire `checkout` et `setup-python` dans test.

## Étape 3 – Contextes

**Q8.** `03-contextes.yml`, step « Afficher les contextes » :
- application : `${{ env.APP_NAME }}` (contexte `env`), soit `fastapi-starter` ;
- acteur : `${{ github.actor }}` ;
- événement : `${{ github.event_name }}` ;
- branche : `${{ github.ref_name }}` ;
- SHA du commit : `${{ github.sha }}` (contexte `github`) ;
- OS du runner : `${{ runner.os }}` (contexte `runner`), soit `Linux`.

**Q9.** Sur un push : `event_name = push`, `ref_name = feature/tp-ci`, `head_ref` et `base_ref` sont vides. Sur la PR vers `main` : `event_name = pull_request`, `head_ref = feature/tp-ci` (branche source), `base_ref = main` (branche cible), et `ref_name` devient `<numéro>/merge`.

**Q10.** Step « Dump github (Q10) » : `toJson(github)` est passé par une variable d'environnement (`GH_CTX`), pour éviter que l'`echo` casse si le message contient une apostrophe. Le message du commit se trouve dans `event.head_commit.message`, soit `"ci: 03 contextes"`. Ce champ n'existe que sur un événement push.
