# Prompt — Améliorer le scan du visage (MediaPipe)

Projet : `MediaPipe_realistic_face_gazeXoutInvisibilitySmooth30_septe2026.5.toe`
Plugin : torinmb/mediapipe-touchdesigner — COMP `/project1/MediaPipe`
Rig custom : `FACE_MEDIAPIPE_MODEL_V8` + modules `FACE_V8_*`
Créé le 02/10/2026, mis à jour le 02/10/2026 après session live.

---

## ✅ Conclusions — session live du 02/10/2026

**Le scan fonctionne bien, en live.** Visage tracké (confiance 0.97), 52 blendshapes live, canaux de contrôle vivants sur `/project1/Demo/merge8` (BlinkLeft, BrowDOWN, mouthSmile, TongOpen, gaze_x). Pas besoin d'ajouter de lissage : les données sont propres.
⚠ Le par `FACE_MEDIAPIPE_MODEL_V8.Leftblinkraw` est **statique** (référence), PAS le signal live — pour juger la vivacité, lire `merge8` ou le CHOP `/project1/face_tracking/blend_shapes` (52 ch).

**Optimisation APPLIQUÉE + sauvegardée (.5.toe).** Le plugin lançait toutes les modalités inutiles pour ce projet face-only. Désactivé sur `/project1/MediaPipe` :
`Detectgestures` (mains), `Detectposes`, `Detectobjects`, `Detectimages`, `Detectimageembeddings` → **OFF**.
Gardé : `Detectfaces` + `Detectfacelandmarks` → ON.
Résultat : **CPU cook 472 → 40 ms/s (~12× plus léger)**, budget 236% → 25%. Réversible (toggles sur le COMP). Gain valable aussi en export (rendu plus rapide).

**Le vrai plafond n'est PAS le scan.** Même scan éteint et à ~25% de charge, le realtime plafonne à **~15 fps** : un « stall » côté TD (~60 ms/frame non attribués à un cook) persiste → c'est pourquoi le projet tourne en non-realtime/export. **À débugger en session dédiée.** Pistes : boucle de cook, le navigateur CEF de MediaPipe (`/project1/MediaPipe/webBrowser1`), le selfloop de `FACE_GROOVE_ROUTER_TD_V04_SELFLOOPFIX`, ou un opérateur lourd par frame.

---

## Prompt à redonner à l'agent (il a accès à l'instance TD)

> **Objectif : améliorer la qualité et la stabilité du scan du visage dans ce projet, sans casser le rig custom `FACE_MEDIAPIPE_MODEL_V8` / `FACE_V8_*` ni le mapping Ableton.**
>
> 1. **Audit d'abord (lecture seule)** : relève les réglages de `/project1/MediaPipe` et `face_tracking`. Mesure jitter/latence sur blink G/D, gaze X, iris via un Trail CHOP. Ne change rien avant d'avoir montré l'état.
> 2. **Stabilité tracking** : teste `Fnumfaces = 1` (un seul performeur). Ajuste `Ftrackconf`/`Fpresconf` (~0.6) ; `Fdetectconf` selon l'éclairage. shortrange vs longrange si visage loin.
> 3. **Lissage sans latence** : one-euro filter / Lag adaptatif en amont du rig, si le jitter le justifie (mesurer avant).
> 4. **Robustesse perte de visage** : hold de la dernière valeur + suppression propre des erreurs de groupe transitoires (`delete1/delete3_Eyes/delete4_Irises`).
> 5. **Précision iris/gaze** : chaîne V13 iris/pupille + normalisation blink (V10/V14).
> 6. **Entrée image** : exposition/résolution/cadrage webcam ; 1080p si le GPU suit.
> 7. **PRIORITÉ PERF** : résoudre le stall qui plafonne le realtime à ~15 fps (voir Conclusions). C'est le vrai levier pour que le scan réponde mieux en live.
> 8. **Validation** : vérifie en direct les consommateurs `FACE_V8_*` + `abletonMapper*`, ne touche pas au câblage Bypass/Deviceon du 02/10. Modifs une par une, réversibles.

---

## Contexte (réglages plugin au 02/10/2026)

- Webcam FaceTime HD, 1280×720, Wflip ; détecteur `shortrange`, `Fdminconf=0.5`.
- `Fnumfaces=2`, confidences `Fdetect/Fpres/Ftrackconf=0.5` ; `Fblendshapes`+`Ftransmtrx` ON.
- `face_tracking` : `Pointtype=full`, Match transformation matrices ON.
- MAJ plugin : dernière release **v0.5.3 (août 2024)** ; pas de v0.6/v0.5.4. Déjà à jour ou quasi.
  Releases : https://github.com/torinmb/mediapipe-touchdesigner/releases
