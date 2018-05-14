---
title: Cloud-Native Schema Migrations
description: A manifesto on headache-free database schema migrations, working well for cloud-native environments and continuous deployment.
version: 0.1
updated: 2018-05-04
layout: default
---

## Introduction

We felt a need to write this document because, in our opinion, the best practices for database schema migrations have evolved in a direction that isn't well suited to cloud-native concepts.

The idea of keeping the database schema and application code close together, and to deploy database schema changes together with application changes, seems to be accepted as best practice. Tools such as Flyway make it very easy to make schema changes, and via migration scripts to automatically upgrade your schema when you deploy the next version of your application.

That's very convenient, but the big downside is that you can't downgrade the application once you have deployed it, because you already deployed the database schema changes. Also, it isn't possible to run multiple versions of the same application using the same database, which makes A/B-deployments and canary testing impossible.

If you follow the following guidelines to do database schema migrations in production, you can be worry-free and benefit from the advantage of easy deployments that cloud-native environments are intended to give you. 

## The 4 Principles of Cloud-Native Schema Migrations

### 1. Avoid unnecessary schema changes

We feel that because database schema migration tools are so effective, developers are often doing more schema changes than necessary. Every backwards-incompatible schema change, should only be considered if it really brings an added value. Do not change your database schema because of cosmetic reasons or naming conventions. Your schema has a history, and that’s fine.

### 2. Always deploy database changes separately from application changes

When you need to deploy both new application code, and database schema changes, never do it together. There are two reasons for this: first, it is much safer to deploy a new version of your application if it's easy to rollback to the old version. Second, you should be able to run both the old and the new version of your application at the same time. This is particularly important for cloud-native environments like Kubernetes, which make A/B and/or canary deployments easy to do and a best practice. You can’t do this if your application also applies backwards-incompatible schema changes during startup.  

### 3. If needed, adapt first the current application

For those times, when the new application version needs a backwards-incompatible schema change, the way to do it is to add an extra step in between: make first your current application compatible with the new database schema. The current application should be able to work with both the old schema and the new schema, possibly auto-detecting which schema version is in use, if necessary. The deployment is then done during 3 steps done at different times:

1. Deploy a minor update to the current application, only adapted to be compatible with both old and new database schema
2. Deploy the database schema change
3. Deploy the new application

Each of these steps should be reversible. Yes, it’s more work, but it’s also a lot less painful both for the users and for yourself during maintenance windows.

### 4. Make database changes reversible

When deploying database schema changes, you should think about what to do in case of problems. You should be able to reverse the database changes as easily as possible. This could be with reverse-changing database migration scripts, or possibly just by restoring the database. Always do a full database dump before doing database schema migrations. 

## Best Practices

### Database schema migration tools

Database migration tools such as Liquidbase and Flyway are actually great to do database schema migrations. Particularly useful is the concept of keeping track of the database schema version in the database itself and of using migration scripts to transition from one to the other. Just don't deploy these tools together with your application.

### Where to store the database schema

There is really no right or wrong for this. Having the database schema definition near to your application has the advantage of keeping things together and making them easier to understand. On the other hand, this can also favor a too tight coupling and becomes a problem as soon as multiple applications start using the same data in the database. Generally, we feel that you should better keep the schema definitions separate.

### Checking the schema version

You should store the version of your schema in the database itself, and check at application startup that it is at least the version that you need. It's better to abort and produce an early failure than cause unexpected issues later.

## Questions and Answers

## Signed-by

- David Schweikert, AdNovum Informatik AG
- Your name? Please create a pull request.
