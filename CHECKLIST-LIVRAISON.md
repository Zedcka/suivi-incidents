# Checklist de livraison — Modèle location / maintenance

> Tous les comptes restent à TON nom (Supabase, Cloudflare, EmailJS, domaine).
> C'est ton levier : si Lucie arrête de payer, tu coupes l'accès.

---

## 1. Domaine & hébergement
- [ ] Acheter un nom de domaine pro à TON nom (ex: `lucie-incidents.fr`) — ~12€/an
- [ ] Rendre le repo GitHub PRIVÉ : Settings → Danger Zone → Change visibility
- [ ] Configurer Cloudflare Pages depuis le repo privé (gratuit, à ton nom)
- [ ] Pointer le domaine vers Cloudflare Pages

## 2. Supabase — projet dédié Lucie (à ton nom)
- [ ] Créer un nouveau projet Supabase dédié (séparer de ton projet perso)
- [ ] Recréer les tables : `incidents`, `boutiques`, `users`, `activity_log`
- [ ] Réappliquer les politiques RLS
- [ ] Redéployer l'Edge Function `manage-users`
- [ ] Recréer le bucket `Factures` avec les 2 policies (INSERT + SELECT authenticated)
- [ ] Mettre à jour `SUPABASE_URL` et `SUPABASE_ANON_KEY` dans le code

## 3. Comptes utilisateurs
- [ ] Créer le compte admin (tu choisis l'email, tu gardes le PIN)
- [ ] Mettre à jour l'email admin dans le code
- [ ] Créer les comptes agents pour leurs employés
- [ ] Renseigner leurs boutiques dans la table `boutiques`

## 4. Zendesk
- [ ] Demander leur adresse email Zendesk support
- [ ] Mettre à jour le "To email" dans le template EmailJS `template_nm9t3s8`

## 5. Branding (optionnel)
- [ ] Mettre à jour couleurs / logo si différents
- [ ] Vérifier qu'il n'y a aucune donnée de démo dans la base

## 6. Tests avant remise
- [ ] Login agent → créer incident → vérifier affichage
- [ ] Login admin → dashboard, exports PDF/CSV
- [ ] Réclamation client → ticket Zendesk créé automatiquement
- [ ] Envoi facture → email reçu + lien PDF fonctionnel
- [ ] Test mobile (iOS + Android)

---

## Ce que tu NE fais PAS en location
- ❌ Tu ne transfères pas le repo GitHub
- ❌ Tu ne transfères pas le compte Supabase
- ❌ Tu ne transfères pas Cloudflare ni EmailJS
- ❌ Tu ne donnes pas le code source
- ❌ Tu ne leur donnes pas les clés d'API

---

## Rappel coûts mensuels réels (à TON nom)
| Service | Coût |
|---------|------|
| Supabase | Gratuit |
| Cloudflare Worker + Pages | Gratuit |
| EmailJS | 0-10€/mois |
| Domaine | ~1€/mois |
| **Total** | **~15€/mois max** |

**Marge nette à 300€/mois : ~285€**
