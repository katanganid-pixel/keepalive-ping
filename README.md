# keepalive-ping

Empêche le service Render **adelineLus** (`srv-dbutir0m7kps73e4at80`) de se mettre
en veille. Le plan gratuit Render endort un service après **15 minutes** sans trafic.

## Ce que ça fait

`.github/workflows/keepalive.yml` :

- se déclenche **toutes les 5 minutes** (cron GitHub) ;
- à chaque déclenchement, ping `https://destin-ia-polyvalent.onrender.com/api/health`
  **toutes les 30 secondes pendant ~4 min 40** (10 pings) ;
- ne fait pas échouer le job sur un ping raté isolé (le réveil à froid prend ~50 s).

Pourquoi une boucle : GitHub Actions ne descend pas sous **5 minutes** de granularité
cron. La boucle interne donne le pas de 30 s demandé.

## Vérifier que ça marche

- **Derniers passages** : https://github.com/katanganid-pixel/keepalive-ping/actions
- **Log d'un passage** : ouvrir le run → job `ping` → étape
  « Ping /api/health toutes les 30 s ». On y lit une ligne par ping :
  `[06:18:50] ping #2 -> 200 OK  {"ok":true}`
- **Le service est-il éveillé ?** ouvrir https://destin-ia-polyvalent.onrender.com
  — s'il répond en moins d'une seconde, il est chaud ; s'il met ~50 s, il dormait.

## Lancer un passage à la main

Onglet **Actions** → workflow **keepalive** → bouton **Run workflow**.

## Arrêter

- **Définitivement** : onglet **Actions** → **keepalive** → menu `…` → **Disable workflow**.
- **Supprimer** : Settings du dépôt → zone *Danger Zone* → **Delete this repository**.

Désactiver le workflow ne touche pas au service Render : il se rendormira simplement
après 15 minutes sans visite.

## Limite à connaître

Le cron GitHub n'est **pas garanti à l'heure** : en période de charge, un passage
planifié peut être retardé de 5 à 20 minutes. Si un trou dépasse 15 minutes, Render
endort le service et la visite suivante paie ~50 s de réveil. Pour une garantie
stricte, utiliser un moniteur externe (cron-job.org, UptimeRobot) ou un
**Render Cron Job** payant.

GitHub désactive aussi les workflows planifiés d'un dépôt **sans activité pendant
60 jours**. Si les passages s'arrêtent, pousser un commit suffit à réactiver.
