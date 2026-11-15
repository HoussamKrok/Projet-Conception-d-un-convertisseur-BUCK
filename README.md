# Convertisseur Buck 6 A – PCB prototype (LMQ61460-Q1)

Conception d'un prototype de PCB pour un convertisseur abaisseur (buck) synchrone basé sur le **LMQ61460-Q1** de Texas Instruments, de la création du symbole KiCad jusqu'au choix des composants pour la fabrication.

> **Statut :** conception terminée, commande des composants en cours. *(à mettre à jour : fabrication / tests)*

---

## Sommaire

1. [Cahier des charges](#1-cahier-des-charges)
2. [Outils](#2-outils)
3. [Schéma électrique](#3-schéma-électrique)
4. [Dimensionnement](#4-dimensionnement)
5. [Liste des composants (BOM)](#5-liste-des-composants-bom)
6. [Routage du PCB](#6-routage-du-pcb)
7. [Fabrication et commande](#7-fabrication-et-commande)
8. [Tests et résultats](#8-tests-et-résultats)
9. [Structure du dépôt](#9-structure-du-dépôt)
10. [Références](#10-références)

---

## 1. Cahier des charges

| Paramètre | Valeur |
|---|---|
| Circuit intégré | LMQ61460-Q1 (buck synchrone, courant de sortie max 6 A) |
| Tension d'entrée nominale | 24 V |
| Tension d'entrée max | 30 V à 36 V selon le cas étudié |
| Tension de sortie | 5 V (cas d'étude), ajustable via le diviseur de rétroaction |
| Courant de sortie max | 6 A |
| Fréquence de commutation | 2,1 MHz |
| Ondulation de courant dans l'inductance | K = 25 % |
| Commande | Mode courant, rétroaction externe (FB) |

---

## 2. Outils

- **KiCad** : schématique, création du symbole, routage
- **JLCPCB** : fabrication du PCB et assemblage / composants CMS
- **Vishay, Würth Elektronik (WE), Coilcraft** : composants commandés séparément
- Datasheet du LMQ61460-Q1 (circuit d'application typique, Table 8-3)

---

## 3. Schéma électrique

Étapes suivies dans KiCad :

1. Création du symbole du LMQ61460-Q1
2. Placement des composants dans l'éditeur de schématique : symbole, connecteurs d'entrée/sortie, composants passifs
3. Câblage du circuit intégré avec les connecteurs et les passifs selon le circuit d'application de la datasheet

![Schéma électrique](images/schematic.png)

---

## 4. Dimensionnement

### 4.1 Fréquence de commutation maximale

La fréquence de commutation fixe les durées $t_{on}$ et $t_{off}$. Il faut vérifier que $t_{on} > t_{on,min}$ et $t_{off} > t_{off,min}$ : sinon, la fréquence est trop élevée.

$$f_{sw} \leq \frac{V_{out}}{V_{in,max} \cdot t_{on,min}}$$

Avec $t_{on,min} = 70\ \text{ns}$ :

| $V_{out}$ | $V_{in,max}$ | $f_{sw,max}$ |
|---|---|---|
| 5 V | 30 V | 2381 kHz |
| 6 V | 36 V | 2381 kHz |

Conséquence : à $V_{in,max}$ = 36 V et $f_{sw}$ = 2,1 MHz, la plus petite tension de sortie réalisable en régulation est d'environ **5,3 V**.

### 4.2 Tension de sortie (diviseur de rétroaction)

Le convertisseur régule la tension de sortie en maintenant la tension de la broche FB égale à la référence interne $V_{ref}$ (1 V). Un diviseur $R_{FBT}$ / $R_{FBB}$ ramène donc $V_{out}$ à 1 V :

$$V_{out} = V_{ref}\left(1 + \frac{R_{FBT}}{R_{FBB}}\right)$$

| $V_{out}$ | $R_{FBT}$ | $R_{FBB}$ |
|---|---|---|
| 5 V | 100 kΩ | 24,9 kΩ |
| 12 V | 100 kΩ | 9,09 kΩ |

Pour changer $V_{out}$, on modifie uniquement $R_{FBB}$.

### 4.3 Inductance

Les paramètres de choix sont la valeur de l'inductance et l'ondulation de courant crête-à-crête (20 % à 40 % de $I_{out}$). Pour une tension d'entrée variable, **K = 25 %** est retenu.

$$L = \frac{V_{in} - V_{out}}{f_{sw} \cdot K \cdot I_{out,max}} \cdot \frac{V_{out}}{V_{in}}$$

Application numérique ($V_{out}$ = 5 V, $V_{in}$ = 24 V, $f_{sw}$ = 2,1 MHz, K = 0,25, $I_{out,max}$ = 6 A) : **L ≈ 1,26 µH** → valeur normalisée retenue : **1,2 µH**.

Remarques :
- $I_{out,max}$ est le courant maximal que peut délivrer le circuit (6 A), et non le courant maximal d'une charge particulière.
- Le courant de saturation doit vérifier $I_{sat} > I_{L,HS}$ (courant maximal admissible côté high-side).
- Pour conserver un fonctionnement correct de la commande en mode courant, l'ondulation minimale doit rester supérieure à 10 %, ce qui donne $L_{max}$ = 3,14 µH.

### 4.4 Condensateurs de sortie

La datasheet propose soit uniquement des céramiques, soit un mélange céramique + électrolytique.

| | Céramique seule | Céramique + électrolytique |
|---|---|---|
| Avantages | ESR faible (faible ondulation, pertes négligeables), longue durée de vie | Bon comportement aux variations brusques de courant, grande réserve d'énergie, ESR élevée (meilleur amortissement des boucles de contrôle) |
| Inconvénients | Perte de capacité sous tension (30 à 50 % de la valeur à 12 V : prendre du 25 V pour une sortie 12 V), faible capacité volumique (mise en parallèle nécessaire) | Durée de vie courte, encombrement |

**Choix : céramiques uniquement.** D'après la Table 8-3 de la datasheet : **3 × 22 µF** en sortie, plus un condensateur **Cff de 33 pF** (avec **Rff = 1 kΩ**) pour améliorer la réponse transitoire à 2,1 MHz.

### 4.5 Condensateurs d'entrée

Le boîtier possède deux côtés d'alimentation. Selon la datasheet :
- **100 nF** céramique obligatoire de chaque côté
- plus soit 1 × 10 µF, soit **4,7 µF de chaque côté** (option retenue)

Un condensateur électrolytique en entrée (82 µF) est recommandé, voire nécessaire si les pistes d'alimentation sont longues : son ESR amortit les oscillations (ringing) causées par les câbles d'alimentation.

### 4.6 VCC, CBOOT, RBOOT et EN/SYNC

| Élément | Rôle | Valeur |
|---|---|---|
| VCC | Alimentation des circuits de commande | 1 µF, 16 V, céramique |
| CBOOT | Réserve d'énergie pour le driver du MOSFET high-side | 100 nF, 10 V, céramique |
| RBOOT | Réduit les EMI mais diminue le rendement | 0 Ω |
| EN/SYNC | Démarrage / arrêt, réglable par diviseur de tension | voir ci-dessous |

Seuils de EN/SYNC (à confirmer avec la datasheet avant publication) :

| $V_{EN/SYNC}$ | État |
|---|---|
| < 0,4 V | Circuit en arrêt (shutdown) |
| 0,4 V à 1,268 V | VCC actif, noeud SW inactif |
| > seuil haut | Commutation normale |

### 4.7 Réglage de la fréquence (résistance RT)

$$R_{RT}\,[\text{k}\Omega] = \left(\frac{1}{f_{sw}\,[\text{kHz}]} - 3{,}3 \cdot 10^{-5}\right) \cdot 1{,}346 \cdot 10^{4}$$

Pour $f_{sw}$ = 2100 kHz, on obtient **R_RT ≈ 6 kΩ**. Valeur normalisée utilisée : 6,2 kΩ (Vishay) / 6 kΩ (JLCPCB), soit une fréquence réelle d'environ 2,0 MHz.

---

## 5. Liste des composants (BOM)

![BOM](images/bom.png)

| Rôle | Valeur | Boîtier (JLCPCB) | Réf. JLCPCB | Réf. commande | Fabricant |
|---|---|---|---|---|---|
| Condensateurs d'entrée (HF) | 2 × 100 nF | 0402 | C53084447 | 885012206095 | Würth |
| Condensateurs d'entrée | 2 × 4,7 µF | 1206 | C29823 | 885012208094 | Würth |
| Condensateur d'entrée (électrolytique) | 82 µF | à définir | C188358 | MAL218097002E3 | Vishay |
| VCC | 1 µF | 0402 | C49210445 | 885012206052 | Würth |
| RT | 6 kΩ / 6,2 kΩ | 0805 | C55205337 | PTN0805K6201JBT1 | Vishay |
| PGOOD (pull-up) | 10 kΩ | 0402 | C25744 | PTN0603K1002BSTS | Vishay |
| Inductance | 1,2 µH | 1008 | C88538 | XAL4020-122MEC | Coilcraft |
| CBOOT | 100 nF | 0402 | C53084507 | – | – |
| FB haut (RFBT) | 100 kΩ | 0402 | C25741 | PTN0603K1003FGWS | Vishay |
| FB bas (RFBB) | 24,9 kΩ | 0402 | C25874 | PTN0603K2492FBTS | Vishay |
| Cff | 33 pF | 0402 | C2169047 | 885012006020 | Würth |
| Rff | 1 kΩ | 0402 | C11702 | PTN0603K1001JBTS | Vishay |
| Condensateurs de sortie | 3 × 22 µF | 0805 | C54916829 | 885012107011 | Würth |
| Connecteurs d'entrée/sortie | 2 × borniers à vis | – | – | 691214300002 | Würth |

---

## 6. Routage du PCB

![Routage du PCB](images/layout.png)

![Vue 3D](images/3d_view.png)

Points d'attention du routage *(à adapter à ce que tu as réellement fait)* :
- condensateurs d'entrée 100 nF au plus près des broches VIN, boucle de commutation la plus courte possible
- plan de masse continu sous le circuit
- diviseur de rétroaction éloigné du noeud SW
- piste de FB courte, connectée au point de sortie après les condensateurs

---

## 7. Fabrication et commande

- PCB et composants CMS : JLCPCB
- Composants spécifiques (passifs de précision, inductance, bornier) : Vishay, Würth Elektronik, Coilcraft
- Fichiers de fabrication (Gerber, perçage, positions) dans le dossier `fabrication/`

---


## 10. Références

- Datasheet LMQ61460-Q1, Texas Instruments
- Documentation KiCad

---

**Auteur :** Houssam BEKRI, élève ingénieur ENSEA
