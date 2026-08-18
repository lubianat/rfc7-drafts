I'll use [0.5](https://ngff.openmicroscopy.org/0.5/) as the base, but it can be rebased

Core: 

* A - string identifiers for channels
* B - channels as an special case of axis-level metadata

Other possible additions:

* C - canonical RGB-in-3-channels representation
* D - tags for channels (slugs or ontology terms) 
* X - handle current `omero.channels` as a fallback default rendering 
* X - channel grouping (m:n) via tags
* X - handling unbound axis (implicit channel=1)
* X - free-text descriptions 

## A - string identifiers for channels

allow `getChannelByName("x")` unambiguously for a single N-D array

<details>

### A0 - identity written in axis 
(dropped due to scenario with no bound channels) 

### A1 - array matching identifiers by index

```json
"channels": {
  "ids": [ "gfp", "dapi"]
}
```

### A2 - array of objects, implicitly linked by index

```json
"channels": [
 { "id": "gfp"}, 
 { "id": "dapi"}
], 
```

### A3 - array of objects, explicitly adding indexes  

```json
"channels": [
 { "id": "gfp", "index": 0}, 
 { "id": "dapi", "index": 1}
], 
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

<details>

### C1 - An explicit "rgb" key + value "R", "G" or "B" 
exactly 3 channels with rgb = oneOf("R", "G", "B")

```json
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

```

### C2 - Reserved ids for RGB

0 or exactly 3 channels with id=oneOf(R,G,B) 

```json
"channels": [
 { "id" : "R",
 },
 { "id" : "G"
  },
 { "id" : "B",
  },
]
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
</details>

## D - Tagging with slugs and ontology terms

optional

allow semantic identification of channels across multiple N-D images 

allow semantic grouping of channels in the same N-D images

allow search engine indexing of channel metadata

<details>
### D1 - Any string as a channel tag

```json
  "channels": [
       { "id" : "channel-0",
          "tags" : ["nuclei", "dapi"]
       },
       { "id" : "channel-1",
          "tags" : ["endogenous-tag", "tp53-gfp" ]
        },
      ]
```

### D2 - Any string as tag + any ontology term as a curie

curies SHOULD use bioregistry.io preferred prefixes

curies and tags MAY be completely different

```json
  "channels": [
       { "id" : "channel-0",
          "tags" : ["dapi"]
          "curies: ["GO:0005634"] 
       },
       { "id" : "channel-1",
          "tags" : ["endogenous-tag", "tp53-gfp" ]
        },
      ]
```

### D3 - ontology small objects

terms SHOULD use bioregistry.io preferred prefixes

labels SHOULD be the canonical label for the term

```json
  "channels": [
       { "id" : "channel-0",
          "ontology_ids" : [{"code":"GO:0005634", "label":"nucleus"}, {"code":"CHEBI:51231", "label": "DAPI"}]

       },
       { "id" : "channel-1",
         "ontology_ids": [{"code": "uniprot:P04637", "label":"TP53"}] 
        },
      ]
```
</details>



