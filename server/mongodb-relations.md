# MongoDB Relations Guide

## Overview

Enfyra supports relational data modeling in MongoDB through application-level logic. Relations are automatically synchronized when you create or update data.

## Relation Types

| Type | Description | Example |
|------|-------------|---------|
| **Many-to-One (M2O)** | Record references one parent | File → Folder |
| **One-to-One (O2O)** | Record references one other record | User → Profile |
| **One-to-Many (O2M)** | Record references many children | Folder → Files |
| **Many-to-Many (M2M)** | Record references many, bidirectional | Posts ↔ Tags |

## Inverse Relations

When you add an `inversePropertyName` to a relation, Enfyra automatically maintains both sides:

**Example:**
```
File has relation "folder" → Folder
Folder has inverse "files" → [File1, File2, ...]
```

When you:
- **Create** a file with folder → Folder's `files` array auto-updated ✅
- **Update** file's folder → Old folder's `files` removes file, new folder's `files` adds file ✅
- **Delete** a file → Folder's `files` removes file ✅

## ⚠️ IMPORTANT: Metadata Changes Behavior

### When You Update or Delete Relations

**MongoDB relations are application-level, not database-level.** When you modify relation metadata (add, edit, or delete relations), Enfyra uses a **drop-and-recreate strategy**.

### What Happens

**Any change to a relation (add inverse, change field name, change type, delete relation) will:**

1. ✅ **DROP the relation field** from ALL records in the source table
2. ✅ **DROP the inverse field** from ALL records in the target table (if inverse exists)
3. ✅ Update the metadata
4. ⚠️ **Your relation data is LOST** (fields are dropped)

### Why This Approach?

**The Problem:**
- In SQL, schema changes (ALTER TABLE) keep data intact
- In MongoDB, there's **no schema enforcement** at the database level
- Existing data doesn't "know" about metadata changes
- Old data + new metadata = inconsistent state

**The Solution:**
- **Drop all relation fields** when metadata changes
- Guarantees **100% consistency** with new metadata
- Simple, reliable, no complex migration logic

**Trade-off:**
- ⚠️ **Temporary data loss** for relation fields
- ✅ **Guaranteed consistency** - no mismatched data
- ✅ **Clean slate** for new schema

### Re-populating Data After Metadata Changes

After updating relation metadata, you need to re-populate the data through the API. The automatic inverse sync will work correctly with the new metadata.

**Example workflow:**
1. Update relation metadata (add `inversePropertyName`)
2. ⚠️ All relation fields dropped
3. Update your records via API with new relations
4. ✅ Both sides sync automatically

## Best Practices

### 1. Plan Relations Before Adding Data
Define all relations and inverse properties **before** adding significant amounts of data. This avoids the need to re-populate later.

### 2. Test in Development First
Always test relation metadata changes in a development environment before applying to production data.

### 3. Backup Before Major Changes
Export your data before making significant relation metadata changes, especially if you have large datasets.

### 4. O2M Requires Inverse
One-to-many relations **must have** `inversePropertyName` defined for proper synchronization.

### 5. Use Inverse Wisely
- **With inverse**: Two-way navigation, automatic sync, but metadata changes drop all data
- **Without inverse**: One-way navigation, simpler, metadata changes only affect source table

## Common Scenarios

### Adding Inverse to Existing Relation

**Scenario:** You have a M2O relation without inverse, now you want to add it.

**What happens:**
```
1. Before: File has "folder" field with data
2. Update metadata: Add inversePropertyName "files"
3. Enfyra drops: "folder" field from files, "files" field from folders
4. You need to: Re-populate via API
5. After re-populate: Both sides synced ✅
```

### Changing Relation Field Name

**Scenario:** You want to rename "folder" to "parent".

**What happens:**
```
1. Before: File has "folder" field with data
2. Update metadata: Change propertyName to "parent"
3. Enfyra drops: "folder" field, "files" field (if inverse exists)
4. You need to: Re-populate via API using new field name "parent"
5. After: Data uses new field name ✅
```

### Deleting a Relation

**Scenario:** You no longer need a relation.

**What happens:**
```
1. Delete relation from metadata
2. Enfyra drops: Relation field + inverse field from all records
3. Done: Clean tables, no orphaned data ✅
```

## Summary

**Key Points:**
- ✅ Relations auto-sync when you work with data (CREATE/UPDATE/DELETE)
- ⚠️ **Metadata changes DROP all relation data** (by design)
- ✅ This ensures consistency between metadata and data
- 💡 Plan your schema carefully to minimize re-population work

**Remember:** Unlike SQL, MongoDB doesn't have schema constraints. Enfyra's drop-and-recreate approach is the safest way to ensure your data matches your metadata.
