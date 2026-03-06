# Guide

## Checklist

- Do not create PR unless specified in this guide.
- When creating PR, always create draft PR.
- When asked to "generate sdk via pipeline", it means run the pipeline (see task `Guide on task "Generate SDK {sdk} via pipeline"`), not generate or build it on local.

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

For each row (not in terminal state) in database, check folder "sdk/{SpecFolder}/azure-resourcemanager-##" in sdk repo. If there is a "tsp-location.yaml" in the folder, update the "Java" column to "AlreadyTypeSpec". -->

## Guide on task "Update tspconfig column"

Add a "tspconfig" column in database, if not exist.

For each row (not in terminal state) in database, check folder "specification/{SpecFolder}/**/*.Management/**" or "specification/{SpecFolder}/resource-manager/**" in specs repo, find the "tspconfig.yaml"

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

For each row (not in terminal state) in database, check folder "sdk/{SpecFolder}/azure-resourcemanager-##" in sdk repo.

If you cannot find the folder of "sdk/{SpecFolder}", read the "tspconfig.yaml" in "tspconfig" column, check the `options/"@azure-tools/typespec-autorest"/emitter-output-dir` property. The folder in sdk repo can be inferred from it.

The "README.md" file should contain word like `Package tag package-2024-05`. Note this "package-2024-05" as "{readme-tag}".

Find the "input-file" from this "{readme-tag}" in the "README.md" in specs repo. (the "README.md" in specs repo would be at the same folder of the "tspconfig" column, or in "specification/{SpecFolder}/resource-manager/**" folder).

The list of "input-file" would be in a form of `stable/2024-05-01/netapp.json`. The api-version should be consistent in all these files. Add it to the "SdkApiVersion" column.
If api-version is not consistent, add "inconsistent" to the column.

List the rows that cannot find the "SdkApiVersion".

## Guide on task "Generate SDK {sdk} via pipeline"

If the ask contains "from specs PR {specs-pull-request}", get the HEAD SHA on the PR as "commit-sha".

If the ask contains "to sdk PR {sdk-pull-request}", get the branch on PR (should be `sdkauto/azure-resourcemanager-xxx`) as "target-sdk-repo-branch".

For the row specified:

1. Run pipeline https://dev.azure.com/azure-sdk/internal/_build?definitionId=7421 via REST API
   - Set "Path to API specification file" as value in "tspconfig" column
   - Set "API version" in "SpecApiVersion" column
   - Set "SDK release type" as beta/stable, depends on whether "SpecApiVersion" contains "-preview"
   - Set "Create SDK pull request" to "true"
   - When specified, set "SDK repository branch" to "target-sdk-repo-branch" (that of sdk PR)
   - When specified, set "commit" as "commit-sha" (that of specs PR)
  Use the token from Azure CLI to call the REST API of the "dev.azure.com" endpoint (preferably using `az rest` and let Azure CLI handle the token, with `Content-Type=application/json` via `--header`)
2. Wait for the pipeline run to complete
3. Check recent PR on https://github.com/Azure/azure-sdk-for-java/pulls, find "[AutoPR {sdk-package}]*", approve the PR, and open it in browser
4. Add the link of PR to "SdkPr" column
5. Set "Validating" to "Java" column

## Guide on task "Release SDK {sdk}"

For the row specified:

1. Find pipeline of "java - {service}" pipeline from https://dev.azure.com/azure-sdk/internal/_build ("{service} is extracted from "SdkFolder" column -- `sdk/{service}/{sdk-package}`)
2. Run the pipeline
  - If the pipeline has "templateParameters", set the parameter of "release_{sdk-package}" to "true", all other parameters to "false"
   - If the pipeline has no "templateParameters", just run it without parameters
3. Open the pipeline run in browser
4. Wait for the pipeline run to complete
5. Check recent PR on https://github.com/Azure/azure-sdk-for-java/pulls, find "Increment versions for {service}", approve the PR, and open it in browser
6. Wait for all CI checks on the PR to pass, then merge the PR.

## Guide on task "Validate CHANGELOG {pullrequest}"

Read CHANGELOG.md from the pull request. Check the change with [Instructions to mitigate breaks](https://github.com/weidongxu-microsoft/java-sdk-tools/blob/main/mcp/assets/migrate-instructions.md)

See if there is more mitigation to be done on the PR.

If no, set the "Java" column to "Done".

## Guide on task "Mitigate breaking changes for {service}"

- DO NOT use MCP tool from azure-mcp-mcp
- Prefer to use MCP tool from java-sdk-tools

All of below is executed on specs repo.

1. Find the service folder under "specification/{service}".
2. Find the "tspconfig.yaml", should be under a "resource-manager" sub-folder, or a "{Service}.Management" sub-folder.
3. Create a "mgmt_java_mitigate-typespec-{service}" branch from local "main" branch, on specs repo. (If this branch already exists in local, just switch to it)
4. Set working directory of the folder of "tspconfig.yaml"
5. Apply "mitigateMigrationTypeSpec" MCP tool
6. When done, only commit "tspconfig.yaml" and .tsp files
7. Create a draft PR, open in browser
8. Add label "PublishToCustomers" and "ARMSignedOff" to the draft PR

- When generate SDK fails with "duplicate-client-name" error, use `@@clientName({model}, "{deduplicated-model-name}", "java")` to rename a model.
