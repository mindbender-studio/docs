#

![dropbox3](https://user-images.githubusercontent.com/2152766/27328354-cd712dd8-55a9-11e7-89b8-bb8b01b9c66d.png)

!!! note "Information-oriented"
    
    This section contain technical reference for APIs and other aspects of Avalon's machinery. They describe how it works and how to use it but assume that you have a basic understanding of key concepts.

Welcome to the Avalon pipeline reference. This is where all important concepts are documented
Mindbender provides a *stateful* API.

State is set and modified by calling any of the exposed registration functions, prefixed `register_*`.

## Schema

![image](https://cloud.githubusercontent.com/assets/2152766/22086121/8c037080-ddd7-11e6-9149-439203c32c6b.png)

Available schemas are organised hierarchically, with the former containing the latter.

- [`asset.json`](#assetjson)
    - [`subset.json`](#subsetjson)
        - [`version.json`](#versionjson)
            - [`representation.json`](#representationjson)

<br>

### `asset.json`

A part of a PROJECT, such as a Character or Shot.

**Example**

```json
{
    "data": {
        "key": "value"
    }, 
    "name": "Bruce", 
    "schema": "mindbender-core:asset-2.0", 
    "silo": "assets", 
    "type": "asset"
}
```

**Definition**

| Key | Value | Required? | Description
|:----|:------|:----------|:------------
| `data` | `dict` | `True` | Document metadata
| `silo` | `str` | `True` | Group or container of asset
| `type` | `str` | `True` | The type of document
| `name` | `str` | `True` | Name of asset
| `schema` | `str` | `True` | Schema identifier for payload

<br>

### `subset.json`

A part of an ASSET, such as a model or a rig.

**Example**

```json
{
    "data": {
        "endFrame": 1201, 
        "startFrame": 1000
    }, 
    "name": "shot01", 
    "schema": "mindbender-core:subset-2.0", 
    "type": "subset"
}
```

**Definition**

| Key | Value | Required? | Description
|:----|:------|:----------|:------------
| `data` | `dict` | `True` | Document metadata
| `type` | `str` | `True` | The type of document
| `name` | `str` | `True` | Name of directory
| `schema` | `str` | `True` | The schema associated with this document

<br>

### `version.json`

An immutable iteration of a SUBSET.

Versions are immutable, in that they never change once made. This is in stark contrast to mutable versions which is when one version may be "updated" such that the same file now contains new information.

**Example**

```json
{
    "data": {
        "author": "marcus", 
        "families": [
            "mindbender.model"
        ], 
        "source": "{root}/f02_prod/assets/BubbleWitch/work/modeling/marcus/maya/scenes/model_v001.ma", 
        "time": "20170510T090203Z"
    }, 
    "locations": [
        "data.mindbender.com"
    ], 
    "name": 12, 
    "schema": "mindbender-core:version-2.0", 
    "type": "version"
}
```

**Definition**

| Key | Value | Required? | Description
|:----|:------|:----------|:------------
| `locations` | `list` | `False` | Where on the planet this version can be found.
| `data` | `dict` | `True` | Document metadata
| `type` | `str` | `True` | The type of document
| `name` | `int` | `True` | Number of version
| `schema` | `str` | `True` | The schema associated with this document

<br>

### `representation.json`

One of many representations of a VERSION.

Think of a representation as one way of storing the same set of data on disk. For example, an image may be stored as both PNG and JPEG. Different files, same data. It could also be stored as a description. *"A picture of my computer."* Much less information is ultimately stored, but it is nonetheless the exact same original data in a different (albeit lossy) representation. The image could also be represented by a feeling (warm, mystical) or a spoken word (muah!).

Representation are very powerful and lie at the heart of assets that are more than just a single file.

As a practical example, a Look is stored as both an MA scene file and a JSON. The JSON stores the shader relationships, whereas the MA file stores the actual shaers. Same data, different representations.

**Example**

```json
{
    "context": {
        "asset": "Bruce", 
        "project": "hulk", 
        "representation": "ma", 
        "silo": "assets", 
        "subset": "rigDefault", 
        "version": 12
    }, 
    "data": {
        "label": "Alembic"
    }, 
    "dependencies": [
        "592d547a5f8c1b388093c145"
    ], 
    "name": "abc", 
    "schema": "mindbender-core:representation-2.0", 
    "type": "representation"
}
```

**Definition**

| Key | Value | Required? | Description
|:----|:------|:----------|:------------
| `name` | `str` | `True` | Name of representation
| `type` | `str` | `True` | The type of document
| `dependencies` | `list` | `False` | Other representation that this representation depends on
| `context` | `dict` | `False` | Summary of the context to which this representation belong.
| `data` | `dict` | `True` | Document metadata
| `schema` | `str` | `True` | Schema identifier for payload

<br>

### `container.json`

An imported VERSION, as yielded from `api.registered_host().ls()`.

**Example**

```json
{
    "asset": "Bruce", 
    "author": "Marcus Ottosson", 
    "name": "modelDefault", 
    "objectName": "modelDefault_CON", 
    "path": "|someParent|someNamespace_:modelDefault_CON", 
    "schema": "mindbender-core:container-2.0", 
    "subset": "modelDefault", 
    "version": 12
}
```

**Definition**

| Key | Value | Required? | Description
|:----|:------|:----------|:------------
| `subset` | `str` | `True` | Name of source subset
| `name` | `str` | `True` | Full name of application object
| `objectName` | `str` | `True` | Name of internal object, such as the objectSet in Maya.
| `author` | `str` | `False` | Name of the author of the published version
| `version` | `int` | `True` | Version number
| `asset` | `str` | `True` | Name of source asset
| `path` | `str` | `True` | Absolute path
| `schema` | `str` | `True` | Schema identifier for payload

Mindbender hosts a series of [graphical user interfaces](#batteries) that aid the user in conforming to the specified [contracts](#contract).

| Name              | Purpose                          | Description
|:------------------|:---------------------------------|:--------------
| **creator**            | control what goes out           | Manage how data is outputted from an application.
| **loader**            | control what goes in             | Keep tabs on where data comes from so as to enable tracking and builds.
| **manager**           | stay up to date                  | Notification and visualisation of loaded data.

<br>
<br>

Public members of `mindbender.api`

| Member | Returns | Description
|:-------|:--------|:--------
| `install` | `null` | Install `host` into the running Python session.
| `uninstall` | `null` | Undo all of what `install()` did
| `schema` | `null` | JSON Schema utilities
| `Loader` | `null` | Load representation into host application
| `Creator` | `null` | Determine how assets are created
| `discover` | `null` | Find and return subclasses of `superclass`
| `register_host` | `null` | Register a new host for the current process
| `register_format` | `null` | Register a supported format
| `register_plugin_path` | `null` | Register a directory of one or more plug-ins
| `register_plugin` | `null` | Register an individual `obj` of type `superclass`
| `register_root` | `null` | Register currently active root
| `registered_root` | `null` | Return currently registered root
| `registered_plugin_paths` | `null` | Return all currently registered plug-in paths
| `registered_host` | `null` | Return currently registered host
| `deregister_plugin` | `null` | Oppsite of `register_plugin()`
| `deregister_plugin_path` | `null` | Oppsite of `register_plugin_path()`
| `deregister_format` | `null` | Deregister a supported format
| `format_staging_dir` | `null` | Return directory used for staging of published assets
| `format_version` | `null` | Produce filesystem-friendly string from integer version
| `find_latest_version` | `null` | Return latest version from list of versions
| `parse_version` | `null` | Return integer version from formatted string
| `logger` | `null` | 
| `time` | `null` | Return file-system safe string of current date and time

<br>
<br>

### Host API

A host must implement the following members.

| Member                                 | Returns    | Description
|:---------------------------------------|:-----------|:--------
| `ls()`                                 | `generator`| List loaded assets
| `create(name, family, options=None)`   | `str`      | Build fixture for outgoing data (see [instance]()), returns instance.
| `load(asset, subset, version=-1, representation=None)` | `None`      | Import external data into [container]()
| `update(container, version=-1)`        | `None`     | Update an existing container
| `remove(container)   `                 | `None`     | Remove an existing container

<br>

#### Information hierarchy

Loaded data is stored in a `container`. A container hosts a loaded asset along with metadata used to associate assets that use other assets, such as a Wheel asset used in a Car asset.

![Host data relationship](https://cloud.githubusercontent.com/assets/2152766/18905784/aa6a3d5c-855c-11e6-9843-b24ebd23c4ac.png)

#### Id

Internally, Pyblish instances and containers are distinguished from native content via an "id". For example, in Maya, the `id` is a user-defined attribute.

| Name                         | Description              | Example
|:-----------------------------|:-------------------------|:----------
| `pyblish.mindbender.container`  | Unit of incoming data    | `...:model_GRP`, `...:rig_GRP` 
| `pyblish.mindbender.instance`   | Unit of outgoing data    | `Strange_model_default`

<br>
<br>

### Project Inventory API

The inventory contains all ASSETs of a project, including metadata.

- [Icon Database](http://fontawesome.io/icons/)

**.inventory.toml**

```ini
# Mandatory, do not touch
schema = "mindbender-core:inventory-1.0"

# Project metadata
label = "The Hulk"
fps = 24
resolution_width = 1920
resolution_height = 1080

# Available assets
[[assets]]
name = "Batman"

[[assets]]
name = "Bruce"
label = "Bruce Wayne"  # (Optional) Nicer name
group = "Character"  # (Optional) Visual grouping
icon = "gear"  # (Optional) Icon from FontAwesome

[[assets]]
name = "Camera"

# Available shots
[[film]]
name = "1000"
edit_in = 1000
edit_out = 1202

[[film]]
name = "1200"
edit_in = 1000  # Optional metadata per shot
edit_out = 1143
```

The above is an example of an "inventory". A complete snapshot of all available assts within a given project, along with optional metadata.

<br>
<br>

### Project Configuration API

The project configuration contains the applications and tasks available within a given project, along with the template used to create directories.

**.config.toml**

```ini
# Mandatory, do not touch
schema = "mindbender-core:config-1.0"

# Available tasks to choose from.
[[tasks]]
name = "modeling"
label = "Character Modeling"
icon = "video-camera"

[[tasks]]
name = "animation"

# Available applications to choose from, the name references
# the executable API (see below)
[[apps]]
name = "maya2016"
label = "Autodesk Maya 2016"

[[apps]]
name = "python"
label = "Python 3.6"
args = ["-u", "-c", "print('Something nice')"]

# Directory layouts for this project.
[template]
work = "{root}/{project}/{silo}/{asset}/work/{task}/{user}/{app}"
publish = "{root}/{project}/{silo}/{asset}/publish/{subset}/v{version:0>3}/{subset}.{representation}"
```

The directory layout have the following members available.

| Member             | Type   | Description
|:-------------------|:-------|:-----------
| `{app}`            | `str`  | The current application directory name, defined in Executable API
| `{task}`           | `str`  | Name of the current task
| `{user}`           | `str`  | Currently logged on user (provided by `getpass.getuser()`)
| `{root}`           | `str`  | Absolute path to root directory, e.g. `m:\f01_project`
| `{project}`        | `str`  | Name of current project
| `{silo}`           | `str`  | Name of silo, e.g. `assets`
| `{asset}`          | `str`  | Name of asset, e.g. `Bruce`
| `{subset}`         | `str`  | Name of subset, e.g. `modelDefault`
| `{version}`        | `int`  | Number of version, e.g. `1`
| `{representation}` | `str`  | Name of representation, e.g. `ma`

<br>
<br>

### Project Executable API

Every executable must have an associated Application Definition file which looks like this.

**maya2016.toml**

```ini
# Required header, do not touch.
schema = "mindbender-core:application-1.0"

# Name of the created directory, available in the 
# `template` of the Configuration API
application_dir = "maya"

# These directories will be created under the
# given application directory
default_dirs = [
    "scenes",
    "data",
    "renderData/shaders",
    "images"
]

# Name displayed in GUIs
label = "Autodesk Maya 2016x64"

# Arguments passed to the executable on launch
arguments = [ "-proj", "{MINDBENDER_WORKDIR}",]

# Name of the executable on the local computer.
# This name must be available via the users `PATH`.
# That is, the user must be able to type this into
# the terminal to launch said application.
executable = "maya2016"
description = ""

# Files copied into the application directory on launch
[copy]
"{MINDBENDER_CORE}/res/workspace.mel" = "workspace.mel"

# The environment variables overrides any previously set
# variables from the parent process.
[environment]
MAYA_DISABLE_CLIC_IPM = "Yes"  # Disable the AdSSO process
MAYA_DISABLE_CIP = "Yes"  # Shorten time to boot
MAYA_DISABLE_CER = "Yes"
PYTHONPATH = [
    "{PYBLISH_MAYA}/pyblish_maya/pythonpath",
    "{MINDBENDER_CORE}/mindbender/maya/pythonpath",
    "{PYTHONPATH}"
]
```

<br>
<br>

<br>

## Creator

Associate content with a family.

The family is what determins how the content is handled throughout your pipeline and tells Pyblish what it should look like when valid.

### API

The creator respects families registered with Mindbender.

```python
from mindbender import api

api.register_family(
    name="my.family",
    help="My custom family",
)
```

For each family, a **common set of data** is automatically associated with the resulting instance.

```python
{
    "id": "pyblish.mindbender.instance",
    "family": {chosen family}
    "name": {chosen name}
}
```

**Additional common** data can be added.

```python
from mindbender import api

api.register_data(
    key="myKey",
    value="My value",
    help="A special key"
)
```

Data may be **associated** with a family.

```python
from mindbender import api

api.register_family(
    name="my.family",
    data=[
        {"key": "name", "value": "marcus", "help": "Your name"},
        {"key": "age", "value": 30, "help": "Your age"},
])
```

<br>
<br>

<img class="ornament" src="https://cloud.githubusercontent.com/assets/2152766/25314676/6405898e-2840-11e7-9a09-3a193d6eaf1f.png">

## Loader

Visualise results from `api.ls()`.

```python
from mindbender import api

for asset in api.ls():
    print(asset["name"])
```

### API

The results from `api.ls()` depends on the currently **registered root**.

```python
from mindbender import api
api.register_root("/projects/gravity")
```

The chosen `ASSET` is passed to the `load()` function of the currently registered host.

```python
from mindbender import api, maya
api.register_host(maya)
```

A host is automatically registered on `api.install()`.

<br>
<br>

<img class="ornament" src="https://cloud.githubusercontent.com/assets/2152766/25314689/8b80cc58-2840-11e7-9bee-a97a40fa830d.png">

## Manager

Visualise loaded assets.

```python
from mindbender import maya

for container in maya.ls():
    print(container["name"])

# The same is true for any host; houdini, nuke etc.
```

### API

The results from `ls()` depends on the currently registered host, such as Maya.

```python
from mindbender import nuke
api.register_host(nuke)
```