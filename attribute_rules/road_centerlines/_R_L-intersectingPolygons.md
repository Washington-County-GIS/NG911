# feature_L and _R Attribute Rules
These rules create an array of polygons that intersect the road centerline feature. 
In the case of more than one possible match, a test line is generated with the same start point as the rcl feature. A
buffer is created around the test line, and rotated slightly to the left or right (based on field) around the rcl
start point. From there a new list of intersecting features is produced, and the one with the greatest area of overlap 
is selected as the right or left feature.

Note that this script can be repurposed for any of the attributes for right and left side of road, such as IncMuni, PostComm, MuniCode, ESN, MSAGComm... simply swap out the feature 
pointed to by "FeatureSetByName" with the appropriate boundary layer. This example uses IncMuni.

## Settings
**Field:** IncMuni_L, IncMuni_R  
**Triggers:** Insert, Update   
**Triggering Fields:** Any or all. Not order dependent.  

## Arcade 
The only difference between these two attribute rules is the rotation angle specified.
### IncMuni_L
```js
// created 6/6/2025 Emma Kraco

// create a buffer of the rcl to grab apparent munis when the placement isn't precise
var rcl_buf = Buffer($feature, 5, "feet")

// get the list of intersecting munis
var munis = Intersects(FeatureSetByName($datastore, "your-municipality-feature", ["Municipality_Name"], true),rcl_buf)
var muniCount = Count(munis)

// get environment and spatial reference for constructing points
var env = GetEnvironment()
var wkid = env.SpatialReference.wkid

// set rcl line start vertex for specifying rotation origin
var start_x = Geometry($feature).paths[0][0].x
var start_y = Geometry($feature).paths[0][0].y
var rcl_pt = Point({"x":start_x, "y":start_y, "spatialReference":{"wkid": wkid}})

if (muniCount > 1){
  // rotate the rcl to the left 5 deg
  var testLine = Rotate($feature, 5, rcl_pt)
  // Create buffer for finding intersection area
  var buf = Buffer(testLine, 10,"feet")
 
  var nameArr = []
  var areaArr = []
  for (var i in munis){
    var areaTest = Area(Intersection(buf, i), 'square-feet')
    Push(nameArr, i.Municipality_Name)
    Push(areaArr, areaTest)
  }
    if(Count(nameArr)>0){
    var max_index = IndexOf(areaArr, Max(areaArr))
    var finalName = nameArr[max_index]
     }else{
      finalName = "error1"
     }

}else{
     if(muniCount> 0){
     finalName = First(munis).Municipality_Name
     }else{
     finalName = "error2"
     }
}

return finalName


```

### IncMuni_R
```js
// created 6/6/2025 Emma Kraco

// create a buffer of the rcl to grab apparent munis when the placement isn't precise
var rcl_buf = Buffer($feature, 5, "feet")

// get the list of intersecting munis
var munis = Intersects(FeatureSetByName($datastore, "your-municipality-feature", ["Municipality_Name"], true),rcl_buf)
var muniCount = Count(munis)

// get environment and spatial reference for constructing points
var env = GetEnvironment()
var wkid = env.SpatialReference.wkid

// set rcl line start vertex for specifying rotation origin
var start_x = Geometry($feature).paths[0][0].x
var start_y = Geometry($feature).paths[0][0].y
var rcl_pt = Point({"x":start_x, "y":start_y, "spatialReference":{"wkid": wkid}})

if (muniCount > 1){
  // rotate the rcl to the right 5 deg
  var testLine = Rotate($feature, -5, rcl_pt)
  // Create buffer for finding intersection area
  var buf = Buffer(testLine, 10,"feet")
 
  var nameArr = []
  var areaArr = []
  for (var i in munis){
    var areaTest = Area(Intersection(buf, i), 'square-feet')
    Push(nameArr, i.Municipality_Name)
    Push(areaArr, areaTest)
  }
    if(Count(nameArr)>0){
    var max_index = IndexOf(areaArr, Max(areaArr))
    var finalName = nameArr[max_index]
     }else{
      finalName = "error1"
     }

}else{
     if(muniCount> 0){
     finalName = First(munis).Municipality_Name
     }else{
     finalName = "error2"
     }
}

return finalName


```

## Caveats
When an oddly shaped road segment falls on a boundary between two features, a higher rotation angle can return unexpected 
results. Test the script results along boundaries, paying special attention to those that do not have uniform shapes. Always
test attribute rules on copies of your data before full implementation. These rules have only been tested on the features 
belonging to Washington County, WI's NG911 dataset. 

If you use this for finding right and left ESN, you may want to replace the error messages with rwo different three-digit
numbers that are not in your domain. eg. 000 and 111. This way you can filter for errors after calculating, but the script
will still run in cases where there isn't a value for your county on one or both sides of the road


## Suggested Modification
If you are finding that you have a lot of instances where the correct intersecting features are not assigned due to unusual geometry and adjusting the angle of rotation isn't helping, 
update the rule to only run when the field is null. Wrap everything starting with the first if statement in a check for null, and default to the current value if not null. 

Before the first if statement: 
```
if(IsEmpty($feature.currentAttribute)){
```
After the return statement: 
```
}else{
return $feature.currentAttribute
}
```

All together, using the IncMuni_L example: 
```js
// created 6/6/2025 Emma Kraco

// create a buffer of the rcl to grab apparent munis when the placement isn't precise
var rcl_buf = Buffer($feature, 5, "feet")

// get the list of intersecting munis
var munis = Intersects(FeatureSetByName($datastore, "your-municipality-feature", ["Municipality_Name"], true),rcl_buf)
var muniCount = Count(munis)

// get environment and spatial reference for constructing points
var env = GetEnvironment()
var wkid = env.SpatialReference.wkid

// set rcl line start vertex for specifying rotation origin
var start_x = Geometry($feature).paths[0][0].x
var start_y = Geometry($feature).paths[0][0].y
var rcl_pt = Point({"x":start_x, "y":start_y, "spatialReference":{"wkid": wkid}})

// add condition to allow user's manual override. Only procedes if the field is already null 
if(IsEmpty($feature.IncMuni_L)){


if (muniCount > 1){
  // rotate the rcl to the left 5 deg
  var testLine = Rotate($feature, 5, rcl_pt)
  // Create buffer for finding intersection area
  var buf = Buffer(testLine, 10,"feet")
 
  var nameArr = []
  var areaArr = []
  for (var i in munis){
    var areaTest = Area(Intersection(buf, i), 'square-feet')
    Push(nameArr, i.Municipality_Name)
    Push(areaArr, areaTest)
  }
    if(Count(nameArr)>0){
    var max_index = IndexOf(areaArr, Max(areaArr))
    var finalName = nameArr[max_index]
     }else{
      finalName = "error1"
     }

}else{
     if(muniCount> 0){
     finalName = First(munis).Municipality_Name
     }else{
     finalName = "error2"
     }
}

return finalName


}else{
//if field is not null, default to the existing field value
return $feature.IncMuni_L
}

```





## Additional Reading
Arcade Rotate function reference: https://developers.arcgis.com/arcade/function-reference/geometry_functions/#rotate
