## SQL Style Guide

Ce guide établit nos standards pour le SQL et est appliqué par le linter SQLFluff et par la revue de code. Les modifications du code cible auxquelles ce guide de style s'applique sont celles effectuées à l'aide de dbt. 

Si vous ne faites pas partie de l'équipe Data ou si vous développez du SQL en dehors de dbt, gardez à l'esprit que les outils de linting peuvent être plus difficiles à appliquer, mais vous êtes invités à suivre les conseils donnés dans ce guide.

### Utilisation

Nous nous attendons à ce que les gens utilisent les règles présentées dans ce guide pendant leur développement.

La mise en application est encore attendue car il n'y a pas de contrôle automatique effectué lors du merge dans le dépôt GitHub.

Plus tard, un contrôle automatique sera exécuté avec chaque changement et sera éventuellement rendu obligatoire pour que le merge passe.

L'intention est que les modèles soient mis à jour dans le nouveau style au fur et à mesure qu'ils sont travaillés, car toutes les règles ne peuvent pas être appliquées automatiquement (comme l'aliasing explicite des colonnes).

Le but est de les mettre à jour au fur et à mesure des interventions à venir sur ces modèles.

### SQLFluff

SQLFLuff est un linter SQL qui fonctionne avec des outils de modélisation comme dbt.

Nous l'utilisons pour définir la structure et le style de base du SQL que nous écrivons et pour confier la révision de cette structure et de ce style aux autres contributrices et contributeur(s) du projet.

SQLFluff est inclus dans l'environnement de développement dbt et utilise le moteur de modélisation dbt pendant le processus de linting. Il peut être utilisé avec la commande suivante:

```console
$ sqlfluff lint models/path/to/file/file-to-lint.sql
```

Une commande dbt peut également être utilisée pour obtenir la liste de fichiers à lint:

```console
$ sqlfluff lint $(dbt list --model model_name --output path)
```

> Si vous écrivez du SQL qui n'est pas modélisé à l'aide de dbt, vous pouvez installer et utiliser SQLFluff directement car il s'agit d'un paquet python autonome.

```console
$ pip install sqlfluff
$ sqlfluff lint path/to/file/file-to-lint.sql
```

SQLFluff inclut une commande `fix` qui appliquera des corrections aux violations de règles quand cela est possible.

**Toutes les violations de règles ne peuvent pas être corrigées automatiquement**; par conséquent, nous vous encourageons à exécuter la commande `lint` après avoir utilisé la commande `fix` pour vous assurer que toutes les violations de règles ont été résolues.

- [Documentation de SQLFluff](https://docs.sqlfluff.com/en/latest/index.html)
- [SQLFluff (Configuration de base)](https://docs.sqlfluff.com/en/latest/configuration.html#default-configuration)


### Quelques Conseils

- Ne pas optimiser pour réduire le nombre de lignes de code, les nouvelles lignes sont bon marché mais [le temps de cerveau ne l'est pas](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/).

- Familiarisez-vous avec [le principe DRY](https://docs.getdbt.com/docs/design-patterns). Exploitez les CTE, le moteur de gabarit jinja et les macros dans dbt, et les snippets dans Sisense. __Si vous tapez deux fois la même ligne, elle doit être gérée à deux endroits.__

- **Soyez cohérent(e).** Même si vous n'êtes pas sûr de la meilleure façon de faire quelque chose, faites-le de la même manière dans tout votre code, il sera plus facile à lire et à modifier si nécessaire.

- **Soyez explicite.** Définir quelque chose de manière explicite permet de s'assurer qu'il fonctionne comme vous l'attendez et c'est plus facile pour la personne suivante, qui peut être vous, lorsque vous êtes explicite en SQL.


### Bonnes Pratiques

- Il ne faut pas utiliser de tabulations, mais uniquement des espaces.
- Enveloppe les longues lignes de code, entre 80 et 100 caractères, sur une nouvelle ligne.
- Comprendre la différence entre les déclarations connexes suivantes et les utiliser de manière appropriée:
  - `UNION ALL` et `UNION`
  - `NOT` et `!` et `<>`
  - `DATE_PART()` et `DATE_TRUNC()`
- Utilisez l'opérateur `AS` pour aliaser une colonne ou une table.
- Préférez `DATEDIFF` pour des modifications de type `date_column + interval_column`. La fonction est plus explicite et fonctionnera pour une plus grande variété de parties de date.
- Préférez `!=` à `<>`. En effet, `!=` est plus courant dans les autres langages de programmation et se lit comme "non égal", ce qui est la façon dont nous sommes plus susceptibles de parler.
- Préférez `LOWER(column) LIKE '%match%'` à `column ILIKE '%Match%'`. Cela réduit le risque que des majuscules égarées entraînent un résultat inattendu.
- Préférez `WHERE` à `HAVING` quand l'un ou l'autre suffirait. C'est généralement plus performant et on peut lire la condition au plus tôt.


### Commentaires

- Pour les commentaires d'une seule ligne dans un modèle, utilisez la syntaxe `--`.
- Pour les commentaires de plusieurs lignes dans un modèle, utilisez la syntaxe `/* */`.
- Respectez la limite de ligne de caractères lorsque vous faites des commentaires. Passez à une nouvelle ligne ou à la documentation du modèle si le commentaire est trop long.
- **Utilisez la documentation du modèle dbt**
- Les calculs effectués en SQL doivent être accompagnés d'une brève description de ce qui se passe et, si possible, d'un lien vers le manuel définissant la métrique (et son mode de calcul).
- Au lieu de laisser des commentaires "TODO", créez une issue GitHub.

> Commentaire SQL = "comment", Documentation dbt = "quoi, pourquoi, pour qui"

### Conventions de nommage


- Un nom de champ ambigu tel que `id`, `name` ou `type` doit toujours être préfixé par ce qu'il identifie ou nomme:

    ```sql
    -- Recommandé
    SELECT
        id    AS account_id,
        name  AS account_name,
        type  AS account_type,
        ...

    -- vs

    -- Pas recommandé
    SELECT
        id,
        name,
        type,
        ...

    ```

- Tous les noms de champs doivent être [snake-cased](https://en.wikipedia.org/wiki/Snake_case):

    ```sql
    -- Recommandé
    SELECT
        dvcecreatedtstamp AS device_created_timestamp
        ...

    -- vs

    -- Pas recommandé
    SELECT
        dvcecreatedtstamp AS DeviceCreatedTimestamp
        ...

    ```
- Les noms de champs booléens doivent commencer par `has_`, `is_`, or `does_`:

    ```sql
    -- Recommandé, comment adapter ça en Français ?
    SELECT
        deleted AS is_deleted,
        sla     AS has_sla
        ...


    -- vs

    -- Pas recommandé
    SELECT
        deleted,
        sla,
        ...

    ```

- Les timestamps doivent se terminer par `_at` et doivent toujours être en UTC..
- Les dates doivent se terminer par `_date`.
- Lorsque vous tronquez les dates, nommez la colonne en fonction de la troncature..

    ```sql

    SELECT
        original_at,                                        -- 2020-01-15 12:15:00.00
        original_date,                                      -- 2020-01-15
        DATE_TRUNC('month',original_date) AS original_month -- 2020-01-01
        ...


    ```

- Évitez les mots clés comme `date` ou `mois` comme nom de colonne.

### Conventions en cas de référence

- Lorsque vous joignez des tables et faites référence à des colonnes des deux tables, tenez compte des points suivants :
  - Référencez le nom complet de la table plutôt qu'un alias lorsque le nom de la table est plus court, peut-être moins de 20 caractères.(essayez de renommer la CTE si possible, et enfin, envisagez d'utiliser un alias descriptif).
  - Toujours qualifier chaque colonne dans l'instruction SELECT avec le nom de la table / l'alias pour faciliter la navigation.  

    ```sql
    -- Recommandé
    SELECT
        budget_forecast_cogs_opex.account_id,
        date_details.fiscal_year,
        date_details.fiscal_quarter,
        date_details.fiscal_quarter_name,
        cost_category.cost_category_level_1,
        cost_category.cost_category_level_2
    FROM budget_forecast_cogs_opex
    LEFT JOIN date_details
        ON date_details.first_day_of_month = budget_forecast_cogs_opex.accounting_period
    LEFT JOIN cost_category
        ON budget_forecast_cogs_opex.unique_account_name = cost_category.unique_account_name

 
    -- vs 

    -- Pas recommandé
    SELECT
        a.account_id,
        b.fiscal_year,
        b.fiscal_quarter,
        b.fiscal_quarter_name,
        c.cost_category_level_1,
        c.cost_category_level_2
    FROM budget_forecast_cogs_opex a
    LEFT JOIN date_details b
        ON b.first_day_of_month = a.accounting_period
    LEFT JOIN cost_category c
        ON b.unique_account_name = c.unique_account_name
    ```

- N'utilisez les guillemets doubles que lorsque cela est nécessaire, par exemple pour les colonnes qui contiennent des caractères spéciaux ou qui sont sensibles à la casse.


    ```sql
        -- Recommandé
        SELECT 
            "First_Name_&_" AS first_name,
            ...

        -- vs

        -- Pas recommandé
        SELECT 
            FIRST_NAME AS first_name,
            ...

    ```

- Préférez les déclarations de jointure explicites.

    ```sql
        -- Recommandé
        SELECT *
        FROM first_table
        INNER JOIN second_table
        ...

        -- vs

        -- Pas recommandé
        SELECT *
        FROM first_table,
            second_table
        ...
    ```


### Common Table Expressions (CTEs)

- Préférez les CTE aux sous-requêtes car [les CTE rendent le SQL plus lisible et sont plus performantes](https://www.alisa-in.tech/post/2019-10-02-ctes/):

    ```sql
    -- Recommandé
    WITH important_list AS (

        SELECT DISTINCT
            specific_column
        FROM other_table
        WHERE specific_column != 'foo'
        
    )

    SELECT
        primary_table.column_1,
        primary_table.column_2
    FROM primary_table
    INNER JOIN important_list
        ON primary_table.column_3 = important_list.specific_column

    -- vs   

    -- Pas recommandé
    SELECT
        primary_table.column_1,
        primary_table.column_2
    FROM primary_table
    WHERE primary_table.column_3 IN (
        SELECT DISTINCT specific_column 
        FROM other_table 
        WHERE specific_column != 'foo')

    ```

- Utilisez les CTE pour référencer d'autres tables.
- Les CTE doivent être placées en haut de vos requêtes.
- Lorsque les performances le permettent, les CTE doivent effectuer une seule opération de sélection.
- Les noms des CTE doivent être aussi concis que possible tout en restant clairs.
    - Evitez les noms longs comme `replace_sfdc_account_id_with_master_record_id` et préférez un nom plus court avec un commentaire dans le CTE. Cela permettra d'éviter les alias de table dans les jointures.
- Les CTE dont la logique est confuse ou complexe doivent être commentées dans le fichier et documentées dans les docs dbt.
- Les CTE qui sont dupliquées à travers les modèles devraient être extraites dans leurs propres fichiers.


### Types de Données

- Utilisez les types de données par défaut et non les alias. Consultez le [résumé BigQuery des types de données] (https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types?hl=fr) pour plus de détails. Les valeurs par défaut sont:
    - `NUMBER` plutôt que `DECIMAL`, `NUMERIC`, `INTEGER`, `BIGINT`, etc.
    - `FLOAT` plutôt que `DOUBLE`, `REAL`, etc.
    - `VARCHAR` plutôt que `STRING`, `TEXT`, etc.
    - `TIMESTAMP` plutôt que `DATETIME`

> L'exception à cette règle concerne les timestamps. Préférez `TIMESTAMP` à `TIME`. Notez que la valeur par défaut de `TIMESTAMP` est `TIMESTAMP_NTZ` qui n'inclut pas de fuseau horaire.

### Fonctions

- Préférer `COALESCE` à `IFNULL`.
- Préférer `IF` à une instruction `CASE` en une seule ligne:

    ```sql
    -- Recommandé
    SELECT 
        IF(column_1 = 'foo', column_2,column_3) AS logic_switch,
        ...

    -- vs 

    -- Non recommandé
    SELECT
        CASE
            WHEN column_1 = 'foo' THEN column_2
            ELSE column_3
        END AS logic_switch,
        ...
    ```
- Préférer `IF` pour sélectionner une instruction booléenne:

    ```sql
    -- Recommandé
    SELECT 
        IF(amount < 10,TRUE,FALSE) AS is_less_than_ten,
        ...
    -- vs

    -- Non recommandé
    SELECT 
        (amount < 10) AS is_less_than_ten,
        ...
    ```

- Simplifier les déclarations répétitives `CASE` lorsque cela est possible:

    ```sql
    -- Recommandé
    SELECT
        CASE field_id
            WHEN 1 THEN 'date'
            WHEN 2 THEN 'integer'
            WHEN 3 THEN 'currency'
            WHEN 4 THEN 'boolean'
            WHEN 5 THEN 'variant'
            WHEN 6 THEN 'text'
        END AS field_type,
        ...

    -- vs 

    -- Non recommandé
    SELECT 
        CASE
            WHEN field_id = 1 THEN 'date'
            WHEN field_id = 2 THEN 'integer'
            WHEN field_id = 3 THEN 'currency'
            WHEN field_id = 4 THEN 'boolean'
            WHEN field_id = 5 THEN 'variant'
            WHEN field_id = 6 THEN 'text'
        END AS field_type,
        ...
    
    ```
- Préférez la fonction date explicite à `date_part`, mais préférez `date_part` à `extract`:

    ```sql
    DAYOFWEEK(created_at) > DATE_PART(dayofweek, 'created_at') > EXTRACT(dow FROM created_at)
    ```

> De manière générale, tout ce qui réduit le temps de cerveau disponible est recommandé.

### Code d'exemple

Cet exemple de code a été traité par SQLFluff linter et le guide de style a été appliqué.

```sql

WITH my_data AS (

  SELECT *
  FROM prod.my_data
  WHERE filter = 'my_filter'

),

some_cte AS (

  SELECT DISTINCT
    id AS other_id,
    other_field_1,
    other_field_2,
    date_field_at,
    data_by_row,
    field_4,
    field_5,
    LAG(
      other_field_2
    ) OVER (PARTITION BY other_id, other_field_1 ORDER BY 5) AS previous_other_field_2
  FROM prod.my_other_data

),
/*
Il s'agit d'un commentaire très long : C'est une bonne pratique de laisser des commentaires dans le code pour code pour expliquer la logique complexe des CTE ou la logique métier qui peut ne pas être quelqu'un qui n'a pas une connaissance approfondie de la source de données. Cela peut aider les nouveaux utilisateurs à se familiariser rapidement avec le code.
*/

final AS (

  SELECT
    -- Il s'agit d'un commentaire mono-ligne
    my_data.field_1 AS detailed_field_1,
    my_data.field_2 AS detailed_field_2,
    my_data.detailed_field_3,
    DATE_TRUNC('month', some_cte.date_field_at) AS date_field_month,
    some_cte.data_by_row['id']::NUMBER AS id_field,
    IFF(my_data.detailed_field_3 > my_data.field_2, TRUE, FALSE) AS is_boolian,
    CASE
      WHEN
        my_data.cancellation_date IS NULL
        AND my_data.expiration_date IS NOT NULL
        THEN my_data.expiration_date
      WHEN my_data.cancellation_date IS NULL
        THEN my_data.start_date + 7 -- There is a reason for this number
      ELSE my_data.cancellation_date
    END AS adjusted_cancellation_date,
    SUM(some_cte.field_4) AS field_4_sum,
    MAX(some_cte.field_5) AS field_5_max
  FROM my_data
  LEFT JOIN some_cte
    ON my_data.id = some_cte.id
  WHERE my_data.field_1 = 'abc'
    AND (my_data.field_2 = 'def' OR my_data.field_2 = 'ghi')
  GROUP BY 1, 2, 3, 4, 5, 6
  HAVING COUNT(*) > 1
  ORDER BY 8 DESC
)

SELECT *
FROM final

```

### Autres Guides populaires

- [Brooklyn Data Co](https://github.com/brooklyn-data/co/blob/master/sql_style_guide.md)
- [Fishtown Analytics](https://github.com/fishtown-analytics/corp/blob/master/dbt_coding_conventions.md#sql-style-guide)
- [Matt Mazur](https://github.com/mattm/sql-style-guide)
- [Kickstarter](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
