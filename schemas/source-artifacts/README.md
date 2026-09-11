# Source artifact catalogues

How the system is packaged, and what each package contributes. See [the catalogue index](../README.md) for the record shape.

| Catalogue | Content | Records |
|---|---|---|
| [`packages.json`](packages.json) | Every capability package: full name, category, summary, dependencies, the entities it defines and extends, and counts of its contents | 620 |
| [`package-dependency-graph.json`](package-dependency-graph.json) | Dependency edges between packages, giving a valid installation order | 1,286 |
| [`chart-templates.json`](chart-templates.json) | The country chart templates and the data sets each ships | 1,078 |

A rebuild does not have to adopt this packaging. The catalogue matters because it records which capability contributes which behaviour, which is what makes a partial rebuild possible. The narrative counterpart is [the package system](../../docs/overview/package-system.md).
