# cloudfront architecture 

cloudfront terms 
- origin: original location of your content
  - s3 origin or a custom origin
  - can have one or more 
- distribution: the configuration unit of cloudfront 
- edge location: local cache of your data 
- regional edge cache: larger version of an edge location.

- cloudfront doesn't perform any write caching

origins are linked to behaviors
- cloudfront always has atleast one behavior 

Cloudfront behaviors 
- all caching behaviors are configured on the behavior level
- behaviors are pattern matched, the default is *
- you set the origins where they are forwarded, and the allowed HTTP methods
- this is also where you configure caching 
- you can restrict viewer access to a behavior

TTL and invalidations
- influence how long objects are stored at the edge locations

TTL -> time to live for objects 
  - default TTL = 24 hours 
  - this TTL applies to all objects that don't have a specific minimum TTL and maximum TTL
  headers: -> these can be set on a custom origin or s3 
  - cache-control max-age (seconds)
  - cache-control s-maxage (seconds)
  - expires (date and time)

cache invalidations
- applies to all edge locations, takes time 
- immediately expires any objects based on the path that you provide for invalidation
- if you're frequently needing to run cache invalidations, you may want to do versioned file names

TLS & SNI 
- alternate domain names allow you to change the default domain name for a distribution
- you need a matching certificate for the alternate domain name
- you need to generate or import a cert in US-EAST-1

- there are two connections that happen when connecting to cloudfront with HTTPS ( viewer and origin )
  - viewer => cloudfront
  - cloudfront => origin
  - both of these connections need a valid public certificate

SNI -> server name indication
- adds the ability for the client to tell the server which domain name its requesting
- this is how we can allow many TLS certs on a shared IP address

Origin types and architecture
- origin group: you can choose 2 or more origins to provide resiliency 

origins:
- s3 buckets
- mediastore container endpoints
- mediapackage endpoints
- HTTP endpoints ( everything else ) -> custom origins, web servers

NOTE: if you configure an S3 bucket for static web hosting, that is viewed as a custom origin by cloudfront

s3 origin
- Origin access -> this feature is only available when you're using s3 bucket origins for cloudfront
  - this is how you restrict an s3 origin so it's only available using the cloudfront distribution
  - origin access identity ( legacy )
  - origin access control ( modern )

custom origin 
- if you want to lock in only connections from cloudfront, you need to add a custom header on your origin

Performance, caching, and optimization

CACHE HIT = matching objects, delivered from cache
CACHE MISS = matching object needs to be grabbed from the origin 

query string parameters and headers:
- these influence the outcomes for caching and performance
- forward only what the application is going to need to the origin, cache only what is going to change the objects 
- default behavior is to not cache or forward headers or query string parameters, this means that if the application is expecting these for serving content, the application could error out 

FORWARD ALL & CACHE ALL -> 
  - all query string parameters are cached
  - all query string parameters are forwarded onto the origin

FORWARD ALL & CACHE whitelist -> 
- we are going to forward everything to the cloudfront origin because those are going to be used in the application decision making 
- we are going to whitelist certain parameters to be cached because they have no impact on what is going to be returned

# cloudfront security + OAI + custom origins 

OAI -> type of identity, associated with cloudfront distributions, CF becomes the OAI, this can be referenced in the s3 bucket policies
  - this is used to lock down an s3 bucket from cloudfront
  - can only be used with s3 origins 

when you have s3 bucket acting as a website, this becomes a custom origin, so how do we lock down access to custom origins?
- utilize custom headers -> add to the viewer policy, configure cloudfront to add custom header, the origin will need to require these headers are present
  - custom headers are injected at the edge location

- we can determine the IP ranges of the cloudfront edge locations 

# private behaviors, signed URLs & Cookies 

- these are specifically for private behaviors, these are not for private distributions 

two modes:
- public: open access to objects
- private: requests require signed cookie or URL 

how to configure private distributions:

OLD -> cloudfront key is created by the account root user
  - the account is added as a trusted signer

NEW -> trusted key groups
  - you don't need to use the root user for these
  - they can be managed using the cloudfront API and identities with the correct permissions 

signed URLs vs signed cookies
signer: trusted key group that you create in cloudfront, or an AWS account that contains a cloudfront key pair 
  - once signer is added to distribution, cloudfront starts to require that viewers use signed URLs or signed cookies to access your files 
  - once a signer is specified, you indirectly specify the files  that require sined URLs or signed cokies by adding the signer to a cache behavior, if you only have one cache behavior, viewers must use a signed URL or a signed cookie to access any files 

steps for creating a signer:
- create a public-private key pair 
- the signer will use its private key to sign the URL or cookies, and CF will use the public key to verify the signature
- in the distro, specify the signer 

URLs 
 - signed URLs provide accss to one object 
 - URLs if your client doesn't support cookies 

 cookies
 - if you want to preserve the applications URL
 - cookies provide access to groups of objects

# cloudfront georestriction

cloudfront georestriction
- whitelist or blacklist architecture - this is countries only 
- GeoIP database is used 
- configured for the entire distribution

3rd party geolocation
- completely customisable  
- we need some kind of compute to be the decider as part of the process 
- cloudfront is private during this, all access will need a signed URL or a signed Cookie
- we can check for any of these different attributes, or just the location 

for the exam: when we are talking about geolocation:
- if it's mentioning restricting based on country -> its cloudfront georestriction
- any other attributes like licensing -> its 3rd party geolocation

# Field level encryption
- happens at the edge, certain fields can be encrypted at the edge location 
- happens separately from the HTTPS tunnel 
- private key is required to decrypt individual fields 

1. generate public / private key pair 
2. the edge location using the public key pair to encrypt the specific field 
3. only the matching private key can be used to decrypt
4. sent encrypted to the custom origin, HTTPs tunnel is removed, but now our data is encrypted
5. data remains encrypted where it's stored, the device of the user will have the private key so it's accessible in the original form 

# Lambda@edge

- run lightweight lambda functions at the edge locations 
- adjust data between the viewer & origin
- cannot access VPC resources
- no lambda layers
- different limits vs normal lambda 

the function can run at any of these 4 parts:
- viewer request
- origin request 
- origin response 
- viewer response

use cases: 
- A/B testing - viewer request 
- migrations between s3 origins - origin request
- different objects based on device - origin request
- content by country

# elasticache

- in-memory database for applications that require high performance 
- data doesn't persist
- redis and memcached 

when to use:
- read heavy workloads with low latency requirements 
- reduces database workloads
- can be used to store session data ( stateless servers )
- this requires application code changes 

difference between redis and memcached

memcached
- simple data structures only ( strings )
- no replication
- multiple nodes can be used with sharding
- no backups 
- multi-threaded by design 

redis 
- advanced data structures 
- supports multi-az replication
- backup and restore
- transactions 

