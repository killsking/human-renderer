[![Build Status](https://travis-ci.com/andres-fr/human-renderer.svg?token=cUXVqzsqAP4ZpSpN77Kh&branch=master)](https://travis-ci.com/andres-fr/human-renderer)

# human-renderer


This repository hosts the `mpiea_mmr` (MPIEA MultiModalRenderer) add-on for Blender 2.80, as well as CI facilities to test, document and package it. It also includes related MakeHumand and Blender files.


### run all tests with coverage:

To be able to run the unit tests within blender, make sure both the module and its utest module are in blender's path (`ln -s <SOURCE> <LINK_PATH>` ). Check allowed paths from Blender with `import addon_utils; print(addon_utils.paths())`.


```
# with coverage:
blender -b --python ci_scripts/utest_with_coverage.py -- -n mpiea_mmr -p 0.9
# without coverage:
blender -b --python mpiea_mmr_utest/__init__.py
```


### Bump version:

Regular work is performed on the `dev` branch. After a milestone commit (and optional merge into `master`), tag it before pushing with:
```
bump2version {major | minor | patch}
```
And then a push will automatically trigger the tagged release.

### Build package:

```
python setup.py clean --all
rm -r *.egg-info
python setup.py sdist bdist_wheel
```


### Build docs:

Run this on repo root:

```
blender -b --python ci_scripts/make_sphinx_docs.py -- -n mpiea_mmr -a "Andres FR" -f .bumpversion.cfg -o docs -l
```


### Branching:

Travis builds get triggered on `master` and tagged builds only. Regular work can be done on a `dev` branch:

```
# create branch right after a commit:
git checkout -b dev
# work normally on it...
...
# track the new branch if you want to implicitly push to it:
git push -u origin dev # the first time, then `git push`

# once a milestone is reached, merge into master:
xx
```



## Setup:



### Blender

Install as follows:

1. Download version 2.80 (currently beta) from `https://www.blender.org/download/`
2. Unpack into `<BLENDERPATH>` (e.g. `~/opt/`)
3. Add to PATH in `.bashrc` or `.profile` (e.g. `export PATH=$HOME/opt/<BLENDER_FOLDER>:$PATH`)
4. now running `blender` will work on any terminal with that environment.


### Blender and pip:

It is possible to install packages into the Blender python via pip:

```
# install and upgrade pip
<BLENDER_PYTHONPATH> -m ensurepip
<BLENDER_PYTHONPATH> -m pip install --upgrade pip

# install the package:
./python3.7m -m pip install 'PATH/TO/dummypackage_dummyname-1.2.1-py3-none-any.whl'

# blender scripts can now import it

# to remove the package, simply do:
./python3.7m -m pip uninstall dummypackage_dummyname
```

Some packages cannot be installed due to a missing `Python.h` error. To fix that in ubuntu:

```
# Check the Blender Python version (in our case 3.7)
sudo apt install python3.7-dev
cp -r /usr/include/python3.7m <BLENDERPATH>/python/include
```

Now installing this repository's `requirements.txt` via `<BLENDERPYTHON> -m pip install -r requirements.txt` should work.

### Blender and custom Python installations:

Blender can execute a Python script upon startup, as follows:

```
blender --python your_script.py
```

It is also possible to call blender and the python script with separate command line arguments. It requires a custom argument parser as covered [here](https://blender.stackexchange.com/a/134596):

```
blender --blender_arg "hello" --python your_script.py -- --script_arg "world"
```

Regarding add-ons: https://docs.blender.org/manual/en/latest/advanced/scripting/addon_tutorial.html

> "there is nothing inherently different about an add-on that allows it to integrate with Blender, such functionality is just provided by the bpy module for any script to access".

An add-on script has only 3 extra requirements:
* a `bl_info` dictionary field with at least `"name"` and `"category"` entries.
* a `def register()` function.
* a `def unregister()` function.

If the add-on is a package instead of a script, the three elements can be in the `__init__.py` file.


To install the add-on, it has to be found by Blender's Python. The paths where it looks can be seen by running the following in the console:

```
import addon_utils
print(addon_utils.paths())
```

To install the add-on,
1. Copy (or place a link e.g. with `ln -s <SOURCE> <PLACE>`) the add-on module (or script) into any of the add_on paths.
2. Open Blender, and enable the add-on under `Edit -> Preferences -> Add-Ons` and save preferences.

**Note** That if the bl_info dict contains a "blender" entry pointing to a given version, other Blender versions won't be able to "see" the plugin. Make sure your Blender version matches!


### Install Makehuman

Install

```
sudo add-apt-repository ppa:makehuman-official/makehuman-11x
sudo apt update
sudo apt install makehuman
# run with makehuman, or (for specific python interpreter):
python2 /usr/share/makehuman/makehuman.py
```

Note that at the moment of writing, there is a bug in the official version `1.1.1`, which is incompatible with `numpy>1.12` (see [this post](http://www.makehumancommunity.org/forum/viewtopic.php?p=44716#p44716)) and throws an exception when trying to export a model. For this reason, and because MH runs on Python2, it is convenient to create a virtual environment just for MH usage:

```
conda update -n base -c defaults conda
conda create -n makehuman_env python=2.7 anaconda
conda activate makehuman_env
conda install pyopengl
conda install pyqt=4
conda install numpy=1.12
```

### Install MHX2 Plugin/Add-On

For best compatibility with Blender, MakeHuman models should be exported in [MHX2 format](https://thomasmakehuman.wordpress.com/2014/10/09/mhx2/). The project is hosted at bitbucket: https://bitbucket.org/Diffeomorphic/mhx2-makehuman-exchange

The README contains the installation details, but basically it involves 2 steps:

1. Copy the export routine into the makehuman plugins folder: `sudo cp -r 9_export_mhx2/ /usr/share/makehuman/plugins/`. This enables exporting MHX2 models from MH.

2. Install the `import_runtime_mhx2` Blender add-on as detailled before.


### Install MakeWalk:

MakeWalk is a Blender add-on that allows to load human motion capture sequences as BVH format and "retarget" them into MHX2 imported models. It is hosted at bitbucket: https://bitbucket.org/Diffeomorphic/makewalk

And can be installed like a regular Blender add-on, as detailled before.



### Basic scene


TODO:

* Sunlight with shadow
* Camlight attached and without shadow
* Floor with chess grid
* Realtime and HD render with OpenGL on GPU





# EXPERIMENTS

### Experiments with the Python console (see https://docs.blender.org/manual/en/latest/editors/python_console.html):


(Blender Python API: `https://docs.blender.org/api/2.79b/`)


https://docs.blender.org/api/2.78a/bpy.types.Object.html#bpy.types.Object.pose


https://docs.blender.org/api/2.78a/bpy.types.Pose.html




```
py.data.objects.keys()
bpy.data.objects["Testmodel"].pose.bones

```


## Pose serialization


### 1. BVH
The [BVH](https://research.cs.wisc.edu/graphics/Courses/cs-838-1999/Jeff/BVH.html) (copy of the link stored under `makehuman_data/BVH_format_explanation.html`) is a format for serialization of skeleton poses. It allows different structures, and the specification of sequences. It is a standard for motion capture compatible with MH, Blender and other editors.

Saved models (`.mhm`) include a line beginning with the `pose` keyword, pointing to the corresponding BVH (built-in poses are in `/usr/share/makehuman/data/poses/`). An example of such `.bvh` is stored under `makehuman_data/poses`.

[This link](https://sites.google.com/a/cgspeed.com/cgspeed/motion-capture/cmu-bvh-conversion) also points to thousands of BVH motion captures (small subset copied to this repo). Importing and playing them in Blender works out-of-the-box.

### 2. MHX2

Makehuman features different kinds of skeletons, the "Default" being the most detailled one. The file `/usr/share/makehuman/data/rigs/default.mhskel` contains the default skeleton definition (a copy is stored in this repo). Models exported as `.mhx2` include a `"skeleton"` entry, that implements the schema defined in `default.mhskel`. It provides the following entries for each bone: `name, parent, head_xyz, parent_xyz, roll_coeff`. Note the following:
    * This file can become very large due to the inefficient storage of the `vertices` information. Saving it as binary helps.
    * The XYZ system in Blender is `(right, deep, upwards)` (see [here](http://www.makehumancommunity.org/forum/viewtopic.php?p=35265&sid=4593dc12911e0b45b9f0342f6a4828d1#p35265)). **Probably** the 3-element vectors for head and tail stand for `(right, upwards, front)`, given `def zup(co): return Vector((co[0], -co[2], co[1]))` defined in `import_rutine_mhx2/utils.py` and used in `importer.py -> buildSkeleton`. I couldn't find any doc to confirm this, experiments will do.
    * It is unclear whether this format stores sequences
v

Load mhx2 into blender:

```
bpy.ops.import_scene.makehuman_mhx2(filepath="~/Desktop/myfile.mhx2")
bpy.ops.import_scene.makehuman_mhx2(filepath="~/github-work/human-renderer/makehuman_data/exported_models/testmodel.mhx2")
```





Related MH files:
  * `/usr/share/makehuman/plugins/3_libraries_skeleton/skeletonlibrary.py`
  * `/usr/share/makehuman/shared/skeleton.py`. The `Bone` class is the node of a tree, and has interesting methods like `getMatrix, get_roll_to, copy_normal...`. Docstring:

```
General skeleton, rig or armature class.
A skeleton is a hierarchic structure of bones, defined between a head and tail
joint position. Bones can be detached from each other (their head joint doesn't
necessarily need to be at the same position as the tail joint of their parent
bone).

A pose can be applied to the skeleton by setting a pose matrix for each of the
bones, allowing static posing or animation playback.
The skeleton supports skinning of a mesh using a list of vertex-to-bone
assignments.
```


BVH files are readily available, e.g. `http://www.makehumancommunity.org/content/figure_skating_sit_spin`. The `.mhm` saved models contain the `pose` pointing to the corresponding BVH (built-in poses are in `/usr/share/makehuman/data/poses/`). An example is stored under `makehuman_data/poses`.

There are some OS iniciatives to parse BVH:

* MH itself
* https://github.com/omimo/PyMO



Before evaluating which one is best, further assessment about the blender sequence rendering and python integration is needed.


### Blender sequences:




`bpy.ops.import_anim.bvh(filepath="/home/a9fb1e/github-work/human-renderer/makehuman_data/poses/cmu_motion_captures/01/01_06.bvh")`





## Makewalk:


To control MHX2 models using BHV within Blender, different sources suggest using the MakeWalk plugin. The PPA doesn't offer it, neither any official page. Older releases seem to have it included: `http://files.jwp.se/archive/releases/1.1.1/` (a copy is saved into this repo):
    1. Extract the makewalk folder and copy it into Blender's plugins folder (e.g. `~/opt/blender-2.79b-linux-glibc219-x86_64/2.79/scripts/addons_contrib`)
    2. Activate it in `Blender -> File -> User Preferences -> Add-ons` and save
    3. 


Set frame end:

```
bpy.data.scenes[0].frame_end = 1234 # the one for the player
bpy.context.scene.McpEndFrame = 2345 # the one for makewalk?
```

png images to mp4:

```
ffmpeg -i %04d.png -vf "transpose=2" output.mp4

```


## ISSUES:

* Time limits: the max nr of frames is about 1 million, i.e. 5 hours at 60 fps. https://docs.blender.org/manual/en/latest/advanced/limits.html
