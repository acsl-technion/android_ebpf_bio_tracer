# Android eBPF Bio Tracer — Patches and Reference Sources

This repository contains **patch files** and reference **Git submodules** for selected Android source components modified to implemented the *eBPF-based BIO Tracer*.

The submodules are included **for reference only** — they show the modified Android source trees that these patches were generated from.  
To use these changes in your own Android source tree, you only need to apply the `.patch` files.

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
Build Android with the applied patches. For example:
```bash
cd $ANDROID_SRC
source build/envsetup.sh
lunch aosp_x86_64-eng
m -j$(nproc)
```

### Notes

You don’t need the submodules to use these patches; they’re provided only as reference.
The .patch files were generated relative to the AOSP source tree layout.
If you’re using a different branch or platform version, minor conflicts may need manual fixes.


## Usage Instructions
 If you're using android emulator, then on one shell start the emualtor:
```bash
emulator
```

On another shell:
```bash
OUTPUT_PATH=/path/to/output/file
echo "echo 1 > /sys/kernel/tracing/tracing_on && cat /sys/kernel/tracing/trace_pipe" | adb shell > $OUTPUT_PATH
```

## Tracer Implementation and Design
The tracer uses an eBPF program which hooks to the `block_rq_issue` tracepoint.
Each time a tracepoint is triggered, the eBPF program records the tracepoint name, device ID, operation type (read or write), target sector, and the number of sectors involved in the operation.
Example output:
<pre>  
 droid.bluetooth-883     [000] d..31    70.443180: bpf_trace_printk: block_rq_issue: dev:7340152 rwflag:R sector: 47232, size: 256
 queued-work-loo-1047    [000] d..31    70.443313: bpf_trace_printk: block_rq_issue: dev:265289760 rwflag:W sector: 18424256, size: 8
    kworker/u4:4-281     [000] d..31    70.443364: bpf_trace_printk: block_rq_issue: dev:265289728 rwflag:R sector: 131096, size: 256
   system_server-530     [000] d..31    70.443719: bpf_trace_printk: block_rq_issue: dev:265289760 rwflag:R sector: 18420408, size: 8
</pre>

In addition to the patch files, the implementation of tracer are referenced as git submodules.
The implementation of the eBPF program is located at [system](/system).
Additionaly, we modified Android to load the new eBPF program. Those changes can be viewed at [frameworks](/frameworks).

