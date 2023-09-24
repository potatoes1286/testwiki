---
layout: default
title: MeatKit Technical Info
parent: MeatKit
grand_parent: Making Mods
---

# 1. Introduction
If you want to learn more about how MeatKit works under the hood,
you've come to the right place. ALL of these articles will be about super
technical information relating to Unity, how assets are stored in bundles,
and the import / export processes for MeatKit. You do not need to learn any
of this to use MeatKit, however should you be interested, having a deeper
understanding can significantly improve your workflow.

The articles will be separated into sections, organized into a typical MeatKit
workflow order. Import, develop, export, etc.

First article: [Understanding the formats](./2_formats.md)

# 2. Understanding the formats
The following articles require some understanding of two main things; .NET Assemblies, and Unity
Asset Bundles. This article will explain what you need to know about them so you can later
understand what we're working with.

## .NET Assemblies
.NET Assemblies (commonly .dll files) is the file which contains the code for the game, or even
your mods. When you write code in C# and click 'build', your code gets compiled into a dll file
which is a more computer efficient way of storing your code so that it can be run later.
Importantly though, during this process, your code translated from the C# that you wrote it in
to another language called IL, and the compiler has very likely applied some optimizations to
your code. This IL requires much less processing to run, and combined with the optimizations
that makes it faster, but there is no way to get back the original C# code that you originally
wrote from just this DLL alone.

Using a _de_-compiler such as dnSpy you can make an attempt are reversing this process but _it
is not perfect_. For simpler bits of code it may produce the same output as the original input
but in larger projects that will certainly not be the case, and often some of the optimizations
that the compiler applies _cannot_ be automatically reversed by the tool _at all_. In the case
of this game, the decompiled code exported to a C# project will not compile. This is fine if 
all you want to do is read the game's code to understand what it is doing though, which is a
very useful tool for modding.

## Unity Asset bundles

When you build an asset bundle with Unity (or when a tool does it for you), Unity packages up
all of your objects, all objects that those objects reference, and every single asset that any
of those objects use. Textures, materials, shaders, etc etc. One thing it does not include
however is any scripts you may have written or applied to objects in the bundle, those are
stored in the compiled assembly of your game. 

For each script that gets used on the objects that Unity includes in any bundle, Unity includes
a special type of asset used internally and then the objects all point at that as the reference
to which script they need. This special asset contains just two bits of data; the name of the
assembly that the script is contained in, and then the name of the script itself. 

So as an example, if you place a .cs script in your project named MyCustomScript and export a
bundle containing an object which uses that script, that bundle will contain one of these special
assets that contains the name of the assembly of the script (`Assembly-CSharp.dll`) and the name
of the script (`MyCustomScript`). Those two pieces of data together uniquely identify any script
that you may want to find later.

## What you need to remember
- Compiling code is not perfectly reversible, decompiled code will likely have errors and will not be re-compilable.
- Unity Asset bundles store references to scripts on objects by the name of the assembly that contains them, alongside the name of the script itself.

With that out of the way, you can continue to the first article about MeatKit: [Importing the game](./3_importing.md).


# 3. Importing the game

When you first open a MeatKit project, the first thing you must do is import the game.
It's a required step and without it you won't be able to do anything. This article will
explain what this process does, why it is required, and the history of alternative methods.

## The history

One of the main goals of MeatKit is to enable an editing experience as close to the original
Unity project as possible. To achieve this, we need to get all of the game's code into our
project.

In the past, what modders did was to decompile the game's code and place the generated C# code
into their Unity projects. Except as explained earlier, the decompilation process is far from
perfect and to make this work, the tool that was used to accomplish it removed everything except
the bare minimum. After it finished, there was no actual code left in the scripts, only the
serialized fields! While it certainly enabled modding, this is not suitable to attain our goal.

## So what do we do

If decompiling the scripts is off the table, then what do we do? Well, luckily, Unity lets
us import arbitrary .dll files into our project and this should give us access to everything
inside them! This is promising but there are a couple issues, the first being that just placing
the assembly into our project doesn't work.

Unity uses the same assembly name for every project, `Assembly-CSharp`. The game's assembly
is named that, but more importantly, the compiled assembly for our own editor project is named
the exact same thing! So just placing the game assembly into our project will make Unity unhappy
because it very much does not like having two assemblies named the same thing loaded at the same
time. To avoid this, the first thing the import process does is make a copy of the game's assembly
and _renames_ it to `H3VRCode-CSharp` using an assembly editing library called Cecil. Now with
a renamed assembly name, we are one step closer to being able to have Unity load this assembly
in our project. It is important to remember that we did this renaming though, because it will
affect us later.

If you had a completely empty Unity project, you would actually have no issues loading the assembly
with just this renaming step. Unfortunately we aren't working in an empty project, we also have
extra things in there like Alloy and possibly Bakery. Both of these contain their own scripts for
tools and such, and some of them even make it into the game's code! This presents a problem because
we already imported Alloy separately and it comes with those scripts again so now we have duplicates.
As you can probably guess, Unity is not happy with this and we need to remove one of them. Removing
the scripts that come with Alloy is not possible since some of the information that the Alloy assets
require are editor-only and get completely removed in the final build of the game. Attempting to use
the scripts inside the imported game DLL will only cause issues with the Alloy editor tools.

So we need to remove the scripts from the game assembly then. This can also be easily accomplished
using the Cecil library, in the code for the importer there is a big list of what needs to be removed
from the imported game assembly.


## Post processing

Since we already have the assembly open for modification, we might as well do some post processing
on it too. When importing the game with MeatKit, anything inside the game with was marked as private
or protected is changed to public, allowing access to those things from anywhere. This is best used
as a last resort option, when something you need is not normally exposed publicly through the code,
but still a very useful option to have.

Additionally, MeatKit provides a way of defining extra modifications just for your project so you can
for example, add new enum values for custom ammo or magazine types. These are applied on each import.

## Why? Legalities.
Wait, but why can't we just ship the project with this imported assembly right from the start?

While there's nothing _technically_ stopping us, legally we would be in the wrong to do so.
Doing this would require us to redistribute the game's code which we of course do not have
permission to do. So despite it being unlikely to happen, to avoid any legal troubles we
require you to do the import process yourself.

There are a couple hidden upsides to this however. The first being that forcing this upon
everyone means that no matter when you download the MeatKit project, you can be sure that
after your first import you are working with the latest version of the game's code. And
secondly, it means that you must actually have a copy of the game. 

# Importing other assemblies
You may have noticed that alongside the import game option, there is another option to import a
single assembly. That option is used for importing assemblies which do not belong to the game and
do not require some of the processing that the game assembly does like removing some things,
publicizing, or other post processing. The key here is that it's important to remember we renamed
the game's assembly, and because we've done that, we also need to update any other assembly that may
want to use code from the game to point to the new name instead so it can find that code without issue.

If you have an assembly that is neither the game's, nor references the game's, it requires no importing
and can be placed in your project as-is.

Next article: [Build pipeline](./4_build_pipeline.md).


# 4. Build pipeline
The MeatKit build pipeline is a general name all of the tools that are used to setup and perform
a build of your mod. The start of which involves the build profile and build items.

## Build profile
A build profile represents a single Thunderstore package and all of the assets inside it to be
exported. The build profile has fields for the Thunderstore metadata, which will not be covered
here as [Thunderstore has its own documentation for that](https://h3vr.thunderstore.io/package/create/docs/).
Alongside the Thunderstore metadata, there are some script related options, and some export options.

The script options are used for one thing; assembly stripping. This will be covered in more detail
in a later article, but the build profile is where you can configure this feature.

The export options is where you configure what build items will be included in this mod, as well
as how MeatKit will build your mod. 

## Build Items
A build item is a representation of a type of mod within your Thunderstore package. Build items
can be a map, a collection of guns, or just an arbitrary collection of assets you wish to use in
your mod via some custom code. Each type of build item will have different fields on it, for
example Atlas Scene Build Items will have a field for the scene file to export, the name of the
scene, the description, author, etc. and an OtherLoader Build Root will have fields for inserting
the individual items for each of the custom objects you want to include.

## Building
When you click build in the build window here's what happens:
1. The build profile, and all build items in it have their validation functions run. If there were
any errors, MeatKit will not allow the build to continue. If there were any warnings, you will get
a message box letting you know there were warnings, with an option to continue if desired.
2. Any files from a previous build are removed.
3. A copy of the compiled scripts assembly is made.
4. The virtual reality supported project setting is forced on to ensure that shaders are compiled correctly.
5. All build items are asked for a list of asset bundles they want to create and then they are all
created at the same time (more on this later).
6. Any unnecessary files from the asset bundle build are deleted.
7. The virtual reality supported setting is reset to what it was before.
8. The script assembly is exported (more on this later).
9. The Thunderstore manifest file is generated from your build profile.
10. The icon for your package is exported. If the icon is not 256x256 it is resized before being written.
11. The readme file is copied to the export folder.
12. If the build action in the selected profile is 'Just build files', the process is done. If the
action is 'Copy to folder', then the built files are copied into a selected thunderstore profile folder.
And if the action is to create a Thunderstore package, then all the files are zipped up, ready to be
uploaded.

Next article: [Exporting](./5_exporting.md).

# 5. Exporting
During the build process, there's two types of special exports. Asset bundles, and script assembly.
In both of these cases, we are required to do some processing on them before they will actually work
in the game. This article will explain how we reverse some of the side effects of the method we use
to import the game scripts.

## Scripts exporting
The editor will compile any .cs files you have in your Unity project into a dll file already, which is
great because it does half the work for us. This is however not ideal for a few reasons:
* The compiled assembly is called 'Assembly-CSharp.dll', which conflicts with the name of the game's assembly.
* The compiled assembly still references 'H3VRCode-CSharp.dll' for all game code rather than 'Assembly-CSharp.dll'
* Unity compiles _all_ scripts in your project into _one_ dll.
  * The compiled assembly will contain duplicate scripts from Alloy and Bakery, which are included in the game itself.
  * If you use your MeatKit project for multiple mods, it may contain bespoke scripts for one of your other mods that you don't intend to include in this one
* Nothing in this compiled assembly will actually cause any of your exported assets to be loaded

So here's what happens to resolve all of those issues:

First we need to rename our compiled assembly to something else to avoid conflicting with the game's,
the name that is used is the thunderstore package name. We also need to rename the references, this
is the reverse of what we did to import the game's assembly so all of that applies, just backwards.

Next we need to remove any unwanted scripts from the assembly. This is where the strip namespaces
option from the build profile comes in. When this is enabled (which it is by default) any scripts
_not_ in an explicitly allowed namespace will be removed from the assembly, ensuring that only
scripts that you intend to be included get included. Any scripts not in a namespace will be included
by default, as well as in a namespace that matches `Author.ModName`. Everything else must be added
to a list of allowed namespaces. In addition to namespace stripping, we also remove the same types
that we removed from the imported game scripts, to ensure that only one copy of them will ever exist
in both the game, and the editor.

Finally, during this export process, we also generate a BepInEx plugin class inside the exported
assembly that will serve to actually load the exported assets. This generated plugin class comes from
a template that is included with MeatKit, but you can replace it with your own adding custom behavior
by making a copy and moving it into the main namespace of your mod. Each build item is given the
chance to generate a small bit of code in the plugin that will load the assets it created, usually
it will generate a single line of code that calls into the main mod (Atlas, OtherLoader) and give it
the path to the asset bundle. The plugin class is also renamed to match your mod's name, and any
required dependencies for it are added here.

## Asset Bundle Exporting
Remember back to [Article 2: Formats](./2_formats.md) how inside asset bundles, Unity stores a small
internal asset that serves as a reference for scripts used on other objects inside the bundle, and in
[Article 3: Importing](./3_importing.md) where we renamed the imported game assembly to `H3VRCode-CSharp`,
and just now how we renamed our exported assembly to something else too. If we were to just export
a bundle without any modifications none of our scripts would work!

The solution to this is to rename the script assembly names in the asset bundles too. In a previous
version of MeatKit, this was done by waiting until Unity built the asset bundles before re-opening
them ourselves to search over the file for these references and apply our modifications. This however
is very slow, as soon as Unity finished building and compressing the bundles, we were opening them
right back up for modification, requiring just as long, if not longer, to apply our modifications
and then save and recompress the bundle. In the newer versions we drop this method entirely, instead
opting to do the replacement just before Unity writes the data into the bundle in the first place.

Part of the MeatKit v2 update is hooking into the editor itself and strategically messing with
function pointers and memory in order to run our code right before Unity does something important.
In this case, right before Unity writes a MonoScript asset into an asset bundle, we swap out the
assembly name with the intended replacement. Unity writes it to the file, and then we swap it back
right after ensuring that we can continue in the editor as normal. Using this new technique we end
up saving a significant amount of time from our builds, sometimes up to 50%.

Also during this process, we keep track of all the scripts that make their way into a bundle. Because
Unity only writes these MonoScript assets for scripts which get used, we can easily put them all
into a list so later when we export the scripts, we can warn the user if a script they used on an
exported object ended up getting removed from the assembly because it wasn't in the allowed namespaces
list.
