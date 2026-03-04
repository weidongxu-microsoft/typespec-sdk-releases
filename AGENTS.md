# Guide

## Checklist

- Do not create PR, unless specified in this the guide.
- When create PR, always create draft PR.

## Database

The database is the "releases.csv" file.

## Required repositories

- [Azure/azure-sdk-for-java](https://github.com/Azure/azure-sdk-for-java), cloned at "../azure-sdk-for-java"
  Call this folder "sdk repo" for short.
- [Azure/azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs), cloned at "../azure-rest-api-specs"
  Call this folder "specs repo" for short.

## Terminal state

"AlreadyTypeSpec", "Done", "Premium", "NO_SPEC" in "Java" column is a terminal state. Skip process on these rows.

- AlreadyTypeSpec
  Java SDK is already generated and released from TypeSpec. We don't need to release again.
- Done
  We've validate the SDK PR from TypeSpec ("SdkPr" column). If both api-version is same, we've also released the SDK.
- Premium
  This is hand-written lib, we have a different agent to process them.
- NO_SPEC
  No TypeSpec source found. Nothing we can do.

<!-- ## Guide on task "Update SDK state"

For each row (not in terminal state) in database, check folder "sdk/<SpecFolder>/azure-resourcemanager-##" in sdk repo. If there is a "tsp-location.yaml" in the folder, update the "Java" column to "AlreadyTypeSpec". -->

## Guide on task "Update tspconfig column"

Add a "tspconfig" column in database, if not exist.

For each row (not in terminal state) in database, check folder "specification/<SpecFolder>/**/*.Management/**" or "specification/<SpecFolder>/resource-manager/**" in specs repo, find the "tspconfig.yaml"

Add the relative path (from specs repo) to the "tspconfig" column.

List the rows that cannot find the "tspconfig.yaml".

## Guide on task "Update api-version on specs"

Add a "SpecApiVersion" column in database, if not exist.

For each row (not in terminal state) in database, check folder of "tspconfig" column in specs repo.

Find the last item in `Version` enum. Add it to the "SpecApiVersion" column.

The `Version` enum can be found as the enum specified in `@versioned` decorator. This usually in "main.tsp" file.

```ts
@versioned(Versions)
namespace Microsoft.NetApp;

enum Versions {
  ...
}
```

## Guide on task "Update api-version on sdks"

Add a "SdkApiVersion" column in database, if not exist.

For each row (not in terminal state) in database, check folder "sdk/<SpecFolder>/azure-resourcemanager-##" in sdk repo.

If you cannot find the folder of "sdk/<SpecFolder>", read the "tspconfig.yaml" in "tspconfig" column, check the `options/"@azure-tools/typespec-autorest"/emitter-output-dir` property. The folder in sdk repo can be inferred from it.

The "README.md" file should contain word like `Package tag package-2024-05`. Note this "package-2024-05" as "<readme-tag>".

Find the "input-file" from this "<readme-tag>" in the "README.md" in specs repo. (the "README.md" in specs repo would be at the same folder of the "tspconfig" column, or in "specification/<SpecFolder>/resource-manager/**" folder).

The list of "input-file" would be in a form of `stable/2024-05-01/netapp.json`. The api-version should be consistent in all these files. Add it to the "SdkApiVersion" column.
If api-version is not consistent, add "inconsistent" to the column.

List the rows that cannot find the "SdkApiVersion".

## Guide on task "Generate SDK <sdk>"

For the row specified:

1. Run pipeline https://dev.azure.com/azure-sdk/internal/_build?definitionId=7421 via REST API
   - Set "Path to API specification file" as value in "tspconfig" column
   - Set "API version" in "SpecApiVersion" column
   - Set "SDK release type" as beta/stable, depends on whether "SpecApiVersion" contains "-preview"
   - Set "Create SDK pull request" to "true"
   Use the token from Azure CLI to call the REST API of the "dev.azure.com" endpoint (preferrable using `az rest` and let Azure CLI handle the token)
2. Wait for the pipeline run to complete
3. Check recent PR on https://github.com/Azure/azure-sdk-for-java/pulls, find "[AutoPR <sdk-package>]*", approve the PR, and open it in browser
4. Add the link of PR to "SdkPr" column
5. Set "Done" to "Java" column

## Guide on task "Release SDK <sdk>"

For the row specified:

1. Find pipeline of "java - <service>" pipeline from https://dev.azure.com/azure-sdk/internal/_build ("<service> is extracted from "SdkFolder" column -- `sdk/<service>/<sdk-package>`)
2. Run the pipeline
   - If the pipeline has "templateParameters", set the parameter of "release_<sdk-package>" to "true", all other parameters to "false"
   - If the pipeline has no "templateParameters", just run it without parameters
3. Open the pipeline run in browser
4. Wait for the pipeline run to complete
5. Check recent PR on https://github.com/Azure/azure-sdk-for-java/pulls, find "Increment versions for <service>", approve the PR, and open it in browser
6. Wait for all CI checks on the PR to pass, then merge the PR.
