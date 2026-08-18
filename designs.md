I'll use [0.5](https://ngff.openmicroscopy.org/0.5/) as the base, but it can be rebased

Core: 

* A - string identifiers for channels
* B - channels as an special case of axis-level metadata

Other possible additions:

* C - canonical RGB-in-3-channels representation
* X - tags for channels (slugs or ontology terms) 
* X - handle current `omero.channels` as a fallback default rendering 
* X - channel grouping (m:n) via tags
* X - handling unbound axis (implicit channel=1)
## A - string identifiers for channels

allow `getChannelByName("x")` unambiguously for a single N-D array

<details>

### A0 - identity written in axis 
(dropped due to scenario with no bound channels) 

### A1 - array matching identifiers by index

```json
{ "ome": 
      { "multiscales": [...], 
         "channels": {
              "ids": [ "gfp", "dapi"], 
      }
}
```

### A2 - array of objects, implicitly linked by index

```json
{ "ome": 
      { "multiscales": [...], 
         "channels": [
             { "id": "gfp"}, 
             { "id": "dapi"}
           ], 
      }
}
```

### A3 - array of objects, explicitly adding indexes  

```json
{ "ome": 
      { "multiscales": [...], 
         "channels": [
             { "id": "gfp", "index": 0}, 
             { "id": "dapi", "index": 1}
           ], 
      }
}
```
</details>


## B - channels as a special case of axis-level metadata

allow for extensions to include other axis-level metadata

<details>
### B1 - only a direct `channels` key 

```json
{ "ome": 
      { "multiscales": [...], 
         "channels": ...
}
```

### B2 - a`channels` key under a key for axes in general

Other metadata blocks may be added for different types of axis ([esp. with RFC 3](https://ngff.openmicroscopy.org/rfc/3/index.html))  

```json
{ "ome": 
      { "multiscales": [...], 
         "indexed_axes": {
             "channels": ...
       }
}
```

### B3 - identifying the channel axis via its name 

where 
```json
"axes": [
  { "name": "t", "type": "time", "unit": "millisecond" },
  { "name": "c", "type": "channel" },
  { "name": "z", "type": "space", "unit": "micrometer" },
```

```json
{ "ome": 
      { "multiscales": [...], 
         "indexed_axes": {
             "c": ...
       }
}
```
</details>

## C - Photometry / RGB processing 

allows for a canonical way of converting RGB images to OME-Zarr

allows for the RGB channels to co-exist with other (e.g. fluorescence) channels in the same N-D array

max of 1 (R, G, B) 3-channel group per N-D image

### C1 - An explicit "rgb" key + value "R", "G" or "B" 
exactly 3 channels with rgb = oneOf("R", "G", "B")

```json
{ "ome": 
      { "multiscales": [...], 
        "channels": [
             { "id" : "channel-0",
                "rgb": "R"
             },
             { "id" : "channel-1",
                "rgb": "G"
              },
             { "id" : "channel-2",
                "rgb": "B"
              },
            ]
       }
}
```

### C2 - Reserved ids for RGB

0 or exactly 3 channels with id=oneOf(R,G,B) 

```json
{ "ome": 
      { "multiscales": [...], 
        "channels": [
             { "id" : "R",
             },
             { "id" : "G"
              },
             { "id" : "B",
              },
            ]
       }
}
```

### C3 - Piggy-back on a `color` key + is_rgb = true

0 or exactly 3 channels with is_rgb = true
if present, colors MUST be encoding R=FF0000, G=00FF00, B=0000FF

```json
{ "ome": 
      { "multiscales": [...], 
        "channels": [
             { "id" : "channel-0",
                "is_rgb" : "true",
                "color": "#FF0000"
             },
             { "id" : "channel-1",
                "is_rgb" : "true",
                "rgb": "#00FF00"
              },
             { "id" : "channel-2",
                "is_rgb" : "true",
                "rgb": ""#0000FF"
              },
            ]
       }
}
```
