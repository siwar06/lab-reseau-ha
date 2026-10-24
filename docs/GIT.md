# Publier sur GitHub

```bash
cd lab-reseau-ha
git init
git add .
git commit -m "Lab réseau HA : documentation, configurations et preuves"
git branch -M main
git remote add origin https://github.com/<ton-compte>/lab-reseau-ha.git
git push -u origin main
```

## Avant le premier push
1. Mettre les captures dans `images/` avec les noms indiqués.
2. Mettre les sorties dans `outputs/`.
3. Relire `configs/` : aucune ligne `secret`, `password`, `community`.
4. `git status` : vérifier qu'aucun `config.xml` n'est ajouté.
