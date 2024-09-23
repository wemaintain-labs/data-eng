# Data Engineering @ WeMaintain

This is a case study for Data Engineering at WeMaintain. It outlines shared architecture and requirements, a starting point for technical discussions.

## Guidelines
> Everything that follows should be taken like:
> 
> *Imagine you encountered that issue while working at WeMaintain and were tasked to facilitate a decision-making meeting with other Tech Leads.*


There are no trick questions, no wrong answers, and no expected answers. The goal is to see how you attack an unknown problem with a real-life set of constraints—not a brand new system but rather a broad improvement to an existing system. 

__We do not expect actual code or a functional system__

The expected output should be way more straightforward and can be in whatever form you deem relevant: 
A simple architecture diagram with the tool of your choice, something like a mock [ADR](https://github.com/joelparkerhenderson/architecture-decision-record) or a simple markdown file with any relevant information. How *you* would approach research and restitution is the main focus. 

The goal of the following interview will be to deep dive into your proposed solution(s). 
You should feel free to ask for any clarifications via e-mail while thinking about this case study.

## Context
At WeMaintain, our data lake handles a variety of sources and use cases.

Our data usage is many-fold:
- BI is used internally to track operational KPIs (examples: Operational quality, supply costs, breakdown origins ...)
- BI is used externally to provide reporting to Customers (examples: Uptime of devices, mean time to repair, % of repeat breakdowns...)
- BI is used programmatically in our tooling and often embedded in our products. (examples: Dashboard to correlate real-world usage & energy usage, track traffic information...)

Some of our data sources include:
- IoT sensors
- Application databases
- CRM
- ERP (Finance, supply, ...)

The IoT sensors' data is pre-processed on edge and then ingested in GCP via a streaming service.
The Application databases' data is ingested in GCP via a streaming service.
Both the CRM & ERP's data are ingested in GCP via a batch service.

DBT is then used to merge, transform & aggregate data from all those sources in GCP. 
The processed data is then made available through our BI platform

[todo diagram]

## New requirement
As part of the ongoing development of our offerings, we want to add a couple of new features:
1. The ability to monitor & create alerting around any measures (example: text message when we detect a breakdown, email when we detect that the usage of a device falls below a threshold, slack message when energy consumption is above a given threshold ...)
2. The ability to stream (near) real-time data to a dashboard (example: See the real-time position of a lift on a dashboard)

To do so, we'd like to introduce a new TimeSeries database to our current data stack. (We use [InfluxDB](https://www.influxdata.com/) internally, but feel free to imagine any similar tooling)

While data from the CRM and ERP will probably never need to live in that TimeSeries DB, many of the tables currently existing in our data lake would be required there. We will focus on a select few to consider potential issues.

- Elevator position data, captured by IoT (to know where the lift was at a given time)
- Elevator trip between floors data (currently computed via dbt based on IoT position data)
- Breakdown detection signals captured by IoT (when the IoT detects that something is going wrong on the Elevator)
- Breakdown data like start, end, type, false alert ... (currently computed via a microservice, based on Breakdown signals & engineer feedback once on site)

## Open questions
Note that you don't have to answer every question; these are just examples of questions we asked ourselves when confronted with this issue. Feel free to add your own; feel free to ignore those that don't seem relevant to you.

- **How would this database fit into the current data architecture ?**
> Is it part of the datalake? Just another source? Is it a middle step between the two?

- **How would we ensure data consistency between this database & our BI ?**
> Should the "trip" calculation be done before the timeseries db ? After ? How do we ensure we don't have two algorithms to maintain in two different places?
> Is the trip use-case (calculating based on position data) & the breakdown data use-case (computed through a variety of different means from various sources) the same? Should they be handled by the same means?

- **Where should what kind of data live? How will we explain the logic of this new system to software engineers / data scientists?**
> They're used to a very simple "if it's software engineering, do it in the backend" / "if it's data, do it in dbt". How will we explain where this new time series db fits?
  
