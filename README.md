# Attractivité Économique des Départements Français - Économétrie Avancée avec le modèle 2SLS

> Version avancée du projet "FFR_Attractivite_Economique_OLS" - même jeu de données, mais méthodologie approfondie : vérification complète des hypothèses de Gauss-Markov, traitement de la multicolinéarité, autocorrélation, et estimation par doubles moindres carrés (2SLS) pour traiter l'endogénéité.

---

## Table des matières

- [Contexte](#-contexte)
- [Différences avec la version de base](#-différences-avec-la-version-de-base)
- [Variables](#-variables)
- [Méthodologie](#-méthodologie)
- [Résultats](#-résultats)
- [Limites](#-limites)
- [Sources](#-sources-des-données)

---

## Contexte

La création d'entreprises est un indicateur clé du dynamisme territorial. Cette étude analyse les **disparités de développement économique** entre les 94 départements métropolitains français pour l'année **2022**, en s'appuyant sur les travaux de **Levratto et al. (2013)**.

La variable dépendante est la proportion de créations d'entreprises en 2022 (`pcENT`), mesurée comme le ratio entre les nouvelles immatriculations et le stock d'entreprises existantes en 2021.

> Les territoires d'outre-mer et la Corse ont été exclus en raison de données manquantes et de problèmes de comparabilité.

Pour plus de détails, lire le **[Rapport complet](./Projet_Econometrie_Avancée.pdf)**.

---

## Différences avec la version de base

Cette version approfondit significativement l'analyse initiale sur plusieurs points :

| Aspect | Version de OLS | Version 2SLS |
|--------|----------------|-----------------|
| Multicolinéarité | Détectée (VIF) | Traitée — suppression de `lnnbENT` |
| Normalité des résidus | Testée (Shapiro-Wilk) | Confirmée (p = 0,15) |
| Hétéroscédasticité | Corrigée (White) | Testée (Breusch-Pagan) + correction HC3 |
| Autocorrélation | Non testée | Testée (Durbin-Watson + Breusch-Godfrey) |
| Endogénéité | Non traitée | Test de Hausman-Wu + estimation 2SLS |
| Variables instrumentales | Aucune | `POP2015`, `POPa` |
| R² ajusté final | 0,758 | 0,601 (MCO robuste) |

---

## Variables

### Variable dépendante

| Variable | Description |
|----------|-------------|
| `pcENT` | Proportion de nouvelles entreprises créées en 2022 / stock 2021 (%) |

### Variables explicatives

| Variable | Description | Unité | Source |
|----------|-------------|-------|--------|
| `nbENT` | Nombre d'entreprises 2021 / population 2021 | % | INSEE |
| `POP` | Population totale en 2021 | Milliers | INSEE |
| `DIPL` | Part des diplômés Bac+3 ou plus | % | INSEE |
| `REV` | Revenu annuel médian | Milliers € | INSEE |
| `txCHOM` | Taux de chômage | % | INSEE |
| `METRO` | Présence d'une métropole (1 = Oui, 0 = Non) | Binaire | collectivites-locales.gouv |

### Variables instrumentales

| Variable | Description | Unité |
|----------|-------------|-------|
| `POPa` | Part de la population active en 2021 | % |
| `POP2015` | Population totale en 2015 | Milliers |

---

## Méthodologie

### 1. Statistiques Descriptives

- Statistiques univariées (moyenne, médiane, écart-type, min, max)
- Graphiques de distribution : `pcENT`, `REV`, `txCHOM` sont approximativement symétriques ; `nbENT`, `POP`, `DIPL` présentent une **asymétrie droite marquée** → transformation logarithmique
- Analyse bivariée : relations positives de `pcENT` avec `nbENT`, `POP`, `DIPL` d'allure légèrement exponentielle

### 2. Spécification et Estimation MCO

Le modèle log-log retenu est :

$$\ln(\text{pcENT}_i) = \beta_0 + \beta_1\ln(\text{nbENT}_i) + \beta_2\ln(\text{POP}_i) + \beta_3\ln(\text{DIPL}_i) + \beta_4\text{REV}_i + \beta_5\text{txCHOM}_i + \beta_6\text{METRO}_i + \varepsilon_i$$

Les coefficients des variables log s'interprètent comme des **élasticités**.

### 3. Vérification des Hypothèses de Gauss-Markov

#### 3.1 Multicolinéarité

| Variable | VIF initial | Décision |
|----------|:-----------:|----------|
| `lnnbENT` | 25,8 | Supprimée (corrélation 0,96 avec `lnPOP`) |
| `lnPOP` | 17,6 → **2,4** | Conservée |
| `lnDIPL` | 6,3 → **4,0** | Conservée |
| `REV` | 4,1 | Conservée |
| `txCHOM` | 1,7 | Conservée |
| `METRO` | 1,7 | Conservée |

Après suppression de `lnnbENT`, tous les VIF sont inférieurs à 5.

#### 3.2 Normalité des résidus

| Test | Statistique | p-value | Conclusion |
|------|:-----------:|:-------:|------------|
| Shapiro-Wilk | W = 0,980 | 0,150 | Normalité non rejetée |

#### 3.3 Homoscédasticité

| Test | Statistique | p-value | Conclusion |
|------|:-----------:|:-------:|------------|
| Breusch-Pagan | BP = 25,7 | 0,0001 | Hétéroscédasticité détectée |

→ Correction par **écarts-types robustes de White (HC3)**

#### 3.4 Autocorrélation

| Test | Statistique | p-value | Conclusion |
|------|:-----------:|:-------:|------------|
| Durbin-Watson | DW = 1,564 | 0,013 | Autocorrélation d'ordre 1 |
| Breusch-Godfrey (ordre 2) | LM = 6,80 | 0,033 | Autocorrélation détectée |

### 4. Endogénéité et Variables Instrumentales

#### 4.1 Identification

`lnPOP` est susceptible d'être endogène : la dynamique entrepreneuriale d'un département peut attirer de nouveaux habitants (causalité inverse).

#### 4.2 Pertinence des instruments (1ère étape 2SLS)

| Instrument | F-statistique | p-value | Conclusion |
|------------|:-------------:|:-------:|------------|
| `POPa` | 0,81 | 0,371 | Instrument faible |
| `POP2015` | 105,5 | < 0,001 | Instrument fort (F > 10) |

→ **`POP2015` retenu** comme instrument principal (exogène par construction).

#### 4.3 Test de Hausman-Wu

| Coefficient testé | t-valeur | p-value | Conclusion |
|-------------------|:--------:|:-------:|------------|
| `v_hat` (résidus 1ère étape) | -1,057 | 0,294 | Absence d'endogénéité — MCO retenu |

#### 4.4 Comparaison MCO vs 2SLS

| Variable | MCO | 2SLS |
|----------|:---:|:----:|
| `ln(POP)` | 0,151*** | 0,172*** |
| `ln(DIPL)` | 0,104 | 0,090 |
| `REV` | -0,005 | -0,009 |
| `txCHOM` | 0,029*** | 0,026** |
| `METRO` | -0,076** | -0,090** |
| R² ajusté | **0,601** | 0,597 |

---

## 📈 Résultats

Le modèle MCO final (avec écarts-types robustes HC3) atteint :

- ✅ **R² ajusté = 0,601** — 60 % de la variabilité inter-départementale expliquée
- ✅ **`lnPOP`** significatif au seuil de 0,1 % : élasticité de **+15,1 %**
- ✅ **`txCHOM`** significatif au seuil de 5 % : effet marginal de **+2,9 %** par point
- ✅ **`METRO`** significatif au seuil de 5 % : effet de **-7,6 %**
- ❌ **`lnDIPL`** et **`REV`** non significatifs après correction HC3

### Interprétation économique

- **Population** : les économies d'agglomération favorisent la création d'entreprises via des marchés plus larges et un tissu d'affaires plus dense.
- **Chômage** : l'effet positif reflète le phénomène d'**entrepreneuriat de nécessité** — face à l'absence d'emploi, certains actifs se tournent vers la création.
- **Métropole** : effet négatif surprenant, possiblement dû à un effet de saturation ou de substitution par le salariat qualifié.
- **Revenu médian** : effet ambigu - un revenu élevé peut signaler un dynamisme économique favorable, mais aussi un coût d'opportunité élevé du salariat.

---

## Limites

- **Variables omises** : culture entrepreneuriale locale, qualité des infrastructures, accès au crédit et densité des structures d'accompagnement.
- **Coupe transversale** : absence de dimension temporelle, impossibilité de contrôler les effets fixes départementaux. Une extension en **données de panel** permettrait de corriger ces biais.
- **Endogénéité résiduelle** : `lnnbENT` (exclue pour multicolinéarité) reste potentiellement endogène si réintégrée.

---

## Sources des Données

| Source | Lien |
|--------|------|
| INSEE - Statistiques locales | [statistiques-locales.insee.fr](https://statistiques-locales.insee.fr) |
| data.gouv - Annuaire des entreprises | [annuaire-entreprises.data.gouv.fr](https://annuaire-entreprises.data.gouv.fr) |
| Collectivités locales | [collectivites-locales.gouv.fr](https://www.collectivites-locales.gouv.fr) |
