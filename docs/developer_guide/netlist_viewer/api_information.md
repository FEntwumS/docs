# Backend API information for frontend developers

## Format of the layouted netlist
The returned JSON document is formatted as ELK-JSON. The exact specification may be found
in [the ELK documentation](https://eclipse.dev/elk/documentation/tooldevelopers/graphdatastructure/jsonformat.html). The
following options are used in the generation of the document:

- `omitZeroDimension` &#x2192; Whenever no dimension is specified for a given element, the receiving program should
  assume that it is `0` instead
- `omitZeroPosition` &#x2192; Whenever no position is specified for a given element, the receiving program should assume
  that it is `0` instead
- `shortLayoutOptionKeys` &#x2192; All layout options are shortened to reduce file size

The document contains some, if not all the options described [here](extra_layout_options.md) in addition to the layout options included in ELK.

## Hash generation

To uniquely identify the netlists on the backend server, the client must send an identifying hash alongside them.
This hash is generated using the [one_at_a_time hash function](https://en.wikipedia.org/wiki/Jenkins_hash_function#one_at_a_time) by Bob Jenkins.
The hash is a 64 bit unsigned integer, where the upper 32 bits are the hashed absolute path of the netlist file, while the lower 32 bits are the hashed contents of the netlist file.

## Creating a graph representation of a given netlist

### `/graphLocalFile`

Parameters:

1. `filename`: The absolute path pointing to the netlist file
2. `hash`: The hash that is to be usd for this file. This hash is generated as described [here](#hash-generation)

Notes:
- This call only works when the file is located on the same system as the running backend executable.

### `/graphRemoteFile`

Parameters:

1. `file`: A multipart file, containing the netlist files' contents
2. `hash`: The hash that is to be usd for this file. This hash is generated as described [here](#hash-generation)

## Modifying or retreiving an existing graph

The following calls only return results if the referenced netlist already exists on the backend server.

### `/expandNode`

This call collapses the given cell, if it is expanded (its' children are visible) or vice-versa.

Parameters:

1. `hash`: The hash that is to be usd for this file. This hash is generated as described [here](#hash-generation)
2. `nodepath`: The path of the node that is to be collapsed or expanded. Can be found in the [location-path option](extra_layout_options.md#dethkoelnfentwumsnetlistbackendlocation-path)

Notes:

- While cells have their `location-path` set, they can neither be expanded or collapsed, as they have no children
- This call forces a partial re-layout of the graph. Depending on the number of visible entities and cells, this may take a while.

### `/get-current-graph`

Parameters:

1. `hash`: The hash that is to be usd for this file. This hash is generated as described [here](#hash-generation)

## Mapping signal names to yosys bitindices

### `/get-net-information`

Parameters:

1. `hash`: The hash that is to be usd for this file. This hash is generated as described [here](#hash-generation)

Notes: 

- The JSON document is formatted as follows:

  ```
  {
    "signals": {
      "<signalname>": {
        "scope": "<hdlname|scopename>",
        "bits": [
          <bitindex>,
          ...
        ]
      },
      ...
    }
  }
  ```