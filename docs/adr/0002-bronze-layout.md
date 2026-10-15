# 0002. Layout de la couche Bronze (Stockage et Partitionnement)

* Date : 2026-10-15
* Statut : Accepté

## Contexte
Dans le cadre de la construction de la plateforme de données pour *Albert's Marketplace*, nous devons stocker les données brutes d'origine (commandes, lignes de commandes) dans la couche Bronze du Data Lakehouse sans altération[cite: 1]. Le système doit absorber les flux d'ingestion tout en garantissant un coût de stockage maîtrisé et une performance de lecture optimale pour les couches ultérieures (Silver et Gold)[cite: 1].

## Décision
Nous implémentons la couche Bronze sur Google Cloud Storage (`europe-west1`) avec ces spécifications[cite: 1] :
1. **Format de fichier :** Apache Parquet[cite: 1].
2. **Codec de compression :** Snappy[cite: 1].
3. **Clé de partitionnement :** Partitionnement à la Hive basé uniquement sur la date (`dt=YYYY-MM-DD`)[cite: 1].

## Options examinées
* **Option 1 : Fichiers CSV d'origine**
  * *Pour :* Aucun traitement à l'ingestion[cite: 1].
  * *Contre :* Format non typé sans métadonnées, obligeant le moteur à scanner l'intégralité du fichier pour lire une seule colonne, générant des coûts et des temps de lecture élevés[cite: 1].
* **Option 2 : Sur-partitionnement (`dt` + `p_id`)**
  * *Pour :* Permet de cibler très précisément certains produits[cite: 1].
  * *Contre :* Génère une explosion du nombre de petits fichiers (plus de 8 000 objets pour un seul mois), faisant exploser les coûts d'écriture et dégradant les performances[cite: 1].

## Justification (Basée sur nos mesures)
* **Compression :** La conversion du CSV (~10,4 Mo) vers le format Parquet (~3,1 Mo) nous a permis d'obtenir un ratio de compression d'environ **3,35x**[cite: 1, 4].
* **Partitionnement :** Nos tests ont montré qu'un partitionnement journalier (`month_by_day`) garantit des requêtes SQL rapides (**490 ms** pour traiter 17 Ko) avec un nombre de fichiers maîtrisé (**31 fichiers** pour un mois)[cite: 1, 4]. En revanche, le sur-partitionnement génère **8 649 fichiers** dont la taille totale explose à 32,6 Mo en raison du surcoût des métadonnées Parquet[cite: 1, 4]. Ce sur-partitionnement rend le coût des opérations d'écriture **68 fois plus élevé** que le coût de stockage mensuel[cite: 1, 4].

## Conséquences
* L'élagage de partitions (*partition pruning*) sera extrêmement performant sur les requêtes filtrant par date[cite: 1].
* Les requêtes nécessitant d'isoler un produit spécifique sans utiliser la date `dt` devront scanner l'ensemble des données de la période[cite: 1].