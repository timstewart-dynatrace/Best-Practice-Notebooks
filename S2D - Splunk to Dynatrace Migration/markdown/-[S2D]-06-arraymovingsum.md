# S2D-06: ArrayMovingSum for Extended Timeframes

> **Series:** S2D — Splunk to Dynatrace Migration | **Notebook:** 6 of 9 | **Created:** January 2026 | **Last Updated:** 06/23/2026

## Overview

Anomaly Detectors have a maximum sliding window of 60 minutes. For alerts that need to evaluate longer timeframes while still using continuous monitoring, the `arrayMovingSum` function provides a solution.

This notebook explains how to use `arrayMovingSum` to create rolling aggregations that can be used with Anomaly Detectors.

---

## Table of Contents

1. [The Problem](#the-problem)
2. [How ArrayMovingSum Works](#how-arraymovingsum-works)
3. [Basic Example](#basic-example)
4. [Limitations](#limitations)
5. [Using with Anomaly Detectors](#using-with-davis-anomaly-detectors)
6. [Dashboard/Report Example (Longer Timeframes)](#dashboardreport-example-longer-timeframes)
7. [Other Array Functions](#other-array-functions)
8. [When ArrayMovingSum Isn't Enough](#when-arraymovingsum-isnt-enough)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail enabled |
| **Permissions** | `logs.read` |
| **Knowledge** | Basic DQL and array functions |

## Learning Objectives

By the end of this notebook, you will be able to:

1. Understand the `arrayMovingSum` function
2. Create rolling sum queries for extended timeframes
3. Apply this pattern to Anomaly Detectors
4. Recognize limitations and alternatives

<a id="the-problem"></a>
## The Problem
Many Splunk alerts evaluate over timeframes longer than 1 hour:

- "Alert if more than 500 errors in the last 4 hours"
- "Alert if average response time exceeds 2s over the past 2 hours"

Anomaly Detectors have a **maximum sliding window of 60 minutes**.

### The Solution: ArrayMovingSum

The `arrayMovingSum` function calculates a rolling sum across timeseries data points, allowing you to aggregate values over longer periods while maintaining 1-minute granularity.

<a id="how-arraymovingsum-works"></a>
## How ArrayMovingSum Works
```
arrayMovingSum(array, window)
```

| Parameter | Description |
|-----------|-------------|
| `array` | The timeseries array to aggregate |
| `window` | Number of data points to sum |

### Timeframe Calculation

The effective timeframe depends on:
- **Interval**: The timeseries interval (e.g., 1m)
- **Window**: The number of points to sum

```
Effective Timeframe = Interval × Window
```

**Example:** `interval:1m` with `window:60` = 60-minute rolling sum

<a id="basic-example"></a>
## Basic Example
Create a 60-minute rolling sum of error logs:

```dql
// 60-minute rolling sum of error logs
fetch logs, from:-24h
| filter loglevel == "ERROR"
| makeTimeseries error_count = count(), interval:1m
| fieldsAdd error_count_last_1h = arrayMovingSum(error_count, 60)
| fieldsRemove error_count
```

### Understanding the Output

Each data point in `error_count_last_1h` represents the **sum of errors in the preceding 60 minutes**.

| Time | error_count (per minute) | error_count_last_1h (rolling 60m) |
|------|--------------------------|-----------------------------------|
| 10:00 | 5 | 120 (sum of 9:01-10:00) |
| 10:01 | 3 | 118 (sum of 9:02-10:01) |
| 10:02 | 8 | 123 (sum of 9:03-10:02) |

<a id="limitations"></a>
## Limitations
### Maximum Window Size: 60

The `arrayMovingSum` window parameter has a **maximum value of 60**.

With a 1-minute interval, this means:
- **Maximum effective timeframe: 60 minutes**

### Increasing the Interval?

You could increase the interval to extend the timeframe:

| Interval | Window | Effective Timeframe |
|----------|--------|---------------------|
| 1m | 60 | 60 minutes |
| 5m | 60 | 300 minutes (5 hours) |
| 10m | 60 | 600 minutes (10 hours) |

**However:** Anomaly Detectors require a **1-minute interval**, so this workaround only applies to dashboards and reports, not continuous alerting.

<a id="using-with-davis-anomaly-detectors"></a>
## Using with Anomaly Detectors
For Anomaly Detectors, you must use `interval:1m` and `window:60`:

```dql
// Query for Anomaly Detector with 60-minute rolling sum
fetch logs, from:-24h
| filter loglevel == "ERROR"
| filter matchesPhrase(k8s.deployment.name, "checkout-service")
| makeTimeseries error_count = count(), by:{dt.entity.cloud_application}, interval:1m
| fieldsAdd error_count_1h = arrayMovingSum(error_count, 60)
| fieldsRemove error_count
```

### Configuring the Detector

When creating the Anomaly Detector:

| Setting | Value | Rationale |
|---------|-------|-----------|
| **Threshold** | Total errors in 60 minutes | e.g., 100 |
| **Sliding Window** | 1 minute | Each point already contains 60m of data |
| **Violating Samples** | 1 | Rolling sum already aggregates |

> **Key Insight:** Since each data point already represents 60 minutes of data, you typically want minimal sliding window and violating samples.

<a id="dashboardreport-example-longer-timeframes"></a>
## Dashboard/Report Example (Longer Timeframes)
For dashboards and reports (not Anomaly Detectors), you can use longer intervals:

```dql
// 4-hour rolling sum for dashboard visualization
// Uses 4-minute interval × 60 window = 240 minutes (4 hours)
fetch logs, from:now()-24h
| filter loglevel == "ERROR"
| makeTimeseries error_count = count(), interval:4m
| fieldsAdd error_count_4h = arrayMovingSum(error_count, 60)
| fieldsRemove error_count
```

<a id="other-array-functions"></a>
## Other Array Functions
Related functions for rolling calculations:

| Function | Description |
|----------|-------------|
| `arrayMovingSum` | Rolling sum |
| `arrayMovingAvg` | Rolling average |
| `arrayMovingMin` | Rolling minimum |
| `arrayMovingMax` | Rolling maximum |
| `arrayMovingMedian` | Rolling median |

```dql
// Rolling average response time over 30 minutes
fetch logs, from:-24h
| filter isNotNull(response_time)
| makeTimeseries avg_response = avg(response_time), interval:1m
| fieldsAdd avg_response_30m = arrayMovingAvg(avg_response, 30)
```

<a id="when-arraymovingsum-isnt-enough"></a>
## When ArrayMovingSum Isn't Enough
If you need timeframes exceeding 60 minutes with Anomaly Detectors, you have these options:

| Approach | Use Case | Trade-offs |
|----------|----------|------------|
| **Workflow-based alerting** | Any timeframe | No Dynatrace Intelligence, consumes workflow hours |
| **Metric extraction** | High-volume queries | Requires OpenPipeline config |
| **Reduce threshold** | Proportional reduction | May increase false positives |

### Proportional Threshold Reduction Example

If Splunk alert is: 500 errors in 4 hours

Convert to 1-hour equivalent:
- 500 ÷ 4 = 125 errors per hour
- Set Dynatrace Intelligence threshold to 125 with arrayMovingSum over 60 minutes

**Caveat:** This assumes errors are evenly distributed, which may not match actual patterns.

## Summary

| Scenario | Solution |
|----------|----------|
| Alert over ≤60 minutes | Standard Anomaly Detector |
| Alert over exactly 60 minutes | ArrayMovingSum with window:60 |
| Alert over >60 minutes | Use Workflows (see S2D-05) |
| Dashboard visualization >60 minutes | ArrayMovingSum with larger interval |

## Next Steps

- **S2D-07: Metric Creation** - For performance-critical alerting queries
- **S2D-05: Workflow Alerts** - For timeframes exceeding 60 minutes

## References

- [ArrayMovingSum Documentation](https://docs.dynatrace.com/docs/shortlink/array-functions#array-moving-sum)
- [Array functions (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/functions/array-functions)
- [anomaly-detection (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/anomaly-detection)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
