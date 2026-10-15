# Dictionnaire de Données (Couche Bronze)

Ce document décrit les données brutes ingérées dans la couche Bronze. Conformément à notre architecture, aucune colonne n'est supprimée à cette étape pour préserver l'historique brut.

## Table : `orders` (Commandes et Clients)
Contient les informations relatives à la transaction globale et au compte client.

| Nom de la colonne | Description | Tag Sensibilité |
| :--- | :--- | :--- |
| `order_id` | Identifiant unique de la commande | |
| `order_date` | Date et heure de la commande | |
| `customer_name` | Nom et prénom du client | **⚠️ PII (Donnée personnelle)** |
| `customer_address`| Adresse physique de livraison | **⚠️ PII (Donnée personnelle)** |
| `credit_card` | Numéro de la carte bancaire utilisée | **🚨 SÉCURITÉ / PII** |
| `password` | Mot de passe du compte client | **🚨 SÉCURITÉ / PII** |

## Table : `order_items` (Lignes de commandes)
Contient le détail des produits achetés pour chaque commande.

| Nom de la colonne | Description | Tag Sensibilité |
| :--- | :--- | :--- |
| `order_id` | Identifiant de la commande (clé étrangère) | |
| `p_id` | Identifiant unique du produit | |
| `qty` | Quantité d'unités achetées | |
| `price_at_purchase`| Prix unitaire du produit au moment de l'achat| |
| `dt` | Clé de partitionnement (dérivée de order_date)| |