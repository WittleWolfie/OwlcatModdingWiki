## About Modding

Modding Kingmaker is primary done by creating .dll libraries written in the C# programming language. It is recommended that you familiarize yourself with the basics of C# before beginning. Some tutorials are [here](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/) and [here](https://www.tutorialspoint.com/csharp/index.htm).

The general workflow for modding is to inspect the game code using decompilers such as [dotPeak](https://www.jetbrains.com/decompiler/), [ILSpy](https://github.com/icsharpcode/ILSpy) or [dnSpy](https://github.com/dnSpy/dnSpy) to find the code you wish to modify, create a C# library that hooks into the game, and makes the desired changes. Unity Mod Manager is used to load libraries into the game at start up, and [Harmony](https://harmony.pardeike.net/) is used to modify the behaviour of methods.

The game uses a blueprint system to store data about game mechanics. Most mechanics, such as classes, abilities, dialogue, kingdom events, units, items, etc use the blueprint system. Blueprints use a type of entity component system, where the blueprints are containers that hold components which contain data about different features. As blueprints are stored in a proprietary unity file format, the main ways of viewing the data are with the [blueprint dump](https://github.com/spacehamster/KingmakerDataminer/releases/tag/blueprints) or the [data viewer](https://www.nexusmods.com/pathfinderkingmaker/mods/106).

![DnSpy](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/DnSpy.png)



## Project Creation

* Download and install Microsoft Visual Studio Community with C#
* Create a new project and select Class Library (.NET Framework)![CreateNewProject](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/CreateNewProject.png)
  * If Class Library (.NET Framework) isn't available, the .NET desktop development workload most likely has not been installed. Select `Get Tools and Features...`  under the tools menu, and then install the  .NET desktop development workload.![InstallDotNet](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/InstallDotNet.png)
* Select .NET Framework 4.6 ![ConfigureNewProject](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/ConfigureNewProject.png)
* Your project should now look like this. ![DefaultProject](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/DefaultProject.png)
* Rename Class1.cs to Main.cs.
* Right click the project in the solution explorer, select properties, navigate to the build pane. By default, the project is in the debug configuration. Set the debug output path to `bin\Debug\MyFirstMod\`, then change the active configuration to release and then set the release output path to `bin\Release\MyFirstMod\`. ![ProjectSettings](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/ProjectSettings.png)
* Right click References and select add References. Then select browse, navigate to the game folder and add the displayed references. ![ReferenceManager](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/ReferenceManager.png)
* The project should now look like this. Select all the project references, right click properties, and set Copy Local to False. ![ReferenceCopyLocal](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/ReferenceCopyLocal.png)
* We now set the contents of Main.cs with our code to initialize the mod after Unity Mod Manager loads it.
```csharp
using UnityEngine;
using UnityModManagerNet;
using UnityEngine.UI;
using HarmonyLib;

namespace MyFirstMod
{
    static class Main
    {
        public static bool Enabled;

        static bool Load(UnityModManager.ModEntry modEntry)
        {
            modEntry.OnToggle = OnToggle
            return true;
        }

        static bool OnToggle(UnityModManager.ModEntry modEntry, bool value)
        {
            Enabled = value;
            return true;
        }
    }
}
```

* We then create our information file, which lets Unity Mod Manager know how to load and manage our mod. Right click the project, select `Add` and then `New Item` and create a text file named `Info.json`![InfoFile](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/InfoFile.png)

Set the contents to 

```json
{
  "Id": "MyFirstMod",
  "DisplayName": "My First Mod",
  "Author": "username",
  "ManagerVersion": "0.21.3",
  "Requirements": [],
  "AssemblyName": "MyFirstMod.dll",
  "EntryMethod": "MyFirstMod.Main.Load"
}
```

and then right click the file, select properties, and then enable `Copy if newer` ![ResourcesCopy](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/ResourcesCopy.png)

* Build with mod with `ctrl+b` or select `Build MyFirstMod` under the build menu. This will create a .dll file in your project's bin directory. `MyFirstMod\bin\Debug\MyFirstMod.dll` when build in debug configuration and `MyFirstMod\bin\Release\MyFirstMod.dll` when built in release configuration. Copy the files to `C:\Program Files (x86)\Steam\steamapps\common\Pathfinder Kingmaker\Mods\MyFirstMod` to install.

* The mod should now show up in the Unity Mod Manager screen at start up.

  ![IngameScreen](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/IngameScreen.png)

## GUI and Settings

* Create a new class called `Settings.cs` 

  ```csharp
  using UnityModManagerNet;
  
  namespace MyFirstMod
  {
      public class Settings : UnityModManager.ModSettings
      {
          public float MyFloatOption = 2f;
          public bool MyBoolOption = true;
          public string MyTextOption = "Hello";
  
          public override void Save(UnityModManager.ModEntry modEntry)
          {
              Save(this, modEntry);
          }
      }
  }
  ```

* Add a new `OnGui` method to draw your settings

  ```csharp
  using UnityEngine;
  using UnityModManagerNet;
  using UnityEngine.UI;
  using HarmonyLib;
  
  namespace MyFirstMod
  {
      static class Main
      {
          public static Settings Settings;
          public static bool Enabled;
  
          static bool Load(UnityModManager.ModEntry modEntry)
          {
              Settings = Settings.Load<Settings>(modEntry);
              modEntry.OnToggle = OnToggle
              modEntry.OnGUI = OnGUI;
              modEntry.OnSaveGUI = OnSaveGUI;
              return true;
          }
  
          static bool OnToggle(UnityModManager.ModEntry modEntry, bool value)
          {
              Enabled = value;
              return true;
          }
  
          static void OnGUI(UnityModManager.ModEntry modEntry)
          {
              GUILayout.BeginHorizontal();
              GUILayout.Label("MyFloatOption", GUILayout.ExpandWidth(false));
              GUILayout.Space(10);
              Settings.MyFloatOption = GUILayout.HorizontalSlider(Settings.MyFloatOption, 1f, 10f, GUILayout.Width(300f));
              GUILayout.Label($" {Settings.MyFloatOption:p0}", GUILayout.ExpandWidth(false));
              GUILayout.EndHorizontal();
  
              GUILayout.BeginHorizontal();
              GUILayout.Label("MyBoolOption", GUILayout.ExpandWidth(false));
              GUILayout.Space(10);
              Settings.MyBoolOption = GUILayout.Toggle(Settings.MyBoolOption, $" {Settings.MyBoolOption}", GUILayout.ExpandWidth(false));
              GUILayout.EndHorizontal();
  
              GUILayout.BeginHorizontal();
              GUILayout.Label("MyTextOption", GUILayout.ExpandWidth(false));
              GUILayout.Space(10);
              Settings.MyTextOption = GUILayout.TextField(Settings.MyTextOption, GUILayout.Width(300f));
              GUILayout.EndHorizontal();
          }
  
          static void OnSaveGUI(UnityModManager.ModEntry modEntry)
          {
              Settings.Save(modEntry);
          }
      }
  }
  ```

## Patching

Harmony patches allow us to change the behavior of methods. As an example, we will patch the game to change the cost of mercenaries. In order to patch a method, we need to know how it is defined, so we start by opening the game assembly (`C:\Program Files (x86)\Steam\steamapps\common\Pathfinder Kingmaker\Kingmaker_Data\Managed\Assembly-CSharp.dll`) in DnSpy. We then need to find out where the behavior we wish to modify is defined, so a good way of doing that is opening the search menu (`ctrl+shift+k`) and trying keywords that seem relevant. In this case `CustomCompanion Cost` brings up the method we wish to change. ![image-20200213164817929](main.assets\image-20200213164817929.png)

Now we can create a patch to change the behavior. There are three kinds of patches, `Prefix` patches that run before the original method, `Postfix` patches that run after the original method, and `Transpiler` patches which modify the the method's IL (intermediate language, a kind of high level assembly) directly. Postfix patches can choose to prevent the original method, and other patches from running, so they may cause compatibility issues with other mods if they prevent their patches from running.

In this case, we will use a postfix patch to set half the cost of the gold.

The simplest way of using harmony is to call the PatchAll method, which scans the mod for any harmony patches, and applies them.

```csharp
using UnityModManagerNet;
using HarmonyLib;
namespace MyFirstMod
{
    static class Main
    {
        public static bool Enabled;

        static bool Load(UnityModManager.ModEntry modEntry)
        {
            modEntry.OnToggle = OnToggle
            var harmony = new Harmony(modEntry.Info.Id);
            harmony.PatchAll(Assembly.GetExecutingAssembly());
            return true;
        }
        static bool OnToggle(UnityModManager.ModEntry modEntry, bool value)
        {
            Enabled = value;
            return true;
        }
    }
}
```

For the patch itself, patches can be located in any file in the mod.

```csharp
using Kingmaker;
using HarmonyLib;

namespace MyFirstMod
{
    //HarmonyPatch attribute allows PatchAll to find the patch
    [HarmonyPatch(typeof(Player), "GetCustomCompanionCost")]
    static class Player_GetCustomCompanionCost_Patch
    {
        //Postfix must be spelt correctly to be applied
        static void Postfix(ref int __result)
        {
            if(!Main.Enabled) return;
            // Harmony parameters are determined by name, __result 
            // is the current cost of the mercenary. Because it is a 
            // ref parameter, we can modify it's value
            __result = __result / 2;
        }
    }
}
```

## Blueprints

### Selectable Groetus

Blueprints are the main way that mechanical data is stored. They are containers that contain data for features such as character classes, races, items, abilities, dialogs, NPCs, AI and many more. As an example of modifying blueprints, we will enable Goblins as a playable race.

The first step is to find out where playable races are stored. One way is to search the json blueprints for references to a race until we find something promising.  ![GroetusBlueprintRoot](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/GroetusBlueprintRoot.png)



As an example of modifying blueprints, we will allow Groetus to be selected as a deity for divine casters other then Harrim. The first step is to find the Groetus blueprint in the json blueprint dump. Any reasonable text editor will do, but we'll use vscode for the example. vscode can search filenames with  `ctrl+p` and file contents with `ctrl+shift+f`. Search Groetus should bring up the file `GroetusFeature.c3e4d5681906d5246ab8b0637b98cbfe.json`.

![GroetusBlueprintSearch](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/GroetusBlueprintSearch.png)

After inspecting the GroetusFeature blueprint, we can see the reason why Groetus is not selectable by other characters is because the blueprint has a `PrerequisiteFeature` requirement that the character have the `HarrimFeatureGroetus` blueprint. If we search the file contents for `9408d7c7953cbd84b80c7c7451252196` (the `HarrimFeatureGroetus` assetGuid) we can see that it isn't referenced by any other blueprints except for the `Groeutus` blueprint

![GroetusBlueprintFeature](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/GroetusBlueprintFeature.png)



We need to modify the GroetusFeature's `Components` list to remove the prerequisite. As GroetusFeature is a kind of BlueprintFeature,  If we take a look at `BlueprintFeature` in DnSpy we can see that it does not contain a field named `Components`. We can click on the parent class `BlueprintFeatureBase`, and after doing that a few times, we finally find out that `Components` is a field of the`BlueprintScriptableObject` class.

![GroetusDnSpy](https://github.com/spacehamster/KingmakerModdingWiki/blob/master/Media/Beginner/GroetusDnSpy.png)

Now that we know what we need to do, we can start modifying the blueprints

```csharp
using UnityModManagerNet;
using Kingmaker.Blueprints;
using Kingmaker.Blueprints.Classes;
using System.Linq;
using Kingmaker.Blueprints.Classes.Prerequisites;
using HarmonyLib;

namespace MyFirstMod
{
    static class Main
    {
        public static bool Enabled;

        static bool Load(UnityModManager.ModEntry modEntry)
        {
            var harmony = new Harmony(modEntry.Info.Id);
            harmony.PatchAll(Assembly.GetExecutingAssembly());
            return true;
        }

        static bool OnToggle(UnityModManager.ModEntry modEntry, bool value)
        {
            Enabled = value;
            return true;
        }

        /// <summary>
        /// We cannot modify blueprints until after the game has loaded them, we patch 
        /// LibraryScriptableObject.LoadDictionary to be able to make our modifications as
        /// soon as the blueprints have loaded.
        /// </summary>
        [HarmonyPatch(typeof(LibraryScriptableObject), "LoadDictionary")]
        static class LibraryScriptableObject_LoadDictionary_Patch
        {
            static bool loaded = false;
            static void Postfix()
            {
                if (loaded) return;
                loaded = true;

                var groetusFeature = ResourcesLibrary.TryGetBlueprint<BlueprintFeature>("c3e4d5681906d5246ab8b0637b98cbfe");
                groetusFeature.ComponentsArray = groetusFeature.ComponentsArray
                    .Where(c => !(c is PrerequisiteFeature))
                    .ToArray();
            }
        }
    }
}
```
