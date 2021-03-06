node-red-contrib-web-worldmap
=============================

A <a href="http://nodered.org" target="_new">Node-RED</a> node to provide world
map web page for plotting "things" on.

![Map Image](https://dceejay.github.io/pages/images/redmap.png)

### Changes
 - v1.0.23 - Add msg.payload.command.heatmap to allow setting of heatmap config.
 - v1.0.22 - Add example how to embed into Node-RED-Dashboard template.
 - v1.0.21 - If you specify range and icon then you get a marker and a range circle, if you just specify range with no icon, you just get a circle, and vice versa.
 - v1.0.20 - Add buildings overlay.
 - v1.0.19 - Add circle mode - specify name, lat, lon and radius.
 - v1.0.18 - Correct .photourl property to match .photoUrl as per docs
 - v1.0.17 - Removed Mapquest maps. Bug fixes - reduced leakage of listeners being added.
 - v1.0.12 - Added ability to set initial start position, zoom level and base map layer.
 - v1.0.x - now uses socket.io to connect to backend - means this node now has an input connection
 (like "proper" nodes should :-), and you no longer need a websocket node in parallel.
 Obviously this is a breaking change hence the major version number bump. Also thus adds a `worldmap in`
 node to handle events coming from the map interaction.

### Install

Run the following command in your Node-RED user directory - typically `~/.node-red`

        npm i --save node-red-contrib-web-worldmap

### Usage

Plots "things" on a map. The map will be served from `{httpRoot}/worldmap`

The minimum **msg.payload** must contain `name`, `lat` and `lon` properties, e.g.

        {name:"Joe", lat:51, lon:-1.05}

`name` must be a unique identifier across the whole map. Repeated location updates to the same `name` move the point.

Optional properties include

 - **layer** : specify a layer on the map to add marker to.
 - **speed** : combined with bearing, draws a vector.
 - **bearing** : combined with speed, draws a vector.
 - **accuracy** : combined with bearing, draws a polygon of possible direction.
 - **icon** : <a href="http://fortawesome.github.io/Font-Awesome/icons/" target="_new">font awesome</a> icon name.
 - **iconColor** : Standard CSS color name or #rrggbb hex value.
 - **photoUrl** : adds an image pointed at by the url to the popup box.
 - **deleted** : set to <i>true</i> to remove the named marker. (default false)
 - **addtoheatmap** : set to <i>false</i> to exlcude point from contributing to heatmap layer. (default true)
 - **intensity** : set to a value of 0.1 - 1.0 to set the intensity of the point on heatmap layer. (default 1.0)
  

Any other `msg.payload` properties will be added to the icon popup text box.

You may select any of the Font Awesome set of [icons](http://fortawesome.github.io/Font-Awesome/icons/).
However there are several specials...

 - **plane** : a plane icon that aligns with the bearing of travel.
 - **ship** : a ship icon that aligns with the bearing of travel.
 - **car** : a car icon that aligns with the bearing of travel.
 - **locate** : a 4 corner outline to locate a point without obscuring it.
 - **friend** : pseudo Nato style blue rectangle.
 - **hostile** : pseudo Nato style red circle.
 - **neutral** : pseudo Nato style green square.
 - **unknown** : pseudo Nato style yellow square.
 - **earthquake** : black circle - diameter proportional to magnitude.

#### Areas and Lines

If the payload contains an **area** property - that is an array of co-ordinates, e.g.

    [ [51.05, -0.08], [51.5, -1], [51.2, -0.047] ]

then rather than draw a point and icon it draws the polygon. Likewise if it contains a
**line** property it will draw the polyline.

 - **iconColor** : can set the colour of the polygon or line.
 - **name** : is used as the id key - so can be redrawn/moved.
 - **layer** : declares which layer you put it on..

#### Circles

If the payload contains a **radius** property, as well as name, lat and lon, then rather
than draw a point it will draw a circle.

As per Areas and Lines you may also specify *iconColor*, and *layer*.

### Drawing

A single *right click* will allow you to add a point to the map - you must specify the `name` and optionally the `icon` and `layer`.  

Right-clicking on an icon will allow you to delete it.

If you select the **drawing** layer you can also add polylines, polygons and rectangles.

All these events generate messages that can be received by using a **worldmap in** node. For example:

    { "action": "connected" }
    { "action": "point", "lat": "50.60634", "lon": "-1.66580", "point": "joe,male,mylayer" }
    { "action": "delete", "name": "joe" }
    { "action": "layer", "name": "Esri Satellite" }
    { "action": "draw", "type": "rectangle", "points": [ { "lat": 50.61243889044519, "lng": -1.5913009643554688 }, { "lat": 50.66665471366635, "lng": -1.5913009643554688 }, { "lat": 50.66665471366635, "lng": -1.4742279052734375 }, { "lat": 50.61243889044519, "lng": -1.4742279052734375 } ] }

### Control

You can also control the map via the node, by sending in a msg.payload containing a **command** object.

Optional properties include

 - **lat** - move map to specified latitude.
 - **lon** - move map to specified longitude.
 - **zoom** - move map to specified zoom level (1 - world, 13 to 20 max zoom depending on map).
 - **layer** - set map to specified layer name.
 - **map** - Object containing details of a new map layer:
   - **name** - name of the map base layer OR **overlay** - name of overlay layer
   - **url** - url of the map layer
   - **opt** - options object for the new layer
 - **heatmap** - set heatmap options object see https://github.com/Leaflet/Leaflet.heat#reference

#### For example

To switch layer, move map and zoom

        msg.payload.command =  {layer:"Esri Relief", lat:51, lon:3, zoom:10 };

To add a new base layer

        msg.payload.command.map = {
            name:"OSMhot",
            url:'http://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png',
            opt:{ maxZoom:19, attribution:"&copy; OpenStreetMap" }
        };

Demo Flow
---------

The following example gets recent earthquakes from USGS, parses the result,
formats up the msg as per above and sends to the node to plot on the map.
It also shows how to zoom and move the map or add a new layer.

        [{"id":"f7950c21.019f5","type":"worldmap","z":"896b28a8.437658","name":"","x":670,"y":680,"wires":[]},{"id":"bb057b8a.4fe2c8","type":"inject","z":"896b28a8.437658","name":"","topic":"","payload":"","payloadType":"none","repeat":"","crontab":"","once":false,"x":110,"y":640,"wires":[["b8545e85.5ba4c"]]},{"id":"b8545e85.5ba4c","type":"function","z":"896b28a8.437658","name":"add new layer","func":"msg.payload = {};\nmsg.payload.command = {};\n\nvar u = 'http://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png';\nvar o = JSON.stringify({ maxZoom: 19, attribution: '&copy; OpenStreetMap'});\n\nmsg.payload.command.map = {name:\"OSMhot\", url:u, opt:o};\nmsg.payload.command.layer = \"OSMhot\";\n\nreturn msg;","outputs":1,"noerr":0,"x":340,"y":640,"wires":[["f7950c21.019f5"]]},{"id":"e6cc0a05.14edd8","type":"function","z":"896b28a8.437658","name":"USGS Quake monitor csv re-parse","func":"msg.payload.lat = msg.payload.latitude;\nmsg.payload.lon = msg.payload.longitude;\nmsg.payload.layer = \"earthquake\";\nmsg.payload.name = msg.payload.id;\nmsg.payload.icon = \"globe\";\nmsg.payload.iconColor = \"orange\";\n\ndelete msg.payload.latitude;\ndelete msg.payload.longitude;\t\nreturn msg;","outputs":1,"noerr":0,"x":460,"y":780,"wires":[["f7950c21.019f5"]]},{"id":"84b8388.5e943c8","type":"function","z":"896b28a8.437658","name":"move and zoom","func":"msg.payload = { command:{layer:\"Esri Terrain\",lat:0,lon:0,zoom:3} };\nreturn msg;","outputs":1,"noerr":0,"x":340,"y":680,"wires":[["f7950c21.019f5"]]},{"id":"5c317188.d2f31","type":"csv","z":"896b28a8.437658","name":"","sep":",","hdrin":true,"hdrout":"","multi":"one","ret":"\\n","temp":"","x":310,"y":720,"wires":[["e6cc0a05.14edd8"]]},{"id":"cfafad11.2f299","type":"inject","z":"896b28a8.437658","name":"","topic":"","payload":"","payloadType":"none","repeat":"","crontab":"","once":false,"x":110,"y":680,"wires":[["84b8388.5e943c8"]]},{"id":"f0d75b03.39d618","type":"http request","z":"896b28a8.437658","name":"","method":"GET","url":"http://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/2.5_day.csv","x":190,"y":780,"wires":[["5c317188.d2f31"]]},{"id":"87da03a.eb8a3","type":"inject","z":"896b28a8.437658","name":"Quakes","topic":"","payload":"","payloadType":"none","repeat":"900","crontab":"","once":false,"x":120,"y":720,"wires":[["f0d75b03.39d618"]]}]


Car icon made by <a href="http://www.freepik.com" title="Freepik">Freepik</a> from <a href="http://www.flaticon.com" title="Flaticon">www.flaticon.com</a> is licensed by <a href="http://creativecommons.org/licenses/by/3.0/" title="Creative Commons BY 3.0" target="_blank">CC 3.0 BY</a>.</div>
