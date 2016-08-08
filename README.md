# UnrealEnginePython
Embed Python in Unreal Engine 4

# How and Why ?

This is a plugin allowing the mapping of Actor and Components to python classes (and a lot more). It is not meant as a way to
avoid blueprints or c++ but as a good companion to them (albeit reducing the amount of c++ required for coding a game could be an interesting thing ;)

Another funny feature is that you can change your python code even after the project has been packaged. You can potentially build a completely new game from an already packaged one.

The plugin embeds the python3 library and automatically add an actor class (PyActor) and a component class (PythonComponent).
Both are available in your editor once you load the plugin.

All of the exposed engine features are under the 'unreal_engine' virtual module (it is completely coded in c into the plugin, so do not expect to run 'import unreal_engine' from a standard python shell)

# Installation on Windows (64 bit)

The installation is pretty long (and boring) as you do not want the final users of the product to be required to install python, so we need to use the 'embedded python distribution' (available for windows in the official python site). Unfortunately the embedded distribution does not contain the python development headers so we need the official system-wide installation too.

* install Unreal Engine 4.12 and Python 3.5 as a system user
* run the unreal editor and create a new blank c++ project
* once the project is setup, close both unreal and visual studio
* move to the project directory (you can right click the problem from the epic launcher and choose 'show in explorer')
* create a 'Plugins' directory into the project directory
* move into the just created Plugins directory and clone the repository:

```sh
git clone https://github.com/20tab/UnrealEnginePython
```

* from the explorer right click on the project main file and choose 'generate visual studio project files'
* open again visual studio, you should now see Plugins/UnrealEnginePython in your solution explorer
* before running the plugin build process you need to copy development headers and libs in the plugin directory (Plugins/UnrealEnginePython).
* create the directory Plugins/UnrealEnginePython/python35
* copy "C:/Program Files/Python35/include" and "C:/Program Files/Python35/libs" into Plugins/UnrealEnginePython/python35
* run the compilation from visual studio
* once the compilation ends, copy the python35.dll (from "C:/Program Files/Python35" or from the embeded distribution) into the Binaries/Win64 directory of your project (you will find the plugin dll too in this directory)
* now you can re-run the unreal engine editor

If all goes well, open the output log and search for the string "Python". You should see the Python VM initialization message. It means your editor has now full python support.

# Installation on MacOSX

On the Mac the installation is easier, as the final user is currently forced to install python on its system (there are obviously dozens of workarounds but at this stage of the project we prefer focusing on the api).

* install the latest official python distribution from python.org (the installation will end in the "Library/Frameworks/Python.framework/Versions/3.5" directory).
* create a new unreal engine blank c++ project
* create a Plugins directory in the project directory
* move to the Projects directory and clone the plugin repository

```sh
git clone https://github.com/20tab/UnrealEnginePython
```

* restart the editor and a popup should appear asking your for confirmation of the build of the plugin.
* Once the plugin is built, go to the output log console and filter for 'Python'. You should see the Python VM banner.

# Installation on other platforms

Currently only Windows and MacOSX are supported, Linux support should be available soon. We are investigating Android support too via the kivy project.

# Using Python with Unreal Engine (finally)

Creating a new blueprint class managed by python
------------------------------------------------

We are going to create a new Actor based on python (instead of C++ or blueprints)

In the content browser click on 'add new' and choose 'blueprint class'

In the classes menu choose 'PyActor':

![Alt text](screenshots/unreal_screenshot1.png?raw=true "Screenshot 1")

You now have a new asset, give it a meaningful name, and double click on it to start configuring it in the blueprint editor

![Alt text](screenshots/unreal_screenshot2.png?raw=true "Screenshot 2")

On the right (in the 'Details' tab) you will find the Python section.

For now only 'Python Module' and 'Python Class' are meaningful.

Go to the Content directory of your project and create a directory named 'Scripts'. This is where all of your python modules will reside. With your favourite text editor create a new python module (like funnygameclasses.py), and define a new class into it:

```py
import unreal_engine as ue

ue.log('Hello i am a Python module')

class Hero:

    def begin_play(self):
        ue.log('Begin Play on Hero class')
        
    def tick(self, delta_time):
        # get current location
        x, y, z = self.uobject.get_actor_location()
        # increase Z honouring delta_time
        z += 100 * delta_time
        # set new location
        self.uobject.set_actor_location(x, y, z)

```

Now, go back to the blueprint editor and set 'funnygameclasses' in the 'Python Module' field, and 'Hero' in 'Python Class'

As you can see the actor will simply move over the z axis, but we need to give it some kind of visual representation to have a feedback in the scene. In the blueprint editor click on 'add component' and add some shape (a spehere, or a cube, or whatever you want). Save and Compile your blueprint.

Now you can drag the bluprint from the content browser to the scene and just click 'Play'.

You should see your actor moving along the 'z' axis at a speed of 1 meter per second

What is 'self.uobject' ?
------------------------

To allows seamless Python integration, each UObject of the engine is automatically mapped to a special Python Object (ue_PyUObject).

Whenever you want to access a UObject from python, you effectively get a reference to a ue_PyUObject exposing (via its methods) the features of the UObject (properties, functions, ....)

This special python object is cached into the UObject (via the __PyObject property) so you do not need to recreate it every time (and more important you can hold state information nto the python class)

To be more clear, a call to:

```py
text_render_component = unreal_engine.find_class('TextRenderComponent')
```

will internally search for the 'TextRenderComponent' class (via c++ reflection) and when found will check if it exposes a '__PyObject' property. If available, it wil lreturn its value to the Python VM, otherwise a new one will be created and mapped to it.

From the previous example the 'text_render_component' maintain a mapping to the UObject (well a UClass in this example).

Pay attention: the python class you map to the PyActor class, is not a ue_PyUObject (it is a classic python class) but hold a reference (via the 'uobject' field) to a ue_PyUObject mapped to the AActor.

Note about 'uobject' from now on
---------------------------------

In the following lines, whenever you find a reference to 'uobject' it is meant as a ue_PyUObject object.

Adding a python component to an Actor
-------------------------------------

This works in the same way as the PyActor class, but it is, well, a component. You can attach it (search for the 'Python' component) to any actor.

Remember that for components, the self.uobject field point to the component itself, not the actor.

To access the actor you can use

```py
actor = self.uobject.get_class()
```

The unreal_engine module
------------------------

This is a generic module, exposing functions not related to a particular 'World' (see below).

You can use these functions from an actor, a component or directly from a module.

Just remember to import it:

```py
import unreal_engine
```

or (as an example)

```py
import unreal_engine as ue
```


```py
uclass = unreal_engine.find_class('name')
```

This function searches for a class in the engine with the specified name and returns its 'uobject' (a c++ UClass object internally).
You can use this uobject to search for objects of that type or to spawn new actors (and lot of more things)



```py
unreal_engine.log('message')
```

log a string into the editor output log (under the LogPython category/class)

```py
unreal_engine.log_warning('message')
```

log a warning string (yellow) into the editor output log (under the LogPython category/class)

```py
unreal_engine.log_error('message')
```

log an error string (red) into the editor output log (under the LogPython category/class)

```py
unreal_engine.add_on_screen_debug_message(key, timeout, 'message')
```

low-level equivalent of blueprint 'print string' function. It disappear after 'timeout' seconds and can get a numeric key (use -1 for disabling keys feature)

TODO: support for colors

```py
unreal_engine.print_string('message')
```

python equivalent of the blueprint 'print string' function. It disappears after 2 seconds and it is wrote in cyan color.

```py
unreal_engine.vector_add_vector(x, y, z[, x1, y1, z1, ...])
```

optimized vector math function, allows adding 3 dimensions vectors exposed as 3 float values

```py
unreal_engine.vector_add_float(x, y, z[, d1, d2, ...])
```

optimized vector math function, allows adding a 3 dimensions vector with one or more floats

```py
unreal_engine.vector_mul_vector(x, y, z[, x1, y1, z1, ...])
```

optimized vector math function, allows multiplyging 3 dimensions vectors exposed as 3 float values

```py
unreal_engine.vector_mul_float(x, y, z[, d1, d2, ...])
```

optimized vector math function, allows multiplying a 3 dimensions vector with one or more floats

The 'World' concept
-------------------

Every uobject is mapped to a world (UWorld in c++). Generally when you play on a Level, that level is your current World, but at the same time there could be multiple worlds (like when streaming new levels, or while testing in the editor there is a world for the editor and one for the simulation)

While it is pretty rare to reference other worlds, you may need to compare the world of two uobject's (for example you may have a reference in your python module to a uobject of a hidden world and you want to check if you need to use it).

The uobject.get_world() functions return a uobject representing the world (the C++ UWorld class)

The uobject api
---------------

Each uobject represent a UObject class of the Engine. This C++ class is basically the root of all the other classes (Actors, components, properties ...). Thanks to Unreal Engine reflection system we do not need to implement a python class for each unreal engine class, but for performance reason we expose the most common methods. The uobject system check for the type of the mapped C++ UObject and will call the method only if it is safe to call it.

Sometime methods are implemented for automatically getting the right object. As an example get_actor_location() when called over a component will automatically retrieve the related actor and will call C++ AActor::GetActorLocation() method over it.

When this automagic approach is too risky, the method will check for the uobject type and will raise an exception in the case of inconsistencies.

```py
x, y, z = uobject.get_actor_location()
```

get the current actor location (automatically retrieve the actor from the component if needed)


```py
uobject.set_actor_location(x, y, z)
```

set the current actor location (automatically retrieve the actor from the component if needed)

```py
pitch, yaw, roll = uobject.get_actor_rotation()
```

```py
uobject.set_actor_rotation(pitch, yaw, roll)
```

```py
x, y, z = uobject.get_actor_forward()
```

```py
x, y, z = uobject.get_actor_right()
```

```py
x, y, z = uobject.get_actor_up()
```

```py
x, y, z = uobject.get_actor_velocity()
```

```py
value = uobject.get_property('name')
```

```py
properties_list = uobject.properties()
```

```py
uobject.call('function')
```

```py
actor = uobject.get_owner()
```

```py
name = uobject.get_name()
```

```py
full_name = uobject.get_full_name()
```

```py
# really slow !
found_uobject = uobject.find_object('name')
```

```py
# really really slow !
found_uobjects = uobject.all_objects()
```

```py
found_actors = uobject.all_actors()
```

```py
uclass = uobject.get_class()
```

```py
uclass = uobject.actor_spawn(uclass[, x, y, z, pitch, yaw, roll])
```

```py
uobject.actor_destroy()
```

```py
yesno = uobject.actor_has_component_of_type(uclass)
```

```py
components = uobject.actor_components()
```

```py
uobject.enable_input()
```

```py
uobject.bind_input_axis('axis')
```

```py
uobject.quit_game()
```

```py
yesno = uobject.is_input_key_down('key')
```

```py
value = uobject.get_input_axis('axis')
```

Automatic module reloading (Editor only)
----------------------------------------

Math functions
--------------

Memory management
-----------------

Physics
-------

Fracturing
----------

Blueprints integration
----------------------

Events
------

Packaging
---------

Known issues
------------