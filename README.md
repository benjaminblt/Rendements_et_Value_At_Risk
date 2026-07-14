# Analyse des rendements et Value at Risk
Étudier les faits stylisés des rendements de Schneider Electric, sélectionner un modèle ARMA-GARCH adapté et valider plusieurs Value at Risk hors échantillon.

## Données

- actif : Schneider Electric, ticker `SU.PA` ;
- cours ajustés quotidiens ;
- période globale : 2010-2024 ;
- estimation : 2010-2019 ;
- validation hors échantillon : 2020-2024 ;
- prévisions glissantes à un jour.

La séparation entre estimation et test permet de vérifier la robustesse des conclusions sur une période différente de celle utilisée pour calibrer les modèles.

## Projet 1 - Faits stylisés

Huit propriétés sont analysées :

- asymétrie perte/gain ;
- queues épaisses ;
- autocorrélation des rendements au carré ;
- clusters de volatilité ;
- queues épaisses conditionnelles ;
- effet de levier ;
- saisonnalité ;
- stationnarité.

Les résultats confirment notamment :

- des queues épaisses ;
- une dépendance de la volatilité ;
- des effets ARCH ;
- un effet de levier ;
- la stationnarité des rendements.

## Projet 2 - Modélisation de la VaR

Plusieurs familles sont comparées :

- GARCH ;
- APARCH ;
- GJR-GARCH ;
- EGARCH ;
- IGARCH ;
- ARCH-M.

Le modèle retenu est un **ARMA(0,5)-APARCH(1,1)** avec innovations NIG.

## Résultats du backtesting à 95 %

| Méthode | VaR moyenne | Violations | Kupiec | Christoffersen |
|---|---:|---:|---:|---:|
| ARMA-APARCH-NIG | ≈ 2,82 % | 62 / 62,5 | **0,948** | **0,275** |
| Normale | 2,911 % | 66 / 62 | 0,657 | 0,070 |
| Cornish-Fisher | 2,844 % | 67 / 62 | 0,568 | 0,077 |
| Historique | 2,961 % | 65 / 62 | 0,752 | 0,063 |

Les dépassements observés restent proches du nombre théorique attendu. Les quatre méthodes sont statistiquement acceptables, mais l’approche ARMA-APARCH-NIG reproduit mieux les caractéristiques réelles de la série.

## Technologies

`R` · `RMarkdown` · ARMA · GARCH · APARCH · innovations NIG · VaR · prévision glissante · backtesting · tests de Kupiec et Christoffersen

## Ce que ce projet démontre

- diagnostic économétrique complet ;
- modélisation de la volatilité ;
- sélection de distributions non gaussiennes ;
- prévision hors échantillon ;
- validation objective d’un modèle de risque.

## Limites et améliorations

L’étude porte sur un seul titre et une VaR à 95 %. Une extension pertinente serait d’ajouter une VaR à 99 %, l’Expected Shortfall, des stress tests et une validation sur plusieurs actifs.

## Auteur

Projet réalisé par **Benjamin BAILLET** dans le cadre du Master IREF - Finance quantitative & Actuariat.
