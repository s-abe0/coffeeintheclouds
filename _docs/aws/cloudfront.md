---
layout: default
---

## CloudFront

AWS CloudFront is a Content Delivery Network (CDN) that improves distribution times for static and dynamic web content (html, css, js & image files) for users. Content is stored on worldwide data centers called edge locations. Requests for content is routed to an edge location with lowest latency. If the content is not in the edge location, CloudFront retrieves it from a defined origin such as an S3 bucket, MediaPackage, or HTTP server.

Configuring CloudFront for content distribution

    1. Specify *origin servers*, such as an S3 bucket or an HTTP server (either EC2 or on premise). The original content is stored on the origin, where CloudFront grabs them. 

    2. Upload content to the origin server, called **objects**. Content can be anything that can be served over HTTP. 

    3. Create a CloudFront distribution, setting up what origin servers CloudFront should grab content from. Details such as logging requests can be set.

    4. A domain name is created for the CloudFront distribution, or an alternate domain name can be used.

    5. Your distributions configuration (not the content) is send to all the CloudFront edge locations, where caches of the content is stored.

**S3 as an Origin** - S3 can be used as a CloudFront origin, where Origin Access Identity is used to allow only CloudFront to access the S3 bucket.

**EC2 instance as an origin** - EC2 instance must be public and allow public IPs of Edge locations so the CloudFront distro can access it.

**ALB as an origin** - An Application Load Balancer can be used as an origin to sit infront of an EC2 instance, which allows the EC2 instance to be private. The ALB must be public and allow public IPs of CloudFront Edge locations.

#### CloudFront Security

**Encryption in Transit** - CloudFront can be configured to require users to use HTTPS. It can also be configured to use HTTPS when grabbing content from the origin. Field-level encryption can also be used to only allow applications which have credentials (apps that need the data) to access the data, such as sensitive user information. Data is encrypted at the edge and remains encrypted throughout the entire application stack.

**Encryption at rest** - CloudFront uses encrypted SSDs for edge location *points of presence (POPs)*, and encrypted EBS volumes for *Regional Edge Caches (RECs)*. 

**Signed URLs or cookies** - Restrict access to content intended for selected users using signed URLs or signed cookies. Signed URLs can be used for single object access to an S3 bucket, whereas cookies can be used for multiple objects such as an http site.

CloudFront Signed URLs allow access to a path, no matter the origin, such as S3, http, etc. They can also be filtered by IP, path, date and expiration. It is an account wide key-pair and manage by the root account.

**Origin Access Identity (OAI)** - Manages direct access to S3 content by using a special CloudFront user identity, associated with the distribution, to secure all or some of the S3 content. This restricts users from accessing the direct S3 url, and only allows CloudFront direct access to S3. An S3 policy on the bucket is created to only allow access for the CloudFront user.

**Geo restriction** - Geo restriction, or geoblocking, can be used to prevent specific geographic locations access to the content served through CloudFront. A whitlelist or blacklist can be used.

#### CloudFront Caching

Cache lives at each CloudFront edge location, and is based on Headers, Session cookies and Query String Parameters. By default, files automatically expire after 24 hours. The distribution cache duration can be set using a *cache policy* and setting the Minimum TTL, Maximum TTL and Default TTL values. TTL for individual files can be set by using the *Cache-Control* or *Expires* headers, such as for an S3 object.

Content in a distribution can also be invalidated before the cache expires, forcing CloudFront to retrieve the latest version from the origin on the next user request.

#### Use Cases

**Accelerate website content delivery** - CloudFront can speed up delivery of content deliver to users across the world, by using edge locations and the AWS backbone network.

**Serve video on demand or live streaming video** - Pre-recorded files and live events can be streamed using CloudFront. 

**Encrypt specific fields throughout system processing** - Field-level encryption can be added to protect specific data throughout system processing in addition to HTTPS. This makes it so that only certain applications at the origin can see the data.