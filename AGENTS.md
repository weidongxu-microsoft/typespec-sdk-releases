# Guide

## Database

The database is the "releases.csv" file.

## Required repositories

- [Azure/azure-sdk-for-java](https://github.com/Azure/azure-sdk-for-java), cloned at "../azure-sdk-for-java"
  Call this folder "sdk repo" for short.
- [Azure/azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs), cloned at "../azure-rest-api-specs"
  Call this folder "specs repo" for short.

## Terminal state

"Done" and "Premium" in "Java" column is a terminal state. Skip process on these rows.

## Guide on task "Update SDK state"

For each row (not in terminal state) in database, check folder "sdk/<SpecFolder>/azure-resourcemanager-##". If there is a "tsp-location.yaml" in the folder, update the "Java" column to "Done".
