# Cache-and-Memory-Hierarchy-Simulator

A C++ simulator for a two-level cache memory hierarchy (L1 and L2). The simulator models configurable L1 and L2 caches with parameters such as block size, cache size, and associativity. It supports both read and write operations and uses the Least Recently Used (LRU) replacement policy.

---

## Simulator Overview

### The simulator models:

- Configurable L1 and L2 caches:
  - Size, associativity, block size
  - LRU replacement
  - Write-back/write-allocate policy

- Detailed metrics collection:
  - Reads, writes, misses, writebacks, memory traffic

<p align="center">
<img width="750" height="500" alt="image" src="https://github.com/user-attachments/assets/8359e17b-b758-41e2-9edd-fb5afbf1f8a5" />
</p>

### Main Steps:
- Allocate memory for cache structures (`Cache_flags`)
- Parse runtime parameters (passed via `CacheParams`)
- Process a memory trace:
  - Each line: `r 0xADDRESS` or `w 0xADDRESS`
  - Dispatch to `readCache` or `writeCache`
  - Update LRU, handle misses, writebacks
- Report aggregated results using `print_output`

---

## Data Structures

```cpp
struct output {
  long read, read_miss;
  long write, write_miss, traffic;
  double miss_rate;
} result[3];
// Collects performance metrics.

struct CacheParams {
  std::string trace_file;
  int BLOCKSIZE, L1_SIZE, L1_ASSOC;
  int L2_SIZE, L2_ASSOC;
};

struct Cache_flags {
  bool valid, dirty;
  int lru, tag, address;
};
// Represents each block's status.

struct para {
  int offset, index, tag, SETS;
};
```
---

## Core Logical Flow
- Memory Allocation:
  - Uses allot_memory(...) to:
    - Allocate L1 and optional L2 caches
- Address Decoding
```bash
para getPara(int address, int SIZE, int ASSOC, int BLOCKSIZE){
  // Calculates offset, index, tag based on parameters
}
```
Splits the memory address into tag, index, and offset based on block size, associativity, and cache size.
This helps identify cache sets and tags for block placement and lookup.

---
## Cache Access Logic

- Read Cache Function
```bash
void readCache(..., char op, int address, ...);
```
- Checks for a hit in L1:
  - if hit: Update LRU
  - else: Query L2 and apply LRU management
    
- Write Cache Function
  - Sets dirty bits on write hits
  - Triggers write-backs on eviction

---
## LRU Management
- updateLRU(...): refreshes LRU counters on hits or insertions
- On every access
  - The accessed block becomes the most recently used (LRU = 0).
  - Other blocks in the set have their LRU counters incremented.
- The block with the highest LRU value is replaced on a miss.

---

## Read/Write Access
- For L1 hit: Update counters, LRU
- For L1 miss: Query L2; on L2 hit, populate L1
- If L2 miss (or no L2), fetch from memory and allocate in cache(s)
- On writes, set dirty bit and manage writebacks when evicting

---

## Compilation and Execution
```bash
g++ -std=c++11 sim.cpp -o sim
./sim <BLOCKSIZE> <L1_SIZE> <L1_ASSOC> <L2_SIZE> <L2_ASSOC> <trace_file>
```

---

## Errors Faced
- Memory traffic is only increased on L2 misses and write-backs.
- Edge Case Bugs: Dirty bit resets may occasionally behave incorrectly if LRU values overlap unexpectedly.

---
For further details reach out to me on ksuthar@ncsu.edu
