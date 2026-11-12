Welcome back ! Lets get right back where we last left off  -> the decoding of `_HEAP_ENTRY`/ies themselves.  

```
┌────────────────────────────────────────┐
│            HEAP                        │
│┌──────────────────┐┌──────────────────┐│
││  Segment         ││  Segment         ││
││┌───────┐┌───────┐││┌───────┐┌───────┐││
│││ Entry ││ Entry ││││ Entry ││ Entry │││
││└───────┘└───────┘││└───────┘└───────┘││
│└──────────────────┘└──────────────────┘│
└────────────────────────────────────────┘
```

In the last post we have took a look at iterating the `HEAP_SEGMENT`s  each `HEAP` holds its own set of segments and most importantly the `Encoding` ( `HEAP_ENTRY`)  at  `0x80` ( x64) 


```
  struct _HEAP_SEGMENT Segment;                                       //0x0
        struct
        {
            struct _HEAP_ENTRY Entry;                                       //0x0
            ULONG SegmentSignature;                                         //0x10
            ULONG SegmentFlags;                                             //0x14
            struct _LIST_ENTRY SegmentListEntry;                            //0x18
            struct _HEAP* Heap;                                             //0x28
            VOID* BaseAddress;                                              //0x30
            ULONG NumberOfPages;                                            //0x38
            struct _HEAP_ENTRY* FirstEntry;                                 //0x40
            struct _HEAP_ENTRY* LastValidEntry;                             //0x48
            ULONG NumberOfUnCommittedPages;                                 //0x50
            ULONG NumberOfUnCommittedRanges;                                //0x54
            USHORT SegmentAllocatorBackTraceIndex;                          //0x58
            USHORT Reserved;                                                //0x5a
            struct _LIST_ENTRY UCRSegmentList;                              //0x60
        };
```

The decoding logic is simple you only get the `FirstEntry` `LastValidEntry` - thats it. My first  confusion came from   thinking the  `HEAP_ENTRY` at `0x0`  is at the start of the user allocated space - despite the pointer of `FirstEntry` pointing somewhere else entirely. Well after some digging I found [Windows Heap Internals learning](https://rolemee.com/posts/windows%E5%A0%86%E5%AD%A6%E4%B9%A0/) that  would confirm that the first traversable block within segment is the `FirstEntry` so question remains what is the `HEAP_ENTRY`  at `0x0` ?? 

From what I gathered the `0x0` `HEAP_ENTRY`  is a chunk that begins at the heap base and contains `_HEAP` plus helpers ( whose payload is the heap's own metadata ) - its allocated by the heap manager itself.  Lets explore it just a little bit. 

### To the end of the `_HEAP` and beyond 

When we look at the size of the first entry of the first embedded segment in `_HEAP` we can see that the size ( in my experience and my windows build ) was always `0x740` (after decoding ) 

```
Heap entries for Segment00 in Heap 000001342a4c0000
                 address: psize . size  flags   state (requested size)
        000001342a4c0000: 00000 . 00740 [101] - busy (73f)
        000001342a4c0740: 00740 . 00070 [100]
```

And the actual `[+0x040] FirstEntry` points to the second entry with the size of 0x70 -> which  we can see here: 
```
    [+0x040] FirstEntry       : 0x1342a4c0740 [Type: _HEAP_ENTRY *]
```

So that sent me down a small rabbit hole  if the `sizeof(_HEAP)` is `0x2c0` and this `0x0 HEAP_ENTRY` has size of `0x740` what actually lives in the difference ? 

```
[+0x134] NonDedicatedListLength : 0x0 [Type: unsigned long]
    [+0x138] BlocksIndex      : 0x1342a4c02e8 [Type: void *]
    .
    .
    .
    [+0x160] LockVariable     : 0x1342a4c02c0 [Type: _HEAP_LOCK *]
	.
	.
    .
    [+0x2b0] TuningParameters [Type: _HEAP_TUNING_PARAMETERS]
0:025> dx -r1 ((ntdll!_HEAP_LOCK *)0x1342a4c02c0)
((ntdll!_HEAP_LOCK *)0x1342a4c02c0)                 : 0x1342a4c02c0 [Type: _HEAP_LOCK *]
    [+0x000] Lock             [Type: <anonymous-tag>]

```

When we take a look at the members of  the original `_HEAP`  struct we can see  some of them. For example :

`LockVariable`  `+0x160` points to `0x1342a4c02c0` ( which is right after the `base` +`0x2c0` ) - that has underlying structure of `_HEAP_LOCK` - main functionality is thread synchronization to prevent memory corruption during concurrent heap operations

`BlocksIndex`  which lives at `0x1342a4c02e8` -> Microsoft doesnt specify underlying structure but both [Understanding the Low Fragmentation Heap](https://illmatics.com/Understanding_the_LFH_Slides.pdf#:~:text=%E2%80%A2%20Chunk%20%3D%20contiguous%20piece%20of%20memory,%3D%20_HEAP_BUCKET%20structure%20used%20as%20size%2Foffset%20reference.) and [Windows Heap Internals learning](https://rolemee.com/posts/windows%E5%A0%86%E5%AD%A6%E4%B9%A0/)  point out that the structure is `_HEAP_LIST_LOOKUP` which we can see for ourselves if we cast the raw address to pointer type of `ntdll!_HEAP_LIST_LOOKUP*` 

```
dx (ntdll!_HEAP_LIST_LOOKUP*)0x1342a4c02e8
(ntdll!_HEAP_LIST_LOOKUP*)0x1342a4c02e8                 : 0x1342a4c02e8 [Type: _HEAP_LIST_LOOKUP *]
    [+0x000] ExtendedLookup   : 0x1342a4c7b70 [Type: _HEAP_LIST_LOOKUP *]
    [+0x008] ArraySize        : 0x80 [Type: unsigned long]
    [+0x00c] ExtraItem        : 0x0 [Type: unsigned long]
    [+0x010] ItemCount        : 0x35 [Type: unsigned long]
    [+0x014] OutOfRangeItems  : 0x0 [Type: unsigned long]
    [+0x018] BaseIndex        : 0x0 [Type: unsigned long]
    [+0x020] ListHead         : 0x1342a4c0150 [Type: _LIST_ENTRY *]
    [+0x028] ListsInUseUlong  : 0x1342a4c0320 : 0x469bfffc [Type: unsigned long *]
    [+0x030] ListHints        : 0x1342a4c0330 [Type: _LIST_ENTRY * *]
```

But lets keep going as this was not the point of this article - so lets get back to the decoding of the regular `HEAP_ENTRY`/ies as promised. 

### Decoding 

If the `EncodeFlagMask` at `0x7c` in the `_HEAP` is set then to get actual useful data from `HEAP_ENTRY`  we need to **XOR**  the new (encoded)  entry with the `Encoding` structure at `0x80`

Validating that the decoding  is the kind of tricky part  and I might be wrong in this so bare with me. 
The structure of `HEAP_ENTRY`  is confusing - especially for someone who has not dealt with `c++ `union in structs - here is the definition of the struct itself -> [\_HEAP_ENTRY](https://www.vergiliusproject.com/kernels/x64/windows-10/22h2/_HEAP_ENTRY)

From what I understood - the byte/bit  placement doesn't change - its just a different view / representation of the bytes - the structures overlap each other 

```
        struct _HEAP_UNPACKED_ENTRY UnpackedEntry;                          //0x0
        struct
        {
            VOID* PreviousBlockPrivateData;                                 //0x0
            union
            {
                struct
                {
                    USHORT Size;                                            //0x8
                    UCHAR Flags;                                            //0xa
                    UCHAR SmallTagIndex;                                    //0xb
                };
                struct
                {
                    ULONG SubSegmentCode;                                   //0x8
                    USHORT PreviousSize;                                    //0xc
                    union
                    {
                        UCHAR SegmentOffset;                                //0xe
                        UCHAR LFHFlags;                                     //0xe
                    };
                    UCHAR UnusedBytes;                                      //0xf
                };
                ULONGLONG CompactHeader;                                    //0x8
            };
        };
```

In this first part we basically see that `HEAP_UNPACKED_ENTRY` (which is just a sub struct of the original `HEAP_ENTRY` ) has  couple different interpretations. As I said the size of the struct doesnt change - its `0x10` -> 16 bytes 

One interpretation could be struct with 2 members - 

```
struct _HEAP_UNPACKED_ENTRY {
            VOID* PreviousBlockPrivateData;                             //0x0
            ULONGLONG CompactHeader;                                    //0x8
}
```

Other interpretation ( **and this is the one we will be working with** ) is -

```
struct _HEAP_UNPACKED_ENTRY{
    VOID* PreviousBlockPrivateData;                                       //0x0
    ULONG SubSegmentCode; (Overlaps Size, Flags, and SmallTagIndex)       //0x8
    USHORT PreviousSize;                                                  //0xc
	UCHAR SegmentOffset;                                                  //0xe
	UCHAR UnusedBytes;                                                    //0xf
}
```

Since I am in habit of making my life more difficult even where I don't have to - I'm doing this in GO  ( and GO doesn't use union in structs ) - so I have to create custom accessors for the individual bits 

```
//subsegment code is 4 byts
//we take last 2  to get the value of size

func (h *HeapUnpackedEntry) Size() uint16 {
	return uint16(h.SubSegmentCode & 0xFFFF)
}

//we move over to the right 2 bytes and get the one byte for flag

func (h *HeapUnpackedEntry) Flags() uint8 {
	return uint8((h.SubSegmentCode >> 16) & 0xFF)
}

func (h *HeapUnpackedEntry) SmallTagIndex() uint8 {
	return uint8((h.SubSegmentCode >> 24) & 0xFF)
}
```

So we got our values ... Now what ? 

The question was how to make sure that the decoding of `HEAP_ENTRY`  header was correct. The answer is the `SmallTagIndex` it holds the XOR'ed checksum of the `Size` and `Flags` ( 3 bytes total ) 

In my case -> 

```
expected := heapEntryDecoded.Size() ^ uint16(heapEntryDecoded.Flags())
if expected = uint16(heapEntryDecoded.SmallTagIndex()){
// We are good .. or are we .. ? 
}
```

The whole point of the checking if the decoding was successful is to make sure we got the right `size` of the entire `HEAP_ENTRY` block  since to advance beyond the `FirstEntry` we take the decoded `size` of the entry and add it to the current entry address -> then move the pointer to that address and cast the raw `0x10` bytes memory as the `HEAP_ENTRY`  decode it  and move to the next entry until we either reach `LastValidEntry` or get to `UCR`(Uncommitted range)  -  if its just a gap we can jump over - or if its the end we can terminate 

The checksum should in **theory** catch if we diverge .. so we don't cast random raw memory as `HEAP_ENTRY` and pretend its a valid block.

But in practice -> I got 2-5 wrong checksums ( I didn't abort ) and then got a correct ones to the end of the segment ( with actual sound payloads ) I might look into this some more  and why does it happen.. but that is for another time. 

### The end 

So that's it .. now you know how to iterate over `HEAP_ENTRIES`  and how to decode them.. or maybe to be more precise -> what worked for me ( and I hope might help you too )

### References :
```
https://rolemee.com/posts/windows%E5%A0%86%E5%AD%A6%E4%B9%A0/

https://illmatics.com/Understanding_the_LFH_Slides.pdf#:~:text=%E2%80%A2%20Chunk%20%3D%20contiguous%20piece%20of%20memory,%3D%20_HEAP_BUCKET%20structure%20used%20as%20size%2Foffset%20reference.
```