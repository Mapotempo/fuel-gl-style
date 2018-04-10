## Fuel map layer

### Build the data

```
bzcat Prix\ des\ carburants\ en\ France-Analyser_Merge_Fuel_FR.byOSM.csv.bz2 | csvcut -c 'name,operator,brand,lon,lat' > fuel3.csv

csvsql --query "
select
 name,
 case
   when mbrand like '%total%' then 'total'
   when mbrand like '%intermarché%' then 'intermarché'
   when mbrand like '%carrefour%' then 'carrefour'
   when mbrand like '%super u%' then 'super u'
   when mbrand like '%avia%' then 'avia'
   when mbrand like '%esso%' then 'esso'
   when mbrand like 'bp' like '% bp' then 'bp'
   when mbrand like '%shell%' then 'shell'
   when mbrand like '%leclerc%' then 'leclerc'
   when mbrand like '%mousquetaires%' then 'intermarché'
   else NULL
 end as brand,
 lon,
 lat
from
 (select coalesce(brand, name, operator) as name, lower(coalesce(brand, '') || coalesce(name, '') || coalesce(operator, '')) as mbrand, lon, lat from fuel3) as t
" fuel3.csv > fuel.csv

./tippecanoe/tippecanoe -o fuel.mbtiles --layer=station -zg --drop-densest-as-needed fuel.csv
```
