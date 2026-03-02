# Guide

## Database

The database is the "releases.csv" file.

## Required repositories

- [Azure/azure-sdk-for-java](https://github.com/Azure/azure-sdk-for-java), cloned at "../azure-sdk-for-java"
  Call this folder "sdk repo" for short.
- [Azure/azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs), cloned at "../azure-rest-api-specs"
  Call this folder "specs repo" for short.

## Terminal state

"Done", "Premium", "NO_SPEC" in "Java" column is a terminal state. Skip process on these rows.

## Guide on task "Update SDK state"

For each row (not in terminal state) in database, check folder "sdk/<SpecFolder>/azure-resourcemanager-##" in sdk repo. If there is a "tsp-location.yaml" in the folder, update the "Java" column to "Done".

## Guide on task "Update tspconfig column"

Add a "tspconfig" column in database, if not exist.

For each row (not in terminal state) in database, check folder "specification/<SpecFolder>/**/*.Management/**" or "specification/<SpecFolder>/resource-manager/**" in specs repo, find the "tspconfig.yaml"

Add the relative path (from specs repo) to the "tspconfig" column.

List the rows that cannot find the "tspconfig.yaml".

## Guide on task "Update api-version on specs"

Add a "SpecApiVersion" column in database, if not exist.

For each row (not in terminal state) in database, check folder of "tspconfig" column in specs repo.

Find the last item in `Version` enum. Add it to the "SpecApiVersion" column.

The `Version` enum can be find as the enum specified in `@versioned` decorator. This usually in "main.tsp" file.

```ts
@versioned(Versions)
namespace Microsoft.NetApp;

enum Versions {
  ...
}
```
