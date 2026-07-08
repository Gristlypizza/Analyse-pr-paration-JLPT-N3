# Mesures DAX
> **Table** : `_Mesures`
---
## Dossier suivi

### jours etude
> Nombre total d'entrées dans le fichier de suivi Excel (peut compter plusieurs fois une même date si plusieurs livres/thèmes travaillés le même jour ex: 2pages Kanji + 1 page Vocab)
```
jours etude = COUNTROWS(Suivi_quotidien)
```
 
### jours etude unique
> Nombre de jours distincts d'étude en complément de COUNTROWS pour calculer le nombre de jours ou il y a eu au moins 1 thème étudié
```
jours etude unique = DISTINCTCOUNT(Suivi_quotidien[Date])
```

### durée d'etude
> période de révision en mois 
```
durée d'etude = 
DATEDIFF(
    MIN( Suivi_quotidien[Date] ),
    MAX( Suivi_quotidien[Date] )
    ,
    MONTH
)
```

### avancement par theme
> volume d'avancée par theme étudié selon la période
```
avancement par theme =
CALCULATE(
    MAX(Suivi_quotidien[Avancement]),
    FILTER(
        ALL(DimDate),
        DimDate[Date] <= MAX(DimDate[Date])
    )
)
```
 
### avancement quotidien
> volume hebdomadaire étudié, visualisation sur le nuage de points volume/résultat mocks
```
avancement quotidien =
VAR semEnCours = [avancement par theme]
VAR semPrec =
    CALCULATE(
        [avancement par theme],
        DATEADD(DimDate[Date], -7, DAY)
    )
RETURN
IF(
    ISBLANK(semEnCours),
    BLANK(),
    semEnCours - semPrec
)
```
 ---
 
## Dossier résultats
 
### nb test fait
> nombre de lignes la table "tests" qui contient tous les tests réalisés avec note OK/NOK/% par date de réalisation et par thème
```
nb test fait = COUNTROWS(tests)
```
 
### taux reussite tests
```
taux reussite tests =
DIVIDE(
    SUM(tests[OK]),
    SUM(tests[Nb qts]),
    0
)
```