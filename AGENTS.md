# Guide

## Database

The database is the "releases.csv" file.

## Required repositories

- [Azure/azure-sdk-for-java](https://github.com/Azure/azure-sdk-for-java), cloned at "../azure-sdk-for-java"
  Call this folder "sdk repo" for short.
- [Azure/azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs), cloned at "../azure-rest-api-specs"
  Call this folder "specs repo" for short.

## Guide on task "Update SDK state"

For each row, check folder "sdk/<SpecFolder>/azure-resourcemanager-##". If there is a "tsp-location.yaml" in the folder, update the "Java" column to "Done".
