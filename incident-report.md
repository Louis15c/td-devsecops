# Rapport d’incident et post‑mortem

## Informations générales

- **Date et heure de l’incident** : 29 juillet 2025, 14:10 CET  
- **Services impactés** : API TODO  
- **Gravité** : faible

## Résumé

Une erreur 500 a été observée lors de plusieurs appels au point de terminaison `/error` de l'API TODO.  
Cette route génère intentionnellement une exception (division par zéro) afin de simuler un incident pour les besoins du TP DevSecOps/SRE.  
L’impact pour les utilisateurs a été limité à cette route spécifique, sans effet sur les fonctionnalités principales de l’API (`/tasks`, `/health`, etc.).

L'incident a permis de tester la remontée des métriques dans Grafana et de valider la bonne configuration du monitoring.

## Timeline

| Heure (CET) | Événement |
|-------------|-----------|
| 14:10 | Appels manuels à `/error` déclenchent des erreurs 500 |
| 14:12 | Les métriques 500 sont observées dans Prometheus via Grafana |
| 14:15 | Le comportement est confirmé comme attendu (simulation volontaire) |
| 14:18 | Vérification du dashboard : montée des erreurs + latence stable |

## Détection et diagnostic

La panne a été détectée manuellement lors de tests via `curl` sur la route `/error`.  
Le dashboard Grafana, connecté à Prometheus, a affiché une augmentation des réponses HTTP 500 via la requête suivante :

rate(flask_http_request_total{status=~"5.."}[1m])

markdown
Copier
Modifier

Les erreurs se sont affichées correctement mais en faible proportion (ex. : 0.222) car d'autres requêtes simultanées généraient des réponses HTTP 200.

## Cause racine

La cause principale est la ligne suivante dans `app.py`, volontairement introduite dans la route `/error` pour générer une erreur :

## Facteurs contributifs :
Aucun gestionnaire d’exception spécifique pour cette route

Le simulateur d’erreur n’est pas conditionné à un flag de débogage

Les métriques initiales étaient difficiles à lire à cause de l’échelle du graphe

## Actions correctives et rétablissement
Aucune action corrective n’a été nécessaire, car il s’agissait d’un test contrôlé.
Cependant, les actions suivantes ont été réalisées :

Ajustement de l’échelle du graphique dans Grafana pour mieux visualiser les erreurs

Vérification de la bonne exposition des métriques HTTP via Prometheus

## Leçons apprises et actions de prévention
Observabilité : les erreurs 500 ont bien été détectées et remontées dans Prometheus, ce qui valide le bon fonctionnement du monitoring.

UX Grafana : une mauvaise configuration des axes peut rendre les erreurs invisibles à l’œil nu.

Sécurité : les routes de test comme /error devraient être désactivées en production ou protégées par une authentification.

Préventions à mettre en place :

Ajouter un gestionnaire global d’exception dans Flask

Supprimer ou restreindre /error en environnement de production

Ajouter des alertes Prometheus sur les taux d'erreur HTTP élevés

Documenter les endpoints de test pour éviter toute confusion

