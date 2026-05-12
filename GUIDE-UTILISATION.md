# Guide d'utilisation — Suivi des Incidents Lucie

---

## Connexion

1. Ouvrir l'application sur votre navigateur ou mobile
2. Sélectionner votre nom dans la liste déroulante
3. Saisir votre code PIN (4 chiffres)
4. Cliquer sur **Connexion**

> Le compte Administrateur dispose d'un accès séparé via l'onglet "Administrateur".

---

## Créer un incident

1. Cliquer sur le bouton **+ Nouvel Incident** (en haut à droite)
2. Remplir le formulaire :
   - **Date / Heure** : pré-remplies automatiquement, modifiables si besoin
   - **Boutique** : sélectionner la boutique concernée
   - **Type d'incident** : choisir dans la liste
   - **Description** : décrire ce qui s'est passé (minimum 3 caractères)
   - **Gravité** : détectée automatiquement par IA après saisie de la description
3. Pour une **réclamation client** : renseigner prénom, nom et contact du client
4. Pour un **vol ou fraude** : indiquer l'estimation du préjudice
5. Cliquer sur **Enregistrer l'incident**

> La gravité (Faible / Moyen / Élevé) est analysée automatiquement. Elle peut prendre 2-3 secondes à apparaître.

---

## Statuts des incidents

| Statut | Signification |
|--------|--------------|
| **Non ouvert** | Incident enregistré, pas encore pris en charge |
| **À traiter** | Incident pris en compte, action en cours |
| **En cours** | Traitement en cours |
| **Résolu** | Incident clôturé (admin uniquement) |
| **Auto-résolu** | Clôturé par l'agent lui-même |

Pour changer le statut : cliquer sur **Ouvrir** → modifier le statut dans la fiche.

---

## Filtres et recherche

La barre de filtres permet de :
- **Rechercher** par mot-clé (description, boutique, type…)
- **Filtrer par période** : aujourd'hui, semaine, mois
- **Filtrer par boutique**, shift, type d'incident
- **Filtrer par date** : sélectionner une plage personnalisée

---

## Envoyer une facture par email

1. Cliquer sur le bouton **📧 Facture** dans le header
2. Renseigner l'email du destinataire
3. Sélectionner la boutique
4. Glisser-déposer le fichier PDF de la facture (ou cliquer pour parcourir)
5. Cliquer sur **Envoyer**

Le destinataire reçoit un email avec un lien de téléchargement valable **7 jours**.

---

## Réclamation client → Zendesk

Lorsqu'un incident de type **"Réclamation client"** est enregistré, un ticket est créé **automatiquement** dans Zendesk avec toutes les informations du client. Aucune action supplémentaire n'est nécessaire.

---

## Pour les Administrateurs

### Dashboard
Accessible via l'onglet **📊 Dashboard** après connexion admin.

- **Sélecteur de période** : 7j / 30j / ce mois / mois dernier / tout
- **KPIs** : total incidents, gravité élevée, taux de résolution, en attente
- **Camembert boutiques** : cliquer sur une boutique pour voir son détail
- **Évolution** : graphique adapté à la période
- **Stats par shift, par type, par agent**
- **Bouton "Rapport PDF"** : exporte un rapport complet de la période

### Gestion via le menu ⚙️ Admin
- **Export PDF / CSV** : exporter les incidents filtrés
- **Archives** : consulter les incidents de plus de 2 mois
- **Utilisateurs** : créer, modifier, supprimer des comptes agents
- **Boutiques** : gérer la liste des boutiques

### Créer un nouvel agent
1. Menu ⚙️ Admin → **Utilisateurs**
2. Cliquer **+ Ajouter un utilisateur**
3. Renseigner prénom, nom, email interne, PIN
4. Valider

### Supprimer un incident
Dans la liste des incidents, cliquer sur **✖** à droite de la ligne.
> Uniquement disponible pour les administrateurs.

---

## Conseils pratiques

- **Mobile** : l'application est optimisée pour iPhone et Android
- **Mode sombre** : bouton 🌙 en haut à droite
- **Cloche 🔔** : affiche l'activité récente de l'équipe
- En cas de problème de connexion : vérifier le PIN avec votre administrateur
