# Overview

The platform-level description of the system: what it is made of and how the pieces fit together, independent of any business domain.

| Document | Content |
|---|---|
| `architecture.md` | Layers, tenancy, process model, request path, the registry, the environment, the record set abstraction, the unit of work |
| `package-system.md` | Capability packages: manifests, dependencies, categories, installation order, data loading, auto-installation, uninstallation, package hooks |
| `entity-and-field-system.md` | Entity kinds (persistent, transient, abstract), field types and attributes, computed and related fields, dependencies and recomputation, defaults, on-change behavior, constraints, ordering, display names, active flag, company scoping, translation |
| `inheritance-and-extension.md` | Classical, prototype and delegation inheritance; extending fields, operations, views, data and security; extension points and their contracts |
| `security-model.md` | Users, groups, privileges, implied groups, access rights, record rules, field groups, superuser, multi-company, portal and public users, sudo semantics |
| `views-and-actions.md` | View types and their declarative grammar, view inheritance, window actions, server actions, client actions, address actions, report actions, menus, binding of actions to entities |
| `messaging-model.md` | Threads, messages, subtypes, followers, notifications, activities, tracking of field changes, templates |
| `design-principles.md` | The recurring design choices that give the system its behavior: everything is an entity, data-driven user interface, configuration as data, external identifiers, soft deletion, audit fields, multi-company by record, currency and precision discipline |
