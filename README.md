# Openpoiservice

Openpoiservice (ops) is a flask application which hosts a highly customizable points of interest database derived from OpenStreetMap.org data.

> OpenStreetMap [tags](https://wiki.openstreetmap.org/wiki/Tags) consisting of a key and value describe specific features of 
> map elements (nodes, ways, or relations) or changesets.  Both items are free format text fields, but often represent numeric 
> or other structured items. 

This service makes use of OSM tags by grouping them into certain categories. If it picks up a node tagged with
one of the osm key's defined in `categories.yml` it will import this point of interest with additional tags
which may be defined in `ops_settings.yml`. Any additional tag, for instance `wheelchair` may then be used
to query the service via the API. 


[![Build Status](https://travis-ci.org/realpython/flask-skeleton.svg?branch=master)](https://travis-ci.org/realpython/flask-skeleton)

## Quick Start

### Basics

1. Create and activate a virtualenv
2. This repository uses [imposm.parser](https://imposm.org/docs/imposm.parser/latest/index.html) to parse the 
OpenStreetMap data. To this end, make sure `google's protobuf` is installed on your system 
- **Ubuntu**: most likely you will have to install protobuf [from source](https://github.com/google/protobuf/blob/master/src/README.md) if 
[https://imposm.org/docs/imposm.parser/latest/install.html#requirements](https://imposm.org/docs/imposm.parser/latest/install.html#requirements) doesn't
do the job.
- **OS X**  Using homebrew` on OS X `brew install protobuf` will suffice.
3. Afterwards you can install the necessary requirements via pipwith `pip install -r requirements.txt`

### Set Environment Variables

Update *openpoiservice/server/ops_settings.yml* with your database, and then run:

```sh
$ export APP_SETTINGS="openpoiservice.server.config.ProductionConfig"
```

(or

```sh
$ export APP_SETTINGS="openpoiservice.server.config.DevelopmentConfig"
```
)


### Create the POI DB

```sh
$ python manage.py create_db
```
### Drop the POI DB

```sh
$ python manage.py drop_db
```

### Parse and import OSM data

```sh
$ python manage.py import_data
```

### Run the Application

```sh
$ python manage.py run
```

Access the application at the address [http://localhost:5000/](http://localhost:5000/)

> Want to specify a different port?

> ```sh
> $ python manage.py run -h 0.0.0.0 -p 8080
> ```

### Testing: TODO

```sh
$ export TESTING=True && python manage.py test
```

### API Documentation

The documentation for this flask service is provided via [flasgger](https://github.com/rochacbruno/flasgger) and can be
accessed via `http://localhost:5000/apidocs/`.

Generally you have three different request types `pois`, `category_stats` and
`category_list`.

Using `request=poi` in the POST body will return a GeoJSON FeatureCollection
in your specified bounding box or geometry. 

Using `request=category_stats` will do the same but group by the categories, ultimately
returning a JSON object with the absolute numbers of pois of a certain group.

Finally, `request=category_list` will return a JSON object generated from 
`openpoiservice/server/categories/categories.yml`.

### Category IDs and configuration

`openpoiservice/server/categories/categories.yml` 

is a list of (note: not all!) OpenStreetMap tags with arbitrary category IDs. If you keep the structure as follows, you can manipulate this list as you wish.
 
 ```yaml
 transport:
    id: 580
    children:
        aeroway:
            aerodrome: 581        
            aeroport: 582 
            helipad: 598         
            heliport: 599 
        amenity:
            bicycle_parking: 583  
            
 sustenance:
    id: 560             
    children:
        amenity:
            bar: 561             
            bbq: 562   
 ...
 ```
 
 Openpoiservice uses this dictionary while its importing pois
 from the OpenStreetMap data and assigns the custom category IDs
 accordingly.

`openpoiservice/server/ops_settings.yml` 

This is where you will configure your spatial restrictions and database connection (find more information within the file). 

Also, these settings file controls which OSM information will be considered in the database and also if 
these may be queried by the user via the API. As an example:

```py
wheelchair:
    common_values: ['yes', 'limited', 'no', 'designated']
    filterable: 'equals'
```

Means that the OpenStreetMap tag [wheelchair](https://wiki.openstreetmap.org/wiki/Key:wheelchair) will be considered
during import and also if a user adds `wheelchair:` as a property and one of the `common_values` as value to the POST body.

### Examples

##### POST body structure

```sh
{
	"request": "pois"|"category_stats"|"category_list",
	"geometry": {
	    "type": "polygon"|"point"|"linestring"
	    "geom": [[lat,lng],[...,...]...],
	    "bbox": [[lat,lng],[...,...]...],
	    "radius": 10000
	}
	"filters": {
	    "wheelchair": "yes"|"no"|"..."
        "smoking": "yes"|"no“|"..."	
        "category_ids": [cat_id_1,cat_id_2,...],
	    "category_group_ids: [cat_group_id_1,cat_group_id_2,...]
        ...    
	}
	"limit": 100,
	"sortby": "distance"
}
```

##### POIs
```sh
curl -X POST \
  http://127.0.0.1:5000/places \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d '{
	"request": "category_stats",
	"geometry": {
	    "type": "point",
	    "geom": [[53.075051,8.798952]],
	    "bbox": [[53.075051,8.798952],[53.080785,8.907160]],
	    "radius": 10000
	},
	"filters": {
        "category_ids": [601, 280] 
	}
}'
```

##### POI Statistics
```sh
curl -X POST \
  http://127.0.0.1:5000/places \
  -H 'content-type: application/json' \
  -d '{
	"request": "pois",
	"geometry": {
	    "type": "point",
	    "geom": [[53.075051,8.798952]],
	    "bbox": [[53.075051,8.798952],[53.080785,8.907160]],
	    "radius": 10000
	},
	"filters": {
        "category_ids": [601, 280] 
	},
	"limit": 100,
	"sortby": "distance"
}'
```

##### POI Category List

```sh
curl -X POST \
  http://127.0.0.1:5000/places \
  -H 'content-type: application/json' \
  -d '{
	"request": "category_list"
}'
```

