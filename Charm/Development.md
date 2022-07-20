Charm is a .NET 6.0 Windows WPF project, with a supporting library project named Field.

### Projects utilised:

* [RevorbStd](https://github.com/xyx0826/revorbstd) for converting RIFF to WEM audio files.
* [3DMigoto](https://github.com/bo3b/3Dmigoto) for CSO HLSL decompilation.
* [Serilog](https://github.com/serilog/serilog) for logging.
* [VersionChecker](https://github.com/hughbe/version-checker) to check for new versions.
* [SharpDX](https://github.com/sharpdx/SharpDX) and [HelixToolkit](https://github.com/helix-toolkit/helix-toolkit) for mathematics and displaying geometry.
* [NAudio](https://github.com/naudio/NAudio) for playing audio.
* [DirectXTexNet](https://github.com/deng0/DirectXTexNet) for processing DDS files, a .NET wrapper of [DirectXTex](https://github.com/microsoft/DirectXTex).
* [FBXSDK](https://www.autodesk.com/developer-network/platform-technologies/fbx-sdk-2020-2-1) and [SWIG](https://www.swig.org/)##  to obtain C# bindings for the C++ FBX SDK.
* [librevorb](https://github.com/xyx0826/librevorb) for extra audio processing.

## How to clone and compile

Cloning should be done using the command `git clone https://github.com/MontagueM/Charm.git --recurse-submodules` as it relies on `RevorbStd`, an included submodule.

If the first compilation generates the error `Could not copy the file "...revorbstd\librevorb.dll" because it was not found.`, try compile again (a quirk of pre-compilation build events when using `xcopy`).

Charm should be built as a console application x64 as this allows logging of unhandled crash exceptions.


## How Charm works

Charms works by treating each game file as a Tag with a header and tables/resources that attach to that header. This is not quite game-accurate, but it makes parsing the files very simple (for more info see [here]()). Each new file is labelled as a Tag with a specified header type. This header is counted as a resource. Resources can point to other resources, and can point to lists of resources. Each resource has a type or "class" which dictates what information is stored inside of it. [Here's an example](https://github.com/MontagueM/Charm/blob/d99369a49a9a7cc9ed1b300b60666209ff780c6a/Field/General/Activity.cs#L22) of a header for a string container: 
```
[StructLayout(LayoutKind.Sequential, Size = 0x78)]
public struct D2Class_8E8E8080
{
    public long FileSize;
    public DestinyHash LocationName;  // these all have actual string hashes but have no string container given directly
    public DestinyHash ActivityName;
    public DestinyHash Unk10;
    public DestinyHash Unk14;
    [DestinyField(FieldType.ResourcePointer)]
    public dynamic? Unk18;  // 6A988080 + 20978080
    [DestinyField(FieldType.TagHash64)]
    public Tag Unk20;  // some weird kind of parent thing with names, contains the string container for this tag
    [DestinyOffset(0x40), DestinyField(FieldType.TablePointer)]
    public List<D2Class_26898080> Unk40;
    [DestinyField(FieldType.TablePointer)]
    public List<D2Class_24898080> Unk50;
    public DestinyHash Unk60;
    [DestinyField(FieldType.TagHash)]
    public Tag Unk64;  // an entity thing
    [DestinyField(FieldType.TagHash64)] 
    public Tag UnkActivity68;
}
```

This is how I define the header for an activity tag. The name of the `struct` specifies its type, given by `8E8E8080`. This is the [LE](https://en.wikipedia.org/wiki/Endianness) version of the class name, where the actual value is 0x80808e8e or 2155908750. For each resource class, the `StructLayout` `Size` must be given. These sizes can be found online [here](https://destiny-pkg-classes.netlify.app/) (supplied by me, thanks Josh for hosting!). The name of the structure must also follow the layout of `D2Class_xxxx8080' as this is used by the custom tag parser.

Practically every resource that is a tag header starts with a `long FileSize`. The rest of the structure must then be figured out by reverse engineering the serialised structures in hexadecimal form. This is the hard part, but trust if you spend long enough trying it becomes like the matrix.

Charm uses a custom `struct` parser that reads the custom attributes and the types of each field to determine how to parse it. If structures are written correctly, a tag file can be fully parsed by only reading the header of that tag. This method heavily reduces the amount of code to implement each tag, and helps in clarity and understanding of what the code is actually doing and how the structures work.

## Writing or editing structures

* `[StructLayout(LayoutKind.Sequential, Size = 0x??)]` is required.
* Structure name must conform to `D2Class_xxxx8080'.
* Namespace of the structure must be `Field`.
* To parse a list of resources, use `List<D2Class_xxxx8080>` and use the `DestinyField(FieldType.TablePointer)` custom attribute.
* To recursively parse a tag, use `Tag<D2Class_xxxx8080>` or a class that inherits from `Tag` with an `override void ParseStructs`.
  * If the hash for the tag is 32 bit, use the `DestinyField(FieldType.TagHash)` custom attribute.
  * If [64 bit](), use the `DestinyField(FieldType.TagHash64)` custom attribute.
* To load a tag but not parse it recursively: set the type as `Tag` with the custom attribute, or set type as `TagHash`, or use the `DestinyField(FieldType.TagHash64, true)]` custom attribute where `true` sets the `disableLoad` flag.
* To skip a region of the structure (eg if unknowns, empty, or for efficiency) use the 'DestinyOffset(0x??)' custom attribute where `??` is the offset to skip to within the structure.
* To identify a resource pointer (a relative address that jumps in front of a class hash), set the type as `dynamic?` and use the `[DestinyField(FieldType.ResourcePointer)]` custom attribute.
* API hashes (which are Fowler-Noll-Vo/FNV hashes) are labelled as `DestinyHash`. The default value for these is `0x811C9DC5`.

There are other types and options available, but they are very rare and generally not useful unless in specific circumstances.

## Key notes for working with the source code

There are several key classes that are important to use in Charm.

* **PackageHandler**: for accessing tags with caching. Create a `Tag<D2Class_xxxx8080>` from a `TagHash` using `PackageHandler.GetTag<D2Class_xxxx8080>(tagHash)`. Do not use `new Tag<D2Class_xxxx8080>` as this does not cache the tag.
* **TagHash64Handler**: interface with the packages for caching and accessing 64 bit tag hashes.
* **InfoConfigHandler**: manages the information about models and maps for importing into third-party programs like UE5.
* **InvestmentHandler**: caches and handles the investment (API) information within the game.

