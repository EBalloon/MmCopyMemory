EAC scans memory with MmCopyMemory
- Sections
- Memory outside a legitimate module
- etc

The easiest way around these checks is to change the bit to 0x2

let's check


![5KPhKr8](https://user-images.githubusercontent.com/29626806/168706720-9d3ee31d-3cf4-4eab-8522-55abb89c4443.png)

# Example

EAC read like this

    auto Buffer = ExAllocatePoolWithTag(PagedPool, PAGE_SIZE, ExTag);
    if (!Buffer)
       return failed;
      
    MM_COPY_ADDRESS Source = { 0 };
    Source.VirtualAddress = Address;
    MmCopyMemory(Buffer, Source, Size, MM_COPY_MEMORY_VIRTUAL, &Bytes);

  	// bypass scans from MmCopyMemory (this only ignores physical scan)
	auto MmCopyMemoryPtr = (uintptr_t)MmCopyMemory;
	for (int index = 0; index < PAGE_SIZE; index++)
	{
		auto Address = MmCopyMemoryPtr + index;
		if (*reinterpret_cast<uint8_t*>(Address) == 0xFC
			&& *reinterpret_cast<uint8_t*>(Address + 1) == 0xFF
			&& *reinterpret_cast<uint8_t*>(Address + 2) == 0xFF
			&& *reinterpret_cast<uint8_t*>(Address + 3) == 0xFF)
		{
			DbgPrintEx(0, 0, "Address: 0x%p\n", (void*)Address);
			// Disable Interruptions
			*reinterpret_cast<uint8_t*>(Address) = 0x02; // set bit to (2 = MM_COPY_MEMORY_VIRTUAL)
			// Enable Interruptions
			break;
		}
	}


# After 30 min, PATCHGUARD!
