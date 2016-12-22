---
layout: post
title:  "How to check if a JIRA upgrade task ran successful?"
date:   2016-12-22 20:10:12 +0100
comments: true
tags: jira
---
To apply changes to existing tables in JIRA Plugins which are using ActiveObjects,
it is necessary to write an upgrade task which describes how to transition the existing 
data of a table into the new datamodel.

Atlassian provides a [detailed documentation][upgrade task] on that topic. As mentioned, the upgrade task
interface expects a `getModelVersion()` method to be implemented, which should return the new datamodel version
after the upgrade ran successful.

But how to find out if the upgrade task actually ran successful? If the upgrade task failed for some reason (uncaught exception anone?),
the datamodel version will not be incremented and all succeeding upgrade task won't run. An additional issue is that 
JIRA will try to run the upgrade task on every restart because the new datamodel version was not applied due to the error.
Depending on the database size and the complexity of the update statements, this
could cause a significant increase of JIRAs startup time.

If JIRAs startup time increased significantly or you experience bugs whose root cause may be related to the datamodel,
it may be useful to take a manual look at the actual datamodel version of your plugin.

To achieve that, issue the following SQL.

```sql
SELECT PROPERTYVALUE
AS DATAMODELVERSION
FROM PROPERTYSTRING, (SELECT DISTINCT ID as ENTRYID FROM PROPERTYENTRY WHERE PROPERTY_KEY = '<YOUR_AO_ID>_#') entry_id
WHERE ENTRYID = ID
```

Substitute `<YOUR_AO_ID>` with the AO ID of your plugin tables. 
If this query does *not* return the model version of your latest upgrad task, it's time to debug your upgrade tasks. 

Be aware that you could also manipulate the existing `PROPERTYVALUE` value in your database to a previous datamodel version. 
This is how you could provoke a rerun of your upgrade tasks starting from that datamodel version.

[upgrade task]: https://developer.atlassian.com/docs/atlassian-platform-common-components/active-objects/developing-your-plugin-with-active-objects/upgrading-your-plugin-and-handling-data-model-updates