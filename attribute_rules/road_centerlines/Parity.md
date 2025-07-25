# Parity_R and Parity_L
This field can have the value Odd, Even, Zero, or Both depending on the address number range entered for a particular RCL segment.

This version of the attribute rule does not take every point along a segment into account. If there are non-conforming points within the
range that are not the min/max, they will not be caught by this check. Streets with a mix of odd and even address numbers on the same side may need to 
have the parity entered manually. See the separate code snippets for parity check using all assiciated points.

## Settings
**Field:** Parity_L or Parity_R  
**Triggers:** Insert, Update   
**Triggering Fields:** Any or all. Not order dependent.  

## Arcade 

### Parity_L
```js
// 7/8/2025 Emma Kraco
// Determine the right/ left parity using the to and from fields

// set variable for from field
var fromNum = $feature.FromAddr_L

//set variable for the to field
var toNum = $feature.ToAddr_L

// create bool variables to store true/false for zero values
var fromZero = (fromNum == 0)
var toZero = (toNum == 0)

// set variable for the parity letter. This helps catch errors/exceptions
var parVal = "none"

function checkEven(addNum){
    var check = (addNum % 2 == 0)
    return check
}

// bool to store even true/false
var toEven = checkEven(toNum)
var fromEven = checkEven(fromNum)

  if (fromZero && toZero){
  parVal = "Z"
  }else{
    if(toEven && fromEven){
    parVal = "E"
    }else{
        if(!toEven && !fromEven){
        parVal = "O"
        }else{ 
        parVal = "B"
        }
    }
}
return parVal
```
