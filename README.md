# Update to instructions Dec 2017

- Started with osmbox

- Then installed kosmtik, see this gist for instructions:
https://gist.github.com/d3netxer/322ec6addf27b289a843c5e50caf10b8

- do a git pull origin of the openstreetmap-carto directory

- There is a new command when importing into a PostGreSQL DB
```
osm2pgsql -G --hstore --style openstreetmap-carto.style --tag-transform-script openstreetmap-carto.lua -d gis ~/path/to/data.osm.pbf
```

# osm-creating-historical-tiles (Before/After Sliders)
Instructions on how to create osm historical tiles

So you want to create OSM historical tiles? Maybe you want to make a before and after OSM slider. This repo aims to help and make things easier for you.

There are various pieces of software ([osm-history-splitter](https://github.com/MaZderMind/osm-history-splitter), [osm2pgsql](https://github.com/openstreetmap/osm2pgsql), osmium, TileMill, and postgis) that need to be tied together to make this work, and they all require a ton of dependencies. To skip a bunch of fun installing things, you need to download the [osmbox](https://github.com/d3netxer/osmbox) and use it as your virtual machine.

##### The osmbox is on [ATLAS](https://atlas.hashicorp.com/omnitom/boxes/osmbox) and is named omnitom/osmbox
##### Install Vagrant and see [Getting Started](http://docs.vagrantup.com/v2/getting-started/index.html) 
##### example:
```Batchfile
$ vagrant init omnitom/osmbox
$ vagrant up
```
##### tips:
- The installed repos are located in the ~/opt directory

- The openstreetmap-carto repo should be updated to reflect the latest styling changes. do into the ~/opt/openstreetmap-carto/ directory and use the command: git pull origin

(username and password is 'vagrant' for virtual machine)

## The overall process
You need to get a historical OSM file from somewhere. You can download an OSM file for a particular area or you can download an OSM Planet file and clip out a smaller section. If you try to do the whole world, your system will most likely crash. The next step is creating a PostGIS database and transferring your historical OSM file into it and styling it with [OpenStreetMap Carto](https://github.com/gravitystorm/openstreetmap-carto) style. Finally, you can use TileMill to create your tiles.

### Getting historical OSM file

OSM file can come in different formats. We are downloading the OSM.pbf file type. This is a super compressed OSM file type that uses Google Protobuf (.pbf) for compression.

For downloads over geographical areas [Geofabrik](http://download.geofabrik.de) is a great site. You can click on various subregions. Notice that on these pages there is a 'raw directory index' link and if you click if you should see older versions of OSM files that go back in time a few months. If you find your interested geographical region and time period download the pbf file.

#### If you cannot find what you are looking for, then download an OSM Planet file from planet.osm.org/pbf/

These planet files have very large file sizes, here is a tip for downloading in Chrome:

-In the chrome address bar type
-chrome://flags
-Experimental features
-Halfway down the page "Enable Download Resumption"

#### (Optional Step if downloading the whole OSM Planet File) Use OSM-history-splitter to cut out a smaller section pbf from world pbf:

To do this you need to create a config file that specifies the bounding box of your area of interest. Look inside the files folder of this repo to see an example config file for Nepal.

tip: when creating config file for softcut the BBOX coordinates work like this: left lon, lower lat, right lon, upper lat

Ex. of osm-history-splitter softcut:

vagrant@vagrant:/opt/osm-history-splitter$sudo ./osm-history-splitter --softcut /shared_folder/osm_planet_file/planet-141105.osm.pbf /home/config_files/nyamira.config

See [osm-history-splitter README](https://github.com/MaZderMind/osm-history-splitter/blob/master/README.md) for more information on osm-history-splitter

### Loading it into PostGIS and Styling

You need to create a database first with postgis and hstore extensions before loading data into the database using osm2pgsql:

(ex. based from https://github.com/openstreetmap/osm2pgsql)

vagrant@vagrant:~$ createdb gis2

vagrant@vagrant:~$ psql -d gis2 -c 'CREATE EXTENSION postgis; CREATE EXTENSION hstore;'


This command loads the extracted pbf into the postgis database and applies the openstreetmap-carto style (need to create DB first):

vagrant@vagrant:$ osm2pgsql --create --database gis2 --user vagrant /shared_folder/osm_planet_file/planet-141105.osm.pbf --style /opt/openstreetmap-carto/openstreetmap-carto.style

(if you get an error about not having enough memory, you may need to add the '--cache-strategy sparse' flag to your command

### Displaying it in TileMill and creating tiles

TileMill directory:

The openstreetmap-carto repo has all of the necessary files to create historical tiles in TileMill but you should download some basemap shapefiles first. Instructions on how to do this are in the repo's Install.md. Essentially go into the openstreetmap-carto directory and run get-shapefiles.sh, this will download the shapefiles. Then you need to run ogr2ogr on the Natural Earth 2.0 populated places shapefile.

TileMill directory:

/home/vagrant/Documents/MapBox

Your TileMill projects are located inside the project directory. You need to launch the TileMill application at least once to see this folder. You should see an openstreetmap-carto folder in here. If your DB was named 'gis' then you should see your historical tiles when you open the TileMill application.

Another option if you need to create multiple tile sets and you don't want to keep on re-writing the gis DB is to create a new DB for each project. Then in the TileMill directory you can copy the 'openstreetmap-cart' project folder for each new historical tile extraction. Important to note that each folder is 1.2 GB in size. Inside the folder you need to edit the project.mml file. At the bottom of the file you can replace the name with a new project name. You also need to do a find&replace to replace the dbname of gis with the dbname you created, for example: "dbname": "gis" with "dbname": "kule_gis"

Also, if you want the name in TileMill to be different that "OpenStreetMap Carto" you can edit the "name": "OpenStreetMap Carto" tag in the bottom of the project.mml file

