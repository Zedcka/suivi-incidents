# Rapport d'Audit — Suivi des Incidents Lucie
*Audit réalisé le 2026-05-01*  
*Fichier audité : `index.html` (3161 → ~3330 lignes après corrections)*

---

## Résumé exécutif

L'application est fonctionnellement solide, bien structurée et couvre les besoins métier.  
L'audit a identifié **3 bugs réels**, **5 problèmes de sécurité/XSS**, **4 manques UX critiques**, **1 problème de performance**, et plusieurs points RGPD à adresser.  
**12 corrections** ont été appliquées et poussées sur GitHub (commit `7dba993`).

---

## 1. Ce qui a été corrigé

### 1.1 Bugs

| # | Problème | Ligne(s) avant | Correction |
|---|----------|---------------|------------|
| B1 | `saveEdit()` ne stockait pas `rawDate` dans `lastModified`, rendant les notifications de modification invisibles pour les utilisateurs | ~1732 | Ajout de `rawDate: new Date().toISOString()` dans `saveEdit` |
| B2 | `pdfRow()` comparait `i.gravite === 'Eleve'` (sans accent) alors que la valeur réelle est `'Élevé'` → les incidents élevés s'affichaient en vert dans le PDF | ~2679 | Corrigé en `i.gravite === 'Élevé'` |
| B3 | `loadBoutiques()` tentait d'insérer les boutiques par défaut même en cas d'erreur réseau (pas seulement si la table est vide), risquant des doublons ou des erreurs en boucle | ~1949 | Séparation des deux cas : erreur réseau vs table réellement vide |

### 1.2 Sécurité & XSS

| # | Problème | Correction |
|---|----------|------------|
| S1 | `inc.description` inséré brut via `innerHTML` dans le modal détail → XSS si un utilisateur saisit du HTML/JS | Escapé via `escHtml()` avec `white-space:pre-wrap` pour conserver les retours à la ligne |
| S2 | Notes de suivi (`e.date`, `e.auteur`, `e.texte`) insérées brutes → XSS | Escapées via `escHtml()` |
| S3 | Tous les champs texte libres de la table principale (`boutique`, `problemeType`, `gravite`, `personne`, `description`, `prejudice`, `shift`) insérés bruts via template literals | Ajout de la fonction globale `esc()` et application systématique |
| S4 | Mot de passe admin en une ligne anonyme difficile à repérer lors d'une revue | Isolé dans la constante `ADMIN_PASSWORD` avec un bloc de commentaires d'avertissement documentant le risque et la solution recommandée |

### 1.3 UX Utilisateur

| # | Problème | Correction |
|---|----------|------------|
| U1 | Suppression d'un incident sans aucune confirmation | `confirm()` avec nom de l'incident avant suppression |
| U2 | Purge des archives sans confirmation | `confirm()` avant purge 6 mois / 1 an |
| U3 | Suppression d'utilisateur sans confirmation | `confirm()` précisant que les incidents sont conservés |
| U4 | Suppression de boutique sans confirmation | `confirm()` précisant l'impact sur les incidents existants |
| U5 | Aucun retour visuel après soumission réussie d'un incident | Toast vert `✅ Incident enregistré avec succès.` pendant 3,5s |
| U6 | Aucun retour visuel après ajout de boutique / utilisateur | Toast vert correspondant |
| U7 | Aucun retour visuel après envoi de facture | Toast vert avec l'email destination |
| U8 | Message d'erreur gravité vide peu explicatif (`"Veuillez sélectionner un niveau de gravité"`) | Message amélioré expliquant la nécessité de saisir une description pour déclencher l'auto-détection |

### 1.4 UX Admin — Export CSV

| # | Ajout | Détail |
|---|-------|--------|
| A1 | Export CSV ajouté dans le modal d'export | Téléchargement direct, séparateur `;`, BOM UTF-8 pour compatibilité Excel, filtres actifs appliqués |

### 1.5 Performance

| # | Problème | Correction |
|---|----------|------------|
| P1 | `setInterval(loadIncidents, 5000)` : polling toutes les 5 secondes en plus du realtime Supabase → 12 requêtes par minute par utilisateur inutiles | Suppression du `setInterval`. Le canal realtime Supabase (`postgres_changes`) gère seul la synchronisation. |

### 1.6 Responsive mobile

| # | Ajout |
|---|-------|
| R1 | Media query `@media (max-width: 768px)` ajoutée couvrant : header, header-buttons, container padding, filters-card, stats-panels, modal padding, form-row, detail-grid, incidents-table, login-card |

---

## 2. Ce qui est suggéré mais non implémenté

### 2.1 Sécurité — Mot de passe admin (CRITIQUE)

**Problème :** `ADMIN_PASSWORD = 'Lucie@@2026!!!'` est en clair dans un repo GitHub public. N'importe qui peut lire ce mot de passe et se connecter en tant qu'administrateur.

**Pourquoi non implémenté :** Nécessite un changement d'architecture backend (Supabase RPC ou Supabase Auth) et une coordination avec la configuration Supabase.

**Solutions recommandées par ordre de priorité :**

1. **Solution court terme (sans refactoring) :** Créer une fonction Supabase RPC `check_admin_password(pwd text)` qui compare avec `crypt(pwd, gen_salt('bf'))` stocké en base. Le mot de passe ne transite jamais en clair dans le code JS.

2. **Solution idéale :** Migrer l'admin vers Supabase Auth avec un compte email/mot de passe dédié. Le login Supabase Auth est côté serveur, rien n'est exposé dans le code.

3. **Atténuation immédiate possible :** Rendre le repo GitHub privé jusqu'à résolution, et changer le mot de passe régulièrement.

### 2.2 Sécurité — Clé anon Supabase publique

**Problème :** `SUPABASE_ANON_KEY` est visible dans le code. C'est acceptable si les Row Level Security (RLS) sont correctement configurées, mais à vérifier.

**Action requise :** S'assurer que les politiques RLS de Supabase sur les tables `incidents`, `boutiques`, `users` restreignent bien :
- Lecture publique limitée au strict nécessaire
- Écriture uniquement via des fonctions RPC authentifiées ou avec des vérifications d'identité

### 2.3 RGPD — Factures dans Supabase Storage

**Problème :** Les factures uploadées dans le bucket `Factures` ne sont jamais supprimées. Elles contiennent potentiellement des données personnelles (nom du client, adresse, montant).

**Obligation RGPD :** Les données doivent être supprimées dès qu'elles ne sont plus nécessaires à la finalité pour laquelle elles ont été collectées.

**Solutions suggérées :**
1. Créer un Edge Function Supabase qui s'exécute quotidiennement (via pg_cron) et supprime les fichiers de plus de 30 jours
2. Ou afficher une bannière dans l'interface rappelant que les factures doivent être purgées manuellement
3. À minima, documenter dans la politique de confidentialité la durée de conservation

**Impact si non traité :** Risque de non-conformité RGPD (CNIL), et accumulation de fichiers en Storage (coût Supabase).

### 2.4 RGPD — Descriptions d'incidents avec données personnelles

**Problème :** Les descriptions des incidents "Vol" / "Tentative de vol" peuvent contenir des descriptions physiques de suspects (clients). Ces données constituent des données personnelles sensibles selon le RGPD.

**Recommandations :**
- Ajouter une mention légale à l'écran de connexion précisant la finalité du traitement
- Limiter les accès : seuls les administrateurs et l'auteur de l'incident peuvent voir la description complète (déjà partiellement implémenté)
- Définir une durée de conservation maximale (l'auto-archivage à 2 mois puis purge manuelle est une approche correcte, mais la durée devrait être documentée)
- Ne pas exporter des données nominatives dans des PDF sans nécessité

### 2.5 UX Admin — Tableau de bord statistiques visuel

**Problème :** L'admin n'a pas de vue synthétique avec graphiques comme les utilisateurs en ont une version.

**Ce qui existe déjà :** Les stats par statut (chips cliquables), l'export PDF "Rapport statistiques".

**Ce qui manquerait pour un vrai dashboard :**
- Graphique d'évolution temporelle (incidents par semaine/mois)
- Carte thermique boutique × gravité
- Alertes automatiques : boutiques avec plus de N incidents élevés non résolus

**Raison non implémenté :** Nécessite une bibliothèque de graphiques (Chart.js ou similaire) et un design détaillé à valider avec le client.

### 2.6 UX — Recherche texte libre dans les incidents

**Problème :** Aucun champ de recherche full-text. L'utilisateur doit filtrer par boutique/type/période mais ne peut pas chercher "Wilson" dans la description.

**Solution simple :** Ajouter un `<input type="search">` dans la barre de filtres qui filtre en JS sur `boutique + problemeType + description + personne`.

**Raison non implémenté :** Petit risque de régression sur les performances si le nombre d'incidents devient très grand (>10 000). Préférable de valider l'approche avant d'implémenter.

### 2.7 Performance — ID basé sur `Date.now()`

**Problème :** `id: Date.now().toString()` pour les nouveaux incidents. Si deux utilisateurs soumettent simultanément au même milliseconde (improbable mais possible), il y aura une collision.

**Solution :** Utiliser `crypto.randomUUID()` disponible nativement dans tous les navigateurs modernes.

```js
id: crypto.randomUUID()
```

**Impact :** Faible, mais à corriger lors d'une prochaine itération.

### 2.8 Accessibilité (a11y)

Les boutons sans label texte (ex : bouton `✖` de fermeture des modals) devraient avoir un `aria-label`. Les icônes emoji utilisées comme contenu informatif devraient avoir `aria-hidden="true"`.

---

## 3. État de sécurité après corrections

| Risque | Niveau avant | Niveau après | Notes |
|--------|-------------|-------------|-------|
| Mot de passe admin en clair | CRITIQUE | ÉLEVÉ | Documenté, isolé en constante. Risque reste présent tant que repo public. |
| XSS via description / suivi | ÉLEVÉ | RÉSOLU | Échappement HTML systématique |
| XSS via table d'incidents | ÉLEVÉ | RÉSOLU | Fonction `esc()` appliquée |
| Clé anon Supabase publique | MOYEN | MOYEN | Acceptable si RLS correctes (à vérifier) |
| Suppression sans confirmation | MOYEN | RÉSOLU | Confirmations `confirm()` partout |

---

## 4. Points RGPD

| Point | Statut | Action recommandée |
|-------|--------|-------------------|
| Factures non supprimées | NON CONFORME | Automatiser la suppression après 30 jours |
| Descriptions avec données personnelles | RISQUE | Ajouter mention légale, vérifier durée de conservation |
| Données utilisateurs (prénom + nom) | ACCEPTABLE | Minimisation déjà respectée (pas d'email ni téléphone) |
| Export PDF avec données nominatives | RISQUE | Réserver aux administrateurs (déjà le cas) |
| Logs de suivi (auteur + date + texte) | ACCEPTABLE | Données opérationnelles légitimes |

---

## 5. Prochaines priorités recommandées

**Semaine 1 (urgent — sécurité) :**
1. Sécuriser l'authentification admin via Supabase RPC ou rendre le repo privé
2. Vérifier et documenter les politiques RLS Supabase

**Semaine 2-3 (RGPD) :**
3. Créer un Edge Function de nettoyage automatique du bucket Factures (> 30 jours)
4. Rédiger la politique de confidentialité et l'afficher à la connexion

**Mois 1 (UX) :**
5. Ajouter une barre de recherche texte libre dans les filtres
6. Remplacer `Date.now()` par `crypto.randomUUID()` pour les IDs

**Mois 2 (évolutions) :**
7. Dashboard admin avec graphique d'évolution temporelle
8. Améliorer l'accessibilité (aria-label sur les boutons icône)

---

## 6. Annexe — Fichiers modifiés

| Fichier | Commit | Description |
|---------|--------|-------------|
| `index.html` | `7dba993` | Toutes les corrections listées ci-dessus |

*Aucun fichier de configuration, secret ou credential n'a été modifié ou créé.*
