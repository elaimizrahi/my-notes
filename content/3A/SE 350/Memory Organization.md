
Trade-offs are made between memory capacity, access time, and cost per bit

#### Access Frequency
$T_s = T_1 \times H + (1-H)(T_1 + T_2)$
$T_s = T_1 + (1-H)T_2$

- **Locality of reference:** memory references by the processor tend to cluster
- **Temporal locality:** if the processor accesses a memory location, it is likely to access the same memory location in the near future
- **Spatial locality:** if the processor accesses a memory location, it is likely to access other locations with close addresses

## Cache
#### Considerations
1. Cost vs. Hit ratio (higher hit rate -> slower)
2. Block size (spacial locality)
3. Mapping and replacement (keeping track of blocks in cache)
4. Write policy (when to write to main memory)


