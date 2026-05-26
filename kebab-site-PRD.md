# 🥙 PRD — Site de commande Kebab interactif
**Stack : Vite.js + React + Supabase + Vercel**

---

## 1. Vision du projet

Site de commande en ligne pour une enseigne de kebab fictive.
Concept central : **le constructeur de kebab interactif** — chaque ingrédient occupe un écran complet en 16:9, présenté comme un pattern seamless photographique, avec une animation cinématographique de clôture.

Identité visuelle : **stérile, technologique, moderne, premium** — à l'opposé du kebab bas de gamme. Ambiance Apple/Nike appliquée à la street food.

---

## 2. Stack technique

| Couche | Technologie |
|---|---|
| Frontend | Vite.js + React + TypeScript |
| Styling | Tailwind CSS |
| Animation | Framer Motion |
| Base de données | Supabase (PostgreSQL) |
| Auth | Supabase Auth |
| Stockage images | Supabase Storage |
| Déploiement | Vercel |
| Domaine | Cloudflare DNS |

---

## 3. Structure des pages

```
/                    → Landing page + Hero animé
/commander           → Constructeur kebab interactif (core feature)
/menu                → Catalogue produits secondaires
/concept             → Présentation de l'enseigne
/contact             → Adresse + Map + Horaires + Réseaux
/compte              → Espace client (commandes, profil)
/confirmation        → Page finale après commande
```

---

## 4. Feature principale — Constructeur kebab interactif

### 4.1 Principe UX

Chaque étape de la composition du kebab = **un écran plein 100vw × 100vh**.
L'utilisateur scrolle (ou swipe) verticalement pour passer d'un ingrédient à l'autre.
Le fond de chaque écran est le **pattern seamless photographique** de l'ingrédient correspondant.

### 4.2 Séquence des écrans

```
Écran 0  → Choix du mode : Livraison ou Click & Collect
           + Vérification zone de livraison (friction point)

Écran 1  → Choix du pain
           [ Lavaş | Pita | Dürum ]

Écran 2  → Choix de la viande
           [ Poulet | Bœuf | Agneau | Nuggets | Falafels | Mixte ]

Écran 3  → Choix des légumes (multi-sélection)
           [ Salade iceberg | Romaine | Tomates | Oignon rouge |
             Oignon blanc | Concombre | Maïs | Poivron frais |
             Poivron rôti | Chou ]

Écran 4  → Choix des sauces (multi-sélection, max 3)
           Présentation : roue circulaire divisée en 8 secteurs
           [ Ail | Harissa | Mayo | Ketchup | Tzatziki |
             Samouraï | BBQ | Miel-moutarde ]

Écran 5  → Choix des extras (produits additionnels)
           [ Frites | Fromage | Œuf | Cornichons marinés ]

Écran 6  → Récapitulatif "Vous êtes sûr de votre choix ?"
           Animation : lavaş ouvert avec tous les ingrédients visibles
           Bouton : CONFIRMER

Écran 7  → Animation cinématographique de fermeture + envol
           (voir section 4.4)

Écran 8  → Page confirmation / panier
```

### 4.3 Comportement des écrans ingrédients

- Le **pattern seamless** de l'ingrédient remplit tout l'écran (background-size: cover, background-repeat: repeat)
- Au centre : le nom de l'ingrédient en typographie large
- En bas : les options cliquables sous forme de cards horizontales
- L'option sélectionnée déclenche une **micro-animation** (scale + glow)
- Transition entre écrans : **scroll snap** ou swipe avec Framer Motion

### 4.4 Animation finale (Écran 6 → 7 → 8)

**Kадр 1 — Question (4 secondes)**
- Lavaş ouvert occupe le bas de l'écran (polukrug bord à bord)
- Rabat supérieur du lavaş = grand clapet ouvert comme une enveloppe
- Texte centré : "Vous êtes sûr de votre choix ?"
- Mouvement très lent : légère respiration cinématographique
- Un filet de vapeur naturel monte doucement

**Kадр 2 — Fermeture (2 secondes)**
- Les rabats du lavaş se referment lentement puis s'accélèrent
- Physique tissu/pâte, éclairage qui glisse sur la surface
- Freeze frame 0.5s à la fermeture complète

**Kадр 3 — Révélation (3 secondes)**
- Pull back caméra, kebab parfaitement fermé centré dans le cadre
- Rotation hero shot 180°, fond blanc stérile
- Lumière froide, détails texture lavaş en macro

**Kадр 4 — Envol (4 secondes)**
- Kebab monte lentement puis accélère vers le haut
- Caméra bascule en low angle sous le kebab
- Fond noir progressif, colonne de lumière divine du dessus
- Slow motion au sommet

**Kадр 5 — Flash final (2 secondes)**
- Bloom lumineux progressif (pas une explosion)
- Écran blanc total — hold 1 seconde
- Transition vers la page confirmation

**Implémentation recommandée :**
- Générer les frames clés avec Kling AI ou Google Veo 3
- Assembler avec Framer Motion ou GSAP dans React
- Ou intégrer directement la vidéo générée comme `<video autoplay muted>`

---

## 5. Point de friction — Zone de livraison

**Placement :** Écran 0, avant toute composition

**Comportement :**
```
1. L'utilisateur choisit : Livraison ou Click & Collect
2. Si Livraison → champ adresse avec autocomplétion Google Maps API
3. Vérification en temps réel si l'adresse est dans le polygone de livraison
4. Si hors zone → message clair + proposition Click & Collect
5. Si dans la zone → validation et suite du parcours
```

**Zone de livraison :** définie dans Supabase sous forme de polygone GeoJSON
**UX Writing :** 
- ✅ "Super, on livre chez vous !"
- ❌ "Votre adresse est dans notre zone de livraison." (trop froid)
- ⚠️ "On ne livre pas encore dans votre quartier — mais vous pouvez récupérer votre commande sur place !"

---

## 6. Point de satisfaction — Offres et promotions

**Placement :** Visible dès la landing page ET dans le récapitulatif panier

**Types d'offres :**
- Offre événementielle → hero section landing page (ex: "Le Kebab de la semaine")
- Promotion récurrente → bandeau sticky dans le constructeur (ex: "-10% le mercredi soir")
- Produit mis en avant → card dédiée sur l'écran menu

**Structure Supabase :**
```sql
promotions (
  id uuid,
  type text, -- 'event' | 'recurring' | 'product'
  title text,
  description text,
  discount_percent int,
  valid_from timestamptz,
  valid_until timestamptz,
  is_active boolean
)
```

---

## 7. Catalogue produits

### 7.1 Types de produits

| Type | Description | Exemples |
|---|---|---|
| Simple | Acheté en l'état | Boisson, dessert seul |
| Variable | Personnalisable | Le kebab (constructeur) |
| Additionnel | Ne peut être acheté seul | Sauce supplémentaire, extra fromage |

### 7.2 Gammes

```
Nos Kebabs          → produit variable (constructeur)
Nos Menus           → kebab + boisson + dessert (produit variable)
Nos Extras          → frites, sauces, suppléments (additionnels)
Nos Boissons        → sodas, jus, eau (simples)
Nos Desserts        → baklava, kunafa, glace (simples)
```

### 7.3 Structure Supabase

```sql
categories (
  id uuid,
  name text,
  slug text,
  display_order int
)

products (
  id uuid,
  category_id uuid references categories,
  name text,
  description text,
  base_price decimal,
  type text, -- 'simple' | 'variable' | 'addon'
  image_url text,
  is_available boolean,
  display_order int
)

ingredients (
  id uuid,
  name text,
  category text, -- 'bread' | 'meat' | 'vegetable' | 'sauce' | 'extra'
  image_pattern_url text, -- URL du pattern seamless 16:9
  extra_price decimal default 0,
  is_available boolean
)

product_ingredients (
  product_id uuid references products,
  ingredient_id uuid references ingredients,
  is_default boolean,
  is_optional boolean
)
```

---

## 8. Ingrédients — Référence complète

### Pains (3)
| Nom | Description |
|---|---|
| Lavaş | Pain plat traditionnel, fin et souple |
| Pita | Pain rond gonflé, plus épais |
| Dürum | Lavaş très fin enroulé serré |

### Viandes (6)
| Nom | Description |
|---|---|
| Poulet grillé | Tranches fines dorées, légèrement grillées |
| Bœuf döner | Tranches caramélisées, bords croustillants |
| Agneau | Viande plus persillée, goût prononcé |
| Nuggets | Panure croustillante dorée |
| Falafels | Boulettes de pois chiches, végétarien |
| Mixte | Mélange poulet + bœuf |

### Légumes (10)
| Nom | Description |
|---|---|
| Salade iceberg | Croquante, fraîche, eau visible |
| Romaine | Feuilles plus longues, vert foncé |
| Tomates | Tranches juteuses, rouge vif |
| Oignon rouge | Rondelles violettes translucides |
| Oignon blanc | Rondelles blanches croquantes |
| Concombre | Tranches rondes, vert clair |
| Maïs | Grains jaunes brillants |
| Poivron frais | Julienne rouge vif |
| Poivron rôti | Bords caramélisés, texture souple |
| Chou | Lanières blanc-vert, croquantes |

### Sauces (8) — roue interactive
| Nom | Couleur | Description |
|---|---|---|
| Ail (blanche) | Blanc crémeux | Fraîche et douce |
| Harissa | Rouge foncé | Épicée, pâte de piment |
| Mayonnaise | Jaune pâle | Classique |
| Ketchup | Rouge vif | Tomate sucrée |
| Tzatziki | Blanc-vert | Yaourt concombre aneth |
| Samouraï | Orange-rouge | Mi-épicée |
| BBQ | Brun très foncé | Fumé et sucré |
| Miel-moutarde | Jaune doré | Douce et acidulée |

### Extras / Additionnels (4)
| Nom | Description |
|---|---|
| Frites | Fines et croustillantes, or vif |
| Fromage fondu | Cheddar coulant, doré |
| Œuf | Tranches d'œuf dur |
| Cornichons marinés | Rondelles vert olive |

---

## 9. Assets visuels — Prompts de génération

> Tous les patterns suivent la même base :
> `Seamless repeating texture pattern of [INGREDIENT], seen directly from above (90-degree top-down view, birds eye view), [DESCRIPTION], fills the entire frame with no gaps, clean sterile modern aesthetic, soft diffused studio lighting, no shadows on edges, no borders, seamless tile, photorealistic, 4K, 16:9 ratio.`

### Pains
**Lavaş**
```
fresh lavash flatbread surface, soft flour-dusted surface, subtle natural folds and texture details, warm beige tones, thin and delicate
```
**Pita**
```
fresh pita bread, round puffy bread, slightly golden, soft surface with air pockets, randomly scattered at different angles and sizes
```
**Dürum**
```
thin durum wheat lavash wraps, very thin and smooth surface, slightly translucent edges, light golden color, minimal texture
```

### Viandes
**Poulet**
```
grilled chicken slices for kebab, thinly cut golden brown chicken with slight char marks, juicy and realistic, various sizes and angles, randomly scattered
```
**Bœuf**
```
grilled beef doner slices, thinly sliced dark brown beef with caramelized edges, rich marbling visible, randomly scattered at different angles
```
**Agneau**
```
grilled lamb kebab slices, thinly sliced lamb meat, deep brown with golden edges, slightly fattier texture, scattered naturally
```
**Nuggets**
```
crispy chicken nuggets, golden brown breaded coating, various oval shapes, randomly scattered at different angles, crispy texture detail
```
**Falafels**
```
golden falafel balls, round crispy exterior, deep golden brown, some slightly cracked revealing green interior on broken ones, scattered randomly
```
**Mixte**
```
mixed doner kebab meat, mix of chicken and beef slices interleaved, various shades of brown, randomly scattered overlapping at different angles
```

### Légumes
**Salade iceberg**
```
shredded iceberg lettuce, crispy pale green shreds, fresh and wet with water droplets, randomly distributed in all directions, natural chaotic feel
```
**Romaine**
```
fresh romaine lettuce leaves, darker green elongated leaves, slightly ruffled edges, torn and shredded naturally, scattered at random angles
```
**Tomates**
```
fresh sliced tomatoes, thin round tomato slices in various sizes, some whole, some half slices, vibrant red with visible juicy seeds, randomly scattered and rotated, natural imperfections, organic feel, no perfect grid
```
**Oignon rouge**
```
thinly sliced red onion rings, vibrant purple-red translucent rings, various sizes, randomly scattered and overlapping at different angles
```
**Oignon blanc**
```
thinly sliced fresh white onion rings, translucent white rings with light yellow edges, crispy texture, randomly scattered at different angles and sizes
```
**Concombre**
```
fresh sliced cucumbers, thin round slices, bright green skin with pale center, visible seeds, some whole rounds some half moons, randomly scattered at different angles
```
**Maïs**
```
sweet corn kernels, bright yellow individual kernels scattered densely, slightly glossy surface, natural size variation, some kernels in small clusters, organic distribution
```
**Poivron frais**
```
fresh red bell pepper strips, thin julienne cut strips, vibrant red, slightly curved, glossy surface, randomly scattered at all angles
```
**Poivron rôti**
```
roasted red bell pepper strips, charred and caramelized edges, deep red with dark spots, soft and slightly wrinkled texture, randomly scattered
```
**Chou**
```
shredded white cabbage, thin crispy white-pale green shreds, slightly translucent, densely packed, randomly oriented in all directions
```

### Sauces
**Roue des sauces (écran unique)**
```
A perfect circular swirl divided into 8 equal pie sections, seen directly from above (90-degree top-down view). Each section is a different sauce texture: white creamy garlic sauce, red spicy harissa, yellow honey mustard, orange ketchup, pale green tzatziki yogurt sauce, dark brown BBQ sauce, light beige mayonnaise, deep red samurai sauce. Each section flows into the next in a seamless swirl motion, sauces look thick, glossy, realistic and appetizing. Clean sterile modern aesthetic, soft diffused studio lighting, centered composition, fills the entire frame. Photorealistic, sharp focus, 4K, 16:9 ratio.
```

**Patterns individuels (pour Supabase)**
- Ail : `white creamy garlic sauce, thick glossy white sauce with subtle swirl patterns, smooth and rich texture`
- Harissa : `red spicy harissa sauce, deep red thick paste with visible chili flakes and texture, glossy surface with natural swirl patterns`
- Mayonnaise : `classic mayonnaise, pale yellow thick creamy texture, smooth with subtle swirls, glossy rich surface`
- Ketchup : `tomato ketchup sauce, bright red glossy thick sauce, smooth with natural flow patterns, vibrant color`
- Tzatziki : `tzatziki yogurt sauce, pale white creamy base with visible green cucumber bits and dill, thick texture with natural swirl patterns`
- Samouraï : `samurai sauce, deep orange-red thick sauce, slightly spicy appearance, smooth glossy texture with subtle swirl patterns`
- BBQ : `dark BBQ sauce, deep dark brown almost black glossy thick sauce, rich molasses-like texture with natural flow patterns`
- Miel-moutarde : `honey mustard sauce, golden yellow thick sauce, warm honey tones, smooth glossy texture with subtle swirl patterns`

### Extras
**Frites**
```
crispy golden french fries, thin cut fries, golden brown and crispy, scattered naturally, various lengths, randomly rotated in all directions
```
**Fromage**
```
melted cheddar cheese, golden yellow melted cheese with natural stretch patterns, slightly bubbly surface, warm tones
```
**Œuf**
```
sliced boiled eggs, round cross-section slices showing white and yellow yolk, various sizes, randomly scattered and rotated
```
**Cornichons**
```
sliced pickled gherkins, thin round slices, olive green with visible seeds, slightly translucent, randomly scattered at different angles
```

---

## 10. Animation finale — Prompts vidéo

### Prompt complet (Kling AI / Google Veo 3)

```
Cinematic commercial grade video, Apple product launch aesthetic,
ultra high end luxury food photography in motion.
No fire, no sparks, no cheap effects. Only light, shadow, texture and space.

Scene 1 - Anticipation (0:00 - 0:05):
Extreme slow motion macro shot of open kebab,
camera glides millimeters above the ingredients surface,
microscopic details: texture of grilled chicken skin,
individual lettuce fibers, tomato juice catching light,
one single elegant wisp of natural steam rises slowly,
shallow depth of field, bokeh background,
silence and stillness, luxury perfume ad aesthetic,
golden ratio composition.

Scene 2 - Decision moment (0:05 - 0:08):
Camera slowly rises and pulls back revealing full kebab,
text "Vous êtes sûr de votre choix?" fades in with elegant typography,
text breathes very slowly — scales 100% to 101% and back,
3 second hold, nothing moves except one slow steam wisp,
tension through stillness not through motion.

Scene 3 - Closing (0:08 - 0:11):
Extremely slow motion lavash folds begin — 10x slower than real time,
fabric-like physics, silk texture quality on the dough,
each fold catches light differently as it moves,
at 75% closed — sudden 4x speed snap to closed,
one frame freeze on perfect sealed wrap.

Scene 4 - Hero reveal (0:11 - 0:14):
Cut to pure white environment, perfectly wrapped shawarma floating still,
camera orbits it at 2cm per second, hyper slow 180-degree arc,
single overhead key light creates one perfect shadow,
surface texture of lavash in extreme detail.

Scene 5 - Ascension (0:14 - 0:16):
Shawarma begins rising imperceptibly slow then exponential acceleration,
camera tilts up following it with perfect smoothness,
background transitions from white to infinite deep black,
single divine light column from above, volumetric Caravaggio chiaroscuro,
shawarma becomes smaller against the void, slow motion at the peak.

Scene 6 - White out (0:16 - 0:18):
Light intensifies gradually over 1.5 seconds,
not a flash but a slow inevitable bloom like sunrise,
white fills the frame completely.
Hold on pure white. Fade complete.

Total duration: 18 seconds.
```

---

## 11. Compte client

### Données à collecter
- Prénom + Nom
- Email
- Téléphone
- Adresse de livraison favorite (optionnel)

### Fonctionnalités
- Voir ses anciennes commandes
- Repasser une commande identique
- Modifier une ancienne commande → devient une nouvelle commande
- Supprimer une commande de l'historique
- Modifier ses données personnelles
- Supprimer son compte

### Structure Supabase
```sql
profiles (
  id uuid references auth.users,
  first_name text,
  last_name text,
  phone text,
  default_address text,
  created_at timestamptz
)

orders (
  id uuid,
  user_id uuid references profiles,
  status text, -- 'pending' | 'confirmed' | 'ready' | 'delivered'
  delivery_type text, -- 'delivery' | 'pickup'
  delivery_address text,
  total_price decimal,
  created_at timestamptz
)

order_items (
  id uuid,
  order_id uuid references orders,
  product_id uuid references products,
  quantity int,
  unit_price decimal,
  customization jsonb -- { bread, meat, vegetables[], sauces[], extras[] }
)
```

---

## 12. Contact

- **Adresse** : à définir (fictive)
- **Google Maps** : embed iframe + lien directions
- **Téléphone** : affiché, cliquable sur mobile (tel:)
- **Email** : formulaire de contact simple (pas d'adresse exposée)
- **Réseaux** : Instagram + TikTok (pertinents pour restauration)
- **Horaires** : tableau par jour, indication si ouvert maintenant (logique temps réel)

---

## 13. UX Writing — Principes

- **Tutoyer** l'utilisateur (proximité, street food)
- **Verbes d'action** clairs : "Ajoute", "Choisis", "Valide"
- **Micro-copy positif** : ne pas dire "Erreur" mais "Oups, on ne te livre pas encore là"
- **Progression visible** : indicateur d'étape dans le constructeur (1/6, 2/6...)
- **CTA** uniques par écran, jamais deux boutons principaux

---

## 14. Déploiement

```
Vercel          → frontend Vite.js (auto-deploy depuis GitHub)
Supabase        → base de données + auth + storage
Cloudflare      → DNS + protection DDoS
```

### Variables d'environnement (.env)
```
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_GOOGLE_MAPS_API_KEY=
```

---

## 15. Checklist brief Mme Pessel

- [x] Commande en ligne, pas de consommation sur place
- [x] Point de friction livraison géré tôt dans le parcours
- [x] Livraison + Click & Collect
- [x] Vérification zone de livraison
- [x] Offre/promotion mise en avant
- [x] Produits simples, variables et additionnels
- [x] Segmentation par gammes
- [x] Ajout au panier direct depuis la gamme
- [x] Présentation de l'enseigne
- [x] Contact + carte + horaires + réseaux
- [x] Pas de tunnel de paiement
- [x] Compte client avec historique
- [x] Parcours utilisateur court et guidé
- [x] UX Writing appliqué
