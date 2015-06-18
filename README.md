# osm-creating-historical-tiles
Instructions on how to create osm historical tiles

So you want to create OSM historical tiles? Maybe you want to make a before and after OSM slider. This repo aims to help and make things easier for you.

There are various pieces of software ([osm-history-splitter](https://github.com/MaZderMind/osm-history-splitter), [osm2pgsql](https://github.com/openstreetmap/osm2pgsql), osmium, TileMill, and postgis) that need to be tied together to make this work, and they all require a ton of dependencies. To skip a bunch of fun installing things, you need to download the [osmbox](https://github.com/d3netxer/osmbox) and use it as your virtual machine.

#####The osmbox is on [ATLAS](https://atlas.hashicorp.com/omnitom/boxes/osmbox) and is named omnitom/osmbox
#####Install Vagrant and see [Getting Started](http://docs.vagrantup.com/v2/getting-started/index.html) 
#####example:
```Batchfile
$ vagrant init omnitom/osmbox
$ vagrant up
```

The overall process is that you need to get a historical OSM file from somewhere. You can download an OSM file for a particular area or you can download an OSM Planet file and clip out a smaller section. If you try to do the whole world, your system will most likely crash. The next step is creating a PostGIS database and transferring your historical OSM file into it and styling it with [OpenStreetMap Carto](https://github.com/gravitystorm/openstreetmap-carto) style. Finally, you can use TileMill to create your tiles.

###Getting historical OSM file

Download world pbf from planet.osm.org/pbf/

or you can download a pbf from geofabrik.de of a smaller area.

tip: 
-In the chrome address bar type
-chrome://flags
-Experimental features
-Halfway down the page "Enable Download Resumption"

OSM-history-splitter to cut out smaller section pbf from world pbf:

Ex. of osm-history-splitter softcut:

vagrant@ubuntu1204-desktop:/home/osm-history-splitter$sudo ./osm-history-splitter --softcut /media/sf_cybergis_data1_hiu/osm_planet_file/planet-141105.osm.pbf /home/vagrantsplitter_config_files/nyamira.config

###Loading it into PostGIS and Styling

You need to create a database first with postgis and hstore extensions before loading data into the database using osm2pgsql:

(ex. based from https://github.com/openstreetmap/osm2pgsql)

vagrant@ubuntu1204-desktop:~$ createdb bohol_gis

vagrant@ubuntu1204-desktop:~$ psql -d bohol_gis -c 'CREATE EXTENSION postgis; CREATE EXTENSION hstore;'


This command loads the extracted pbf into the postgis database and applies the openstreetmap-carto style (need to create DB first):

vagrant@ubuntu1204-desktop:~/osm$ osm2pgsql --create --database kathmandu_gis --user vagrant /home/osm-history-splitter/kathmandu.osh.pbf --style ~/osm/openstreetmap-carto/openstreetmap-carto.style

###Displaying it in TileMill and creating tiles

TileMill directory:

/home/vagrant/Documents/MapBox

Your TileMill projects are located inside the project directory. You should see an openstreetmap-carto folder in here. If your DB was named 'gis' then you should see your historical tiles when you open the TileMill application.

Another option if you need to create multiple tile sets and you don't want to keep on re-writing the gis DB is to create a new DB for each project. Then in the TileMill directory you can copy the 'openstreetmap-cart' project folder for each new historical tile extraction. Important to note that each folder is 1.2 GB in size. Inside the folder you need to edit the project.mml file. At the bottom of the file you can replace the name with a new project name. You also need to do a find&replace to replace the dbname of gis with the dbname you created, for example: "dbname": "gis" with "dbname": "kule_gis"

Also, if you want the name in TileMill to be different that "OpenStreetMap Carto" you can edit the "name": "OpenStreetMap Carto" tag in the bottom of the project.mml file

