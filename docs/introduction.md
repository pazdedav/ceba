# Introduction to CEBA

Welcome to CEBA project.

CEBA stands for **Cloud Environments Builder for Azure**. This tools is not an official tool provided or supported by Microsoft, but it is based on real-life experience and number of public sources documenting design patterns and recommended practices.

This tool is designed to help you to build your **cloud platform foundations** in Azure. It can be seen as an extension of the Cloud Adoption Framework (CAF) for Azure, but instead of focusing on concepts and the entire adoption journey, it will **focus** on:

- platform architecture design
- providing common design patterns
- converting your **key design decisions** into implementation artifacts, helping you to develop a Target Architecture
- making your environment **production ready** from get go!

In the context of CAF framework, this tool fits into three parts:

1. Ready
2. Govern
3. Manage

The tool is designed using a modular approach. There are several "pillars" on which your cloud platform foundations should stand on, each pillar is represented in form of a **deployable module**. You can use all modules to build your platform completely, or decide to pick only some, based on your assessment of your existing environment.

Since the goal is to provide you with a tool that will actually build your platform, there were a number of decisions made beforehand (e.g. what tools to use), so CEBA represents an **opiniated view** on how you could build your platform. Should you want to deviate from those (e.g. use Terraform instead of ARM templates), feel free to fork this project and "replace" parts of the code with your own.

## Module Composition

Each module represents a pillar for your cloud platform foundatations (also sometimes referred as the __Enterprise Cloud Control Plane__). The intention is to consolidate both recommended practices and proven design patterns and "serve" them in form of a **structured guide** that you can use to make important design decisions based on your business, technical, security & compliance, and management & operational requirements. Those decisions will be recorded in form of a code and used as an input to the tool to "build out" that particular pillar.

## Key Design Areas / Modules

1. Subscription organization and governance
2. Landing zones
3. Network
4. Shared services
5. IAM
6. Policy management
7. Platform management and monitoring
8. Automation

A. Platform Architecture - Compute, Storage, Network, Shared Services, Regions / AZs, Capacity Management
B. Security and Compliance - IAM, Encryption and Secret Management, Policies, Security monitoring and auditing, Platform-level security, Compliance management
C. Ops (management and maintenance) - subscription organization and governance, App Management and Monitoring, Platform Management and Monitoring, BCDR, ITSM, Cost Management
D. App Architecture - App Archetypes, App Migration Paths, Landing Zones, App Platform and Integrations, Data Platform

## Key Architecture Principles

1. Agility - delegate control to subscription/business owners and build measures for compliance and control, instead of doing centralized provisioning or building "overlays".
2. Azure Native - use declarative and idempotent tools, instead of building "abstractions" like provisioning portals with a limited set of options (service catalog). Provide full access to Azure APIs to the users.
3. Avoid microsegmentation - provide your customers with entire subscriptions, rather than slicing the environments into several Resource Groups and using them as administrative boundary. Use subscriptions as "scale units". This will help you overcoming various constraints and limitations.
4. Scalable design - your environment will start small, but it will (most likely) grow over time. Your platform must be designed to scale and grow accordingly. Using the right design patterns for e.g. your overall hierarchy (Management Groups, subscriptions), network architecture, operations.
5. Platform as Code - everything you do in Azure should be done in form of reproducible code that uses declarative semantics and is idempotent. CEBA honors this principle to full extend (exceptions from this rule are clearly documented with explanation). Everything from the creation and lifecycle management of subscriptions, management groups, roles (definitions and assignments), policies (definitions and assignments)
6. Policy driven platform - use Azure native constructs and control plane capabilities, instead of building an overlay.

## Main sources of recommendations

- assessment (MRIO)
- Az DevOps Security Kit
- CIS
- ++

## Concept of Landing Zones

A Landing Zone is essentialy a __subscription class__ (type) that is provisioned for a particular customer based on their requirements and needs. Before your Landing Zones can be delivered in some shape and form, they need to be designed, so they can delivery necessary "plumbing" to your workloads you intend to run there (either by building them from scratch or migrating them from on-premises). Each applicatio archetype will most likely have different requirements, so analyzing what archetypes you have (today and in future) is an important pre-requisite.

The requirements will be translated to a __blueprint__ that will represent all common resources (components) that will deliver that "plumbing" to customers' workloads and applications. Examples of components or artifacts could be: networking components, monitoring workspace, role and policy assignments.

## TODO & Links

- [Azure Management](https://docs.microsoft.com/en-us/azure/governance/azure-management)
- [CAF Journey](https://azure.microsoft.com/en-us/cloud-adoption-framework/#cloud-adoption-journey)
- [CAF Landing page](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/)
- [CAF Ready](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/)
