## Fuel map layer

A Mapbox GL overlay style to display fuel station, with colour scheme based on stations brands common in France.

### Build the data

Get the conflated data set of stations between www.prix-carburants.gouv.fr and OpenStreetMap provided by Osmose-QA osmose.openstreetmap.fr

Install `csvcut` from csvkit and `tippecanoe` from https://github.com/mapbox/tippecanoe.

```
wget http://osm110.openstreetmap.fr/~osmose/Prix%20des%20carburants%20en%20France-Analyser_Merge_Fuel_FR.byOSM.csv.bz2
bzcat "Prix des carburants en France-Analyser_Merge_Fuel_FR.byOSM.csv.bz2" | csvcut -c 'name,operator,brand,lon,lat' > fuel3.csv

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

tippecanoe -o fuel.mbtiles --layer=station -zg --drop-densest-as-needed fuel.csv
```
