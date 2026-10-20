# Secure Static Website Hosting Using Amazon S3 and CloudFront

## Project Overview

This project demonstrates how to securely deliver a static website using Amazon S3 and Amazon CloudFront.

The website is stored in a **private S3 bucket**, while CloudFront provides HTTPS-based access and content caching.

Rather than making the S3 bucket publicly accessible, I configured **Origin Access Control (OAC)** to allow CloudFront to retrieve website files securely.

I also performed hands-on experiments to understand CloudFront caching, cache invalidation, cache hits, cache misses, and Time to Live (TTL).

## AWS Services Used

- **Amazon S3:** Stores the static website files.
- **Amazon CloudFront:** Delivers website content over HTTPS using AWS edge locations.
- **AWS Identity and Access Management (IAM):** Provides the policy framework used to control access to S3 resources.

## Architecture

```text
                Website Visitor
                       |
                    HTTPS
                       |
                       v
                Amazon CloudFront
                - HTTPS delivery
                - Content caching
                - Origin Access Control
                       |
                Authorized S3 access
                       |
                       v
                Private S3 Bucket
                - Block Public Access ON
                - SSE-S3 encryption
                - index.html
```

## Security Configuration

The following security measures were implemented:

1. Created an S3 bucket in the Mumbai (`ap-south-1`) AWS Region.
2. Kept S3 Block Public Access enabled.
3. Used Amazon S3 server-side encryption (SSE-S3).
4. Created a CloudFront distribution with the S3 bucket as its origin.
5. Configured Origin Access Control (OAC).
6. Used an S3 bucket policy allowing the CloudFront service principal to retrieve objects, restricted to the authorized distribution using `AWS:SourceArn`.
7. Configured CloudFront to redirect HTTP requests to HTTPS.

### Security Verification

- Accessing the website through CloudFront over HTTPS: **Successful**
- Attempting anonymous direct access to the S3 object URL: **AccessDenied**
- Verified that the S3 bucket policy grants object-read access to the authorized CloudFront distribution.

## CloudFront Caching Experiments

### Cache Hit

Inspected the `X-Cache` HTTP response header using Chrome Developer Tools.

Observed:

`X-Cache: Hit from cloudfront`

This demonstrated CloudFront serving a valid cached response.

### Cache Miss

Created a targeted CloudFront invalidation for `/index.html`.

After the invalidation completed, requested `/index.html` and observed:

`X-Cache: Miss from cloudfront`

This demonstrated CloudFront retrieving content when a valid cached response was unavailable.

### Cache Invalidation

Updated `index.html` in S3 and observed that CloudFront continued displaying the previously cached version.

Created an invalidation using:

`/*`

After the invalidation completed, verified that CloudFront displayed the updated website.

Also practised targeted invalidation using:

`/index.html`

### Time to Live (TTL)

Inspected the AWS-managed `CachingOptimized` cache policy.

| Setting | Value |
|---|---|
| Minimum TTL | 1 second |
| Default TTL | 86,400 seconds (24 hours) |
| Maximum TTL | 31,536,000 seconds (365 days) |

Used Chrome Developer Tools to inspect the `Age` response header.

Observed that the `Age` value increased across subsequent Cache Hits.

Actual TTL expiration was not tested.

## Key Learnings

- Securely hosting static website files in a private S3 bucket.
- Using CloudFront OAC to restrict origin access.
- Delivering static website content over HTTPS.
- Understanding how CloudFront caches responses.
- Distinguishing Cache Hits from Cache Misses.
- Using cache invalidations to refresh updated website content.
- Understanding TTL and the HTTP `Age` header.
- Troubleshooting website delivery using browser Developer Tools.

## Project Structure

```text
aws-s3-cloudfront-secure-static-website/
├── README.md
├── docs/
├── screenshots/
└── website/
    └── index.html
```

## Deployment and Cleanup

This project was deployed and tested using the AWS Management Console.

The repository documents the implementation and verification steps so the architecture can be recreated for learning purposes.

AWS resources will be removed after documentation and verification to avoid unnecessary ongoing usage charges.

## Disclaimer

This is a hands-on AWS learning and portfolio project. The sample NovaTech website is used for demonstration purposes only.
