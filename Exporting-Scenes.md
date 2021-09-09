## Exporting Process

One of the more difficult things to do when modding KM/Wrath has to do with working with the content of scenes. This page goes over how to export scenes for viewing locally via Unity.

Steps:
* Export the scene via [utinyripper](https://github.com/mafaca/UtinyRipper)
* Grab the [utinyripper Exporter](https://github.com/spacehamster/UtinyRipperExporter)

* (In a command prompt) - run `extract --g "C:\Program Files (x86)\Steam\steamapps\common\Pathfinder Kingmaker\Kingmaker_Data" -o C:\KingmakerExport` (path will need to be changed based on where your Kingmaker or Wrath folder is)

Theres two main reasons to use the exporter which is a basic CLI wrapper around UtinyRipper
1. utinyripper exports invalid shaders, so the exporter creates dummy shades instead
2. all assets exported by utinyripper have randomly generated GUIDs, the exporter gives script stubs and shaders deterministic GUIDs 

* Create a new empty project using unity editor 2018.4.10 for Kingmaker, or 2019.4.0 for Wrath, and copy in all the assets from C:\KingmakerExport except for C:\KingmakerExport\ScriptableObject. Importing assets goes quicker if you import the ScriptableObject in batches for some reason, like first import everything except for the blueprint components and then import all the blueprint components (blueprint componentshave a filename starting with $)

You will probably get a bunch of script errors from the script stubs utinyripper exported. You'll need to manually fix those up. 
(Spacehamster is working on a tool to export script stubs without needing manual fixing up, but it isn't finished yet)

The unity editor project should now be usable to inspect scenes