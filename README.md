# Hive — Open Source SIEM Platform

A fully open-source Security Information and Event Management (SIEM) platform built as a final year undergraduate project. Hive integrates widely-used community tools to deliver real-time threat detection, log aggregation, incident response, and threat intelligence capabilities.

The project is structured around two independent tech stacks, each representing a different architectural approach to building a functional SOC environment. Both stacks use **Wazuh** as the core agent-based detection engine and extend it with complementary tools for alerting, visualization, and case management.

---

## Repository Structure

```
Hive/
├── Forenzy_Manan_Docs/        # Cortex + MISP + theHive
└── Forenzy_Manan_Docs_2/        # Graylog + Grafana + DFIR-IRIS
```

---

## Stack 1 — Threat Intelligence & Case Management

Focuses on integrating threat intelligence and incident response workflows alongside Wazuh.

| Component | Role |
|-----------|------|
| Wazuh | Agent-based intrusion detection and log collection |
| Cortex | Automated observable analysis and response actions |
| MISP | Threat intelligence platform and IOC sharing |
| theHive | Security incident case management |

→ See [`Stack-1/`](./Forenzy_Manan_Docs/overview.md) for setup guide.

---

## Stack 2 — Log Aggregation, Visualization & DFIR

Focuses on centralized log management, dashboarding, and digital forensics/incident response.

| Component | Role |
|-----------|------|
| Wazuh | Agent-based intrusion detection and log collection |
| Graylog | Centralized log aggregation and search |
| Grafana | Real-time dashboards and alerting |
| DFIR-IRIS | Digital forensics and incident response platform |

> **Note:** In this stack, Wazuh, Graylog, and Grafana are deployed on the same machine to optimize hardware resource utilization.

→ See [`Stack-2/`](./Forenzy_Manan_Docs_2/overview.md) for setup guide.

---

## Wazuh Agents

Both stacks rely on Wazuh agents installed on monitored endpoints to forward logs and security events to the Wazuh server. Agent installation guides are included in each stack's documentation.

---

## Purpose

This project was built to explore how open-source security tools can be combined to replicate core SOC capabilities without commercial licensing costs. Each stack was independently deployed and tested as part of the final year project evaluation.
