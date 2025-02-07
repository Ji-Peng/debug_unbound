This repository is used to replicate the [issue-1045](https://github.com/NLnetLabs/unbound/issues/1045) of low throughput of DNS over TLS (DoT) with `unbound` on Ubuntu 22.04.

Before installing `unbound`, I customized some OpenSSL installations because my research requires exploring the impact of different OpenSSL implementations. The specific steps can be found at [this link](https://github.com/Ji-Peng/eng25519_artifact/tree/main/ENG25519/eng25519).

In summary, I installed `openssl 1.1.1q` in the `/root/local-eng25519` directory.

The commands I used to install and configure `unbound` include:

```bash
git clone https://github.com/NLnetLabs/unbound.git
./configure --with-ssl=/root/local-eng25519
make && make install
```
To start `unbound`: `taskset -c 0 unbound -c /root/eng25519/test/unbound.conf`

The configuration file used above can be found [here](https://github.com/Ji-Peng/eng25519_artifact/tree/main/ENG25519/eng25519/test).

For the DoT client, I used [DoT Timer](https://github.com/Ji-Peng/eng25519_artifact/tree/main/ENG25519/DoT_timer).

The command to start the client is:

```bash
for i in {1..100}; do taskset -c 0 dot_timer -n 10000 -connect $PEER_LOCAL_AD:853 -sigalgs ed25519 -client_sigalgs ed25519 -groups X25519 -no_ssl3 -no_tls1 -no_tls1_1 -no_tls1_2 >> unbound_x25519.txt; done
```

Here, `PEER_LOCAL_AD` is the server's IP address. In this replication, it equals `127.0.0.1`, meaning I ran `unbound` and `DoT Timer` on the same machine.

The repository contains standard outputs and standard error outputs of the `configure` command, `make` command, and `make install` command.

The `unbound.log` file is a fragment copied from `/var/log/syslog`, indicating that the number of requests processed in 60 seconds is `1352`, resulting in a throughput of 23 connections/second, which is exceptionally low.

The `DoTTimer.log` file contains the output generated by the client.

The output of the `unbound -V` command is:
```bash
Version 1.19.4

Configure line: --with-ssl=/root/local-eng25519
Linked libs: mini-event internal (it uses select), OpenSSL 1.1.1q  5 Jul 2022
Linked modules: dns64 respip validator iterator

BSD licensed, see LICENSE in source package for details.
Report bugs to unbound-bugs@nlnetlabs.nl or https://github.com/NLnetLabs/unbound/issues
```

My operating system information is:
```bash
Linux jipengzhang 6.5.0-14-generic #14~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Nov 20 18:15:30 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

My CPU information is:
```bash
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 167
model name      : 11th Gen Intel(R) Core(TM) i7-11700K @ 3.60GHz
stepping        : 1
microcode       : 0x5d
cpu MHz         : 800.322
cache size      : 16384 KB
physical id     : 0
siblings        : 16
core id         : 0
cpu cores       : 8
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 27
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single ssbd ibrs ibpb stibp ibrs_enhanced fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid mpx avx512f avx512dq rdseed adx smap avx512ifma clflushopt intel_pt avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp hwp_pkg_req avx512vbmi umip pku ospke avx512_vbmi2 gfni vaes vpclmulqdq avx512_vnni avx512_bitalg avx512_vpopcntdq rdpid fsrm md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass swapgs mmio_stale_data retbleed eibrs_pbrsb gds
bogomips        : 7200.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
power management:
```