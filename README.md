# Static Destiny Datamining Documentation
A documentation repo for understanding the basics of static Destiny 2 datamining.

You're free to skip any bits you think are unnecessary. This documentation presumes you are a capable programmer, up to object-oriented programming. The code provided will be in Python for ease of understanding.

### What this is for:
:white_check_mark: Introducing static techniques
 
:white_check_mark: Providing a basic understanding of reading binary in hex form

:white_check_mark: An intro to how the Tiger engine for Destiny uses binary to store data (headers, terminators, file references)

:white_check_mark: Some basic examples of analysing classes and table structures

### What this is not for:
:x: Dynamic reverse engineering, including debugging (against Bungie ToS)

:x: A library for existing Destiny datamining classes

:x: Finding or grouping files


If anyone asks you to or provides a tutorial on how to do any of the following with regards to Destiny *in any way*:
* Memory/RAM access
* Debugging (even having one open while Destiny is running)
* Network manipulation (including Wireshark, NetLimiter)
* Virtual Machines (VM)
* VPN

**do not** follow through with it. It will likely get your account banned permanently with zero way to revert. This is dynamic reverse engineering and is against Bungie ToS. Also do not trust datamining closed-source executables as there is no guarantee this is safe.

## Reading binary in hex format
Before even working with decompiled Destiny binary files, it is important to understand how to read binary files. Without this, understanding and parsing any of the decompiled files from Destiny will be very difficult.

### **Understanding hexadecimal**

Computers store file data as binary, but this is very difficult to read. Reverse engineers usually opt to read binary files in hexadecimal form, which is otherwise known as base 16. This means instead of each column ranging from 0-9 in decimal (such as the decimal number 109), hexadecimal ranges from 0-F. Hence 10=A, 11=B, 15=F, and so 16=10 and 109=6D. This can be confusing and difficult to distinguish between hexadecimal and decimal, so hexadecimal numbers are usually prefixed with 0x - so 6D becomes 0x6D and 10 becomes 0x10.

A single hexadecimal digit represents 4 bits, so two hexadecimal digits together represents a byte. The maximum number is therefore 0xFF, which is 255 - the same maximum binary.

We can then look at what a binary file might look like. The below image will turn out to be very useful throughout this documentation so I will reference it consistently.

*Figure 1:*

!["HxD Texture Header in hexadecimal"](https://i.imgur.com/pJgy3rd.png "HxD Texture Header in hexadecimal")

This is a binary file, represented using hexadecimal. The width used is 16 bytes, helpfully separated into each byte.

### **Endianness**

It is then important to introduce endianness. Endianness is simply the order that the bytes should be read. In most languages, numbers are written right to left, so that leading zeros that don't matter are on the left: for example 000040 = 40. In hexadecimal, this is called "big endian". We can read the first 2 bytes in the image as 0xA000 this way - which represents the number 40,960.

There is also another endian format - "little endian". This means reading the columns in reverse order, so instead of reading "A0 00" as 0xA000, we instead read it as 0x00A0. All of Destiny 1 from ROI and Destiny 2 uses this format, so it is very important to understand. I even get it wrong sometimes, so making mistakes with this is very common as day-to-day we always read "big endian".

Endianness also changes with the length of the hex string. Below are some examples of little endianness:
| String      | Hex |
| ----------- | ----------- |
| "50" | 0x50 |
| "12 A6" | 0xA612 |
| "00 51 05" | 0x055100 |
| "95 AF FF FE" | 0xFEFFAF95 |

Naturally, we also hide the leading zeros of a number so that 00032 becomes 32 in decimal. This also applies to hex, but can make it more complicated when working with endianness:
| String      | Hex |
| ----------- | ----------- |
| "50 00" | 0x50 |
| "50 00 00 00" | 0x50 |
| "00 00 50 00" | 0x50000 |
| "00 05 00 00" | 0x500 |


This means it is important to know how long your number is meant to be, as it can change the final result quite easily. This introduces of the first difficulties of reverse engineering, which is identifying how long each meaningful number is meant to be. This will be covered later, but it relates to storage efficiency - why store every number as a 32 bit number when it only holds a few bits?

## How the Tiger engine uses binary to store data

From here on I will be using three files in the Resources/ directory: header.bin, data.bin, and classes.bin. These files are directly from the game and will be useful in explaining by example. To open and view these files I recommend the program [HxD](https://mh-nexus.de/en/hxd/).

### Packages
The Tiger engine, like many other video game engines, stores its data in a binary format like in figure 1. The game stores all of this data in package files, which are found in the Destiny 2 directory and named like "w64_investment_globals_client_0173_2.pkg". The majority of these packages are encrypted and compressed for security and speed respectively.

These packages can be read in the same way the Destiny 2 game will read them to extract the actual data within, which requires understanding everything in the pkg files. I have a working decompiler [here](https://github.com/MontagueM/DestinyUnpacker) which is a good stepping stone from this document to a real example of reverse engineering.

### Headers
We'll use header.bin and data.bin here, so open both of these files in HxD as shown: ![](https://i.imgur.com/DZgDJB9.png). HxD is useful as you can click on the "Offset (h)" text and it will change between hex and decimal, making it easy to do mental maths at times. The first file we'll look at is the header file, which means it provides a group of important information about a group of data or a table. A header can be separated from its data, like in this case, or it can be attached to the beginning of a file. We'll see an example of this in *Classes and 8080, my favourite number*. 

When reverse engineering game files, you usually have very little information about what a file is or does. When looking at this header.bin, it probably looks like a mess of random hex. However, if we know this is a header for texture data, we know a bit more information already. For example, we know at a minimum the header must contain this data:
* Texture Height
* Texture Width
* Reference to the texture data

With a bit more research into textures, it's also possible it stores:
* File type (compression type? PNG? JPG?)
* Mipmaps
* Array size for cubemaps

So we can try looking for this data in here. We can presume that a lot of textures will hold a texture of width or height as 2^n, such as 256, 1024, or 2048 bytes long. We can also presume the value will be unsigned, as height and width cannot be negative. However, we cannot presume the length of these numbers - it's possible it uses uint16 to have a maximum height or width or 65,535 which is a safe assumption, but we can't presume it.

If we look at the offset 0x22 (click/highlight parts of the .bin and look at the bottom left for an offset), we can see that the value on right gives a uint16 as 256 - a power of two, which is a good sign. The number next to it, 0x24 is also uint16 256. This is probably our height and width. Realistically, you would check multiple files of the same type to check if you're idea is correct, but here I can confirm this is correct. So here we can say we know that:

| Offset      | Desc |
| ----------- | ----------- |
| 0x22 | Texture Width/Height |
| 0x24 | Texture Width/Height |

we can't say for sure if one or the other is width or height, so this would have to be checked in a few other ways (e.g. extracting it and testing, or finding a 1920x1080 image which will always be width x height).

### Referencing

### Classes and 8080, my favourite number

### The Unknown


## A *very* basic overview of where things are

### Packages
We touched on packages a bit earlier in how the Tiger engine stores binary data. These packages are named like "w64_investment_globals_client_0173_2.pkg" or "w64_ghost_ship_02e2_0.pkg". 
We can decipher what this title means:

*  "w64" means Windows x64, which is unimportant - but packages from a console may say ps4 or ps3 for example. 
* The English string is an internal dev name, which is the most useful part of the package name
* The hexadecimal string 0x02E2 or 0x0173 represent the package ID and is not very useful here
* The final numeration is a patch ID, which is by itself not very useful. However, it can be used to see when something has been updated which may be useful

The most useful part is the English string. Most of them are quite obvious, but some vague names are described here:

| Term      | Desc |
| ----------- | ----------- |
| investment | table data that is relevant to the API manifest - whether it be strings, inventory items, perks, images |
| client_startup |  mostly covers UI information and images |
| planet_x | nessus |
| luna | moon |
| sandbox | player gear data |
| activities | misc. game data |
| combatants | enemy data |
| prison of elders | the Forsaken opening mission and Warden of Nothing strike |
