---
name: cosmos-db
description: Create and configure Azure Cosmos DB. Use when working with Cosmos DB, NoSQL on Azure, document databases, partition keys, RU/s, or when the user asks about Cosmos DB API (NoSQL, MongoDB, Gremlin) or change feed.
---

# Cosmos DB Skill

## Quick Start

Cosmos DB is a globally distributed, multi-model database. APIs: NoSQL (SQL), MongoDB, Gremlin (graph), Cassandra, Table.

## Bicep: Cosmos DB Account + Database + Container

```bicep
param accountName string
param databaseName string
param containerName string
param partitionKey string = '/id'

resource account 'Microsoft.DocumentDB/databaseAccounts@2023-11-15' = {
  name: accountName
  location: resourceGroup().location
  kind: 'GlobalDocumentDB'
  properties: {
    databaseAccountOfferType: 'Standard'
    consistencyPolicy: {
      defaultConsistencyLevel: 'Session'
    }
    locations: [
      { locationName: resourceGroup().location, failoverPriority: 0 }
    ]
  }
}

resource database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2023-11-15' = {
  parent: account
  name: databaseName
  properties: {
    resource: {
      id: databaseName
    }
  }
}

resource container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2023-11-15' = {
  parent: database
  name: containerName
  properties: {
    resource: {
      id: containerName
      partitionKey: {
        paths: [partitionKey]
        kind: 'Hash'
      }
      throughput: 400  // or use autoscale
    }
  }
}
```

## Naming

- Account: `cosmos-<name>-<env>` (3-44 chars, lowercase)
- Database/container: per API; NoSQL uses SQL-style names

## Partition Key

- Choose a property with high cardinality and even distribution
- Common: `/id`, `/userId`, `/tenantId`, `/category`
- Avoid low-cardinality keys (e.g., boolean)

## Throughput

- **Manual**: 400 RU/s minimum per container
- **Autoscale**: 1000-1000000 RU/s; scales between 10% and 100% of max
- **Serverless**: Pay per request; no RU provisioning

## Best Practices

1. **Design partition key** before creating containers; changing it requires migration
2. **Use bulk operations** (SDK bulk API) for many documents
3. **Consider change feed** for event sourcing, ETL, or sync
4. **Use managed identity** for app-to-Cosmos auth when possible
