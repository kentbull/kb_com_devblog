+++
draft = true
title = "Expose Database Schema through ReST API – Good Design or Anti-pattern?"
slug = "database-schema-rest-api"
date = "2026-01-04"

[taxonomies]
tags=["api-design", "database", "rest", "architecture"]

[extra]
comment = true
+++

# Expose Database Schema through ReST API – Good Design or Anti-pattern?


## Tradeoffs

### Benefits

#### Initial speed

### Disadvantages

#### Tight coupling between the API contract and the database model

Breaks the dependency inversion principle, “D” in [SOLID](https://en.wikipedia.org/wiki/SOLID) since the high-level API contract is bound to the concrete database schema. This is also known as the concept of encapsulation.

A database schema reflects the specific design decisions oriented around how data is stored in a specific datastore. This is a very different concern than the concern of how clients or other consumers interact with a REST API.

The ability to independently change each layer without worrying about the other layer is a major architectural benefit. Such a design enables each to change separately to meet the needs of each specific layer and will work as long as the interface of the REST API layer is met. The underlying implementation details of the data model should be hidden from the REST API consumer rather than being exposed to them. For example, the REST API consumer should never have to know that you use [PostgreSQL](https://www.postgresql.org/) or [MySQL](https://www.mysql.com/) in your database layer, or any other technology such as [MongoDB](https://www.mongodb.com/), [LMDB](http://www.lmdb.tech/doc/), [Datomic](https://www.datomic.com/), [Cassandra](https://cassandra.apache.org/_/index.html), or [Oracle](https://www.oracle.com/database/technologies/).

Similarly, you do not want the concerns of the REST API to pollute the database entities in the domain layer of the application. The primary concerns of a REST API are facilitating the user experience, or UX of either web user interfaces or service contracts to other API services which often may use aggregations of data and derived fields. These aggregations of data can come from multiple database tables. The derived fields may not exist in a database table or view

The API becomes driven by a technical data model, the underlying database schema, rather than the business purpose. Every change in the data model will require a change in the API. Change relations in SQL and you have to change the API as well.

Everything is not reducible down to CRUD, GET POST PUT DELETE.

Encourages the Anemic Domain Model anti-pattern

ORM is a leaky abstraction

This constrains the request and response bodies, by default to be domain objects.

Using SQL isn’t that hard

### Data Type Optimizations

### Entities are not business events

Operations supported by an API should be business-domain driven. The business domain does not always have a one-to-one correspondence with classes or database tables. Database tables do not have a one-to-one correspondence with business events. Sometimes one business event will cause the creation of a number of records in a normalized, relational database schema.

### Long Term Maintenance Costs

Every data attribute exposed through the REST API becomes a part of the API contract for that API service and requires long-term maintenance. Usually, if a data attribute is exposed, some consumer or client will come to depend on that data attribute so that

## Practical Comparison – FastAPI versus Falcon

### FastAPI

- Encourages coupling the database domain entities to the ReST API request and response bodies.

See the [SQL Databases](https://fastapi.tiangolo.com/tutorial/sql-databases/) section of the docs.

### Falcon

## Principles

- Single Responsibility Principle
- Open-Closed Principle: open for extension, closed for modification
- Liskov-substitution principle: subclasses should be substitutable for superclasses. Design by Contract.
- Interface Segregation Principle: client should never be forced to implement an interface it doesn’t use, or clients shouldn’t be forced to depend on methods they do not use.
- Dependency Inversion Principle: Entities must depend on abstractions, not on concretions. The high level module must not depend on the low level module, rather on abstractions.

Low Coupling, High Cohesion

References

- [Web API Design Anti-Pattern: Exposing your database model](https://shekhargulati.com/2021/10/15/web-api-design-anti-pattern-exposing-your-database-model/)
- [The CRUD Anti-Pattern in REST APIs](https://codeboje.de/the-crud-anti-pattern-in-rest-apis/)
- [SOLID: The First 5 Principles of Object Oriented Design](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
- [CQRS – Martin Fowler](https://martinfowler.com/bliki/CQRS.html)
- [REST anti-patterns](https://marcelocure.medium.com/rest-anti-patterns-b128597f5430)

