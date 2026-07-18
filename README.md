# Order — Portail de commande Free Spirits Distribution

Portail de commande B2B pour les clients de Free Spirits Distribution (cavistes, CHR).
Application statique (HTML/JS vanilla, sans build) hébergée sur GitHub Pages,
backend Supabase — le **même projet** (`dlpzxngnphxuvopcxenf`) que le CRM interne.

**Dépôt frère :** [`FreeSpirits-distribution/CRM`](https://github.com/FreeSpirits-distribution/CRM) — CRM interne (gestion des clients Order via l'onglet Admin).

---

## Parcours client

1. **Inscription** : formulaire (enseigne, email, téléphone, mot de passe) → `auth/v1/signup` avec métadonnées `role: client_order`. Un email de confirmation est envoyé (dépend du SMTP configuré côté Supabase, voir plus bas).
2. **Activation** : le client envoie Kbis + RIB à `commande@free-spirits.fr` (lien mailto pré-rempli). L'équipe FSD active/complète la fiche `clients_order` depuis le CRM (onglet Admin).
3. **Commande** : catalogue produits (table `produits`), panier par colis (PCB), notes de commande.
4. **Envoi** : `submitOrder()` génère un **email `mailto:` récapitulatif** vers `commande@free-spirits.fr` (la commande n'est pas écrite en base — traitement manuel côté FSD).

---

## Stack

- HTML/CSS/JS vanilla — un seul fichier (`index.html`), aucun build
- Supabase : Auth (email/password + confirmation), tables `produits` (lecture) et `clients_order` (profil client)
- GitHub Pages : déploiement automatique à chaque push sur `main`
- PWA installable (manifest + icônes)

---

## Structure

```
.
├── index.html             # App complète (auth, catalogue, panier, envoi commande)
├── manifest.webmanifest   # PWA
├── icon-*.png             # Icônes
└── README.md
```

---

## Démarrage local

```bash
git clone https://github.com/FreeSpirits-distribution/Order.git
cd Order
npx serve .   # ou ouvrir index.html directement
```

La clé Supabase utilisée est la clé **anon** (publique par design). Ne jamais y placer la `service_role`.

---

## Déploiement

Push sur `main` → GitHub Pages publie en ~1 minute (Ctrl+F5 / rouvrir la PWA après publication).

---

## Emails d'inscription (important)

La confirmation d'inscription part via le SMTP configuré dans Supabase → Authentication → SMTP :

- **SMTP par défaut Supabase** : ~2 emails/heure, uniquement vers les adresses membres de l'équipe Supabase → **les inscriptions de vrais clients ne reçoivent pas l'email**. Dépannage : confirmer manuellement le compte dans Dashboard → Authentication → Users → ⋯ → Confirm email.
- **SMTP custom (Brevo / OVH)** : requis pour un fonctionnement réel en production (domaine `free-spirits.fr` authentifié SPF/DKIM chez le fournisseur).

---

## Sécurité

- Signup public par nature : le trigger Supabase `handle_new_user` ne doit **jamais** accepter un rôle privilégié depuis les métadonnées d'inscription (seul `client_order` — ou `agent` par défaut — est légitime ici). Voir `db/` du dépôt CRM pour l'audit RLS.
- `produits` est lisible par `anon` : n'y stocker aucune donnée sensible (prix d'achat, marges…).
- RLS : un client ne doit voir que sa propre ligne `clients_order`.
- Token de session en `localStorage` : maintenir l'échappement (`esc()`) de toute donnée affichée.
