# How to read this 

This is a design document for `channels` metadata on OME-NGFF. 

It lists design options for different features, providing a blueprint for decision making and Architectural Decision Records. 

Each feature has a letter ("A") and each of the possible designs get a letter+number code ("A1"). 

Examples use [0.5](https://ngff.openmicroscopy.org/0.5/) as a base, but should be interpreted as version-agnostic. 

Any designs that are incompatible with OME-Zarr 1.0 (e.g. conflicting with other works in progress) are avoided. 
If you find such design flaws, please let me know.

you may contribute by: 

* sharing which of the features you care the most about
* providing arguments for and against particular designs
* reporting design flaws
* suggesting new features

# Features
## Core: 

* A - string identifiers for channels
* B - channels as an special case of axis-level metadata

## Other:

* C - canonical RGB-in-3-channels representation
* D - tags for channels (slugs or ontology terms)
* E - free-text descriptions 
* F - advance discussion on the `omero.channels` block 
* G - channel grouping (m:n) --> currently designed as part of D 
* H - handling unbound axis (implicit channel=1)

# Out of scope:

* detailed acquisition metadata 
* detailed biological metadata
* detailed rendering metadata

## A - string identifiers for channels

allow `getChannelByName("x")` unambiguously for a single N-D array

<details>

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

## C - Photometric interpretation / RGB processing 

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
                "rgb": "#0000FF"
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

## E - Free-text descriptions for channels 

allow humans and language models to use expressive natural language information 

<details>
### E1 - A `label` and a `description`

optional 
```json
  "channels": [
       { "id" : "channel-0",
          "label": "H2A-mCherry nuclei",
          "description": "nuclei stained with via the endogenous chromatin marker H2A-mCherry"
       },
       { "id" : "channel-1",
          "label": "drl-GCamp6s",
          "description": "Calcium concentration in mesodermal cells, imaged using GCamp6s expressed via genetic construct with regulatory elements of the draculin (drl) gene."
        },
      ]
```

### E2 - Only a `label` key

```json
  "channels": [
       { "id" : "channel-0",
          "label": "H2A-mCherry nuclei",
       },
       { "id" : "channel-1",
          "label": "drl-GCamp6s",
        },
      ]
```

### E3 - Conflate the "id" and the "label" in a single element 


```json
  "channels": [
       { "id" : "H2A-mCherry nuclei",
       },
       { "id" : "drl-GCamp6s",
        "label": "",
        },
      ]
```
</details>

### F - advance discussion on the `omero.channels` block 

unambiguously describe the relation of `channels` and `omero.channels` 

non-goals: 
 - a full rendering spec
 - a backbone for a full rendering spec

stretch goals: 
 - a minimal joint rendering spec as a fallback (_not_ transitional, but semantically inferior in relation a future rendering spec)

<details>

### F1 - move `omero.channels` as it is, don't touch other `omero` keys

`omero.channels` gets deprecated

all `omero.channels` keys become valid `channels` keys

```json
  "omero": {
           "id": 1,                             
           "name": "example.tif",
            }
  "channels": [
    {   "id": "channel-0", 
        "active": true,
        "color": "0000FF",
        "inverted": false,
        "label": "H2A-mCherry",
        "window": {
            "end": 1500,
            "max": 65535,
            "min": 0,
            "start": 0
        }
    }
      ]
```

### F2 - don't touch `omero.channels` 

```json
  "omero": {
           "id": 1,                             
           "name": "example.tif",
            "channels": [
                    {   "id": "channel-0", 
                        "active": true,
                        "color": "0000FF",
                        "inverted": false,
                        "label": "H2A-mCherry"
                    }
                ]
            },
  "channels": [
    {   "id": "channel-0" },
    {   "id": "channel-1"}
]
```
### F3 - move `omero.channels` and apply changes to keys 

`omero.channels` gets deprecated

`omero.channels` keys are used only when necessary

keys may be renamed for clarity 

(example below out of many possible)

```json
  "omero": {
           "id": 1,                             
           "name": "example.tif",
            }
  "channels": [
    {   "id": "channel-0", 
        "color": "0000FF",
        "inverted": false,
        "label": "H2A-mCherry",
        "value_limits": {
                "min": 0,
                "max": 1234
           },
        "contrast_limits": {
               "start": 50,
               "end": 1000
           }
    }
      ]
```

</details>

## G - grouping channels 

(see ## D - Tagging with slugs and ontology terms) 

## H - handling unbound axis (implicit channel=1)

<details>
  
### H1 - implicit assume that if channel is not bound, channels[0] refers to it

```json
"channels": [
       { "id" : "channel-x",}
]
```

### H2 - `"unbound":"true"`

```json
"channels": [
       { "id" : "channel-x",
         "unbound": true        
       }
]
```

### H3 - explicit `"index": null`

```json
"channels": [
       { "id" : "channel-x",
         "index" : null        
       }
]
```
</details>

