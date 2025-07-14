# Travel Direction
This is an optional field that can help determine if your streets are digitized in the correct direction, aid in label placement for SSAPs, and aid in determining left and right parity.
Please note that this usage of the field differs from what is outlined in the WLIA standards. Here the field is used to indicate the general direction of address number increase using
the cardinal and intercardinal directions. 

## Settings
**Field:** Travel_Dir  
**Triggers:** Insert, Update   
**Triggering Fields:** Any or all. Not order dependent.  

## Arcade 

### Travel_Dir
```js
// Created 6/9/2025 by Emma Kraco

// Calculate the travel direction of a street using the direction of house number increase. For road centerline segments without address points, default to the start and end 
// points of the line segment
// Calculating by using line feature start and end point only works for perfectly straight lines digitized in the direction of street number increase

// attributes used for determining r/left muni
var uniqueID = $feature.RCL_NGUID

// get environment and spatial reference for constructing points
var env = GetEnvironment()
var wkid = env.SpatialReference.wkid



var query = "(RCL_NGUID = @uniqueID)"

// coordinates of current rcl start and end points
var start_x = Geometry($feature).paths[0][0].x
var start_y = Geometry($feature).paths[0][0].y

var end_x = Geometry($feature).paths[-1][-1].x
var end_y = Geometry($feature).paths[-1][-1].y

var rclStart = Point({"x":start_x, "y":start_y, "spatialReference":{"wkid": wkid}})
var rclEnd = Point({"x":end_x, "y":end_y, "spatialReference":{"wkid": wkid}})

// Get the address point features 
var all_pts = FeatureSetByName($datastore, "AddressMaster",['RCL_NGUID','Add_Number','Inc_Muni','Easting','Northing'], true)
// address points filtered by RCL NGUID
var add_pts = Filter(all_pts, query)
var add_count = Count(add_pts)

// array of all address numbers attached to the current RCL
var add_num = []
var add_es = []  //eastings of the points
var add_ns = []  //northings of the points

// populate parallel arrays with address numbers, eastings and northings
  for (var i in add_pts){
    Push(add_num, i.Add_Number)
    Push(add_es, i.Easting)
    Push(add_ns, i.Northing)

  }

// find the maximum address number and its index to match to the easting and northing associated with the point
var max_add_num = Max(add_num)
var max_index = IndexOf(add_num, max_add_num)

// minimum add num
var min_add_num = Min(add_num)
var min_index = IndexOf(add_num, min_add_num)

if(add_count>0){
var max_add_es = add_es[max_index]
var max_add_ns = add_ns[max_index]
var min_add_es = add_es[min_index]
var min_add_ns = add_ns[min_index]
}else{
max_add_es = 0
max_add_ns = 0
min_add_es = 0
min_add_ns = 0
}


// reconstruct point geometry using the max and min  address number information
var max_add_point = Point({"x": max_add_es, "y": max_add_ns, "spatialReference":{"wkid":wkid} })
var min_add_point = Point({"x": min_add_es, "y": min_add_ns, "spatialReference":{"wkid":wkid} })

// bearing for min and max point, with min point as the start pt
var rcl_bearing = Round(Bearing(rclStart, rclEnd),0)

if (add_count>1){
var add_pt_bearing = Round(Bearing(min_add_point, max_add_point),0)
}else{
add_pt_bearing = rcl_bearing
}


var dir = ""

// use bearing to get the direction
function direction(bears){
   var dir = When(
   bears < 22.5, "N",

   bears >= 22.5 && bears < 67.5,"NE",

   bears >= 67.5 && bears < 112.5,"E",

   bears >= 112.5 && bears < 157.5,"SE",

   bears >= 157.5 && bears < 202.5,"S",

   bears >= 202.5 && bears < 247.5,"SW",

   bears >= 247.5 && bears < 292.5,"W",

   bears >= 292.5 && bears < 337.5,"NW",

   bears >= 337.5,"N",
   "none"

    )
  return dir
}


var pt_dir = direction(rcl_bearing)

return Concatenate(pt_dir)


```
