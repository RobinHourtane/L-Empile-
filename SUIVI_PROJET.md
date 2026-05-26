# L'EM PILÉ — Suivi du projet

> Dernière mise à jour : 2026-05-26 (session 3)

---

## 📋 Contexte

Site de commande en ligne pour **L'EM PILÉ**, kebab artisanal basé à Toulon.
Brief pédagogique : EX commande en ligne / Restauration — noté UX, UI, UX Writing.

---

## ✅ Réalisé

### Système de commande (`commander.html`)
- [x] Scroll-snap full-screen (10 écrans, `scroll-snap-type: y mandatory`)
- [x] Barre de progression dynamique (IntersectionObserver)
- [x] Mini-panier flottant (pill fixe bas-droite, visible dès 1er choix)
- [x] Step 0 — Mode : Livraison / Click & Collect (avec photos Gemini)
- [x] Step 1 — Pain : Lavash / Pita / Dürum (photos réelles)
- [x] Step 2 — Taille : S/M/L avec cercles visuels (72/104/136px), prix 6.90/7.90/9.90 €
- [x] Step 3 — Viande : 6 options avec photos (Poulet, Bœuf, Mixte, Agneau, Falafels, Nuggets)
- [x] Step 4 — Légumes : 10 options, multi-select limité à 4, bouton Continuer
- [x] Step 5 — Sauces : 8 sauces avec roue décorative, descriptions saveur + points piment
- [x] Step 6 — Extras : 6 options avec prix (+), bouton Continuer
- [x] Step 7 — Boissons & Desserts : 5 options, bouton Continuer
- [x] Step 8 — Récapitulatif dynamique avec boutons Modifier par étape
- [x] Step 9 — Confirmation avec animation kebab flottant
- [x] États visuels : selected (rouge + glow + badge ✓ animé), hover (lift), unavailable (opacity + grayscale + ✕)
- [x] `calcTotal()` : SIZE_PRICES + EXTRA_PRICES + DRINK_PRICES

### Design system (`style.css`)
- [x] Palette terracotta : --btn #8C1A18, --bg #FAF3EB, --frame #F2DDD0
- [x] Typographie : Barlow Condensed (display) + Barlow (body)
- [x] Composants : boutons, tags, cards, navbar, scrollbar custom

### Assets
- [x] 10 images Gemini mapées aux bons contenus
- [x] Sous-dossiers images : `Sauses/`, `Secondary choose  ingredient/`, `Dessert or drink/`

---

## 🔴 En cours / Critique

### Vérification zone de livraison (Point de friction #1 du brief)
**Statut : ✅ Implémenté**
**Solution choisie : Leaflet.js + Nominatim (OpenStreetMap)**

#### Décisions techniques
| Choix | Option retenue | Raison |
|---|---|---|
| Carte | **Leaflet.js** (CDN, gratuit) | Léger, pas d'API key, open-source |
| Géocodage adresse | **Nominatim OSM** (`nominatim.openstreetmap.org/search`) | Gratuit, pas de clé, précis pour la France |
| Zone de livraison | **Cercle 5 km** centré restaurant | Simple, visuel, cohérent pour Toulon |
| Localisation restaurant | `43.1252, 5.9275` (Toulon centre) | À affiner avec l'adresse réelle |
| Rayon de livraison | **5000 m** | Couvre le centre de Toulon + quartiers proches |
| Marker restaurant | Logo SVG custom | Identité visuelle forte sur la carte |
| Fallback hors zone | Proposition Click & Collect | UX non-frustrant, on garde l'utilisateur |

#### Flux utilisateur
```
Step 0 : Choix mode
  └─ LIVRAISON sélectionné
      └─ Carte Leaflet s'affiche (inline, dans step 0)
          ├─ Cercle rouge translucide = zone de livraison
          ├─ Marker logo = restaurant L'EM PILÉ
          └─ Champ saisie adresse + bouton "Vérifier"
              ├─ ✅ Dans la zone → go step 1
              └─ ❌ Hors zone → message + proposition Click & Collect
```

---

## 🟠 À faire (Important)

- [x] Afficher adresse + horaires restaurant dans mode Click & Collect
- [x] Bannière promo / offre du moment (dans flux commande ou step 0)
- [x] Compteur légumes visible « 2/4 choix » (pas seulement shake) — dots animés
- [x] Renommer « Commander · Payer » → « Confirmer ma commande »
- [x] Ajouter mention mode de paiement sur écran confirmation
- [x] Navigation retour (← bouton ou swipe) entre steps

## 🟡 À faire (Souhaitable)

- [x] Sélection date/heure de livraison (commande immédiate ou planifiée)
- [x] Choix de quantité (plusieurs kebabs)
- [ ] Relance « Commander pour quelqu'un d'autre ? »
- [ ] Page/section contact : adresse, Google Map, téléphone, horaires, réseaux sociaux
- [ ] Page catalogue (gammes : kebabs / menus formule / extras seuls / boissons seules)
- [ ] Section présentation du concept (About L'EM PILÉ, valeurs, histoire)

## 🟢 Bonus / créativité

- [ ] Animations de transition entre steps (particules, flammes, etc.)
- [ ] Mode sombre
- [ ] Estimation temps de livraison dynamique selon heure
- [ ] Partage de commande (lien ou QR code)

---

## 📁 Structure des fichiers

```
Colline_kebab_site/
├── index.html              — Page d'accueil
├── commander.html          — Constructeur kebab (10 écrans)
├── style.css               — Design system global
├── SUIVI_PROJET.md         — Ce fichier
├── lavash.png, poule.png… — Images produits (racine)
├── Sauses/                 — 8 sauces (noms avec espaces trailing)
├── Secondary choose  ingredient/  — Légumes & viandes (double espace dans nom)
└── Dessert or drink/       — Boissons & desserts
```

---

## 🗺️ Carte Leaflet — Notes techniques

```js
// Coordonnées restaurant (à confirmer)
const RESTAURANT = { lat: 43.1252, lng: 5.9275 };
const DELIVERY_RADIUS_M = 5000; // 5 km

// Géocodage adresse utilisateur
// GET https://nominatim.openstreetmap.org/search
//   ?q={adresse}&format=json&limit=1&countrycodes=fr

// Calcul distance Haversine pour vérifier si dans zone
function isInZone(userLat, userLng) {
  // Formule Haversine
  const R = 6371000;
  const dLat = (userLat - RESTAURANT.lat) * Math.PI / 180;
  const dLng = (userLng - RESTAURANT.lng) * Math.PI / 180;
  const a = Math.sin(dLat/2)**2 +
            Math.cos(RESTAURANT.lat * Math.PI/180) *
            Math.cos(userLat * Math.PI/180) *
            Math.sin(dLng/2)**2;
  return (R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a))) <= DELIVERY_RADIUS_M;
}
```

---

## 🎨 Identité visuelle — Valeurs de l'enseigne

| Valeur | Expression dans le design |
|---|---|
| Artisanal & authentique | Palette terracotta chaude, photos réelles |
| Généreux | Grandes photos immersives full-screen |
| Rapide & efficace | Scroll-snap, parcours 9 étapes sans errance |
| Moderne & premium | Glassmorphism, animations spring, typographie condensed |
| Convivial | Emojis intégrés, ton direct et chaleureux |

---

*Généré automatiquement le 2026-05-26*
