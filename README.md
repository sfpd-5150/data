# data for sfmentalcrisis 

link to sqlite file:

http://stash.compjour.org/theses/sfmentalcrisis/sfpddata.sqlite

## Incidents data

Counting overall incidents

```sql
SELECT SUBSTR(date, -4) AS year, COUNT(*)
FROM incidents
WHERE year < '2017'
GROUP BY year
```

Time-series of 5150 cases:

```sql
SELECT SUBSTR(date, -4) AS year, COUNT(*)
FROM incidents
WHERE year < '2017'
  AND descript = 'AIDED CASE, MENTAL DISTURBED'
GROUP BY year
ORDER BY year 
```

2007 to 2016 delta count:

```sql
WITH tx(shape_name, xcounts) AS 
        (SELECT shapes.name10 AS shape_name,
            COUNT(*)
        FROM
            sf_tracts AS shapes
        LEFT JOIN
            sf_aided_mental AS dots
                ON ST_CONTAINS(shapes.the_geom, dots.the_geom)
                    AND dots.year = '2007'
        GROUP BY shape_name),

    ty(shape_name, ycounts) AS 
        (SELECT shapes.name10 AS shape_name,
            COUNT(*)
        FROM
            sf_tracts AS shapes
        LEFT JOIN
            sf_aided_mental AS dots
                ON ST_CONTAINS(shapes.the_geom, dots.the_geom)
                    AND dots.year = '2016'
        GROUP BY shape_name)

SELECT
    shapes.name10 AS shapename,
    xcounts,
    ycounts,
    ycounts - xcounts / xcounts AS delta,
    shapes.cartodb_id,
    shapes.the_geom_webmercator
FROM
    sf_tracts AS shapes
LEFT JOIN tx 
    ON tx.shape_name = shapes.name10
LEFT JOIN ty
    ON ty.shape_name = shapes.name10

where ABS(ycounts - xcounts / xcounts) > 10
ORDER BY delta DESC
```






