---
layout: post
title: DDD fundamentals
description: Introduce the fundamentals of Domain-driven design (DDD) and benefits of using DDD, learn what DDD can do for your projects.
location: Taiwan
tags:
- DDD
---

## DDD fundamentals

### What is DDD?
DDD (Domain-Driven Design) is a software development approach to help succeed at achieving high-quality software model design, it gives a way to gain deep insight into the business domain, with the prospect of carefully crafted sofware models.
DDD provides the software techniques that address strategic and tactical design, stragegic design helps us understand what are the most important software investments to make; Tactical design helps us craft a single elegant model of a solution using time-tested, proven software building blocks.

### Terminology
**Ubiquitous language**: A language of a model such that each term is unambiguous and no rules contradict.

**Domain**: A sphere of knowledge, influence or activitly around which the application logic resolves.

**Subdomain**: A subset of the domain divide the entire complexity of the company's domain into smaller parts such as Catalog, CDP (Customer Data Platform) and so on.

**Bounded context**: is simply boundary within a domain where a particular domain models applies.

**Context map**: it's a map to relate different bounded context together to identity the relationship and translation of data access from one model to the other subsystem. Types of context maps:
- **Partnership**: The team have a mutual dependency on each other for delivery.
- **Shared Kernel**: Both referenced and owned by multiple bounded contexts, each team is free to modify the compiled library that defines the integration contract.
- **Customer-supplier**: Upstream and downstraem relationship.
- **Conformist**: The downstream team can accept the upstream team's model.
- **Open host service (OHS) / Published language (PL)**: The upstream supplier decouples its implementation model from the public interface to protect the customers from changes.
- **Anticorruption-layer (ACL)**: Downstream bounded context is not willing to conform the upstream, instead translate the upstream bounded context's model into a model.
- **Separate ways**: not collaborate at all.

**Repository**: A mechanism for encapsulating storage, retrieval and search behavior which emulates a collection of objects.

**Factory**: A mechanism for encapsulating complex creation logic and obstracting teh type of a created object for the sake of a client.
