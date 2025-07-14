# Rotation
This is an optional field to assign a rotation value to an address point for the purpose of labeling. The 
field is called in labeling properties to vary the angle by point.

The first version depends on the existence of a populated field called Travel_Dir in the RCL feature data. 
There are 8 rotation angles given, but if you prefer less variation you can adjust the When function to 
group the cardinal/ intercardinal directions as desired. 

## Settings
**Field:** Rotation  
**Triggers:** Insert, Update   
**Triggering Fields:** Any or all. Not order dependent.  

## Arcade 

### Rotation
```js
// Created 6/9/2025 by Emma Kraco

// var for the point's rcl ID
var uniqueID = $feature.RCL_NGUID

// query for filtering RCL features
var query = "(RCL_NGUID = @uniqueID)"

// get the RCL feature that matches point
var rcl = FeatureSetByName($datastore, "yourRCL Feature",['RCL_NGUID', 'TravelDir'], true)

var filterRCL = Filter(rcl, query)

// There should only be one RCL_NGUID returned, but treat as array due to nature of FeatureSet functions
var dirList = []
for (var i in filterRCL){
 Push(dirList, i.travel_dir)
}
// get the travel dir only


var travDir = First(dirList)


var rot = When(
travDir == "N", 360,
travDir == "NE",45,
travDir == "E",90,
travDir == "SE",135,
travDir == "S",180,
travDir == "SW",225,
travDir == "W",270,
travDir == "NW",315,
0
)

return rot


```

### Rotation with only 4 angles

```js
// var for the point's rcl ID
var uniqueID = $feature.RCL_NGUID

// query for filtering RCL features
var query = "(RCL_NGUID = @uniqueID)"

// get the RCL feature that matches point
var rcl = FeatureSetByName($datastore, "yourRCL feature",['RCL_NGUID', 'travel_dir'], true)

var filterRCL = Filter(rcl, query)

var dirList = []
for (var i in filterRCL){
 Push(dirList, i.travel_dir)
}
// get the travel dir only


var travDir = First(dirList)


var rot = When(
travDir == "N"||travDir == "S", 0,
travDir == "NE" ||travDir == "SW",45,
travDir == "E" || travDir == "W",90,
travDir == "SE" || travDir == "NW",135,
 null
)

return rot
```