# AWS Architecture — Secure Static Website Hosting

## 1. Architecture Overview

This project uses Amazon CloudFront and Amazon S3 to securely deliver a static website.

The website files are stored in a private S3 bucket. Visitors access the website through CloudFront rather than directly accessing S3.

CloudFront provides HTTPS delivery and content caching, while Origin Access Control (OAC) allows CloudFront to securely retrieve files from the S3 origin.

## 2. Architecture Diagram

```text
                   Website Visitor
                          |
                    HTTPS Request
                          |
                          v
                  Amazon CloudFront
                  -----------------
                  HTTPS Redirection
                  Content Caching
                  Origin Access Control
                          |
                          |
              Authorized Origin Request
                          |
                          v
                   Amazon S3 Bucket
                   ----------------
                   Private Bucket
                   Block Public Access ON
                   SSE-S3 Encryption
                   index.html
```

## 3. AWS Components

### Amazon S3

- Stores the static website file `index.html`.
- Bucket deployed in the Mumbai Region (`ap-south-1`).
- Block Public Access enabled.
- Bucket owner enforced for object ownership.
- Server-side encryption enabled using SSE-S3.
- The bucket is not publicly accessible.

### Amazon CloudFront

- Acts as the public entry point for the website.
- Uses the S3 REST endpoint as its origin.
- Redirects HTTP requests to HTTPS.
- Uses `index.html` as the default root object.
- Uses the AWS-managed `CachingOptimized` cache policy.
- Caches eligible responses to reduce repeated origin requests.

### Origin Access Control (OAC)

Origin Access Control allows CloudFront to authenticate its requests to the private S3 bucket.

The S3 bucket policy permits s3:GetObject requests from the CloudFront service principal. It uses an ArnLike condition on AWS:SourceArn to restrict access to the authorized CloudFront distribution ARN.

This prevents anonymous visitors from directly retrieving website objects through the S3 URL.

## 4. Website Request Flow

1. A visitor enters the CloudFront website URL in their browser.
2. HTTP requests are redirected to HTTPS.
3. CloudFront checks whether it has a valid cached response.
4. If a valid cached response is available, CloudFront serves it.
5. Otherwise, CloudFront retrieves the content through its caching infrastructure, contacting the private S3 origin when necessary.
6. When contacting S3, CloudFront uses OAC to authenticate the origin request.
7. S3 evaluates the bucket policy and returns the permitted object.
8. CloudFront delivers the response to the visitor and can cache it for subsequent requests.

## 5. Security Design

The architecture follows these principles:

- Keep the S3 bucket private.
- Use CloudFront as the public website delivery endpoint.
- Restrict S3 object retrieval to the authorized CloudFront distribution.
- Use HTTPS for browser-to-CloudFront communication.
- Encrypt S3 objects at rest using SSE-S3.
- Avoid exposing AWS credentials or sensitive information in website files.

## 6. Verification

The following checks were performed:

| Test | Observed result |
|---|---|
| Access website through CloudFront HTTPS URL | Successful |
| Access S3 object anonymously using its direct URL | AccessDenied |
| Inspect S3 bucket policy | CloudFront access restricted by distribution ARN |
| Inspect CloudFront cache headers | Cache Hit and Cache Miss observed |
| Invalidate cached website content | Updated version displayed successfully |

## 7. Architecture Limitations

This is a learning-focused architecture for a static website.

- No custom domain or ACM certificate was configured.
- AWS WAF was not enabled.
- Origin Shield was disabled.
- No CI/CD pipeline or Infrastructure as Code deployment was implemented.
- The deployment was configured manually through the AWS Management Console.

These features could be explored in future projects.
