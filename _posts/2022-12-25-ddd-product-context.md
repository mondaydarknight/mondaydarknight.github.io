---
layout: post
title: Design EC product with DDD
description: Design ECommerce product functionality involves DDD architectural pattern
location: Taiwan
tags:
- DDD
- EC
---

## Design EC product with DDD

### Summary
We recently had a bottleneck about product functionality with complicated business logic and large technical debt, apply DDD architectural pattern to divide problems and implement appropriate technical strategies.

![product_class_diagram-separate-bounded-context](https://user-images.githubusercontent.com/32161174/209455486-678b1a12-e0b1-4b86-ab09-7b4e707ef158.png)


### Terminology
[`Catalog`](https://blog.miva.com/digital-catalog-ecommerce) showcases all the products you sell and allows buyers to shop and purchase. Catalog is used for managing item data that is an attribute information of a product for sellers e.g. brand name,  color, material, categories, etc.

[`Inventory`](https://www.investopedia.com/terms/i/inventory.asp) refers to the raw materials used in production as well as the goods produced that are available for sale.

### Bounded context
Define a product aggregate root within the inventory context.

![product-bounded context drawio](https://user-images.githubusercontent.com/32161174/209455484-944c0b17-62bf-48b2-849f-e98dd34bd06f.png)

Pros:
- Dramastically reduce complexities and overheads of cross-context interaction e.g. context map, event-driven communication, act inventory domain act as entrypoint to build item lifecycle.
- Avoid premature decomposition of aggregates can lead to expensive refactoring.

Cons:
- some fields like keywords, highlights for catalog information could make the intension ambiguous.

Issues
- Is there any benefits to separate offer informations e.g. price, description, highlights into other bounded context?
  According to [DDD aggregate principles](https://www.alibabacloud.com/blog/an-in-depth-understanding-of-aggregation-in-domain-driven-design_598034), Consistency of lifecycle (means domain model should be unified, internally consistent) and consider the following issues.
  - Should the lifecycle refers to the objects within the aggregation boundary that have a personal dependency relationship with the aggregate root (means objects lifecycle creation/update/destroy within the context)
  - Is an object remain meaningful after the aggregate root disappear
  - Consistency of the problem domain
  - Is constraint of the bounded context within the same bounded context
- How to interact contexts between catalog and inventory?
  It depends on whether the catalogue product info should be within aggregate root of inventory context, if we’re going to separate to different contexts, they might result in the following problems.
  - How to deal with transaction to make consistency? 
    For example, Save item stock (quantity, price) in inventory transaction, handle product info in catalog transaction.
  - How to handle make sure the consistency between tow contexts when either of one failed?
    - Saga pattern (distributed system solution) → too complex, unrealistic
    - How to map/relate data? (since they both require the same identity) -> Add a fake ID (Hack)
      - Hard to debug, troubleshooting, side-effects across bounded context, besides ID is highly-coupling cross context.
      - Increase the complexity of catalog info, inventory item stock.
      - Worse to scale up item service

Above solutions would increase complexities of technique and reduce the maintainability (it’s hard to make sure consistency between different contexts) just for divide the responsibilities of inventory, catalogue contexts. Seems the product just a context that contain catalog info, details (qty, pricing, SKU), Keywords for recommendation.

[Sometimes, bounded context might both contain unrelated concepts and shared concepts for integration.](https://www.martinfowler.com/bliki/BoundedContext.html)

Make the comparison between contexts
1. Coupling bounded context

![product_class_diagram-inventory_context drawio (1)](https://user-images.githubusercontent.com/32161174/209455641-007d5dd7-fa4b-4150-90b8-45c94fc5a2ad.png)
Advantages:
- Can divide fields into different contexts
Downside
- ID corruption and mismatched within contexts
  - Different contexts mean they have their corresponding lifecycles.
    e.g. Catalog Item, inventory item requires the same identity, so they cannot create directly.
    Def: Aggregate root always has its own unique identity    
- Confused contexts to catalog, fulfillment
  Vital info: ID ASIN, Title, variant options (size, color, attributes), Qty, Barcode(UP)
- Invariant inconsistency due to `partial update`  
  Aggregate root doesn’t enforce invariants (e.g. image optional)
  Result in an defect of cleaning up the field for existing products (Requires some hack ways to handle)

2. Separate contexts

![product_class_diagram-upstream_downstream_item_bounded_context drawio](https://user-images.githubusercontent.com/32161174/209455794-8e12f7f5-4fe3-4d8e-9909-9c0f61fdf2df.png)
Advantages:
- Enforce consistency for product/item lifecycle (creation → update → archive)
- Centralized fields management
- Efficiently delegate contexts between catalog, fulfillment
- Single source of truth
Downside:
- Requires a team to maintain functionalities of product management.
- How’s team collaboration for product context?
  - Relevant engineer maintain context thoroughly (inventory → catalog)
  - Upstream, downstream collaboration
- If we’re going to use microservice, how to interact between catalog and product context?
  - CQRS pattern (implementation details)
  - OHS (Open-host service): Provide a REST API to communicate product information.
- Is pricing within product model context?
  It conforms the 1, 2 aggregate root principles, so the ideal way is within the product boundary context.
- What’s the relationship between Item, Curation, Groupbuy?
  According the previous answer, if pricing is within the product context, other resources e.g. curation, groupbuy cannot access pricing directly, they must access an aggregate root of product context to modify pricing info to keep consistent.
- What if we have additional requirements that want to modify a part of product info e.g. pricing, quantity?
  Declare an additional aggregate root within the bounded context, a bounded context can have more than one aggregate.
- Can a single bounded context covers multiple subdomains?
  
  It would increase the risk of terms overloading since multiple subdomains would be expressed through the same ubiquitous language (UL). 
  Sub-domain is problem space and bounded context is solution space, but in general case they should be 1:1. For legacy code it is ok to have more than one bounded context per sub-domain, but NOT the other way around - one bounded context should not cover more than one sub-domain.
  
  When the BC models more than one (sub)domain with overlapping concepts it would be wise to consider splitting that BC. A shared kernel could be used to deduplicate the overlap that is equal in concept in both (sub)domains.
  It may also be that a single BC contains multiple sub-domains. This is usually not ideal because it likely indicates a conflated BC.
  
  A subdomain is a problem of a larger domain and a bounded context is the solution space where that problem will get solved in practice. You should strive to have a 1 to 1 alignment between subdomains and bounded contexts, just like you would ideally have a 1 to 1 alignment between questions and answers.
  
  There are more than one global goals, more than one motivation for change that result in inconsistency state within each subdomain, decomposing your domain along nouns and putting all stuff related somehow to that noun **into its own context, Cyrille Martraire stresses the importance of extracting bounded contexts based on their responsibilities and behavior. As a result, these contexts most probably would have different evolutionary forces and different motivation for change.
  - [Bounded contexts and subdomains](https://stackoverflow.com/questions/18625576/confused-about-bounded-contexts-and-subdomains)
  - [Inventory, pricing, catalog communication](https://softwareengineering.stackexchange.com/questions/368699/inventory-pricing-and-product-catalog-communication)
  - [Domain Driven Design and Cross Domain interaction](https://softwareengineering.stackexchange.com/questions/308946/domain-driven-design-and-cross-domain-interaction)
  - [Orders&Inventory ddd where should allocation reservation](https://stackoverflow.com/questions/53108190/orders-inventory-ddd-where-should-allocation-reservation-be-handled)
  - [DDD UI composition across multiple context on creation](https://stackoverflow.com/questions/66510146/ddd-ui-composition-across-multiple-contexts-on-creation/66596061#66596061)
  - [DDD UI composition across multiple contexts](https://stackoverflow.com/questions/66510146/ddd-ui-composition-across-multiple-contexts-on-creation/66596061#66596061)
  - [DDD strategy pattern: how to define bounded context](https://codeburst.io/ddd-strategic-patterns-how-to-define-bounded-contexts-2dc70927976e)
- Instead, divide into bounded contexts into subdomains to make responsibilities

![product_class_diagram-subdomains drawio](https://user-images.githubusercontent.com/32161174/209455768-edd4aac0-e61f-4feb-840c-1410a3ed08f4.png)

- Bounded contexts and subdomains ~ Robert Basic, software developer making web applications better
  - Subdomains and bounded contexts go hand in hand and I think one can’t be understood without the other. The optimal solution would be to have one bounded context in one subdomain. The world is not a perfect place, software even less so, so it might happen that one bounded context spans multiple subdomains, or that one subdomain has multiple bounded contexts.
- Sub-domains and bounded contexts in Domain\-Driven Design \(DDD\) \- Lev Gorodinski
- Should UI composition across multiple subdomains (catalog , inventory)?
  If we’ve considered that catalog and inventory should have their own responsibilities, it’s required to integrate together on the application UI, or dividing into separate UI pages. Make sure you’d find the main owner of the item ( in the case is catalog sub-domain), don’t waste to build a ambiguous relationship between contexts.
