**Hisilicon SoC PMU (Performance Monitoring Unit)**
=================================================
The Hisilicon SoC hip05/06/07 chips consist of various independent system
device PMU's such as L3 cache(L3C), Miscellaneous Nodes(MN) and DDR
controllers. These PMU devices are independent and have hardware logic to
gather statistics and performance information.

Hip0x chips are encapsulated by multiple CPU and IO die's. The CPU die is
called as Super CPU cluster (SCCL) which includes 16 cpu-cores. Every SCCL
is further grouped as CPU clusters (CCL) which includes 4 cpu-cores each.
Each SCCL has 1 L3 cache and 1 MN units.

The L3 cache is shared by all CPU cores in a CPU die. The L3C has four banks
(or instances). Each bank or instance of L3C has Eight 32-bit counter
registers and also event control registers. The hip05/06 chip L3 cache has
22 statistics events. The hip07 chip has 66 statistics events. These events
are very useful for debugging.

The MN module is also shared by all CPU cores in a CPU die. It receives
barriers and DVM(Distributed Virtual Memory) messages from cpu or smmu, and
perform the required actions and return response messages. These events are
very useful for debugging. The MN has total 9 statistics events and support
four 32-bit counter registers in hip05/06/07 chips.

The DDR controller supports various statistics events. Every SCCL has 2 DDR
channels and hence 2 DDR controllers. The hip05/06/07 has support for a
total of 13 statistics events.

There is no memory mapping for L3 cache and MN registers. It can be accessed
by using the Hisilicon djtag interface. The Djtag in a SCCL is an independent
module which connects with some modules in the SoC by Debug Bus.

**Hisilicon SoC (hip05/06/07) PMU driver**
--------------------------------------
The hip0x PMU driver shall register perf PMU drivers like L3 cache, MN, DDRC
etc.
The available events and configuration options shall be described in the sysfs.
The "perf list" shall list the available events from sysfs.

The L3 cache in a SCCL is divided as 4 banks. Each L3 cache bank have separate
PMU registers for event counting and control. So each L3 cache bank is
registered with perf as a separate PMU.
The PMU name will appear in event listing as hisi_l3c<bank-id>_<scl-id>.
where "bank-id" is the bank index (0 to 3) and "scl-id" is the SCCL identifier
e.g. hisi_l3c0_2/read_hit is READ_HIT event of L3 cache bank #0 SCCL ID #2.

The MN in a SCCL is registered as a separate PMU with perf.
The PMU name will appear in event listing as hisi_mn_<scl-id>.
e.g. hisi_mn_2/read_req. READ_REQUEST event of MN of Super CPU cluster #2.

For DRR controller separate PMU shall be registered for each channel in a
Super CPU cluster.
The PMU name will appear in event listing as hisi_ddrc<ch-id>_<scl-id>.
where "ch-id" is the channel identifier.
eg: hisi_ddrc0_2/flux_read/ is FLUX_READ event of DDRC channel #0 in
Super CPU cluster #2.

The event code is represented by 12 bits.
	i) event 0-11
		The event code will be represented using the LSB 12 bits.

The driver also provides a "cpumask" sysfs attribute, which shows the CPU core
ID used to count the uncore PMU event.

```bash
Example usage of perf:
$# perf list
hisi_l3c0_2/read_hit/ [kernel PMU event]
------------------------------------------
hisi_l3c1_2/write_hit/ [kernel PMU event]
------------------------------------------
hisi_l3c0_1/read_hit/ [kernel PMU event]
------------------------------------------
hisi_l3c0_1/write_hit/ [kernel PMU event]
------------------------------------------
hisi_mn_2/read_req/ [kernel PMU event]
hisi_mn_2/write_req/ [kernel PMU event]
------------------------------------------
hisi_ddrc0_2/flux_read/ [kernel PMU event]
------------------------------------------
hisi_ddrc1_2/flux_read/ [kernel PMU event]
------------------------------------------

`$# perf stat -a -e "hisi_l3c0_2/read_allocate/" sleep 5`

`$# perf stat -A -C 0 -e "hisi_l3c0_2/read_allocate/" sleep 5`
```

The current driver doesnot support sampling. so "perf record" is unsupported.
Also attach to a task is unsupported as the events are all uncore.

Note: Please contact the maintainer for a complete list of events supported for
the PMU devices in the SoC and its information if needed.

Known Issues:

1. As Hisilicon hardware counters are not CPU core specific, the counter values maynot be accurate.

2. As the counter registers in Hisiilicon are config and accessed via Djtag interface, it can affect the
   event counter readings as the access is not atomic.

3. Counter are 32 bit and overflow handling is currently not supported.
