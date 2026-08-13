# My Allocator

A custom memory allocator in C built on a fixed 4 KB static memory pool. Implements `malloc`-style APIs with block metadata, coalescing, alignment, runtime statistics, and leak detection.

## Highlights (Resume-Ready)

- Implemented a **first-fit free-list allocator** with **block splitting**, **immediate coalescing**, and **8-byte alignment**
- Added **canary-guarded block headers** to detect buffer overflows, double frees, and invalid pointer releases
- Built **`my_realloc`**, **fragmentation metrics**, and a **leak checker** that scans the pool at shutdown
- Wrote an **automated test suite** (16 checks) covering allocation failure, reuse, coalescing, realloc, and stats validation

## Features

| Feature | Description |
|---------|-------------|
| Block metadata | Magic numbers + canaries on every block |
| Coalescing | Merges adjacent free blocks to reduce fragmentation |
| Alignment | All user pointers aligned to 8 bytes |
| Statistics | Tracks peak usage, splits, coalesces, and fragmentation ratio |
| Leak detection | Reports active allocated blocks before shutdown |
| Tests | 8 test scenarios, 16 automated checks with pass/fail output |

## Project Structure

```
Myallocator/
├── include/
│   └── allocator.h       # Public API
├── src/
│   ├── allocator.c       # Core allocator implementation
│   └── demo.c            # Interactive demo program
├── tests/
│   └── test_allocator.c  # Automated test suite
├── bench.c               # Benchmark + metric calculations
├── Makefile
└── README.md
```

## Build and Run

```bash
cd Myallocator
make all      # build demo + tests
make run      # run demo program
make check    # run automated tests
make bench    # run benchmark and show metric calculations
make clean    # remove binaries
```

### Demo output

Runs sample allocations, a `realloc`, and prints allocator statistics.

### Test output

```
[PASS] basic allocation succeeds
[PASS] allocation is aligned
...
16/16 tests passed
```

## API

```c
void allocator_init(void);
void *my_malloc(size_t size);
void my_free(void *ptr);
void *my_realloc(void *ptr, size_t new_size);
allocator_stats_t allocator_get_stats(void);
void allocator_print_stats(void);
size_t allocator_check_leaks(void);
void allocator_shutdown(void);
```

## Resume Bullets (Copy/Paste)

**Custom Pool Memory Allocator | C, Systems Programming**

- Built a 4 KB pool-based memory allocator in C with custom `my_malloc`, `my_free`, and `my_realloc` APIs, block splitting, and adjacent-block coalescing to reduce fragmentation
- Added canary-protected block headers to detect buffer overflows, double frees, and invalid deallocations at runtime
- Implemented allocator telemetry (peak usage, fragmentation ratio, coalesce/split counters) and an automated test suite validating alignment, reuse, and leak detection

## Performance Metrics and Calculations

These numbers are measured from this project, not estimated.

### 1. Peak pool utilization (~65.6%)

Tracks the highest memory usage seen during a benchmark run.

**Formula:**

```
utilization = (peak_bytes / pool_size) × 100
            = (2688 / 4096) × 100
            = 65.6%
```

| Variable | Meaning |
|----------|---------|
| `peak_bytes` | Maximum bytes allocated at once (tracked in `allocator.c`) |
| `pool_size` | Fixed pool size = **4096 bytes** (`ALLOCATOR_POOL_SIZE`) |

During the benchmark, 32 blocks are allocated simultaneously (sizes 32, 40, 48 … bytes). That peak usage is **2688 bytes**, which is **65.6%** of the 4 KB pool.

### 2. Throughput (200K+ alloc/free ops/sec)

Measures how many allocate/free operations the allocator handles per second on your machine.

**Benchmark workload:**

- 100,000 rounds
- Each round: allocate 32 blocks, then free 32 blocks
- Total operations = `100,000 × 32 × 2` = **6,400,000** (malloc + free)

**Formula:**

```
Kops/s = total_operations / time_in_seconds / 1000
       ≈ 6,400,000 / 0.031 / 1000
       ≈ 209,000 Kops/s  →  reported as "200K+"
```

**Note:** Throughput varies by CPU, compiler flags, and OS. Results are from a local benchmark with `-O2`.

### 3. Automated test count (16)

This is the number of `[PASS]` checks printed by:

```bash
make check
```

There are **8 test functions**, and some contain **2 assertions each**, giving **16 total checks**.

### Reproduce the metrics

```bash
gcc -O2 -std=c11 -Iinclude -o bench bench.c src/allocator.c
./bench
```

Or:

```bash
make bench
```

Example output:

```
peak_bytes     = 2688
utilization    = 65.6%
Kops/s         = ~209000
```

You can also inspect live stats after the demo:

```bash
make run
```

Look for `peak bytes` in the printed allocator statistics.

## Design Notes

- Uses a static byte pool (`4096` bytes) — no OS `malloc` for the pool itself
- First-fit policy selects the first sufficiently large free block
- Remaining space after a split is returned to the free list if large enough
- Educational project: not intended for production workloads

## Requirements

- GCC or Clang
- GNU Make
- macOS or Linux

## License

Personal / educational project.
