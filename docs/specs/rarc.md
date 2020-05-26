# Archives (.rarc)
RARCs or Resource ARChives contain files grouped together in a filesystem for game use.

## The Main File
The main file consists of file information, nodes and entries, a string table, and a dump of file data.

| **Type** | **Description** |
|----------|-----------------|
|FileHeader|File Header|
|InfoSection|Contains info about the format|
|Node[NumNodes]|Holds directory information|
|Entry[NumEntries]|Holds information about the filesystem of directories and files|
|StringTable|Contains all the names|
|byte[NumFiles][]|File data, each file is padded to 0x20 bytes|

### File Header
Header for the file.

| **Type** | **Description** |
|----------|-----------------|
|char[4]|Magic (RARC)|
|u32|File Size|
|u32|Relative Offset|
|u32|Data Offset (Relative to Relative Offset)|
|u32|Size of All File Data|
|u32|Size of All File Data|
|u32[2]|Padding|

### Info Section
Contains information about the archive. All offsets are relative to the Relative Offset.

| **Type** | **Description** |
|----------|-----------------|
|u32|Number of Nodes|
|u32|Relative Offset|
|u32|Number of Entries|
|u32|File Entries Offset|
|u32|String Table Size (With Padding)|
|u32|String Table Offset|
|u16|Number of Entries Again? This only seems to appear sometimes|
|bool|True if Number of Entries Appear Again. I think this is set when there is at least one file in the root folder|
|u8[5]|Padding|

### Node
Represents a directory. There will always be a ROOT one, no matter what. They are sorted alphabetically with ROOT always being the first, and each parent folder's entries being added after it.

| **Type** | **Description** |
|----------|-----------------|
|char[4]|Type of node|
|u32|Directory Name Offset (Relative to start of String Table)|
|u16|Name Hashcode|
|u16|Number of Entries|
|u32|First Entry Index|

### Entry
An item contained within a node. Each folder, including the root, has two directory entries labelled . and .. at the end which link back to current and parent node respectively. It first iterates through the files in the folder followed by the folders, and the files and folders each folder contains. It is known what is contained inside each folder, since each node has a file entry offset and number of file entries.

| **Type** | **Description** |
|----------|-----------------|
|u16|ID. Either the File Entry ID, or 0 if a directory|
|u16|Name Hashcode|
|u8|Flags. Bit 0 means it is a file, bit 1 is a directory. If this is a file, it seems to always have a value of 0x11|
|u8|Padding|
|u16|File Name Offset (Relative to start of String Table)|
|u32|Data Offset (Offset to file data, node index if this is a directory)|
|u32|Data Size (Always 0x10 if a directory)|
|u32|Padding|

### String Table
First two entries are always . and .. with the table consisted of null-terminated strings padded to 0x20 bytes.

### Name Hashcodes
The hash starts at 0. The multiplier is the length of the string plus one, with a max multiplier of 3. Then, iterate through each char in the string multiplying the current hash by the multiplier and adding the char value.