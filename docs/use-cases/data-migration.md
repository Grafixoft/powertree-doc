---
id: data-migration
title: Data Migration
---

### Background

Data migration is one of the most common problems modern businesses encounter.
Whether it's to back up your data, or to load it to a data warehouse, building and maintaining an ETL process is hard.
Data commonly has to go through different transformations and validations in order to be written to the target.
In many cases the data needs to be enriched through different sources, e.g. an external API, increasing the complexity and time requirements.

While working serially with the data initially does the job, it becomes increasingly more challenging when working with larger volumes of data.

### How can PowerTree help?

PowerTree allows you to offload the bulk of the processing work and execute it in parallel.
This allows you to scale as long as your data sources scale and is a good fit for scenarios where either data has to go through multiple processing stages, needs to be collected from multiple sources, or generally speaking, takes a long time to process.
PowerTree guarantees the state durability of your workflow and seamlessly handles work distribution and long-running operations.
