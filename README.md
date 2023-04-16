# Cpufreq Governor Anxious

- This Governor is based on Performance Governors.
- It aims to solve Broken HALs(unfixable).
- This is still work in progess(Alpha) more features are still on the pipeline.

## Tunable Parameters
- `max_throttle_step` - This parameter defines the maximum steps of frequency for the respective cluster can be throttled down.
- `idle_frequency` - This paraemter defines the frequency to be used when the CPU is 'idle'.
- `idle_threshold` - This parameter defines 'idle' for the respective cluster. The threshold is average load of all cores in that cluster and if average is lower than defined threshold `idle_threshold` then the `idle_frequency` is applied to the CPU.
```
Steps/Levels  |     Frequency(Mhz)           Governor Parameters
 --------------------------------------------------------------------
 0             |      652000            -----> idle_frequency (default)
 1             |      1036000
 2             |      1401000
 3             |      1689000                                               __
 4             |      1804000           -----> tunable->max_throttle_step     |
 5             |      1958000                                                 |
 6             |      2016000                                                 |-> cluster>cur_level can move up within this
 7             |      2150000           -----> cluster>cur_level              | 
 8             |      2208000           -----> cluster->nr_levels           __|
 ```
 ## Working
Using the above representation we will see how the governor works. The `cluster->nr_levels` is the total number of frequency steps available for the current cluster.
`cluster->cur_level` holds the current frequency step the CPU is at, initially it's at highest step in the unthrottled state. 

To explain this let us take an example using the above representation,<br />
`cluster->nr_level` = 8 <br />
`tunable->max_throttle_step` = 4 <br />
`throttle_temperature` = 45 <br />
<br />
Lets say the temprature of CPU has risen to 45 degrees,<br />
`thermal_monitor->cur_temps` = 45 <br />

Using this, Initially the `cluster->cur_level` is equal to `cluster->nr_levels`. Now the 
`thermal_monitor->cur_temps` have increased to 45 which is also the Trip point (`throttle_temperature`) for the governor so the  `govern_cpu()` does the first throttle
which any temprature rise will have no affect on frequency of the CPU. Same is true if the CPU is already throttled and temperature is falling, it thottle's 'up' CPU for 
every drop equal to `temprature_diff`, until it reaches max level `cluster->nr_levels` at which point the CPU is no more Hot and temprature has dropped below 
`throttle_temperature`.

I spent a lot of time on this, resolved all conflicts, updated old sources and added callers for latest sources from 3.10 to 4.9 but it's a fun and fun way to learn how cpu works and cpuferq governor linux :v
