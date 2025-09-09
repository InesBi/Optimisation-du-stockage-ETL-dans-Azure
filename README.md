Une **procédure opérationnelle complète (checklist pas à pas)** pour mettre en place ce module d’optimisation du stockage dans **Azure**. Vous pourrez l’utiliser directement comme guide pratique.


##  Étape 1 : Vérifier ou créer le compte de stockage
1. Connectez-vous au **[Azure Portal](https://portal.azure.com/)**.
2. Dans la barre de recherche, tapez **Storage Accounts**.
3. Vérifiez si un compte de stockage est déjà disponible pour l’ETL.

   * Si **oui** → passer à l’étape suivante.
   * Si **non** → cliquez sur **+ Create**, configurez :

     * Region : même que vos services ETL
     * Performance : Standard (suffisant pour logs ETL)
     * Redundancy : LRS (moins cher) ou GRS (si besoin haute disponibilité)

##  Étape 2 : Configurer un conteneur Blob pour les données ETL

1. Dans votre **Storage Account**, allez dans **Containers**.
2. Cliquez sur **+ Container** et créez par exemple :

   * Nom : `etl-data`
   * Public access : Private (important pour sécurité)

##  Étape 3 : Activer la gestion du cycle de vie

1. Dans le menu de gauche du compte de stockage → **Lifecycle Management**.
2. Cliquez sur **+ Add rule**.
3. Configurez les conditions :

   * **Scope** : Apply rule to all blobs (ou prefix `etl-data/`).
   * **Actions** :

     * Move to **Cool** after 7 days
     * Move to **Archive** after 30 days
     * Delete after 365 days (ou délai adapté à vos besoins)
4. Validez → La règle est activée automatiquement.

Alternative via **Azure CLI** :

```bash
az storage account management-policy create \
  --account-name MyStorageAccount \
  --resource-group MyResourceGroup \
  --policy @policy.json
```

##  Étape 4 : Intégrer la compression dans l’ETL

Dans votre script ETL (Python ou autre langage) :

* Après récupération/transformation → **sauvegarder en Parquet ou CSV compressé (gzip)**.

Exemple Python :

```python
import pandas as pd

# Charger données
df = pd.read_csv("data.csv")

# Export compressé
df.to_csv("data.csv.gz", index=False, compression="gzip")

# OU en Parquet (format optimisé)
df.to_parquet("data.parquet", compression="snappy")
```

##  Étape 5 : Vérifier la politique de rétention

1. Dans **Lifecycle Management**, vérifier que la suppression automatique est bien activée.
2. Exemple : suppression des blobs après 365 jours → pas de stockage inutile.
3. Si conformité légale → adapter la durée (ex. 3 ans).

##  Étape 6 : Tests & validation

1. Charger quelques fichiers CSV compressés dans `etl-data/`.
2. Vérifier dans **Blob Explorer** qu’ils apparaissent correctement.
3. Contrôler dans **Lifecycle Management** → onglet "Simulation" pour voir à quelle date chaque fichier changera de tier ou sera supprimé.
4. Documenter la configuration (screenshots + description des règles).

#  Résultats attendus

* Les fichiers récents (moins de 7 jours) → **Hot tier** (accès rapide).
* Après 7 jours → **Cool tier** (moins cher).
* Après 30 jours → **Archive tier** (très faible coût).
* Après 365 jours → **Suppression automatique**.
* Stockage global réduit grâce à la **compression CSV/Parquet**.
