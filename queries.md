# Requêtes Power Query M
===================================================
- **tests** : données des tests réalisés : OK/NOK/%/Theme (fact et chargée)
- **Suivi_quotidien** : saisie quotidienne de l'avancée des livres par page (fact et chargée)
- **Dim_Theme** : duplication de Suivi_quotidien pour dimensions sur les themes étudiés (chargée)
===================================================

## tests
- **Source** : suivi_N3.xlsx
- **Rôle** : scores réalisés sur les tests Kanji/Lecture/Vocabulaire/Grammaire et comparaison de l'évolution dans le temps et coorélation avec l'apprentisage quotidien

```
let
  Source = Excel.Workbook(File.Contents("D:\suivi_N3.xlsx"), null, true),
  #"Navigation 1" = Source{[Item = "tests", Kind = "Table"]}[Data],
  #"Type de colonne changé" = Table.TransformColumnTypes(#"Navigation 1", {{"Type", type text}, {"Date", type date}, {"OK", Int64.Type}, {"NO", Int64.Type}, {"Nb qts", Int64.Type}, {"ratio", type number}}, "fr"),
    #"Type modifié" = Table.TransformColumnTypes(#"Type de colonne changé",{{"ratio", Percentage.Type}, {"Test N°", type text}}),
    #"Requêtes fusionnées" = Table.NestedJoin(#"Type modifié", {"Type"}, Dim_Theme, {"Theme"}, "Dim_Theme", JoinKind.LeftOuter),
    #"Dim_Theme développé" = Table.ExpandTableColumn(#"Requêtes fusionnées", "Dim_Theme", {"ThemeId"}, {"ThemeId"}),
    #"Colonnes supprimées" = Table.RemoveColumns(#"Dim_Theme développé",{"Type"})
in
  #"Colonnes supprimées"
```

## Suivi_quotidien
-  **Source** : suivi_N3.xlsx
- **Rôle** : suivi quotidien de l'avancée de chaque livre par le nombre de pages étudiées 

```
let
  Source = Excel.Workbook(File.Contents("D:\suivi_N3.xlsx"), null, true),
  #"Navigation 1" = Source{[Item = "Suivi_quotidien", Kind = "Table"]}[Data],
  #"Type de colonne changé" = Table.TransformColumnTypes(#"Navigation 1", {{"Jour", type text}, {"Date", type date}, {"Soumatome Kanji", Int64.Type}, {"Soumatome Vocab", Int64.Type}, {"TRY! Grammaire", Int64.Type}, {"Soumatome Lecture", Int64.Type}, {"Soumatome Ecoute", Int64.Type}, {"Shinkenzen Master Dokkai", Int64.Type}, {"nihongobu", type number}}, "fr"),
    #"Colonnes supprimées" = Table.RemoveColumns(#"Type de colonne changé",{"Jour"}),
    #"Supprimer le tableau croisé dynamique des autres colonnes" = Table.UnpivotOtherColumns(#"Colonnes supprimées", {"Date"}, "Livre", "Avancement"),
    #"Requêtes fusionnées" = Table.NestedJoin(#"Supprimer le tableau croisé dynamique des autres colonnes", {"Livre"}, Dim_Theme, {"Livre"}, "Dim_Theme", JoinKind.LeftOuter),
    #"Dim_Theme développé" = Table.ExpandTableColumn(#"Requêtes fusionnées", "Dim_Theme", {"ThemeId"}, {"ThemeId"}),
    #"Colonnes supprimées1" = Table.RemoveColumns(#"Dim_Theme développé",{"Livre"})
in
    #"Colonnes supprimées1"
```

## Dim_Theme
> 
- **Source** : suivi_N3.xlsx
- **Rôle** : Création d'une dimension par Thème d'étudie et relier les 2 fact

```
let
  Source = Excel.Workbook(File.Contents("D:\suivi_N3.xlsx"), null, true),
  #"Navigation 1" = Source{[Item = "Suivi_quotidien", Kind = "Table"]}[Data],
  #"Type de colonne changé" = Table.TransformColumnTypes(#"Navigation 1", {{"Jour", type text}, {"Date", type date}, {"Soumatome Kanji", Int64.Type}, {"Soumatome Vocab", Int64.Type}, {"TRY! Grammaire", Int64.Type}, {"Soumatome Lecture", Int64.Type}, {"Soumatome Ecoute", Int64.Type}, {"Shinkenzen Master Dokkai", Int64.Type}, {"nihongobu", type number}}, "fr"),
    #"Supprimer le tableau croisé dynamique des autres colonnes" = Table.UnpivotOtherColumns(#"Type de colonne changé", {"Jour", "Date"}, "Livre", "Valeur"),
    #"Autres colonnes supprimées" = Table.SelectColumns(#"Supprimer le tableau croisé dynamique des autres colonnes",{"Livre"}),
    #"Doublons supprimés" = Table.Distinct(#"Autres colonnes supprimées"),
    #"Colonne conditionnelle ajoutée" = Table.AddColumn(#"Doublons supprimés", "Theme", each if [Livre] = "Soumatome Kanji" then "Kanji" else if [Livre] = "Soumatome Vocab" then "Vocabulaire" else if [Livre] = "TRY! Grammaire" then "Grammaire" else if [Livre] = "Soumatome Ecoute" then "Ecoute" else if [Livre] = "Soumatome Lecture" then "Lecture" else if [Livre] = "Shinkenzen Master Dokkai" then "Lecture" else if [Livre] = "nihongobu" then "Grammaire" else null, type text),
    #"Index ajouté" = Table.AddIndexColumn(#"Colonne conditionnelle ajoutée", "ThemeId", 1, 1, Int64.Type)
in
    #"Index ajouté"
```