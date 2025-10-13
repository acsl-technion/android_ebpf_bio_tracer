# Android eBPF Bio Tracer — Patches, Manual, and Reference Sources

This repository contains **patch files**, a guide on how to use them, and reference **Git submodules** for the Android source components modified to implement the *eBPF-based BIO Tracer*.

The submodules are included **for reference only** — they show the modified Android source trees from which these patches were generated. The tracer was built on top of `android-14.0.0_r1`.

**To use the tracer in your own Android source tree, just apply the `.patch` files.**

This tracer was used in the SYSTOR'24 paper [Space-efficient FTL for Mobile Storage via Tiny Neural Nets](https://dl.acm.org/doi/10.1145/3688351.3689157). Citation instructions are provided below.

---

## Installation Instructions

### 1. Prepare a Clean Android Source Tree

Download and sync the Android sources using the `repo` tool, for example:

```bash
repo init -u https://android.googlesource.com/platform/manifest -b android-14.0.0_r1
repo sync -j$(nproc)
```

### 2. Apply the patch files

```bash
# Paths
ANDROID_SRC=/path/to/aosp
PATCH_REPO=/path/to/android_ebpf_bio_tracer

# Apply patches
git -C $ANDROID_SRC/frameworks/native apply $PATCH_REPO/patches/frameworks_native.patch
git -C $ANDROID_SRC/system/bpfprogs apply $PATCH_REPO/patches/system_bpfprogs.patch
```

### 3. Build Android
Build Android with the applied patches. Example:
```bash
cd $ANDROID_SRC
source build/envsetup.sh
lunch aosp_x86_64-eng
m -j$(nproc)
```

> **Note:**  
You don't need the submodules to use these patches; they're provided only as reference.
The `.patch` files were generated relative to the AOSP source tree layout.
If you're using a different branch or platform version, you may need to resolve minor conflicts manually.


## Usage Instructions
 If you're using the Android emulator, start it in one terminal:
```bash
emulator
```

In a new terminal:
```bash
OUTPUT_PATH=/path/to/output/file
echo "echo 1 > /sys/kernel/tracing/tracing_on && cat /sys/kernel/tracing/trace_pipe" | adb shell > $OUTPUT_PATH
```

## Tracer Implementation and Design
The tracer uses an eBPF program that hooks into the `block_rq_issue` tracepoint.
Each time a tracepoint is triggered, the eBPF program records:
- The tracepoint name
- Device ID
- Operation type (read or write)
- Target sector
- The number of sectors involved in the operation.

Example output:
<pre>  
 droid.bluetooth-883     [000] d..31    70.443180: bpf_trace_printk: block_rq_issue: dev:7340152 rwflag:R sector: 47232, size: 256
 queued-work-loo-1047    [000] d..31    70.443313: bpf_trace_printk: block_rq_issue: dev:265289760 rwflag:W sector: 18424256, size: 8
    kworker/u4:4-281     [000] d..31    70.443364: bpf_trace_printk: block_rq_issue: dev:265289728 rwflag:R sector: 131096, size: 256
   system_server-530     [000] d..31    70.443719: bpf_trace_printk: block_rq_issue: dev:265289760 rwflag:R sector: 18420408, size: 8
</pre>

In addition to the patch files, the implementation of tracer are referenced as git submodules.

The modifications required for implementing the eBPF program are located at ['system'](./system), specifically, the eBPF program is located here: [timeInState.c](https://github.com/OriBenZur/android_system_bpfprogs-bio-tracer/blob/bio-tracer/timeInState.c).

Additionally, we modified Android to load the new eBPF program. Those changes can be viewed at ['frameworks'](./frameworks), specifically, the loading code can be viewed here: [cputimeinstate.cpp](https://github.com/OriBenZur/android_frameworks_native-bio-tracer/blob/main/libs/cputimeinstate/cputimeinstate.cpp).

## Citation

If you use the tracer in your work, please cite our paper _Space-efficient FTL for Mobile Storage via Tiny Neural Nets_.

```bibtex
@inproceedings{10.1145/3688351.3689157,
author = {Marcus, Ron and Rashelbach, Alon and Ben-Zur, Ori and Lifshits, Pavel and Silberstein, Mark},
title = {Space-efficient FTL for Mobile Storage via Tiny Neural Nets},
year = {2024},
isbn = {9798400711817},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/3688351.3689157},
doi = {10.1145/3688351.3689157},
abstract = {We present RQFTL, a demand-based FTL for mobile storage controllers that boosts the effective Logical-To-Physical (L2P) address translation cache capacity over state-of-the-art techniques. RQFTL stores a large part of the L2P cache in a compressed form, and employs a learned data structure called RQRMI that leverages tiny neural nets to quickly find the correct translation entry in the cache. RQFTL uses neural network inference for cache lookups, and rapidly retrains the neural nets to efficiently handle L2P cache updates. It is specifically optimized to achieve high coverage for scattered read accesses, making it suitable for popular read-skewed workloads such as mobile gaming.We evaluate RQFTL on hours-long real-world I/O traces of popular modern mobile apps, including games, video editing, and social networking apps collected on Google Pixel 6a phone. We show that RQFTL outperforms all the state-of-the-art FTLs in these workloads, increasing the effective L2P cache capacity by over an order of magnitude compared to DFTL and up to 5x over the recent LeaFTL. As a result, it achieves 65\%, and 25\% lower miss rate compared to DFTL and LeaFTL respectively, under the same SRAM capacity, and allows reduction of the total SRAM capacity of a controller by about a third of that of LeaFTL.},
booktitle = {Proceedings of the 17th ACM International Systems and Storage Conference},
pages = {146–161},
numpages = {16},
keywords = {FTL, Mobile Storage, Range Matching, SSD},
location = {Virtual, Israel},
series = {SYSTOR '24}
}
```
