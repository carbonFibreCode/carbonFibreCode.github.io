---
layout: post
title: "Beyond the Fix: Orchestrating Data Migrations in Twenty (YC23)"
date: 2026-02-18
categories: engineering open-source
excerpt: "How a 'simple' date filter fix evolved into a cross-repo coordination of feature flags, data migrations, and timezone-aware engineering."
---

Contributing to **Twenty**, the leading open-source CRM alternative to Salesforce, is rarely just about fixing a bug. It’s about understanding the architectural ripples of every change. My recent work on the `DATE_TIME` filtering logic is a prime example of how a simple UI friction point can lead to a multi-stage deployment strategy involving data migrations and feature flags.

### The Problem: Precision vs. Intent

The core issue ([core-team-issues #2027](https://github.com/twentyhq/core-team-issues/issues/2027)) centered on the `DATE_TIME IS` operand. Technically, it was "correct"—it filtered for a specific minute. But users don't think in minutes; they think in days. 

If a user filtered for "Records on February 18," they expected everything from midnight to 11:59 PM. Instead, they were stuck with a `DateTimePicker` that felt like a hurdle rather than a tool. Furthermore, filtering on a precise point in time is notoriously brittle across global teams where "now" varies by 24 hours.

### The Strategy: Two PRs, One Goal

After discussing the approach with [Lucas Bordeau](https://github.com/lucasbordeau), we decided to split the resolution into two distinct Pull Requests to ensure a safe, zero-downtime rollout.

#### Phase 1: Foundational Logic & Feature Flags ([PR #17529](https://github.com/twentyhq/twenty/pull/17529))
The first step was building the infrastructure for the "new way" while maintaining compatibility for the "old way." 
- **Timezone Awareness**: I updated the backend (`view-query-params.service.ts`) to fetch the active workspace member's timezone instead of defaulting to UTC.
- **Operand Refactoring**: Modified the GraphQL operation generator to transform a single date into a `gte` / `lt` range (Start of Day to End of Day).
- **Feature Flagging**: We introduced a feature flag, initially set to `false`. This allowed the code to be merged without immediately affecting production users before their data was ready.

#### Phase 2: The Data Migration ([PR #17564](https://github.com/twentyhq/twenty/pull/17564))
Existing users already had thousands of saved view filters stored in ISO format (e.g., `2024-01-31T12:00:00Z`). Simply flipping the code logic would have broken these saved views.
- **The Migration Command**: I authored a migration command that identified old `DATE_TIME` filter values containing an ISO 'T' and converted them to plain date strings (`2024-01-31`).
- **Cache Flushing**: To ensure the UI updated instantly, the migration was designed to flush the virtualized table metadata cache.
- **Rollout**: The strategy involved running this migration and concurrently setting the feature flag to `true` within the same process.

### Testing for the Extremes

Engineering for global users means testing for edge cases. Per Lucas's suggestion, I performed QA by setting my local machine to **Auckland (UTC+13)** and the Twenty workspace setting to **Samoa (UTC-11)**. This 24-hour shift is the ultimate "stress test" for date logic—if your filter still catches the right records here, it works anywhere.

### Lessons in Open Source Stewardship

This contribution reinforced a vital lesson for any DevX or Core Engineer: **Implementation is only 50% of the work.** The other 50% is ensuring that the transition for existing users is seamless. 

By coordinating a feature-flagged rollout and a robust data migration, we moved the needle on Twenty’s UX without a single broken saved view.

***

- **See the Code Fix**: [Twenty PR #17529](https://github.com/twentyhq/twenty/pull/17529)
- **Check the Data Migration**: [Twenty PR #17564](https://github.com/twentyhq/twenty/pull/17564)
