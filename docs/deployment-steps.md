# Deployment Guide — Secure Static Website Using Amazon S3 and CloudFront

## 1. Project Objective

Deploy a static website using Amazon S3 as a private storage origin and Amazon CloudFront as the public website delivery service.

The goal is to make the website accessible over HTTPS without allowing anonymous visitors to access the S3 objects directly.

**Deployment method:** AWS Management Console

**S3 Region:** Asia Pacific (Mumbai) — `ap-south-1`

## 2. Prerequisites

- An AWS account with permissions to manage S3 and CloudFront
- A sample `index.html` website file
- Access to the AWS Management Console
- A web browser for testing

AWS service usage may incur charges. Review current AWS pricing and account billing information before deployment.

## 3. Create the Private S3 Bucket

1. Open the AWS Management Console.
2. Navigate to Amazon S3.
3. Select **Create bucket**.
4. Choose the Mumbai Region (`ap-south-1`).
5. Enter a globally unique bucket name.
6. Keep **ACLs disabled (Bucket owner enforced)**.
7. Keep **Block all public access** enabled.
8. Use default server-side encryption with Amazon S3 managed keys (SSE-S3).
9. Create the bucket.

### Verification

Confirm that the bucket exists and that Block Public Access is enabled.

## 4. Upload the Website File

1. Open the newly created S3 bucket.
2. Select **Upload**.
3. Add the `index.html` file.
4. Complete the upload.
5. Verify that `index.html` appears in the bucket's object list.

The bucket remains private. Do not enable public access or configure public S3 website hosting for this architecture.

## 5. Create the CloudFront Distribution

1. Open Amazon CloudFront.
2. Select the option to create a distribution.
3. Choose the single-website configuration in the AWS Console wizard.
4. Select the private S3 bucket as the origin.
5. Use the regular S3 REST endpoint, not the S3 static website endpoint.
6. Enable the option to grant CloudFront access to the origin.
7. Use **Origin Access Control (OAC)** for authenticated access to S3.
8. Keep Origin Shield disabled for this lab.
9. Leave AWS WAF disabled for this learning project.
10. Use the pay-as-you-go configuration.
11. Create the distribution.

The AWS Console may present these options in a different order depending on its interface version.

## 6. Configure the S3 Bucket Policy

The S3 bucket policy must allow the authorized CloudFront distribution to read website objects.

The policy uses:

- Principal: `cloudfront.amazonaws.com`
- Action: `s3:GetObject`
- Resource: The objects in the website bucket
- Condition: `AWS:SourceArn` matching the authorized CloudFront distribution

When the AWS Console configures OAC automatically, inspect the generated bucket policy to confirm these settings.

Do not replace the distribution ARN with an unrestricted wildcard.

## 7. Configure CloudFront Website Settings

Verify the following distribution settings:

| Setting | Configuration |
|---|---|
| Origin | Private S3 bucket |
| Origin access | OAC |
| Default root object | `index.html` |
| Viewer protocol policy | Redirect HTTP to HTTPS |
| Cache policy | `Managed-CachingOptimized` |
| Origin request policy | None |
| Response headers policy | None |
| Origin Shield | Disabled |
| AWS WAF | Disabled |

Wait until the distribution configuration has finished deploying before testing.

## 8. Test the Website

### Test A — CloudFront Website Access

1. Copy the CloudFront distribution domain name.
2. Open the domain in a web browser using HTTPS.
3. Confirm that the NovaTech website loads successfully.

**Observed result:** Website loaded successfully.

### Test B — Direct S3 Access

1. Copy the S3 object URL for `index.html`.
2. Open the URL anonymously in a browser.
3. Confirm that the object cannot be retrieved publicly.

**Observed result:** `AccessDenied`.

This verifies that the website is accessible through CloudFront while direct anonymous S3 access remains restricted.

## 9. Test CloudFront Caching

Open Chrome Developer Tools → Network → select the document request → Headers.

Inspect the `X-Cache` response header.

### Cache Hit

Observed:

`X-Cache: Hit from cloudfront`

### Cache Miss

Created an invalidation for `/index.html`.

After the invalidation completed, requested `/index.html` again and observed:

`X-Cache: Miss from cloudfront`

### Cache Invalidation

1. Modify and reupload `index.html` to S3.
2. Observe that CloudFront may continue serving the previously cached version.
3. Create an invalidation for `/*`.
4. Wait until the invalidation completes.
5. Reload the website and verify that the updated content appears.

**Observed result:** Updated website content displayed successfully.

For targeted homepage invalidations, consider both `/` and `/index.html` because CloudFront may cache them separately.

## 10. Inspect TTL and Cache Age

Open CloudFront → Policies → Cache policies → Managed → `CachingOptimized`.

Verify:

| Setting | Value |
|---|---|
| Minimum TTL | 1 second |
| Default TTL | 86,400 seconds |
| Maximum TTL | 31,536,000 seconds |

In Chrome Developer Tools, inspect the `Age` response header for cached responses.

**Observed result:** The `Age` value increased across subsequent Cache Hits.

This experiment demonstrates the reported age of a cached response. It does not independently verify TTL expiration.

## 11. Deployment Result

The NovaTech static website was successfully deployed using:

- A private Amazon S3 bucket
- Amazon CloudFront
- Origin Access Control
- A restricted S3 bucket policy
- HTTPS delivery
- CloudFront caching and invalidation

The deployment and tests were performed manually using the AWS Management Console.

## 12. Cleanup

After collecting the required project evidence:

1. Disable the CloudFront distribution.
2. Wait for the disable operation to finish deploying.
3. Delete the CloudFront distribution.
4. Empty the S3 bucket, including any object versions if versioning is enabled.
5. Delete the S3 bucket.
6. Verify resource removal.

Record the actual cleanup results in the project documentation once completed.
