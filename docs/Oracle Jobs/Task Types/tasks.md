---
layout: nodes.liquid
date: Last Modified
title: "Tasks"
permalink: "docs/tasks/"
---

## What is a Task?

> ✅ NOTE
> This page refers to tasks in the latest version of Chainlink jobs (otherwise known as TOML, or v2 jobs). For documentation on the legacy job format, see [v1 job specs](/docs/job-specifications/).

A task is essentially the equivalent of the old [adapters](/docs/core-adapters/) but more flexible. Tasks can be composed in arbitrary order into [pipelines](/docs/jobs/task-types/pipelines/). Pipelines consist of one or more threads of execution where tasks are executed in a well-defined order.

Chainlink has a number of built-in tasks which are listed below. You can also create your own [external adapters](/docs/external-adapters/) for tasks which are accessed through a `bridge`.

## Shared attributes

All tasks share a few common attributes:

`index`: when a task has more than one input (or the pipeline overall needs to support more than one final output), and the ordering of the values matters, the index parameter can be used to specify that ordering.

```dot
data_1 [type="http" method="get" url="https://chain.link/eth_usd"       index=0]
data_2 [type="http" method="get" url="https://chain.link/eth_dominance" index=1]
multiword_abi_encode [type="eth_abi_encode" method="fulfill(uint256,uint256)"]

data_1 -> multiword_abi_encode
data_2 -> multiword_abi_encode
```

`timeout`: The maximum duration that the task is allowed to run before it is considered to be errored. Overrides the `maxTaskDuration` value in the job spec.


