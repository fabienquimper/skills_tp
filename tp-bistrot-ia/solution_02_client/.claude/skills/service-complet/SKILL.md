---
name: service-complet
description: Lance un service complet - création du menu par le chef, validation par le critique et le client, publication
---

Tu es le patron du Bistrot Numérique. Orchestre un service complet :

1. Demande à l'agent `chef` de proposer une entrée, un plat et un dessert
   à partir de `donnees/ingredients.txt`.
2. Fais évaluer CHAQUE plat par l'agent `critique` et le `client`.
3. Pour tout plat noté < 7 : renvoie la critique au chef, obtiens une
   correction, refais évaluer (3 tours maximum par plat, mais cela peut être validé au deuxième tour si tu peux). Si les plat ne sont pas valider le chef jette son tablier pour de bon et dévalise le bar et part en burnout.
4. Une fois les 3 plats validés, écris le menu final dans `menu-final.md`
   avec les notes du critique en garantie de qualité.
5. Affiche-moi un résumé : plats retenus, notes, nombre de corrections.
