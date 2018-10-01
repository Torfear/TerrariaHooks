# TerrariaHooks

### License: MIT

----

HookGen "recipe" for Terraria, resulting in a TML mod that other mods can depend on.

Built with [MonoMod](https://github.com/0x0ade/MonoMod).

## Features

### For Players

This mod is a helper for other mods. Just install it if a mod needs it, it won't change Terraria on its own.

### For Modders

- "Hook" any arbitrary method you want easily:
```cs
On.Terraria.Player.CheckMana += (orig, player, amount, pay, blockQuickMana) => {
	// We can either make this act as a method replacement
	// or just call our code around the original code.

	// Let's double the mana cost of everything.
	// We can pass on custom values.
	bool spendable = orig(player, amount * 2, pay, blockQuickMana);

	// ... but give back half of it if it was spent.
	if (spendable && pay) {
		player.statMana += amount / 2;
	}

	// We can return custom values.
	return spendable;
}
```

For an extended example, take a look at [this gist](https://gist.github.com/0x0ade/1d1013d6ae1ff450aa76f252b0f3b62c).

- Use "hooks" to detect when a method runs, change how it runs or even replace it.  
"Hooks" are automatically undone whenever your mod unloads. This is handled by TerrariaHooks for you.

- Manipulate any method at runtime via IL... += (...) => {...} using Cecil and many helpers.  
Special thanks to Chicken-Bones for the great ideas and feedback along the way!

- Use RuntimeDetour to quickly port your existing "method swapping" code.  
Note that this requires you to undo your detours on unload.

If you need more info, read the [MonoMod RuntimeDetour README](https://github.com/0x0ade/MonoMod/blob/master/README-RuntimeDetour.md).

## Instructions

If you want to use TerrariaHooks in your mod:

- Download `TerrariaHooks.dll` from the [latest release](https://github.com/0x0ade/TerrariaHooks/releases).
- Put it into your mod `lib` folder.
- Add it as a reference in Visual Studio.
- In your `build.txt`, add `modReferences = TerrariaHooks`
- Add your hooks in your mod `Load` method.
- TerrariaHooks will automatically undo your hook when your mod unloads.

### Bundling TerrariaHooks

If you need to avoid dependencies and want to include TerrariaHooks with your mod, follow these additional instructions.

**Note:** The copy of TerrariaHooks bundled with your mod will get outdated sooner or later.

#### Preparing Terraria / tML

You'll need to perform a few preparations.

tModLoader only supports compiling mods against a fixed set of libraries. This means that both your `Windows.dll` and `Mono.dll` will use `TerrariaHooks.dll` for Windows.

TerrariaHooks comes with a wrapper for tModLoader's `CompileMod` and `RoslynCompile` methods. When compiling your mod against `lib/SomeLibrary.dll`, it'll also check `lib/SomeLibrary.Windows.dll` and `lib/SomeLibrary.Mono.dll`

- Download the TerrariaHooks mod in the Terraria mod browser.
- Download `tModLoaderServer_TerrariaHooks.exe` from the [latest release](https://github.com/0x0ade/TerrariaHooks/releases).
- Place it into your Terraria directory. Don't replace the original file.

#### Preparing your mod

- Change your `.csproj` to use `tModLoaderServer_TerrariaHooks.exe`
- Download `TerrariaHooks.Mono.dll` from the [latest release](https://github.com/0x0ade/TerrariaHooks/releases).
- Put it into your mod `lib` folder, next to `TerrariaHooks.dll`
- In your `build.txt`, add the following:
    - `dllReferences = TerrariaHooks` (`dll` instead of `mod`)
    - `languageVersion = 6`
- Run `TerrariaHooksContext.Init(this)` at the beginning of your mod constructor.

## Building TerrariaHooks yourself

- `git clone --recursive https://github.com/0x0ade/TerrariaHookGen.git`
    - Alternatively, pull in the MonoMod submodule manually
- Update the files in `_input`
	- The files redistributed in this repo are stripped.
	- The files in that directory will be stripped automatically on each run.
- Build the solution.
- Run `./TerrariaHookGen/bin/Debug/TerrariaHookGen.exe _input .`
- In the Terraria directory, run `./tModLoaderServer.exe -build 'path/to/TerrariaHooks' -eac`

This repository overcomplicates the entire procedure. It boils down to:
```bash
./MonoMod.RuntimeDetour.HookGen.exe --private Terraria.exe TerrariaHooksPre.dll
./ILRepack.exe /out:TerrariaHooks.dll TerrariaHooksPre.dll MonoMod.*.dll MonoMod.exe
```

When running the above two lines in the Terraria directory (with all dependencies present), it generates `TerrariaHooks.dll` for your `Terraria.exe`.

The sole purpose of this repository is to automate the process entirely, and to allow publishing `TerrariaHooks.dll` as a TML mod.
