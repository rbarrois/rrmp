=====================================
Remote Resource Manipulation Protocol
=====================================

Introduction
============

Context
-------

With the evolution of cloud services, applications are increasingly designed as
set of clients interacting concurrently with the same resources.

Each client needs to retrieve a specific set of resources, act on them, and be
notified when they get modified by another action.


Goal
----

The Remote Resource Manipulation Protocol intends to provide a robust structure
on top of which application-specific protocols may be built. Its core features
are:

- Strongly typed description of resources and their supported actions;
- Update notifications;
- Resource actions as a first-class citizen;
- Optimistic concurrency control (or allow MVCC, pessimistic. etc.? FIXME);
- Built-in authentication and authorization primitives;
- Recover from short or long disconnections.


Inspiration
-----------

* ReST
* IMAP
* GraphQL

Questions
---------

* Concurrency
* Locales / translation: client, server? META?

High level
==========

Headers
-------

Each request contains a set of headers:
- Accepted content-types
- Expected locale
- Authentication
- ?


Read
----

A client may send a 'read' query. Those queries MUST contain:

- The target resource type;
- A set of filters for the resource;
- A cursor limit for results;
- The list of fields to fetch.

The server returns the list of resources, including allowed actions for each one, and their etag.

.. XXX group actions? Compression? Performance?
.. XXX Actual structure is a batch, including several subqueries.
.. XXX Streaming fetches?


Action
------

A client may send a 'DO' query. Those queries MUST contain:

- The target resource ID;
- The selected action;
- The parameters for the action;
- The resource etag used for the action.

.. XXX partial etag (e.g. I don't care about changes on fields x,y,z)?
.. XXX batch actions, transactions

The server returns the updated resource, or an error code (standardized).


Schema
------

Fetch the schema, for one, some or all resources.
.. XXX Schema is a resource itself? With standardized fields, to get part of resources: name/title/fields/descriptions/translations/...?
Include actions, and their parameters (parameter name, type, requirements, human description)

.. XXX Maybe include translations for enums and fields and actions in the schema?
.. XXX Custom types / checkers?


Subscription
------------

- Send filters for target + fields
- Standard format for add, update, delete
- Send a combined 'subscribe + read' query, incl. pagination

.. XXX Use bi-directionnal channel for updates? Or separated?
.. XXX Disconnection/reconnection? Timestamp? Heartbeat?



Examples
========

Simple read
-----------

1. The client connects to the server, through a TLS channel;
2. The client requests a set of resources from the server, based on a few criteria,
   identifying requested fields.
   That request might be authenticated.
3. The server checks credentials, and provides the requested data.
4. The client disconnects.


Auto-discovery
--------------

1. The client connects to the server, through a TLS channel;
2. The client requests the schema from the server. That request might be authenticated.
3. The server returns the subset of the schema allowed for the client criteria.
4. The client requests resources, including allowed actions.
5. Based on that data, the client might build forms to handle those resources.
