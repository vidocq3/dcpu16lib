alloc.dasm16 description
V 0.1 , by Vidocq

-----------------------------------------------------------
* Synopsis
-----------------------------------------------------------
- provides a dynamic memory allocation implementation for the dcpu-16
- provides C standard library malloc() , calloc() and free() functions (http://en.wikipedia.org/wiki/C_dynamic_memory_allocation)
- conforms to the ABI as described at  https://github.com/0x10cStandardsCommittee/0x10c-Standards/blob/master/ABI/Draft_ABI_1.txt

-----------------------------------------------------------
* functions
-----------------------------------------------------------

unsigned fc_msys_init( void *heap, unsigned len )
A = heap word start address
B = len in bytes ( BYTES )
must be called once to initialise heap.
returns actual heapsize in bytes in A

- void *fc_msys_malloc ( unsigned size )
A = size needed in bytes ( BYTES )
allocaties the requested number of bytes on the heap
returns word address of allocated memory in A or 0x0000 if allocation failed

- void *fc_msys_calloc ( unsigned size )
same as malloc() but also resets all words in allocated memory to 0x0000

- void fc_msys_free( void *ptr )
A = word address to memory to release
release a previously allocated memory block


-----------------------------------------------------------
* description of the memory allocation algorithm
-----------------------------------------------------------

the heap memory is mapped to 1 or more adjacant memory blocks , every memory block contains a header followed by the actual allocated memory. The header describes the state & length of the memory block.
In the initial state there is one memory block covering the whole heap and in the UNUSED state.

Memory allocation is achieved by finding one or more adjacant UNUSED memory blocks that together are large enough for the request, the total memory the UNUSED blocks occupied are merged and split up in one new USED block with the requested size and in a new UNUSED block to span the remaining memory. The caller the receives the address of the allocated memory in the newly create USED block.

Freeing memory is done by setting the memory block in the UNUSED state.
Iterating the memory blocks is possible since they are allways mapped adjacant , and the address of the next block can thus be deduced from the length value ( much like a single linked list ).

-----------------------------------------------------------
* implementation details
-----------------------------------------------------------

- the header is one word long 
     > UNIT STRUCT { unsigned m_length }
- the m_length holds the word length of the allocated memory in the lower 15 bits.
    ( not counting the length of the header )
- the 16th bit of m_length is used as in use flag.
     > inUse = ((unit.m_length & 0x8000) == 0x8000)
- get address 'm' of next block when current block with address 'n' is in USE
     > m = n + (n->length & 0x7FFF ) + size_of(UNIT)
- get address 'm' of next block when current block with address 'n' is NOT in USE
     > m = n + n->length + size_of(UNIT)
- finding an unused memory block is sped up by maintaining the address of a unused block .
- Searching for unused memory is done from lowest address ( start address of the heap ) to highest , doing so any series of adjacant unused memory blocks are merged .