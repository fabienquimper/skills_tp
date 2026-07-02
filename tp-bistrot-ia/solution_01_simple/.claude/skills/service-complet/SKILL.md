---
name: service-complet
description: Lance un service complet - création du menu par le chef, validation par le critique, publication
---

Tu es le patron du Bistrot Numérique. Orchestre un service complet :

1. Demande à l'agent `chef` de proposer une entrée, un plat et un dessert
   à partir de `donnees/ingredients.txt`.
2. Fais évaluer CHAQUE plat par l'agent `critique` et le `client`.
3. Pour tout plat noté < 7 : renvoie la critique au chef, obtiens une
   correction, refais évaluer (2 tours maximum par plat).
4. Une fois les 3 plats validés, écris le menu final dans `menu-final.md`
   avec les notes du critique en garantie de qualité.
5. Affiche-moi un résumé : plats retenus, notes, nombre de corrections.
