---
title: "Apple Silicon GPU Monitoring"
date: 2026-06-30
draft: false
tags: ["gpu", "engineering", "apple", "monitoring", "observability"]
description: "A look at the tools available for monitoring GPU workloads on Apple silicon, covering PowerMetrics' built-in stats and how to get clean, scrapeable metrics out of macmon."
---

If you're running inference, training, AI experiments or a lab locally on Apple silicon, you'll likely want to be able to monitor your workloads.

If you're running your workloads on CPUs, you have the normal `*top` family of tools available like `top`, `htop` and `btop`. But for GPUs, you'll need to reach for a different set of tools. The two that I've been testing are:

* PowerMetrics
* macmon

## PowerMetrics

### GPU Metrics

The built-in tool on macOS is called PowerMetrics. This works, but there are a few problems. First off, you have to run it with `sudo`, which is pretty tedious. The other problem is that Apple hasn't documented it very well, so I've had to make some assumptions in interpreting the data.

An example below:

```bash
sudo powermetrics --samplers gpu_power
```

This should give you output similar to this:

```bash
*** Sampled system activity (Sat Jun 27 20:18:22 2026 +0100) (5009.31ms elapsed) ***=
**** GPU usage ****
GPU HW active frequency: 392 MHz
GPU HW active residency:  38.91% (389 MHz:  39% 486 MHz: .09% 648 MHz: .08% 778 MHz: .21% 972 MHz: .01% 1296 MHz:   0%)
GPU SW requested state: (P1 :  99% P2 : .24% P3 : .35% P4 : .46% P5 : .03% P6 :   0%)
GPU SW state: (SW_P1 :   0% SW_P2 :   0% SW_P3 :   0% SW_P4 :   0% SW_P5 :   0% SW_P6 :   0%)
GPU idle residency:  61.09%
GPU Power: 149 mW
```

This can be a little confusing to decipher:

#### GPU HW active frequency

`GPU HW active frequency: 392 MHz`: the currently active hardware (HW) frequency of the GPU. Apple silicon operates at a number of different discrete frequencies, which are dynamically selected based on active workloads, power consumption, thermals etc. These frequencies are approximately `[0, 389, 486, 648, 778, 972, 1296]` MHz (which you will see listed in the next line item). Note that `392 MHz` isn't one of the discrete values; it's computed as a weighted average over the sampling period.

#### GPU HW active residency

`GPU HW active residency:  38.91% (389 MHz:  39% 486 MHz: .09% 648 MHz: .08% 778 MHz: .21% 972 MHz: .01% 1296 MHz:   0%)`: here you can see a breakdown of active residency by discrete frequency. Again, the numbers look confusing, and I haven't quite worked out how they are derived. Adding the discrete percentages gives a total of 39.39%, not 38.91%. This could be due to various factors such as non-atomic sampling, different underlying hardware counters being used etc, but the logic isn't documented anywhere I have been able to find.

#### GPU SW requested state

`GPU SW requested state: (P1 :  99% P2 : .24% P3 : .35% P4 : .46% P5 : .03% P6 :   0%)`: here you will see what the software (SW) is requesting, meaning the Metal driver and macOS power management deciding what they want. When a workload is submitted, the driver looks at the demand and requests a P-state. This is a policy decision, something like "we think we need P4 right now". This ranges from P1 (lowest demand) to P6 (highest demand).

#### GPU SW state

`GPU SW state: (SW_P1 :   0% SW_P2 :   0% SW_P3 :   0% SW_P4 :   0% SW_P5 :   0% SW_P6 :   0%)`: this is the actual time spent in each software state. Everything shows 0%, which is unexpected given the SW requested state above. This is likely a rounding or counter sampling artefact.

#### GPU idle residency

`GPU idle residency:  61.09%`: fairly self-explanatory: the percentage of time over the sample period that the GPU has been idle. This at least tallies to 100% against the `GPU HW active residency`.

#### GPU Power

`GPU Power: 149 mW`: again, fairly self-explanatory: the power consumption of the GPU over the sampling period in mW.

---

All useful stats, despite some skewed numbers. Here we can compare things like software demand vs hardware residency, performance states and resource demand, and a comparison between the two. However, this snapshot isn't overly useful by itself. We want to hook into something more sophisticated and track these metrics over time during our training or inference runs. This is where `macmon` comes in.

## macmon

`macmon` is an open source tool from `vladkens` available on GitHub here: https://github.com/vladkens/macmon.

You can install it via `brew`:

```bash
brew install macmon
```

And you can run a nice little TUI with:

```bash
macmon
```

This gives some basic graphing of your vitals:

* `E-CPU  USAGE% @ FREQ MHz`
* `P-CPU  USAGE% @ FREQ MHz`
* `RAM USAGE / CAPACITY GB (USAGE%) - SWAP USAGE / CAPACITY G`
* `GPU  USAGE% @  FREQ MHz`
* `Power: PWR W (avg PWR W, max PWR W) - Total PWR W (TOTAL_PWR_MIN, TOTAL_PWR_MAX)`
* `CPU CPU_PWR W (CPU_PWR_MIN, CPU_PWR_MAX) ─ TEMP °C`
* `GPU GPU_PWR W (GPU_PWR_MIN, GPU_PWR_MAX) ─ TEMP °C`
* `ANE ANE_PWR W (ANE_PWR_MIN, ANE_PWR_MAX)`

Very cool, super useful for ad hoc monitoring. But even better is that if you run:

```bash
macmon serve
```

You get two endpoints (served by default on port 9090, though note that Prometheus also uses this port by default, so check for conflicts if you're running both):

* `/json`: JSON output of the metrics
* `/metrics`: Prometheus-compatible output for scraping

The JSON output shows more than just the GPU stats we looked at earlier from PowerMetrics, although you can easily get the full set and more from PowerMetrics with `sudo powermetrics`. There are also many different filtering flags and options you can get with `powermetrics --help`. It's nice that you can get clean JSON output of the main metrics from macmon with:

```bash
curl -s http://localhost:9090/json
```

Which shows something like:

```json
{
  "all_power": 0.494245052337647,
  "ane_power": 0,
  "cpu_power": 0.339286744594574,
  "cpu_usage_pct": 0.0710111409425736,
  "ecpu_usage": [1512, 0.255764722824097],
  "gpu_power": 0.154958307743073,
  "gpu_ram_power": 0.0064940438605845,
  "gpu_usage": [388, 0.132904022932053],
  "memory": {
    "ram_total": 68719476736,
    "ram_usage": 33498726400,
    "swap_total": 2147483648,
    "swap_usage": 762118144
  },
  "pcpu_usage": [1359, 0.0248227454721928],
  "ram_power": 1.08980870246887,
  "soc": {
    "chip_name": "Apple M1 Max",
    "ecpu_cores": 2,
    "ecpu_freqs": [600, 972, 1332, 1704, 2064],
    "ecpu_label": "E",
    "gpu_cores": 32,
    "gpu_freqs": [0, 388, 486, 648, 777, 972, 1296],
    "mac_model": "MacBookPro18,2",
    "memory_gb": 64,
    "pcpu_cores": 8,
    "pcpu_freqs": [600, 828, 1056, 1296, 1524, 1752, 1980, 2208, 2448, 2676, 2904, 3036, 3132, 3168, 3228],
    "pcpu_label": "P"
  },
  "sys_power": 9.14718246459961,
  "temp": {
    "cpu_temp_avg": 47.0178909301758,
    "gpu_temp_avg": 45.5415573120117
  },
  "timestamp": "2026-06-27T19:32:07.409680+00:00"
}
```

You can see that we have things like the GPU frequencies listed, the power, thermals etc. Very nice. You can pipe this to `jq` for a snapshot, read it from custom tooling etc.

Even better though, is that you can set up a quick Prometheus and Grafana with `docker compose` and build out some dashboards. I've done a quick demo here: https://github.com/code-engine/apple-silicon-prometheus-grafana that you can use as a template to try it out.

## Summary

I hope you'll find this useful, whether experimenting with serving inference on your Mac Mini or doing AI / ML workloads on your MacBook.
