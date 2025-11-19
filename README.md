# GCP Landing Zone – Enterprise-Scale Architecture (Terraform)

# WORK IN PROGRESS

This repository will contain an enterprise-grade Google Cloud Landing Zone, built with Terraform, designed to simulate a multi-domain healthcare organization with realistic networking, security, and workload isolation requirements. And then it will expanded to use for a data engineering project with streaming and batch pipelines. And to make things a little more interesting we will use Azure devops for CI-CD. 

We will work on setting the network architecture properly, then vpc perimeters, then org policies, then IAM. 


## Overview

Large healthcare organizations typically operate:

- Multiple business domains (Clinical, Enterprise, Billing, HR, Research/ML, etc.)
- Thousands of projects across internal, external, and restricted workloads
- Strict PHI segmentation
- Multiple development environments (dev, qa, test, prod)
- Regulatory-driven network segmentation
- Shared VPC networking managed by a central cloud networking team

This repository implements a scalable, realistic Landing Zone that mirrors what a large healthcare enterprise should be running — with a cleaner, modern, well-architected network design.

## Core Principles

### Infrastructure-as-Code (Terraform)
Everything — org, folders, networking, projects, IAM, service accounts, automation — is created and governed via Terraform.

### Hub-and-Spoke Network Model
Shared VPC host projects (“hubs”) provide network, routing, firewalling, and DNS for dozens or hundreds of service projects (“spokes”).

### Domain-Aligned Network Isolation
Workloads are grouped by domain:
- Clinical
- Enterprise (Finance, HR, RCM)
- Research / ML

Each domain has its own network hubs.

### Prod Split Into Three Zones
Every domain’s production network is divided into:

1. Internal – core apps, integrations, batch, BI
2. External – internet-facing APIs, portals, DMZ
3. Restricted – PHI-heavy ML/analytics, highly sensitive workloads

This model provides isolation without creating hundreds of standalone VPCs.

### Nonprod Split Into Two Zones
For dev/qa/test environments:
- internal
- external

Restricted nonprod nets are optional.

## Networking Architecture (Primary Focus)

The networking foundation supports the entire Landing Zone.  
We implement:

- 3 PROD Shared VPC hubs per domain  
- 2 NONPROD Shared VPC hubs per domain  
- Consistent regional pattern (example uses usc = us-central1)
- Well-defined subnets inside each VPC

### Networking Structure

```
fd-networking
└─ fd-net-usc (region: us-central1)
   ├─ fd-net-clinical
   │  ├─ prj-net-clin-prod-int-usc (PROD – internal)
   │  │  └─ VPC: vpc-clin-prod-int-usc
   │  │     ├─ sn-clin-prod-int-core-usc
   │  │     ├─ sn-clin-prod-int-de-usc
   │  │     ├─ sn-clin-prod-int-bi-usc
   │  │     ├─ sn-clin-prod-int-bu1-usc
   │  │     └─ sn-clin-prod-int-bu2-usc
   │  │
   │  ├─ prj-net-clin-prod-ext-usc (PROD – external/DMZ)
   │  │  └─ VPC: vpc-clin-prod-ext-usc
   │  │     ├─ sn-clin-prod-dmz-usc
   │  │     └─ sn-clin-prod-partner-dmz-usc
   │  │
   │  ├─ prj-net-clin-prod-res-usc (PROD – restricted PHI)
   │  │  └─ VPC: vpc-clin-prod-res-usc
   │  │     ├─ sn-clin-prod-res-ml-usc
   │  │     └─ sn-clin-prod-res-analytics-usc
   │  │
   │  ├─ prj-net-clin-nonprod-int-usc (NONPROD – internal)
   │  │  └─ VPC: vpc-clin-nonprod-int-usc
   │  │     ├─ sn-clin-nonprod-int-core-usc
   │  │     ├─ sn-clin-nonprod-int-de-usc
   │  │     ├─ sn-clin-nonprod-int-team-usc
   │  │     └─ sn-clin-nonprod-int-sbx-usc
   │  │
   │  └─ prj-net-clin-nonprod-ext-usc (NONPROD – external)
   │     └─ VPC: vpc-clin-nonprod-ext-usc
   │        ├─ sn-clin-nonprod-dmz-usc
   │        └─ sn-clin-nonprod-partner-usc
   │
   ├─ fd-net-enterprise
   │  ├─ prj-net-ent-prod-int-usc (PROD – internal)
   │  │  └─ VPC: vpc-ent-prod-int-usc
   │  │     ├─ sn-ent-prod-int-core-usc
   │  │     ├─ sn-ent-prod-int-fin-usc
   │  │     ├─ sn-ent-prod-int-hr-usc
   │  │     ├─ sn-ent-prod-int-rcm-usc
   │  │     └─ sn-ent-prod-int-analytics-usc
   │  │
   │  ├─ prj-net-ent-prod-ext-usc (PROD – external)
   │  │  └─ VPC: vpc-ent-prod-ext-usc
   │  │     ├─ sn-ent-prod-dmz-usc
   │  │     └─ sn-ent-prod-partner-usc
   │  │
   │  ├─ prj-net-ent-prod-res-usc (PROD – restricted)
   │  │  └─ VPC: vpc-ent-prod-res-usc
   │  │     ├─ sn-ent-prod-res-fin-usc
   │  │     └─ sn-ent-prod-res-hr-usc
   │  │
   │  ├─ prj-net-ent-nonprod-int-usc (NONPROD – internal)
   │  │  └─ VPC: vpc-ent-nonprod-int-usc
   │  │     ├─ sn-ent-nonprod-int-core-usc
   │  │     ├─ sn-ent-nonprod-int-fin-usc
   │  │     ├─ sn-ent-nonprod-int-hr-usc
   │  │     └─ sn-ent-nonprod-int-rcm-usc
   │  │
   │  └─ prj-net-ent-nonprod-ext-usc (NONPROD – external)
   │     └─ VPC: vpc-ent-nonprod-ext-usc
   │        ├─ sn-ent-nonprod-dmz-usc
   │        └─ sn-ent-nonprod-partner-usc
   │
   └─ fd-net-research
      ├─ prj-net-res-prod-int-usc (PROD – internal)
      │  └─ VPC: vpc-res-prod-int-usc
      │     ├─ sn-res-prod-int-core-usc
      │     ├─ sn-res-prod-int-ml-usc
      │     └─ sn-res-prod-int-exp-usc
      │
      ├─ prj-net-res-prod-ext-usc (PROD – external)
      │  └─ VPC: vpc-res-prod-ext-usc
      │     ├─ sn-res-prod-dmz-usc
      │     └─ sn-res-prod-partner-usc
      │
      ├─ prj-net-res-prod-res-usc (PROD – restricted)
      │  └─ VPC: vpc-res-prod-res-usc
      │     ├─ sn-res-prod-res-ml-usc
      │     └─ sn-res-prod-res-analytics-usc
      │
      ├─ prj-net-res-nonprod-int-usc (NONPROD – internal)
      │  └─ VPC: vpc-res-nonprod-int-usc
      │     ├─ sn-res-nonprod-int-core-usc
      │     ├─ sn-res-nonprod-int-ml-usc
      │     └─ sn-res-nonprod-int-exp-usc
      │
      └─ prj-net-res-nonprod-ext-usc (NONPROD – external)
         └─ VPC: vpc-res-nonprod-ext-usc
            ├─ sn-res-nonprod-dmz-usc
            └─ sn-res-nonprod-partner-usc
```

## Terraform Module Structure

```
/terraform
  /org
  /folders
  /networking
  /projects
  /iam
  /services
  /bootstrap
```

## VPC Service Controls and Organization Policies

Once the base networking is in place, the next layer of the Landing Zone is **data-plane isolation and governance**:

- **VPC Service Controls (VPC-SC)** to reduce data exfiltration risk for Google-managed services (BigQuery, Cloud Storage, Pub/Sub, etc.).  
- **Organization policies (Org Policy)** to enforce global guardrails at the org/folder/project level.  
- **Policy Intelligence** (Policy Analyzer, Policy Simulator) to test and understand policies at scale.

### VPC Service Controls – Perimeter Strategy

VPC Service Controls lets us define **service perimeters** around groups of projects and Google-managed services to reduce data exfiltration risk (for example, blocking reads of BigQuery tables from outside our perimeter, or from untrusted networks). :contentReference[oaicite:0]{index=0}  

We follow the same **domain + env + zone** model used for networking:

- **Domains**: clinical, enterprise, research  
- **Environments**: prod, nonprod  
- **Zones**: internal, external, restricted

At the VPC-SC layer, we define **logical perimeters** around our data/analytics projects, not around every little app. Example (compressed):

```text
VPC-SC Access Policy (org-level)
└─ Service Perimeters
   ├─ perim-clin-prod-int
   │  ├─ Projects:
   │  │  ├─ prj-de-synthea-prod
   │  │  ├─ prj-de-claims-prod
   │  │  ├─ prj-de-member360-prod
   │  │  ├─ prj-bi-core-prod
   │  │  └─ prj-hl7-gateway-prod
   │  └─ Protected services:
   │     ├─ bigquery.googleapis.com
   │     ├─ storage.googleapis.com
   │     ├─ pubsub.googleapis.com
   │     └─ healthcare.googleapis.com (FHIR/HL7)
   │
   ├─ perim-clin-prod-res
   │  ├─ Projects:
   │  │  ├─ prj-ml-readmission-prod
   │  │  ├─ prj-ml-nlp-notes-prod
   │  │  └─ prj-ml-imaging-prod
   │  └─ Protected services:
   │     ├─ bigquery.googleapis.com
   │     ├─ storage.googleapis.com
   │     └─ aiplatform.googleapis.com
   │
   ├─ perim-ent-prod-int
   │  ├─ Projects:
   │  │  ├─ prj-fin-core-prod
   │  │  ├─ prj-fin-payroll-prod
   │  │  └─ prj-hr-core-prod
   │  └─ Protected services:
   │     ├─ bigquery.googleapis.com
   │     └─ storage.googleapis.com
   │
   ├─ perim-res-prod-res
   │  ├─ Projects:
   │  │  ├─ prj-ml-imaging-prod
   │  │  └─ prj-ml-exp-001 … 010
   │  └─ Protected services:
   │     ├─ bigquery.googleapis.com
   │     ├─ storage.googleapis.com
   │     └─ aiplatform.googleapis.com
   │
   └─ perim-clin-nonprod-int
      ├─ Projects:
      │  ├─ prj-de-synthea-dev
      │  ├─ prj-de-synthea-qa
      │  ├─ prj-fhir-api-dev
      │  └─ prj-ml-readmission-dev
      └─ Protected services:
         ├─ bigquery.googleapis.com
         └─ storage.googleapis.com
```

