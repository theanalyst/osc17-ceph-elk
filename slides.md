## Ceph
 Basic ceph arch diagram slide
At a 10,000 ft view ceph is a unified storage solution that can provide block, object & file access
[insert diagram]
* Basic ELK explanation

---

## Logging


---

## ElasticSearch

---

## RGW
+ Object storage client to the ceph cluster, exposes a S3 & Swift API
+ Also implmenents user accounts, acls 
+ heavy ecosystem of s3/swift client tooling can be leveraged against RGW
+ From Jewel we support multisite which allows geographical redundancy 

--

### Multisite 
+ Disaster Recovery, CDN kind of usecase 
+ Key concepts: 
  - Zone -> local ceph cluster, with one or many RGWs, one of the zones needs to be master eg. aws us-east-1
  Data is replicated across zones bidirectionally, metadata always relies on master zone
  - Zonegroups -> collection of zones, zonegroups only share metadata among them

--

## RGW Metadata search with ES
### Motivation
+ Objects have metadata associated with them that is often interesting to analyze
+ Since it is an "object storage" you don't have any traditional filesystems tool at your disposal
+ No du, df & friends, and either way these are hard on a dist. storage system

--

### Motivation
+ Some existing support with admin API, however the problems with this:
- returns metadata when for eg. the user_ids or bucket_ids are provided, which would make it a not so simple application if you want to understand statistics about your object store
- also permissions for users to access the admin API is tricky, since admin API was meant for administering
+ As an storage administrator you'd be interested in finding out for eg. the top 10 users, average object size etc., no of objects held on a user account etc.

--

## Design
+ Requires multiple RGWs and multiple  zones
+ A zone is made read only and essentially forwards metadata to configured ES cluster, this zone will not service any metadata
+ essentially gets metadata from other zones and pushes them to ES, deletion is also handled automatically
+ We have an attribute mentioning owners for an object and this is used to service user req,.
+ ES unfortunately doesn't have an off the shelf authentication module, so RGWs in other zones can actually query metadata and service user requests (since user auth is handled by RGW)

