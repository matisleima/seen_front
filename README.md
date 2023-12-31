# seen_front

## description
This is the front end code of a prototype web app for sharing mushroom picking spots which is 
an absolutely unwanted and never occuring thing in the life of mushroom-loving
people of Estonia. These places have been family secrets for decades and this app
is here to shamefully break that tradition.

The app makes use of OpenStreetMap and Leaflet JS library for interactive maps. Note that
there is a known and unsolved bug regarding zoom animation (see below). 

## instructions
The app features a map of Estonia, a text box and buttons for adding,
editing and deleting mushroom picking locations. 

### view locations
Hover on the pins with the cursor to read about the existing spots.
Zoom and drag the map for a more detailed navigation.

### add a new location
Simply click on the map, fill the text box with info about what can be found on spot and clik 'SALVESTA'.

### edit an existing location
Click on an existing pin, drag it to a new location and/or change its description. Finally click 'MUUDA'.

### delete a location
Click on an existing pin and 'KUSTUTA'.

### misc
Clicking on the backgrounds cancels all ongoing operations.

### BUG 
There is a known and unsolved Leaflet + Vue.js 3 bug that throws an error in some
map navigation combinations, most notably click on pin and zoom on this map.

Read more here: https://stackoverflow.com/questions/73542576/leaflet-error-when-zooming-after-closing-popup

### NB!
NEVER share the whereabouts of a mushroom picking spot that you didn't discover yourself!