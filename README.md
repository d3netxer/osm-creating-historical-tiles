# Instructions (September 2018)

1. clone the openstreetmap-carto repo (https://github.com/gravitystorm/openstreetmap-carto) and enter inside the directory.
2. Use Docker Compose to import data and start up Kosmtik
- follow the instructions for the most part from the 'DOCKER.md' readme. Except...
- edit the 'docker-compose.yml' file:
  - for the kosmtik service replace the "127.0.0.1:6789:6789" ports with "6789:6789". This maps the same ports between the container and the host server so that the kosmtik server is accesible to the outside world. 
- In order to keep the container running after the docker-compose command make following edits to the 'scripts/docker-startup.sh' file:
  - comment out this line ```kosmtik serve project.mml --host 0.0.0.0```
  - add this line ```tail -f /dev/null```
  - now if you running this for the first time ```sudo docker-compose up import``` before you start kosmtik with ```sudo docker-compose up kosmtik```
- tip: if you need to re-build the container, you can use the ```sudo docker-compose build``` command (https://github.com/docker/compose/issues/1487)
- now you can find the running container and enter into it in bash and run kosmtik commands (tip: open a new terminal)
  - find running containers with this command: ```sudo docker ps```
  - enter running container with this command: ```sudo docker exec -u 0 -it containerID /bin/bash```
-it should produse a running kosmtik instance with your rendered tiles with a command like this: ```kosmtik serve project.mml --host 0.0.0.0```
- you should be able to see your tiles within the kosmtik server on a web browser, ex: http://xx.xxx.xx.xx:6789/openstreetmap-carto/#8/2.205/31.904
3. Use the tilelive command-line (tl) (https://github.com/mojodna/tl, https://github.com/posm/OpenMapKitAndroid/wiki/Creating-Your-Own-MBTiles-Basemap-File, https://github.com/mojodna/tl/issues/15) to save the tiles as mbtiles. (note: http://bboxfinder.com is good for finding bbox. bbox format seems to be minY,minX,maxY,maxX). Install NPM and NVM.

```
nvm use 4
npm install tl tilelive-http mbtiles
node_modules/.bin/tl copy http://54.159.75.39:6789/openstreetmap-carto/tile/{z}/{x}/{y}.png mbtiles://./northern_uganda.mbtiles -b "30.151978 0.911827 35.310059 4.351889" -z 6 -Z 14
```

- Note: another option I tried was using the kosmtik plug-ins kosmtik-tiles-export and kosmtik-mbtiles-export. kosmtik-tiles-export seemed to have worked for generating tiles, but not kosmtik-mbtiles-export for generating mbtiles, Here are example commands I used (bbox should be in minX,minY,maxX,maxY format):

- kosmtik-tiles-export command:
```
kosmtik export project.mml --format tiles --output export2 --tileformat 'png' --minZoom 3 --maxZoom 10 --workers 2 --tileSize 256 --metatile 2 --bounds 2.361392,30.824890,3.692966,33.441010
```
- kosmtik-mbtiles-export command:
```
kosmtik export project.mml --format mbtiles --output export/output.mbtiles --tileformat 'png' --minZoom 3 --maxZoom 8 --workers 4 --tileSize 256 --metatile 2 --bounds 2.361392,30.824890,3.692966,33.441010
```


4. upload mbtiles to MapBox to host

