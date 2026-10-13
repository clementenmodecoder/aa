# Tapster Sidekick (v1.5, moteurs pas-à-pas) – BOM reconstituée

Tapster n'a jamais publié de BOM officielle pour le Sidekick. Cette liste est
reconstituée à partir de :

- le code de [tapsterbot/sidekick-controller-server](https://github.com/tapsterbot/sidekick-controller-server)
  (version 1.5.0, 2017-2019, Johnny-Five + Firmata) ;
- le code de [tapsterbot/controller](https://github.com/tapsterbot/controller)
  (première version à servos, 2017) ;
- la photo du robot et les vidéos YouTube
  ([Controller Demo](https://www.youtube.com/watch?v=MAu_O9O79Sc),
  [Sidekick 1.5 – 720 Taps Per Minute](https://www.youtube.com/watch?v=2mKfu5inglc)).

Colonne **Source** :

- **Code** : confirmé par le code source (fichier cité).
- **Photo** : visible sur la photo ou dans les vidéos.
- **Estimation** : déduit ou choix raisonnable. À valider.

Les quantités sont données **pour un robot** (un « tap »). Les gestes à deux
doigts (pinch/zoom) utilisent **deux robots** (`tap0` et `tap1`, chacun avec son
propre firmware dans `firmware/`), donc doublez les quantités.

---

## 1. Électronique

| Qté | Composant | Spécification | Source | Notes |
|---:|---|---|---|---|
| 1 | Arduino Uno (ou compatible ATmega328P) | Firmware « tapsterfirmata 2.10.0 » (StandardFirmata modifié) | Code (`firmware/*.ino`, `server.js`) | Pins utilisés : D2–D12, A3, A4. Pilotage USB depuis Node.js. |
| 1 | CNC Shield V3 (format Arduino Uno) | 4 emplacements de driver (X, Y, Z, A) | Code (`tapper.js` : « CNC Shield – 4th Axis (A) ») | Brochage standard : step X/Y/Z = D2/D3/D4, dir = D5/D6/D7, enable = D8. |
| 3 | Driver de moteur pas-à-pas Pololu | A4988 ou DRV8825, réglé en **1/16 micro-pas** | Code (`arm.js` : `stepsPerRev = 3200`) | 3200 = 200 pas × 16. Un A4988 en 1/16 colle directement. Un DRV8825 doit être cavalié en 1/16 et non en 1/32. |
| 3 | Fin de course mécanique (micro-switch) | NO/NF, câblé avec pull-up (`invert: true`) | Code (`arm.js`, `z-axis.js`) | Bras A → D9, bras B → D10, Z → D12 (et non D11 : voir § 5). |
| 1 | Alimentation 12 V | ≥ 2 A (3 moteurs NEMA 17 + solénoïde) | Estimation | Entrée bornier du CNC Shield. |
| 1 | Câble USB A–B | Arduino ↔ PC hôte | Estimation | |
| 1 | Étage de puissance pour le solénoïde | MOSFET logique (ex. IRLZ44N) + diode de roue libre (1N4007), ou module MOSFET | Estimation | Le code commande le solénoïde via les pins dir/step de l'axe A (A3 = 17, A4 = 18). Le schéma exact n'est pas publié. |

## 2. Moteurs et actionneurs

| Qté | Composant | Spécification | Source | Notes |
|---:|---|---|---|---|
| 2 | Moteur pas-à-pas NEMA 17 (bras A et B) | 1,8°/pas (200 pas/tour), bipolaire | Code (`arm.js`) + Photo | Entraînement direct des bras proximaux (aucun réducteur visible dans le code). |
| 1 | Moteur pas-à-pas (axe Z) | NEMA 17 (ou NEMA 14), pilote un mouvement vertical | Code (`z-axis.js`, step D4, dir D7) | Course logicielle : 190 pas depuis la fin de course haute. La mécanique (vis-mère, crémaillère…) n'est pas documentée. |
| 1 | Solénoïde push (end effector « tapper ») | Petit solénoïde 12 V à course courte | Code (`tapper.js`) + vidéo « 720 taps/min » | Impulsion de 45 ms, période de 90 ms en mode rafale. |
| 1 | Stylet capacitif / embout conducteur | Embout caoutchouc conducteur, relié à la masse | Photo + Estimation | Monté au bout du solénoïde. Sert aussi à appuyer sur les boutons latéraux du téléphone. |

## 3. Mécanique (pièces imprimées / usinées)

Géométrie tirée de `server.js` et `site/js/models/sidekick.js`. Les unités ne
sont pas indiquées dans le code, mais ce sont très probablement des mm (multiples
de 8 mm).

| Qté | Pièce | Dimension | Source | Notes |
|---:|---|---|---|---|
| 2 | Bras proximal (« upper arm ») | Entraxe **72 mm** (8 × 9) | Code (`r0: 8*9`) | Un par moteur (A, B). |
| 2 | Bras distal (« forearm ») | Entraxe **104 mm** (8 × 13) | Code (`r1: 8*13`) | Se rejoignent au point effecteur : cinématique parallèle à 5 barres. |
| 1 | Support moteurs A/B | Axes moteurs écartés de **48 mm** (x = −24 / +24) | Code (`x0: -24`, `x0: 24`) | La v1 à servos utilisait 64 mm (y = ±32). |
| 1 | Boîtier / tour moteur | Impression 3D (PLA/PETG noir, logo Tapster) | Photo | Contient l'Arduino + le shield. |
| 1 | Chariot Z + guide | Guide vertical pour les bras et le solénoïde | Estimation | Non documenté. |
| 1 | Porte-effecteur | Fixe le solénoïde au point de jonction des bras distaux | Photo | |
| 1 | Plaque de base / porte-téléphone | Plaque blanche (PEHD ou acrylique), trous de fixation | Photo | |
| 4 | Cales d'angle pour le téléphone | Impression 3D (rouges sur la photo) | Photo | Dimensionnées pour le téléphone testé. |
| 6–8 | Pieds / entretoises | Vis + entretoises sous la plaque de base | Photo | |

## 4. Visserie et roulements (estimation)

| Qté | Composant | Notes |
|---:|---|---|
| ~4 | Roulement à billes (ex. 623ZZ ou 683ZZ) | Articulations passives : coude A, coude B et articulation de l'effecteur. |
| ~4 | Axes / vis épaulées M3 | Pour les articulations. |
| ~16 | Vis M3 tête cylindrique (CHC) | Moteurs NEMA 17 (4 par moteur) et fixations diverses. |
| ~20 | Écrous M3 + rondelles | |
| 2 | Moyeu / adaptateur arbre 5 mm → bras | Fixe le bras proximal à l'arbre moteur (vis de pression). |

## 5. Brochage récapitulatif (`sidekick-controller-server/server.js`)

| Fonction | Pin Arduino | Emplacement sur le CNC Shield |
|---|---|---|
| Bras A – STEP / DIR | D2 / D5 | Axe X |
| Bras B – STEP / DIR | D3 / D6 | Axe Y |
| Z – STEP / DIR | D4 / D7 | Axe Z |
| Enable drivers | D8 | EN |
| Fin de course A | D9 | X-endstop |
| Fin de course B | D10 | Y-endstop |
| Fin de course Z | D12 | Z-endstop (D12 sur les shields câblés « GRBL ≥ 0.9 », D11 sinon) |
| Solénoïde (via étage MOSFET) | A3 (17) / A4 (18) | Pins DIR/STEP de l'axe A |

Paramètres de mouvement :

- angle de repos bras A = 240°, bras B = −60° ;
- débattement : A de 240° à 30°, B de −60° à 150° ;
- ordre de homing : Z, puis A, puis B.

---

## Ce qui reste inconnu

- Fichiers CAO/STL des pièces imprimées (non publiés).
- Mécanique exacte de l'axe Z.
- Références commerciales exactes (moteurs, solénoïde, drivers).
- Schéma de l'étage de puissance du solénoïde.

Pour ces points, il faut contacter Tapster Robotics (Jason Huggins) ou
rétro-concevoir à partir des photos et vidéos.
