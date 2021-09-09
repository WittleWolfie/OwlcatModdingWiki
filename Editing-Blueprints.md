**IMPORTANT FOREWORD:** Once units (and other things) are loaded into an area, most (all?) of their properties and fields are locked in. If you want to test changes to a Unit's blueprint or otherwise, you will need to go through an area load beforehand.

**Helpful Advice:** It is much easier to access protected members of a WoR or Kingmaker class via publicizing the C# assembly - check out the page [here](https://github.com/TylerGoeringer/OwlcatModdingWiki/wiki/Publicise-Assemblies) for more details. If you do not do this, you will need to access many fields via reflection.

# General

Generally, in order to edit blueprints in the game, the typical process is to use a postfix harmony patch after the point at which the game loads the blueprint library. However, note that in Pathfinder: WoR, the process is slightly difference, due to blueprints being loaded and unloaded dynamically, which can allow the garbage collector to eat our changes.

Pathfinder WoR Example:
```C#
[HarmonyPatch(typeof(ResourcesLibrary), "InitializeLibrary")]
static class ResourcesLibrary_InitializeLibrary_Patch
{
	static bool Initialized;
	static bool Prefix()
	{
		if (Initialized)
		{
			// When wrath first loads into the main menu InitializeLibrary is called by Kingmaker.GameStarter.
			// When loading into maps, Kingmaker.Runner.Start will call InitializeLibrary which will
			// clear the ResourcesLibrary.s_LoadedBlueprints cache which causes loaded blueprints to be garbage collected.
			// Return false here to prevent ResourcesLibrary.InitializeLibrary from being called twice 
			// to prevent blueprints from being garbage collected.
			return false;
		} 
                else
		{
			return true;
		}
	}
	static void Postfix()
	{
                if(Initialized) return;
		Initialized = true;
		try
		{
			Main.logger.Log("Library patching initiated");
			DoSomethingHere.FixTest();
		}
		catch (Exception ex)
		{
			Main.logger.Log("Error while patching library");
			Main.logger.Log(ex.ToString());
		}
	}
}
```

Once you have set up the above, modifications made to blueprints will persist for the session, so if you do not want to replace original blueprints, you should copy before editing them.

Blueprints can be grabbed by using something like
```C#
public static T Get<T>(this LibraryScriptableObject library, String assetId) where T : BlueprintScriptableObject
        {
            //return (T)ResourcesLibrary.BlueprintsByAssetId[assetId]; //Kingmaker
            return (T)ResourcesLibrary.TryGetBlueprint(assetId); // Wrath of the Righteous
        }
```
After grabbing the blueprint asset, you can further edit it/duplicate/etc..

# Editing Units
Units are the NPCs and other actors in the game that have statistics. At their core, they have a brain, an inventory, and a fairly wide variety of other properties needed to function. Alongside that, they have a list of facts, and a componentsList that defines more details.

Generally you will usually:
* Modify core values, such as race, portrait, faction, size, alignment, etc...
* Add or remove facts from the unit
* Edit their inventory
* Add or remove components from their componentList

## Modifying Core Values
Modifying core values of a unit is quite simple - you can usually just directly access the value and replace it.
```C#
//Grab your Unit blueprint
public static BlueprintUnit AasimarRedMask => ResourcesLibrary.TryGetBlueprint<BlueprintUnit>("20fcfa451598dfe48a2e90088effc766");
//Change your Unit's dex stat
AasimarRedMask.Dexterity = 28;
```
For other values, you can use dnspy/ilspy and decompile BlueprintUnit to see what types are accepted by different stats, and how they can be accessed. Note that protected fields and similar will need reflection to access, unless you have gone through the steps of creating and using a publicized assembly.


## Modifying Fact Arrays
Fact arrays can include quite a wide number of things, from buffs to features and feats. Facts are "built-in" to the Unit and, in the example of pushing a spell in as a fact, the Unit will be able to use the spell an unlimited amount of times, similar to how SLAs work in PnP. The general process for adding things to the Fact Array looks something like:

```C#
BlueprintUnitFactReference[] refArr;
refArr = Units.UnitYouAreModifying.AddFacts; // Copy the original AddFacts array of your Unit
refArr = refArr.AddToArray(SomeFactFeatureBuffEtc.ToReference<BlueprintUnitFactReference>()); //Add a buff, feature.. whatever
refArr = refArr.AddToArray(DR20.ToReference<BlueprintUnitFactReference>()); //Add some DR to the Unit
Units.UnitYouAreModifying.m_AddFacts = refArr; //Publicized assembly allows us to directly push the updated array back into the AddFacts protected member.
```


## Inventory Editing
Similar to directly modifying stats, you can access the body of a Unit and change what it's wearing by simply going:

```C#
//Replace whatever is in the primary hand of your unit with a Longsword +4 (note that it takes a reference, so we need to go ToReference)
//Again note that we do not need to use reflection because we have publicized the assembly
Units.UnitYouWantToChange.Body.m_PrimaryHand = Equipment.Yaniel_Longsword4HolyAvenger.ToReference<BlueprintItemEquipmentHandReference>();
```


## ComponentList Editing
Components are things like the class levels a unit has (add feats, spells, etc... that come from it's class), working with it's tactical/army statistics (for the army battles), or a few other things.

As class levels and the class-related information on a unit is something that is commonly edited, that will be the part mainly focused on here.

The general way to access a component is via something like:
```C#
var CRtest = YourBlueprintUnit.GetComponent<Experience>().CR;
```

In this case, we are getting the CR value of the Experience component which is located in the componentsList of YourBlueprintUnit. Other components and their fields can be accessed and changed in the same way. Take a look at the fields in both the dumped blueprint as well as the dnspy/ilspy decompiled code for complete understanding (eg. the "Experience" component here is under Kingmaker.Blueprints.Classes.Experience - if you access that via dnspy/ilspy, you will see all of the fields and such that make up the component)