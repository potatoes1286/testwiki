---
layout: default
title: Getting Started with Meatkit
parent: MeatKit
grand_parent: Making Mods
---

# 1. Getting Started with MeatKit
This is the first article in a series which will guide you through the basics of MeatKit.
In this part, you will install Unity, download MeatKit, and confirm everything works by building the sample mod.

## Setup
To use MeatKit, you must have a copy of the game and [Unity Editor 5.6.7f1](https://unity.com/releases/editor/whats-new/5.6.7) installed. Previously H3VR and MeatKit both used 5.6.3p4, but they have since been updadted to be 5.6.7f1 instead. MeatKit will still run on 5.6.3p4.

### Installing Unity
If you do not have Unity 5.6.7f1 installed, you must go and [download it from their archive](https://unity.com/releases/editor/whats-new/5.6.7), select the `Unity Editor Windows (X86-64)` option. Run the installer when it finishes downloading and let everything install with the default options.

You may also install the editor with Unity Hub, if you prefer to do it that way. If you choose to use Unity Hub, know that the first time you try and open a project it may hang on the splash screen. To resolve this you have to open it directly (not through Unity Hub) and register your license from there instead.

{: .note }
> If you already have another version of the Unity Editor installed, you may want to change the install location for 5.6.7f1 to avoid conflicts with other versions.

When the Editor is finished installing, go ahead and open it. It will ask you to log in with your Unity account and then let you register your license. If you do not have a license, just select the 'personal' option and continue. You won't be missing out on any features by doing so. Once you get to the screen for creating a project, you can move on.

### Downloading MeatKit
1. Visit the [MeatKit repository](https://github.com/H3VR-Modding/MeatKit) and download the current version by clicking the green 'Code' button and selecting 'Download ZIP'.
2. Unzip MeatKit and open the folder with your installed version of Unity.
3. With the project open, select `MeatKit > Scripts > Import Game` on the menu bar at the top and point it to your `h3vr/h3vr_Data/Managed` folder.

> [!WARNING]
> If you are getting compile errors for missing the game's assembly and / or cannot see the MeatKit menu bar item, go to `Edit > Project Settings > Player` and in the `Other Settings` tab, clear the `Scripting Define Symbols` field and hit enter, then re-import the game's scripts.

## Import Core Packages
The tools to build different types of mods are created and maintained independently from MeatKit itself, so you will need to download and import one or more of these packages to work on mods:

* Atlas package for mapping: https://github.com/H3VR-Modding/AtlasSampleScenes/releases
* OtherLoader package for creating guns and other items: https://github.com/devyndamonster/MeatKit/releases

Import one or both of the packages depending on what kind of modding you wish to do by using the `Assets > Import Package > Custom Package` menu item. 

## Confirm everything works
To confirm everything is setup correctly, open MeatKit build window (`MeatKit > Build Window`) and click the circle beside the build profile field. At least one of the sample profiles will show up, depending on which packages you imported, select one of them.
You won't need to modify anything to get these to work, however you will want to change the build action to 'copy to profile' and then select one of your r2mm profile folders.

Now click the big build button. When the build is finished, the 'last build time' in the build window will update to reflect the current time, and because of the selected build action, the mod has been inserted into the selected profile and is ready to test.

> [!NOTE]
> The mod will not show up in r2mm's installed list as it doesn't take into account mods installed outside the app, but it is installed and running that profile will cause it to be loaded.

## Updating MeatKit
As of MeatKit v3, an updater window is built into MeatKit. You can check for updates to MeatKit by opening it on the menu bar `MeatKit > Check for updates` and clicking the check for updates button. If an update is available another button will appear to update to the latest version. 

> [!WARNING]
> MeatKit updates itself by removing and re-downloading the entire `Assets/MeatKit` folder in your project. _Do not_ store your own files in there or make any changes to files expecting them to be permanent.

## Summary
Congratulations, you've built your first mod using MeatKit!  
Next: [Changing the build settings](2_build_profile.md)

# 2. MeatKit build profiles
In MeatKit, there are files which store your build profiles. Each profile is a different configuration which represents a single Thunderstore package and the assets which are set to be included in it. 

In this second article of the MeatKit getting started guide, you will configure your build settings to make your mod appear as your own.

## Accessing the build profile
If you are starting by using one of the samples, you may just select it and in either the inspector window or the build window edit its values. Alternatively, you can create new empty ones by right clicking in the project window and selecting `Create > MeatKit > Build Profile`.

## Rebranding your mod
For now, just change the `Package Name`, `Author`, `Version`, and `Description`. These should all be self-explanatory. If you have an icon for your mod, or a website for it, you can set those now too. The README is just a plain text file that shows up on your Thunderstore page, you can read more about that [over on this page](../../thunderstore/uploading.md).

Just changing the values here is all you need to do to rebrand your mod. If you'd like, you can re-build your mod from the build window to verify these changes were indeed made.

> [!NOTE]
> Using the copy to profile action, if you change the name or author of your mod the previous mod will not be removed from the profile. This can be done manually by going into that profile's plugins folder and removing the old mod folder.

> [!NOTE]
> Sometimes a message box will appear beneath settings that you edit. They will contain a symbol to indicate the importance of the message (info, warning, error), as well as some text. If there is an error message anywhere in any of your settings, MeatKit will not let you build your package until you correct it. 

## Build Items
Build items are the base of the MeatKit build pipeline. Each build item represents some kind of asset in your project that MeatKit will build and export.
As part of the samples, there should be at least one build item in there already. Click the little triangle by the Build Items tag to show the list of them.
By double-clicking on one, it will select the object and show it in the inspector. Each type of build item has different options and is meant to represents 
some kind of asset, like a custom scene, weapon, or just an asset bundle to store assets.

## Dependencies
This is just a list of the Thunderstore packages your mod will depend on. Some build items require certain dependencies, like the `AtlasSceneBuildItem` requiring Atlas, so if you have a custom scene build item in your list, Atlas will appear automatically in the dependencies and you will not be able to remove it.

You can also add your own here if you are making a more complex mod and need to set one up manually.

## Summary
After rebranding your package, you can [move on to creating your first mod](3_creating.md).

# 3. Creating A Mod
Congratulations, you now know the basics of MeatKit and can start making your own mod!

Currently, MeatKit supports both Atlas and OtherLoader.
To make custom maps, check out the [Atlas section of the wiki](../../mapping/atlas/intro.md).
To make custom guns, check out the [video playlist guide for OtherLoader](https://www.youtube.com/watch?v=BScDQiGCRAM&list=PL4xZPb3t-cEFkCxo648hdTulxKYy08thY)

