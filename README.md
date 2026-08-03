# Schneider Electric - Rendements, volatilité, Value at Risk et backtesting

![R](https://img.shields.io/badge/R-Analyse%20statistique-276DC3?logo=r&logoColor=white)
![R Markdown](https://img.shields.io/badge/R%20Markdown-Rapports-75AADB)
![Économétrie](https://img.shields.io/badge/Économétrie-Séries%20temporelles-0B1F3A)
![Finance](https://img.shields.io/badge/Finance-Value%20at%20Risk-D8A633)
![Statut](https://img.shields.io/badge/Statut-Terminé-2E8B57)

Projet académique en deux volets complémentaires consacré à l’étude des **rendements logarithmiques**, de la **volatilité** et du **risque de marché** de l’action Schneider Electric.

L’analyse porte sur l’action Schneider Electric cotée à Paris sous le symbole :

```text
SU.PA
```

Le premier projet étudie les principales propriétés statistiques et économétriques des rendements logarithmiques.

Le second projet prolonge cette analyse par la modélisation de la moyenne et de la volatilité conditionnelles, l’estimation de plusieurs Value at Risk à 95 % et leur validation par backtesting.

> Projet réalisé par Benjamin Baillet dans le cadre d’un enseignement d’économétrie des séries temporelles, de Value at Risk et de backtesting à l’Université de Bordeaux.

---

## Sommaire

- [Vue d’ensemble](#vue-densemble)
- [Articulation des deux projets](#articulation-des-deux-projets)
- [Objectifs](#objectifs)
- [Données](#données)
- [Découpage des échantillons](#découpage-des-échantillons)
- [Projet 1 - Caractéristiques des rendements](#projet-1---caractéristiques-des-rendements)
- [Projet 2 - Value at Risk et backtesting](#projet-2---value-at-risk-et-backtesting)
- [Principaux résultats](#principaux-résultats)
- [Modèle final retenu](#modèle-final-retenu)
- [Résultats des Value at Risk](#résultats-des-value-at-risk)
- [Backtesting](#backtesting)
- [Méthodologie](#méthodologie)
- [Outils et packages](#outils-et-packages)
- [Structure du dépôt](#structure-du-dépôt)
- [Reproduire le projet](#reproduire-le-projet)
- [Compétences démontrées](#compétences-démontrées)
- [Limites](#limites)
- [Auteur](#auteur)

---

# Vue d’ensemble

Les deux projets forment une étude progressive du risque de marché de l’action Schneider Electric.

```text
Données boursières quotidiennes de Schneider Electric
                         ↓
Calcul des rendements logarithmiques
                         ↓
Projet 1 : étude des propriétés des rendements
                         ↓
Sélection d’un modèle pour la moyenne conditionnelle
                         ↓
Projet 2 : modélisation ARMA-GARCH de la volatilité
                         ↓
Estimation de plusieurs Value at Risk à 95 %
                         ↓
Backtesting avec les tests de Kupiec et Christoffersen
```

Les rapports ont été réalisés en **R Markdown** afin de réunir dans un même document :

- le code R ;
- les résultats statistiques ;
- les graphiques ;
- les équations ;
- les interprétations ;
- les conclusions.

---

# Articulation des deux projets

## Projet 1

Le premier projet étudie huit propriétés classiques des rendements logarithmiques des actifs financiers :

1. asymétrie entre pertes et gains ;
2. queues de distribution épaisses ;
3. autocorrélation des rendements au carré ;
4. clusters de volatilité ;
5. queues épaisses conditionnelles ;
6. effet de levier ;
7. saisonnalité ;
8. stationnarité.

Il permet également de sélectionner un modèle pour représenter la moyenne conditionnelle des rendements.

## Projet 2

Le deuxième projet utilise les conclusions du premier projet pour :

- sélectionner une distribution adaptée aux rendements ;
- tester plusieurs familles de modèles GARCH ;
- modéliser la volatilité conditionnelle ;
- estimer une Value at Risk paramétrique ;
- comparer cette estimation à des VaR plus classiques ;
- réaliser un backtesting hors échantillon.

---

# Objectifs

L’étude cherche à répondre aux questions suivantes :

- Les rendements logarithmiques suivent-ils une loi normale ?
- Leur moyenne est-elle statistiquement nulle ?
- La distribution est-elle symétrique ?
- Les valeurs extrêmes sont-elles plus fréquentes que sous une loi normale ?
- Les rendements sont-ils autocorrélés ?
- Les rendements au carré sont-ils autocorrélés ?
- La volatilité se regroupe-t-elle en périodes de faible et forte intensité ?
- Les nouvelles négatives ont-elles davantage d’effet sur la volatilité ?
- Existe-t-il une saisonnalité journalière ou mensuelle ?
- Les séries de rendements sont-elles stationnaires ?
- Quel modèle ARMA représente le mieux la moyenne conditionnelle ?
- Quel modèle GARCH représente le mieux la volatilité conditionnelle ?
- Quelle perte quotidienne maximale peut être estimée avec un niveau de confiance de 95 % ?
- Les violations de la VaR sont-elles suffisamment peu nombreuses et indépendantes ?

---

# Données

## Actif étudié

```text
Entreprise : Schneider Electric
Ticker Yahoo Finance : SU.PA
Fréquence : quotidienne
Type de cours : cours ajusté
Type de rendement : rendement logarithmique
```

Les données sont récupérées avec le package R `yfR`.

```r
df_yf <- yf_get(
  tickers = "SU.PA",
  first_date = "2010-01-04",
  last_date = "2024-11-13",
  freq_data = "daily",
  type_return = "log"
)
```

Le rendement logarithmique est défini par :

```math
r_t = \log(P_t) - \log(P_{t-1})
```

avec :

- \(P_t\) : cours ajusté à la date \(t\) ;
- \(r_t\) : rendement logarithmique entre \(t-1\) et \(t\).

Le Projet 1 utilise une extraction allant jusqu’au 23 septembre 2024.

Le Projet 2 actualise l’extraction jusqu’au 13 novembre 2024.

Le dossier `donnees` contient une version locale des observations afin de conserver une photographie reproductible des données utilisées.

---

# Découpage des échantillons

Les données sont séparées en deux ensembles chronologiques.

## Ensemble d’estimation

```text
Du 4 janvier 2010 au 31 décembre 2019
```

Cet ensemble est utilisé pour :

- étudier les caractéristiques statistiques ;
- sélectionner les modèles ;
- estimer les paramètres ;
- construire les prévisions de volatilité et de VaR.

Il est noté :

```text
rte
```

## Ensemble de test

```text
Du 2 janvier 2020 au 13 novembre 2024
```

Cet ensemble est utilisé pour :

- vérifier la stabilité des propriétés observées ;
- tester les modèles hors échantillon ;
- réaliser les prévisions glissantes ;
- comparer les VaR estimées aux pertes réellement observées.

Il est noté :

```text
rtt
```

Cette séparation évite de tester le modèle uniquement sur les données ayant servi à son estimation.

---

# Projet 1 - Caractéristiques des rendements

## 1. Analyse descriptive

Le projet commence par représenter :

- l’évolution du cours ajusté ;
- les variations du cours ;
- les rendements logarithmiques ;
- la distribution des rendements ;
- la comparaison avec une loi normale.

Le cours de Schneider Electric présente une tendance haussière marquée sur la période.

Les rendements fluctuent autour de zéro, avec des périodes pendant lesquelles leur amplitude augmente fortement.

Ces périodes constituent des clusters de volatilité, notamment autour des années marquées par de fortes perturbations financières.

---

## 2. Moyenne empirique

Un test t de Student est réalisé afin de vérifier si la moyenne des rendements est statistiquement nulle.

Pour l’ensemble d’estimation :

```text
p-value = 0,2379
```

La moyenne n’est donc pas significativement différente de zéro au seuil de 5 %.

---

## 3. Asymétrie perte/gain

La symétrie de la distribution est étudiée avec le test de D’Agostino.

### Ensemble d’estimation

```text
Skewness ≈ -0,047
p-value = 0,3298
```

La skewness n’est pas significativement différente de zéro.

### Ensemble de test

```text
Skewness ≈ -0,3866
p-value ≈ 9,029 × 10⁻⁸
```

La distribution de l’ensemble de test est significativement asymétrique.

La propriété d’asymétrie n’est donc pas identique sur les deux sous-périodes.

---

## 4. Queues de distribution épaisses

Le test d’Anscombe est utilisé pour comparer la kurtosis à celle d’une loi normale.

### Ensemble d’estimation

```text
Kurtosis ≈ 5,5215
p-value < 2,2 × 10⁻¹⁶
```

### Ensemble de test

```text
Kurtosis ≈ 9,622
p-value < 2,2 × 10⁻¹⁶
```

Dans les deux ensembles, la kurtosis est nettement supérieure à 3.

Les distributions sont donc leptokurtiques et présentent davantage de valeurs extrêmes qu’une loi normale.

---

## 5. Autocorrélation des rendements au carré

Les fonctions d’autocorrélation montrent que :

- les rendements présentent peu d’autocorrélation directe ;
- les rendements au carré présentent une autocorrélation nettement plus forte.

Le test de Ljung-Box confirme la présence d’autocorrélation dans les rendements au carré.

Cette propriété justifie l’utilisation de modèles de volatilité conditionnelle de type ARCH ou GARCH.

---

## 6. Modélisation de la moyenne conditionnelle

L’EACF est utilisée pour proposer différentes valeurs de \(p\) et \(q\) dans les modèles ARMA.

Plusieurs modèles sont testés :

- MA(1) ;
- MA(2) ;
- MA(3) ;
- MA(4) ;
- MA(5) ;
- ARMA(1,1) ;
- ARMA(1,2) ;
- ARMA(2,2) ;
- ARMA(3,1).

Les modèles sont comparés selon :

- la significativité des coefficients ;
- la nullité de la moyenne des résidus ;
- l’absence d’autocorrélation résiduelle ;
- le critère d’information BIC.

Le modèle retenu sur l’ensemble d’estimation est :

```text
MA(5)
```

Son BIC est :

```text
-13 275,85
```

Il s’agit du modèle viable présentant le BIC le plus faible parmi les modèles conservés.

---

## 7. Clusters de volatilité

Les tests ARCH-LM sont significatifs aux ordres 1, 2 et 20.

Les rendements présentent donc :

- des effets ARCH ;
- une hétéroscédasticité conditionnelle ;
- des regroupements temporels de volatilité.

Les périodes de forte volatilité ont tendance à être suivies par d’autres périodes de forte volatilité.

---

## 8. Queues épaisses conditionnelles

Plusieurs spécifications sont étudiées :

- GARCH(1,1) ;
- GARCH(1,2) ;
- GARCH(2,1).

Les modèles d’ordres supérieurs ne fournissent pas de coefficients entièrement significatifs.

Le modèle GARCH(1,1) est conservé pour l’analyse, mais il ne parvient pas à éliminer complètement l’hétéroscédasticité conditionnelle résiduelle.

La conclusion concernant les queues épaisses conditionnelles reste donc prudente et non concluante dans le Projet 1.

---

## 9. Effet de levier

L’effet de levier correspond à une réaction potentiellement plus importante de la volatilité après une mauvaise nouvelle qu’après une bonne nouvelle de même amplitude.

Les tests réalisés concluent à la présence d’un effet de levier dans les deux sous-échantillons.

---

## 10. Saisonnalité

Deux formes de saisonnalité sont étudiées :

- effet lié au jour de la semaine ;
- effet janvier.

Aucune saisonnalité suffisamment robuste n’est retenue dans les deux ensembles.

---

## 11. Stationnarité

Plusieurs tests de racine unitaire et de rupture sont utilisés :

- Dickey-Fuller ;
- Dickey-Fuller augmenté ;
- sélection par MAIC et BIC ;
- procédure de Perron ;
- Zivot-Andrews ;
- Lee-Strazicich.

Les conclusions retiennent la stationnarité des rendements logarithmiques dans les ensembles étudiés.

---

## Résumé des huit propriétés

| Propriété | Ensemble d’estimation | Ensemble de test |
|---|---:|---:|
| Asymétrie perte/gain | Validée | Non validée |
| Queues épaisses | Validée | Validée |
| Autocorrélation des carrés | Validée | Validée |
| Clusters de volatilité | Validée | Validée |
| Queues épaisses conditionnelles | Non concluant | Non concluant |
| Effet de levier | Validé | Validé |
| Saisonnalité | Non détectée | Non détectée |
| Stationnarité | Validée | Validée |

---

# Projet 2 - Value at Risk et backtesting

Le deuxième projet prolonge directement le premier.

Les résultats précédents indiquent notamment :

- des queues de distribution épaisses ;
- des effets ARCH ;
- des clusters de volatilité ;
- un effet de levier ;
- une absence de saisonnalité robuste ;
- une série de rendements stationnaire.

Ces observations justifient l’utilisation de modèles ARMA-GARCH avec des distributions capables de représenter les valeurs extrêmes.

---

## Distributions étudiées

Plusieurs distributions sont comparées :

- loi normale ;
- loi de Student asymétrique ;
- loi gaussienne inverse asymétrique, ou NIG ;
- loi hyperbolique asymétrique ;
- loi hyperbolique généralisée ;
- loi hyperbolique généralisée skew-t, ou GHST.

La distribution NIG est celle qui se rapproche le mieux de la distribution empirique des rendements parmi les distributions initialement comparées.

Les distributions NIG et GHST sont ensuite conservées pour les estimations GARCH.

---

## Familles de modèles GARCH étudiées

Six familles de modèles sont testées :

1. APARCH ;
2. GJR-GARCH ;
3. EGARCH ;
4. IGARCH ;
5. ARCH-M ;
6. GARCH standard.

Chaque famille est estimée avec différentes distributions afin de comparer :

- la significativité des coefficients ;
- l’autocorrélation des résidus standardisés ;
- les effets ARCH résiduels ;
- la stabilité des paramètres ;
- l’asymétrie de l’impact des nouvelles ;
- l’adéquation de la distribution ;
- les critères d’information.

---

# Modèle final retenu

Le modèle retenu est :

```text
ARMA(0,5)-APARCH(1,1)
```

avec une distribution :

```text
NIG - Gaussienne inverse asymétrique
```

Les contraintes retenues sont :

```text
delta = 1
ma1 = 0
ma3 = 0
mu = 0
```

Ce modèle associe :

- un modèle MA d’ordre 5 pour la moyenne conditionnelle ;
- un modèle APARCH(1,1) pour la volatilité conditionnelle ;
- une distribution NIG pour mieux représenter les queues épaisses et l’asymétrie.

Le modèle est ensuite utilisé dans une procédure de prévision glissante hors échantillon.

---

# Value at Risk

La Value at Risk représente la perte maximale potentielle qui ne devrait être dépassée qu’avec une probabilité donnée sur un horizon donné.

Le projet étudie une VaR quotidienne à :

```text
95 % de confiance
```

Une VaR de 2,9 % signifie qu’avec les hypothèses retenues, la perte quotidienne ne devrait dépasser environ 2,9 % que dans 5 % des observations.

Quatre méthodes sont comparées :

1. VaR paramétrique issue du modèle ARMA-APARCH ;
2. VaR normale ;
3. VaR de Cornish-Fisher ;
4. VaR historique.

---

# Résultats des Value at Risk

| Méthode | VaR estimée | Violations attendues | Violations observées |
|---|---:|---:|---:|
| Paramétrique ARMA-APARCH | En moyenne 2,82 % | 62,5 | 62 |
| Normale | 2,911399 % | 62 | 66 |
| Cornish-Fisher | 2,844005 % | 62 | 67 |
| Historique | 2,961336 % | 62 | 65 |

La VaR paramétrique varie quotidiennement avec la volatilité conditionnelle estimée.

Sa moyenne sur l’ensemble de test est d’environ :

```text
2,82 %
```

---

# Backtesting

Le backtesting consiste à comparer chaque prévision de VaR avec le rendement effectivement observé le jour suivant.

Une violation apparaît lorsque la perte réelle dépasse la perte maximale prévue par la VaR.

Deux tests sont utilisés.

## Test de Kupiec

Le test de Kupiec vérifie si la proportion totale de violations correspond au taux attendu.

À 95 % de confiance, environ 5 % des observations peuvent dépasser la VaR.

## Test de Christoffersen

Le test de Christoffersen vérifie notamment si les violations sont indépendantes dans le temps.

Une VaR peut avoir un nombre total de violations correct tout en concentrant ces violations sur une même période. Le test de Christoffersen permet de détecter ce problème.

## Résultats

| Méthode | P-value Kupiec | P-value Christoffersen |
|---|---:|---:|
| Paramétrique ARMA-APARCH | 0,9480 | 0,2750 |
| Normale | 0,6572 | 0,0703 |
| Cornish-Fisher | 0,5680 | 0,0772 |
| Historique | 0,7521 | 0,0626 |

Toutes les p-values du test de Christoffersen sont supérieures à 5 %.

Dans le cadre des tests réalisés, l’hypothèse de validité des différentes VaR n’est donc pas rejetée.

La VaR paramétrique fournit le résultat le plus proche du nombre théorique de violations :

```text
62 violations observées pour 62,5 attendues
```

---

# Principaux résultats

## Projet 1

- moyenne des rendements statistiquement nulle ;
- distributions non normales ;
- queues épaisses sur les deux sous-périodes ;
- autocorrélation des rendements au carré ;
- présence de clusters de volatilité ;
- effet de levier ;
- absence de saisonnalité robuste ;
- stationnarité des rendements ;
- sélection d’un modèle MA(5).

## Projet 2

- distribution NIG adaptée aux rendements ;
- comparaison de six familles de modèles GARCH ;
- sélection d’un ARMA(0,5)-APARCH(1,1)-NIG ;
- VaR paramétrique moyenne d’environ 2,82 % ;
- VaR normale d’environ 2,91 % ;
- VaR Cornish-Fisher d’environ 2,84 % ;
- VaR historique d’environ 2,96 % ;
- résultats de backtesting acceptables pour les quatre méthodes.

---

# Méthodologie

## Tests statistiques et économétriques

Le projet mobilise notamment :

- test t de Student ;
- test de D’Agostino ;
- test d’Anscombe ;
- ACF ;
- EACF ;
- test de Ljung-Box ;
- test ARCH-LM d’Engle ;
- critères AIC et BIC ;
- test de Dickey-Fuller ;
- test de Dickey-Fuller augmenté ;
- test de Zivot-Andrews ;
- test de Lee-Strazicich ;
- tests de stabilité de Nyblom ;
- tests d’asymétrie et de sign bias ;
- test d’adéquation de Pearson ;
- test de Kupiec ;
- test de Christoffersen.

## Modèles utilisés

- AR ;
- MA ;
- ARMA ;
- ARCH ;
- GARCH ;
- APARCH ;
- GJR-GARCH ;
- EGARCH ;
- IGARCH ;
- ARCH-M.

## Méthodes de VaR

- VaR paramétrique ;
- VaR normale ;
- VaR de Cornish-Fisher ;
- VaR historique ;
- estimation par fenêtre glissante ;
- backtesting hors échantillon.

---

# Outils et packages

## Outils

- R ;
- RStudio ;
- R Markdown ;
- HTML ;
- PDF ;
- Yahoo Finance ;
- GitHub.

## Packages du Projet 1

```r
library(yfR)
library(scales)
library(forecast)
library(moments)
library(TSA)
library(lmtest)
library(FinTS)
library(tseries)
library(CADFtest)
library(urca)
library(fGarch)
library(rcompanion)
```

## Packages du Projet 2

```r
library(yfR)
library(ghyp)
library(SkewHyperbolic)
library(rugarch)
library(parallel)
library(zoo)
library(PerformanceAnalytics)
```

---

# Structure du dépôt

```text
schneider-electric-rendements-var-backtesting/
│
├── README.md
│
├── projet-1-caracteristiques-rendements/
│   ├── BAILLET_Projet1.Rmd
│
├── projet-2-var-backtesting/
│   ├── BAILLET_Projet2.Rmd
│
└── documentation/
    └── Synthese_Value_At_Risk.pdf
```

---

# Reproduire le projet

## Prérequis

- R ;
- RStudio ;
- R Markdown ;
- Pandoc ;
- connexion Internet pour télécharger les données avec `yfR`.

## Installer les packages

```r
install.packages(c(
  "rmarkdown",
  "yfR",
  "scales",
  "forecast",
  "moments",
  "TSA",
  "lmtest",
  "FinTS",
  "tseries",
  "CADFtest",
  "urca",
  "fGarch",
  "rcompanion",
  "ghyp",
  "SkewHyperbolic",
  "rugarch",
  "zoo",
  "PerformanceAnalytics"
))
```

## Générer le Projet 1

```r
rmarkdown::render(
  "projet-1-caracteristiques-rendements/BAILLET_Projet1.Rmd"
)
```

## Générer le Projet 2

```r
rmarkdown::render(
  "projet-2-var-backtesting/BAILLET_Projet2.Rmd"
)
```

Le Projet 2 utilise des estimations glissantes et plusieurs modèles GARCH. Son exécution peut donc être sensiblement plus longue.

Le package `parallel` est utilisé pour mobiliser plusieurs cœurs du processeur pendant certaines estimations.

---

# Reproductibilité des données

Les scripts téléchargent automatiquement les données de marché avec `yfR`.

Les résultats peuvent évoluer légèrement si :

- Yahoo Finance corrige son historique ;
- la date de fin de l’extraction est modifiée ;
- les versions des packages changent ;
- les méthodes d’optimisation convergent vers une solution différente.

Le dossier `donnees` contient donc une copie locale des données utilisées afin de faciliter la reproduction des résultats historiques.

---

# Compétences démontrées

## R et R Markdown

- récupération de données financières ;
- manipulation de séries temporelles ;
- automatisation de rapports ;
- production de graphiques ;
- création de tableaux HTML ;
- intégration du code et des résultats ;
- génération de documents HTML et PDF.

## Statistiques

- analyse descriptive ;
- tests d’hypothèses ;
- analyse de la skewness ;
- analyse de la kurtosis ;
- autocorrélation ;
- sélection de modèles ;
- analyse de résidus ;
- comparaison de distributions.

## Économétrie des séries temporelles

- modèles ARMA ;
- modèles ARCH et GARCH ;
- volatilité conditionnelle ;
- effet de levier ;
- stationnarité ;
- ruptures structurelles ;
- estimation hors échantillon ;
- critères d’information.

## Risque de marché

- Value at Risk ;
- volatilité ;
- quantiles de pertes ;
- fenêtre glissante ;
- exceptions de VaR ;
- couverture non conditionnelle ;
- indépendance des violations ;
- backtesting.

## Communication

- interprétation des résultats ;
- rédaction mathématique ;
- documentation des hypothèses ;
- présentation de modèles complexes ;
- synthèse des conclusions.

---

# Limites

- Le projet étudie une seule action.
- Le risque de liquidité n’est pas modélisé.
- Les coûts de transaction ne sont pas pris en compte.
- La VaR ne renseigne pas sur l’ampleur des pertes au-delà du quantile.
- Une Expected Shortfall pourrait compléter l’analyse.
- Les résultats dépendent de la fenêtre temporelle retenue.
- Les modèles sont sensibles aux choix de distribution et de spécification.
- La période de test inclut des événements exceptionnels, notamment la crise sanitaire.
- Les données récupérées en ligne peuvent être révisées.
- Certains diagnostics du Projet 1 restent non concluants.
- L’absence de rejet d’un test de backtesting ne garantit pas qu’un modèle soit parfait.
- Les résultats ne doivent pas être interprétés comme une prévision certaine des pertes futures.

---

# Pistes d’amélioration

- calculer l’Expected Shortfall ;
- comparer plusieurs niveaux de confiance ;
- étudier une VaR à horizon de plusieurs jours ;
- comparer plusieurs actions ou indices ;
- construire un portefeuille diversifié ;
- intégrer des corrélations dynamiques ;
- utiliser des modèles DCC-GARCH ;
- réaliser un stress testing ;
- utiliser une validation croisée temporelle ;
- automatiser la mise à jour des données ;
- comparer les performances de prévision sur plusieurs sous-périodes.

---

# Avertissement

Ce dépôt présente un travail académique réalisé à des fins pédagogiques.

Il ne constitue :

- ni un conseil en investissement ;
- ni une recommandation d’achat ou de vente ;
- ni une mesure exhaustive du risque financier ;
- ni une garantie concernant les pertes futures.

---

# Auteur

**Benjamin Baillet**

Master IREF - Université de Bordeaux

Compétences principales :

- R ;
- RStudio ;
- séries temporelles ;
- économétrie ;
- Value at Risk ;
- backtesting ;
- analyse financière ;
- Python ;
- SQL ;
- Power BI.
