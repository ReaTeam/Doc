# Markers and Regions in ReaScript API

Markers and Regions iteration is quite confusing cause there is several kinds of index.
 
Note that markers and regions are undifferencied in this function. A regions is basically a marker with and end_position and isrgn = true.

Beware : `retval = idx +1`. This allows to check `retval > 0` to see if the enumeration worked for a certain idx.

## IDX (Timeline)

IDX is according to the timeline position. It is 0-based. If a marker/region move, it's IDX will also move.
 
For eg, if you have Marker A, Marker B, Region A, Marker C,
You will neeed IDX = 2 to get Region A.
 
This IDX is used in the following functions:

```lua
retval, isrgn, pos, rgnend, name, markrgnindexnumber, color = reaper.EnumProjectMarkers3( proj, idx )
```
 
```lua
reaper.SetProjectMarkerByIndex2( proj, markrgnidx, isrgn, pos, rgnend, IDnumber, name, color, flags )
```
 
```lua
reaper.DeleteProjectMarkerByIndex( proj, markrgnidx )
```

```lua
id = 1
retval, region_guid = reaper.GetSetProjectInfo_String( 0, "MARKER_GUID:" .. id, "", false )
```

```lua
markeridx, regionidx = reaper.GetLastMarkerAndCurRegion( proj, time )
```

```lua
 reaper.NF_GetSWSMarkerRegionSub( markerRegionIdx )
```
 
## markrgnindexnumber (Ruler)

This is the number associated to the region or marker, which is displayed in the ruler.
You can then have Marker 1 and Region 1 in the same project.

This index doesn't change when you move the markers.

This index is used in the following functions:

```lua
retval = reaper.AddProjectMarker2( proj, isrgn, pos, rgnend, name, wantidx, color )
```
 
```lua
reaper.SetProjectMarker4( proj, markrgnindexnumber, isrgn, pos, rgnend, name, color, flags )
```
 
```lua
reaper.DeleteProjectMarker( proj, markrgnindexnumber, isrgn )
```

It has not Get function (v6.13). You need to firts iterate all markers in a table to have a quick way to get them.

## Marker GUID

```lua
id = 1
retval, region_guid = reaper.GetSetProjectInfo_String( 0, "MARKER_GUID:" .. id, "", false )
```

This is a unique string to identify a marker/region. it doesn't change if the marker/region move. id is timeline based like EnumProjectMarkers3.
There is no GetMarkerByGUID native function (v6.13). You need to iterate all markers/regions in a table first to have a quick way to get them.

## Colors

`color` parameter can sometimes needs 25th bit: `color_int | 1<<24`, or `color_int | 0x1000000`.

```lua
reaper.SetProjectMarker3(0,1,false,0,0,"test", reaper.ColorToNative( 255, 0, 0 ) |0x1000000 )
```

```lua
reaper.SetProjectMarker3(0,1,false,0,0,"test", reaper.ColorToNative( 255, 0, 0 ) | 1<<24 )
```

## Looping

Using while/repeat loops are perfect for iterating in all markers+regions. A simple conditions can allows filter markers by name, positions, type (marker or region) or even more complex things. Consider storing marker/region infos in a table in you plan to move them.

```lua
idx = 0
repeat
  retval, isrgn, pos, rgnend, name, markrgnindexnumber, color = reaper.EnumProjectMarkers3( proj, idx ) -- get marker by idx
  if not isrgn then -- if it is a marker and not a region
    -- do something
  end
  idx = idx + 1 -- increment idx
until retval == 0
```
