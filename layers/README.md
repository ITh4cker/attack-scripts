# layers

This folder contains modules and scripts for working with ATT&CK Navigator layers. ATT&CK Navigator Layers are a set of annotations overlayed on top of the ATT&CK Matrix. For more about ATT&CK Navigator layers, visit the ATT&CK Navigator repository. The core module allows users to load, validate, manipulate, and save ATT&CK layers. A brief overview of the components can be found below. All scripts adhere to the MITRE ATT&CK Navigator Layer file format, [version 3.0](https://github.com/mitre-attack/attack-navigator/blob/develop/layers/LAYERFORMATv3.md).

#### Core Modules
| script | description |
|:-------|:------------|
| [filter](core/filter.py) | Implements a basic [filter object](https://github.com/mitre-attack/attack-navigator/blob/develop/layers/LAYERFORMATv3.md#filter-object-properties). |
| [gradient](core/gradient.py) | Implements a basic [gradient object](https://github.com/mitre-attack/attack-navigator/blob/develop/layers/LAYERFORMATv3.md#gradient-object-properties). |
| [layer](core/layer.py) | Provides an interface for interacting with core module's layer representation. A further breakdown can be found in the corresponding [section](#Layer) below. |
| [layout](core/layout.py) | Implements a basic [layout object](https://github.com/mitre-attack/attack-navigator/blob/develop/layers/LAYERFORMATv3.md#layout-object-properties). |
| [legenditem](core/legenditem.py) | Implements a basic [legenditem object](https://github.com/mitre-attack/attack-navigator/blob/develop/layers/LAYERFORMATv3.md#legenditem-object-properties). |
| [metadata](core/metadata.py) | Implements a basic [metadata object](https://github.com/mitre-attack/attack-navigator/blob/develop/layers/LAYERFORMATv3.md#metadata-object-properties). |
| [technique](core/technique.py) | Implements a basic [technique object](https://github.com/mitre-attack/attack-navigator/blob/develop/layers/LAYERFORMATv3.md#technique-object-properties). |

#### Manipulator Scripts
| script | description |
|:-------|:------------|
| [layerops](manipulators/layerops.py) | Provides a means by which to combine multiple ATT&CK layer objects in customized ways. A further breakdown can be found in the corresponding [section](#layerops.py) below. |

#### Exporter Scripts
| script | description |
|:-------|:------------|
| [to_excel](exporters/to_excel.py) | Provides a means by which to export an ATT&CK Layer to an excel file. A further breakdown can be found in the corresponding [section](#to_excel.py) below. |
##### Utility Modules
| script | description |
|:-------|:------------|
| [excel_templates](exporters/excel_templates.py) | Provides a means by which to convert a matrix into a clean excel matrix template. |
| [matrix_gen](exporters/matrix_gen.py) | Provides a means by which to generate a matrix from raw data, either from the ATT&CK TAXII server or from a local STIX Bundle. |

## Layer
The Layer class provides format validation and read/write capabilities to aid in working with ATT&CK Navigator Layers in python. It is the primary interface through which other Layer-related classes defined in the core module should be used. The Layer class API and a usage example are below.

| method [x = Layer()]| description |
|:-------|:------------|
| x.from_str(_input_) | Loads an ATT&CK layer from a string representation of a json layer. |
| x.from_dict(_input_) | Loads an ATT&CK layer from a dictionary. |
| x.from_file(_filepath_) | Loads an ATT&CK layer from a file location specified by the _filepath_. |
| x.to_file(_filepath_) | Saves the current state of the loaded ATT&CK layer to a json file denoted by the _filepath_. |
| x.to_dict() | Returns a representation of the current ATT&CK layer object as a dictionary. |
| x.to_str() | Returns a representation of the current ATT&CK layer object as a string representation of a dictionary. | 

#### Example Usage

```python
example_layer_dict = {
    "name": "example layer",
    "version": "3.0",
    "domain": "mitre-enterprise"
}

example_layer_location = "/path/to/layer/file.json"
example_layer_out_location = "/path/to/new/layer/file.json"

from layers.core import Layer

layer1 = Layer(example_layer_dict)              # Create a new layer and load existing data
layer1.to_file(example_layer_out_location)  # Write out the loaded layer to the specified file

layer2 = Layer()                                # Create a new layer object
layer2.from_dict(example_layer_dict)           # Load layer data into existing layer object
print(layer2.to_dict())                        # Retrieve the loaded layer's data as a dictionary, and print it

layer3 = Layer()                                # Create a new layer object
layer3.from_file(example_layer_location)        # Load layer data from a file into existing layer object
```

## layerops.py
Layerops.py provides the LayerOps class, which is a way to combine layer files in an automated way, using user defined lambda functions. Each LayerOps instance, when created, ingests the provided lambda functions, and stores them for use. An existing LayerOps class can be used to combine layer files according to the initialized lambda using the process method. The breakdown of this two step process is documented in the table below, while examples of both the list and dictionary modes of operation can be found below.

##### LayerOps()
```python
 x = LayerOps(score=score, comment=comment, enabled=enabled, colors=colors, metadata=metadata, name=name, desc=desc, default_values=default_values)
```
 
 Each of the _inputs_ takes a lambda function that will be used to combine technique object fields matching the parameter. The one exception to this is _default_values_, which is an optional dictionary argument containing default values to provide the lambda functions if techniques of the combined layers are missing them.

##### .process() Method
```python
x.process(data, default_values=default_values)
```
The process method applies the lambda functions stored during initialization to the layer objects in _data_. _data_ must be either a list or a dictionary of Layer objects, and is expected to match the format of the lambda equations provided during initialization. default_values is an optional dictionary argument that overrides the currently stored default
 values with new ones for this specific processing operation.

#### Example Usage
```python
from layers.manipulators.layerops import LayerOps
from layers.core.layer import Layer

demo = Layer()
demo.from_file("C:\Users\attack\Downloads\layer.json")
demo2 = Layer()
demo2.from_file("C:\Users\attack\Downloads\layer2.json")
demo3 = Layer()
demo3.from_file("C:\Users\attack\Downloads\layer3.json")

# Example 1) Build a LayerOps object that takes a list and averages scores across the layers
lo = LayerOps(score=lambda x: sum(x) / len(x), 
              name=lambda x: x[1], 
              desc=lambda x: "This is an list example")     # Build LayerOps object
out_layer = lo.process([demo, demo2])                       # Trigger processing on a list of demo and demo2 layers
out_layer.to_file("C:\demo_layer1.json")                    # Save averaged layer to file
out_layer2 = lo.process([demo, demo2, demo3])               # Trigger processing on a list of demo, demo2, demo3
visual_aid = out_layer2.to_dict()                           # Retrieve dictionary representation of processed layer

# Example 2) Build a LayerOps object that takes a dictionary and averages scores across the layers
lo2 = LayerOps(score=lambda x: sum([x[y] for y in x]) / len([x[y] for y in x]), 
               color=lambda x: x['b'], 
               desc=lambda x: "This is a dict example")      # Build LayerOps object, with lambda
out_layer3 = lo2.process({'a': demo, 'b': demo2})            # Trigger processing on a dictionary of demo and demo2
dict_layer = out_layer3.to_dict()                            # Retrieve dictionary representation of processed layer
print(dict_layer)                                            # Display retrieved dictionary
out_layer4 = lo2.process({'a': demo, 'b': demo2, 'c': demo3})# Trigger processing on a dictionary of demo, demo2, demo3
out_layer4.to_file("C:\demo_layer4.json")                    # Save averaged layer to file

# Example 3) Build a LayerOps object that takes a single element dictionary and inverts the score
lo3 = LayerOps(score=lambda x: 100 - x['a'],
               desc= lambda x: "This is a simple example")  # Build LayerOps object to invert score (0-100 scale)
out_layer5 = lo3.process({'a': demo})                       # Trigger processing on dictionary of demo
print(out_layer5.to_dict())                                 # Display processed layer in dictionary form
out_layer5.to_file("C:\demo_layer5.json")                   # Save inverted score layer to file

# Example 4) Build a LayerOps object that combines the comments from elements in the list, with custom defaults
lo4 = LayerOps(score=lambda x: '; '.join(x),
               default_values= {
                "comment": "This was an example of new default values"
                },
               desc= lambda x: "This is a defaults example")  # Build LayerOps object to combine descriptions, defaults
out_layer6 = lo4.process([demo2, demo3])                      # Trigger processing on a list of demo2 and demo0
out_layer6.to_file("C:\demo_layer6.json")                     # Save combined comment layer to file
```

## to_excel.py
to_excel.py provides the ToExcel class, which is a way to export an existing layer file as an Excel 
spreadsheet. The ToExcel class has an optional parameter for the initialization function, that 
tells the exporter what data source to use when building the output matrix. Valid options include using live data from cti-taxii.mitre.org or using a local STIX bundle. 

##### ToExcel()
```python
x = ToExcel(domain='enterprise', source='taxii', local=None)
```
The ToExcel constructor takes domain, server, and local arguments during instantiation. The domain can 
be either `enterprise` or `mobile`, and can be pulled directly from a layer file as `layer.domain`. The source argument tells the matrix generation tool which data source to use when building the matrix. `taxii` indicates that the tool should utilize the `cti-taxii` server when building the matrix, while the `local` option indicates that it should use a local bundle respectively. The local argument is only required if the source is set to `local`, in which case it should be a path to a local stix bundle.

##### .to_file() Method
```python
x.to_xlsx(layer=layer, filepath="layer.xlsx")
```
The to_xlsx method exports the layer file referenced as `layer`, as an excel file to the 
`filepath` specified. 

#### Example Usage
```python
from layers import Layer
from layers import ToExcel

lay = Layer()
lay.from_file("path/to/layer/file.json")
# Using taxii server for template
t = ToExcel(domain=lay.layer.domain, source='taxii')
t.to_xlsx(layer=lay, filepath="demo.xlsx")
#Using local stix data for template
t2 = ToExcel(domain='mobile', source='local', local='path/to/local/stix.json')
t2.to_xlsx(layer=lay, filepath="demo2.xlsx")
```