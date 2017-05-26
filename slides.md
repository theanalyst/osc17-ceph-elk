## Ceph
 Basic ceph arch diagram slide
At a 10,000 ft view ceph is a unified storage solution that can provide block, object & file access

* Basic ELK explanation

---

## Logging

---

## RGW
+ Object storage client to the ceph cluster, exposes a S3 & Openstack
Swift API 
<p align="center"><img src="img/rgw.png"></p>

--

### RGW
+ Implements Buckets, User Accounts, ACLs 
+ S3 is a very well known Object Storage Rest API, RGW's
  implementation is similar
+ Heavy ecosystem of s3/swift client tooling can be leveraged against
  RGW
+ Supports S3 features like websites, object lifecycle, versioning

--

### RGW
+ From Jewel we support multisite which allows for data to be
  replicated across geographies
+ From Kraken we support sync plugins which allow for interesting
  operations that can work based on object data & metadata
  notifications

---

## ElasticSearch
+ At its core a search & analytics engine with a REST api
+ Understand trends & patterns in data
+ Really simple & intuitive REST API for many std. queries

---

## RGW Metadata search with ES
### Motivation
+ Objects have metadata associated with them that is often interesting to analyze
+ Since it is an "object storage" you don't have any traditional filesystems tool at your disposal
+ No du, df & friends, and either way these are hard on a dist. storage system

--

### Motivation
+ Some existing support with admin API, however the problems with this:
- returns specific metadata, not ideal for aggregation
- no notifications when new objects/buckets/accounts are created
- also permissions for users to access the admin API is tricky, since admin API was meant for administering
+ As an storage administrator you'd be interested in finding out for eg. the top 10 users, average object size etc., no of objects held on a user account etc.

--

## Design
+ Built atop of the multisite architecture, where metadata forwarding was already impelmented
+ Requires multiple RGWs and multiple  zones
+ A zone is made read only and essentially forwards metadata to configured ES cluster, this zone will not service any data
+ essentially gets metadata from other zones and pushes them to ES, deletion is also handled automatically


--

## Design
+ We have an attribute mentioning owners for an object and this is used to service user req,.
+ ES unfortunately doesn't have an off the shelf authentication module, so RGWs in other zones can actually query metadata and service user requests (since user auth is handled by RGW)

--

## Example Queries
As an administrator:
- Average object size in the cluster, by user etc.
- Queries across user accounts, largest object sizes etc
As a user:
- total uploads over the last {week,hour,month...}
- objects with a certain metadata tag
- the full range of ES type queries are still accessible by connecting to RGW

--

## Status in OpenSUSE
- openSUSE Factory: we already have Luminous (Ceph 12.0.2), 42.3, TW
- devel package: Filesystems:Ceph

#Backup Slides

### Multisite 
+ Disaster Recovery, CDN kind of usecase 
<section data-background-color="#ffffff">
<p align="center"><img src="img/zone-sync2.png"></p>
</section>


--

### Multisite
+ Zone -> local ceph cluster, with one or many RGWs, one of the zones needs to be master eg. aws us-east-1
  Data is replicated across zones bidirectionally, metadata always relies on master zone
+ Zonegroups -> collection of zones, zonegroups only share metadata among them
