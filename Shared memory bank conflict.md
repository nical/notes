Some architectures have memory groken into chunks of memory that can only be accessed (TODO: r/w ?) one word at a time, causing access to N consecutive locations to be linearized into the cost of N sequential instructions 

The memory is distributed among banks in such a way that each 32-bit word in a sequence is sequentially assigned to one of N banks, where N depends on the device (typically 16 or 32 for Nvidia hardware) 

The number of threads accessing the bank is called the degree of conflict. If the degree of conflict is `D`, then access is carried out `D` times slower than if there was no conflict.

# Takeaway 

Avoid concurrent access to locations at the same index where the index is the offset in words (usually 32bits) modulo bank size. this is somewhat analogous to false cache sharing performance issues, but with reads as well as writes. And a weirder memory layout.


Cuda devices version 2.0
Bank number = (Address in bytes / 4)% 32

Cuda devices version 1.x
Bank number = (Address in bytes / 4)% 16

# hardware 

TODO

On some generations of Nvidia hardware banks were the size of half a warp 16x32bits, on others they were warp sized (64x32bits) 

