---
title: Migrate to the Azure Monitor agent (AMA) from the Log Analytics agent (MMA/OMS) for Microsoft Sentinel
description: Learn about migrating from the Log Analytics agent (MMA/OMS) to the Azure Monitor agent (AMA), when working with Microsoft Sentinel.
author: yelevin
ms.topic: reference
ms.date: 04/03/2024
ms.author: yelevin
---

# AMA migration for Microsoft Sentinel
This article describes the migration process to the Azure Monitor Agent (AMA) when you have an existing Log Analytics Agent (MMA/OMS), and are working with Microsoft Sentinel. 

> [!IMPORTANT]
> The Log Analytics agent will be [retired on **31 August, 2024**](https://azure.microsoft.com/updates/were-retiring-the-log-analytics-agent-in-azure-monitor-on-31-august-2024/). If you are using the Log Analytics agent in your Microsoft Sentinel deployment, we recommend that you start planning your migration to the AMA.

## Prerequisites
Start with the [Azure Monitor documentation](/azure/azure-monitor/agents/azure-monitor-agent-migration) which provides an agent comparison and general information for this migration process. 

This article provides specific details and differences for Microsoft Sentinel.


## Gap analysis between agents

The Azure Monitor agent provides extra functionality and a throughput that is 25% better than legacy Log Analytics agents. Migrate to the new AMA connectors to get higher performance, especially if you are using your servers as log forwarders for Windows security events or forwarded events.

The Azure Monitor agent provides the following extra functionality, which is not supported by legacy Log Analytics agents:

| Log type | Functionality |
| --- |---|
| **Windows logs** | Filtering by security event ID <br>Windows event forwarding |
| **Linux logs** | Multi-homing | 

The only logs supported only by the legacy Log Analytics agent are Windows Firewall logs.

## Recommended migration plan

Each organization will have different metrics of success and internal migration processes. This section provides suggested guidance to consider when migrating from the Log Analytics MMA/OMS agent to the AMA, specifically for Microsoft Sentinel.

**Include the following steps in your migration process**:

1. Make sure that you've reviewed necessary prerequisites and other considerations as [documented here](/azure/azure-monitor/agents/azure-monitor-agent-migration#before-you-begin) in the Azure Monitor documentation.

1. Run a proof of concept to test how the AMA sends data to Microsoft Sentinel, ideally in a development or sandbox environment.

    1. To connect your Windows machines to the [Windows Security Event connector](data-connectors/windows-security-events-via-ama.md), start with **Windows Security Events via AMA** data connector page in Microsoft Sentinel. For more information, see [Windows agent-based connections](connect-services-windows-based.md).

    1. Go to the **Security Events via Legacy Agent** data connector page. On the **Instructions** tab, under **Configuration** > Step 2, **Select which events to stream**, select **None**. This configures your system so that you won't receive any security events through the MMA/OMS, but other data sources relying on this agent will continue to work. This step affects all machines reporting to your current Log Analytics workspace.

    > [!IMPORTANT]
    > Ingesting data from the same source using two different types of agents will result in double ingestion charges and duplicate events in the Microsoft Sentinel workspace. 
    >
    > If you need to keep both data connectors running simultaneously, we recommend that you do so only for a limited time for a benchmarking, or test comparison activity, ideally in a separate test workspace.
    >

1. Measure the success of your proof of concept. 

    To help with this step, use the **AMA migration tracker** workbook, which displays the servers reporting to your workspaces, and whether they have the legacy MMA, the AMA, or both agents installed. You can also use this workbook to view the DCRs collecting events from your machines, and which events they are collecting.

    For example:

    :::image type="content" source="media/ama-migrate/migrate-workbook.png" alt-text="Screenshot of the AMA migration tracker workbook." lightbox="media/ama-migrate/migrate-workbook.png" :::

    Success criteria should include a statistical analysis and comparison of the quantitative data ingested by the MMA/OMS and AMA agents on the same host:

    - Measure your success over a predefined time period that represents a normal workload for your environment.

    - While testing, make sure to test each new feature provided by the AMA, such as Linux multi-homing, Windows event filtering, and so on.

    - Plan your rollout for AMA agents in your production environment according to your organization's risk profile and change processes.

3. Roll out the new agent on your production environment and run a final test of the AMA functionality.

4. Disconnect any data connectors that rely on the legacy connector, such as Security Events with MMA. Leave the new connector, such as Windows Security Events with AMA, running.

    While you can have both the legacy MMA/OMS and the AMA agents running in parallel, prevent duplicate costs and data by making sure that each data source uses only one agent to send data to Microsoft Sentinel.

5. Check your Microsoft Sentinel workspace to make sure that all your data streams have been replaced using the new AMA-based connectors.

6. Uninstall the legacy agent. For more information, see [Manage the Azure Log Analytics agent ](/azure/azure-monitor/agents/agent-manage#uninstall-agent).

## FAQs
The following FAQs address issues specific to AMA migration with Microsoft Sentinel. For more information, see [Frequently asked questions for Azure Monitor Agent](/azure/azure-monitor/agents/agents-overview#frequently-asked-questions) in the Azure Monitor documentation.
  
## What happens if I run both MMA/OMS and AMA in parallel in my Microsoft Sentinel deployment?
Both the AMA and MMA/OMS agents can co-exist on the same machine. If they both send data, from the same data source to a Microsoft Sentinel workspace, at the same time, from a single host, duplicate events and double ingestion charges will occur.

For your production rollout, we recommend that you configure either an MMA/OMS agent or the AMA for each data source. To address any issues for duplication, see the relevant FAQs in the [Azure Monitor documentation](/azure/azure-monitor/agents/agents-overview#frequently-asked-questions).

## The AMA doesn’t yet have the features my Microsoft Sentinel deployment needs to work. Should I migrate yet?
The legacy Log Analytics agent will be retired on 31 August 2024.

We recommend that you keep up to date with the new features being released for the AMA over time, as it reaches towards parity with the MMA/OMS. Aim to migrate as soon as the features you need to run your Microsoft Sentinel deployment are available in the AMA.

While you can run the MMA and AMA simultaneously, you may want to migrate each connector, one at a time, while running both agents.



## Next steps

For more information, see:

- [Overview of the Azure Monitor agents](/azure/azure-monitor/agents/agents-overview)
- [Migrate from Log Analytics agents](/azure/azure-monitor/agents/azure-monitor-agent-migration)
- [Windows Security Events via AMA](data-connectors/windows-security-events-via-ama.md)
- [Security events via Legacy Agent (Windows)](data-connectors/security-events-via-legacy-agent.md)
- [Windows agent-based connections](connect-services-windows-based.md)
