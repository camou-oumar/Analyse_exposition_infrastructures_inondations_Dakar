# Exposition des infrastructures aux inondations dans la région de Dakar

Analyse géospatiale qui croise les **infrastructures publiques** avec les **zones inondables** afin d'estimer combien d'équipements (écoles, lieux de culte, marchés, services de santé...) se trouvent en zone à risque.

Le travail est réalisé dans un notebook Jupyter (`gpd.ipynb`) à partir de données stockées dans **PostgreSQL / PostGIS**.

## Contenu du dépôt

| Fichier | Description |
|---|---|
| `gpd.ipynb` | Notebook d'analyse, commenté et structuré en sections |
| `README.md` | Ce document |

## Données

Base PostGIS : `inondk` (CRS **EPSG:32628**, WGS 84 / UTM 28N, en mètres).

**`infrastructure_publiq`** (1 611 lignes, 1 586 après nettoyage), géométries de type *Point*

| Colonne | Description |
|---|---|
| `id_inf` | Identifiant de l'infrastructure |
| `nom_infra` | Nom (25 valeurs manquantes, supprimées) |
| `id_zone` | Identifiant de la zone associée |
| `geom` | Point (position) |

**`zone_inondable`**, géométries de type *Polygone*

| Colonne | Description |
|---|---|
| `id_zone` | Identifiant de la zone |
| `nom_quarti` | Nom du quartier |
| `commune` | Commune |
| `niveau_risque` | `faible`, `modere`, `eleve` ou `tres eleve` |
| `geom` | Polygone |

## Méthode

1. **Chargement** des deux tables avec GeoPandas (`read_postgis`).
2. **Nettoyage** : suppression des valeurs manquantes, contrôle des doublons et des types de géométrie.
3. **Exploration** : répartition par niveau de risque et par commune, liste des quartiers à risque très élevé, surfaces en hectares.
4. **Cartographie** : carte statique (matplotlib) et cartes interactives (folium via `.explore()`).
5. **Zone tampon** de 100 m autour des zones à risque très élevé, et « anneau » périphérique.
6. **Jointure spatiale** (`sjoin`, prédicat `within`) pour trouver les infrastructures situées dans une zone.
7. **Classification par type** (école, santé, culte, marché, administration, eau/énergie) à partir du nom, par expressions régulières, puis calcul du **taux d'exposition** par type.

## Résultats principaux

- 380 zones sur 1 475 sont classées « tres eleve » (~11 579 ha au total).
- **501 infrastructures sur 1 586 (31,6 %)** sont situées en zone à risque très élevé.

| Type | Total | Exposées | Taux (%) |
|---|---:|---:|---:|
| Administration | 17 | 7 | 41,2 |
| Autre | 945 | 319 | 33,8 |
| École | 128 | 40 | 31,2 |
| Eau/Énergie | 62 | 19 | 30,6 |
| Culte | 352 | 97 | 27,6 |
| Marché | 45 | 12 | 26,7 |
| Santé | 37 | 7 | 18,9 |

## Prérequis

- Python 3.10+
- PostgreSQL avec l'extension PostGIS, contenant la base `inondk`
- Bibliothèques :

```bash
pip install geopandas sqlalchemy psycopg2-binary matplotlib folium mapclassify jupyter
```

## Utilisation

1. Définir le mot de passe PostgreSQL dans une variable d'environnement (il n'est **pas** écrit dans le notebook) :

   ```powershell
   # Windows (PowerShell)
   $env:PGPASSWORD="votre_mot_de_passe"
   # windows (cmd)
   set PGPASSWORD=votre_mot_de_passe
   ```
   ```bash
   # Linux / macOS
   export PGPASSWORD="votre_mot_de_passe"
   ```

2. Adapter si besoin `host`, `port`, `username` et `database` dans la cellule de connexion.
3. Lancer Jupyter depuis le même terminal, puis exécuter les cellules dans l'ordre :

   ```bash
   jupyter notebook gpd.ipynb
   ```

## Limites et pistes d'amélioration

- **Niveau de risque :** à partir de la section 6, `gdf4` est filtré sur « tres eleve ». Les taux d'exposition ne concernent donc que ce niveau. Pour comparer tous les niveaux, utiliser une copie du GeoDataFrame au lieu de l'écraser.
- **Classification :** 945 infrastructures restent en « Autre ». Les règles gagneraient à couvrir pharmacies, cimetières, garages, etc., et à tolérer les fautes de frappe (ex. « Ecoel »).
- **Faux positif :** le mot-clé `station` classe les stations d'essence dans « Eau/Énergie ».
- **Nettoyage :** les lignes incomplètes sont supprimées (`dropna`), ce qui retire 25 infrastructures et 15 zones.
- **Sécurité :** ne jamais écrire d'identifiants en clair dans un notebook partagé. Si le mot de passe d'origine a déjà été diffusé, il est conseillé de le changer.
