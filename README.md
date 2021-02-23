# Static Destiny Datamining Documentation
A documentation repo for understanding the basics of static Destiny 2 datamining.

## What this is for:
* Introducing static techniques
* Providing a basic understanding of reading binary in hex form
* An intro to how the Tiger engine for Destiny uses binary to store data (headers, terminators, file references)
* Some basic examples of analysing classes and table structures

## What this is not for:
* Dynamic reverse engineering, including debugging (against Bungie ToS)
* A library for existing Destiny datamining classes
* Finding or grouping files

You're free to skip any bits you think are unnecessary. This documentation presumes you are a capable programmer, up to object-oriented programming. The code provided will be in Python for ease of understanding.

## Reading binary in hex format
Before even working with decompiled Destiny binary files, it is important to understand how to read binary files. Without this, understanding and parsing any of the decompiled files from Destiny will be very difficult.

### **Understanding hexadecimal**

Computers store file data as binary, but this is very difficult to read. Reverse engineers usually opt to read binary files in hexadecimal form, which is otherwise known as base 16. This means instead of each column ranging from 0-9 in decimal (such as the decimal number 109), hexadecimal ranges from 0-F. Hence 10=A, 11=B, 15=F, and so 16=10 and 109=6D. This can be confusing and difficult to distinguish between hexadecimal and decimal, so hexadecimal numbers are usually prefixed with 0x - so 6D becomes 0x6D and 10 becomes 0x10.

A single hexadecimal digit represents 4 bits, so two hexadecimal digits together represents a byte. The maximum number is therefore 0xFF, which is 255 - the same maximum binary.

We can then look at what a binary file might look like. The below image will turn out to be very useful throughout this documentation so I will reference it consistently.

*Figure 1:*

!["HxD Texture Header in hexadecimal"](https://i.imgur.com/8G5gGHa.png "HxD Texture Header in hexadecimal")

This is a binary file, represented using hexadecimal. The width used is 16 bytes, helpfully separated into each byte.

### **Endianness**

It is then important to introduce endianness. Endianness is simply the order that the bytes should be read. In most languages, numbers are written right to left, so that leading zeros that don't matter are on the left: for example 000040 = 40. In hexadecimal, this is called "big endian". We can read the first 2 bytes in the image as 0x5000 this way - which represents the number 20,480.

There is also another endian format - "little endian". This means reading the columns in reverse order, so instead of reading "50 00" as 0x5000, we instead read it as 0x0050. All of Destiny 1 from ROI and Destiny 2 uses this format, so it is very important to understand. I even get it wrong sometimes, so making mistakes with this is very common as day-to-day we always read "big endian".

Endianness also changes with the length of the hex string. Below are some examples of little endianness:
* "50" -> 0x50
* "12 A6" -> 0xA612
* "00 51 05" -> 0x055100
* "95 AF FF FE" -> 0xFEFFAF95

Naturally, we also hide the leading zeros of a number so that 00032 becomes 32 in decimal. This also applies to hex, but can make it more complicated when working with endianness:
* "50 00" -> 0x50
* "50 00 00 00" -> 0x50
* "00 00 50 00" -> 0x50000
* "00 05 00 00" -> 0x500

This means it is important to know how long your number is meant to be, as it can change the final result quite easily. This introduces of the first difficulties of reverse engineering, which is identifying how long each meaningful number is meant to be. This will be covered later, but it relates to storage efficiency - why store every number as a 32 bit number when it only holds a few bits?

## How the Tiger engine uses binary to store data

### Packages
The Tiger engine, like many other video game engines, stores its data in a binary format like in figure 1. The game stores all of this data in package files, which are found in the Destiny 2 directory and named like "w64_investment_globals_client_0173_2.pkg". The majority of these packages are encrypted and compressed for security and speed respectively.

These packages can be read in the same way the Destiny 2 game will read them to extract the actual data within, which requires understanding everything in the pkg files. I have a working decompiler [here](https://github.com/MontagueM/DestinyUnpacker) which is a good stepping stone from this document to a real example of reverse engineering.

### Headers

### Classes and 8080, my favourite number

### Referencing

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
