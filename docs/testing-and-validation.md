# Testing and Validation — Amazon S3 and CloudFront

## 1. Objective

Verify that the NovaTech static website is securely delivered through Amazon CloudFront, while its Amazon S3 origin remains private.

Also validate CloudFront caching behavior, cache invalidation, and Time to Live (TTL) configuration.

**Testing tools:** AWS Management Console, Google Chrome, Chrome Developer Tools

## 2. Test Results Summary

| Test | Expected Result | Actual Result | Status |
|---|---|---|---|
| CloudFront website access | Website loads over HTTPS | Website loaded successfully | PASS |
| Direct anonymous S3 access | Access denied | AccessDenied | PASS |
| S3 bucket policy verification | Access restricted to CloudFront distribution | CloudFront service principal and SourceArn condition verified | PASS |
| CloudFront Cache Hit | Cached response served | Hit from cloudfront | PASS |
| CloudFront Cache Miss | Response retrieved after invalidation | Miss from cloudfront | PASS |
| Cache invalidation | Updated website becomes visible | Version 2 displayed | PASS |
| TTL policy inspection | Expected policy settings visible | Minimum, Default and Maximum TTL verified | PASS |
| Cache Age observation | Age increases for a reused cached response | Age increased | PASS |

## 3. HTTPS Website Access

**Procedure:**

1. Open the CloudFront distribution URL in a browser.
2. Confirm the website loads using HTTPS.
3. Verify that the NovaTech homepage displays correctly.

**Actual result:** The website loaded successfully over HTTPS.

**Status:** PASS

## 4. Private S3 Origin Verification

**Procedure:**

1. Copy the direct S3 object URL for `index.html`.
2. Open it anonymously in a browser.
3. Check whether the object is publicly accessible.

**Actual result:** S3 returned `AccessDenied`.

**Status:** PASS

This confirms that anonymous visitors cannot retrieve the object directly through its S3 URL.

## 5. Origin Access Control and Bucket Policy

**Procedure:**

1. Open the S3 bucket's Permissions tab.
2. Inspect the bucket policy.
3. Confirm the policy grants `s3:GetObject` to the CloudFront service principal.
4. Verify that `AWS:SourceArn` restricts access to the authorized distribution.

**Actual result:** The expected CloudFront service principal, object-read permission, and distribution ARN condition were present.

**Status:** PASS

## 6. Cache Hit Test

**Procedure:**

1. Open the CloudFront website.
2. Open Chrome Developer Tools.
3. Navigate to Network → Headers.
4. Inspect the `X-Cache` response header.

**Observed response:**

```text
x-cache: Hit from cloudfront
```

**Status:** PASS

**Learning:** CloudFront can serve a valid cached response without retrieving the object from the S3 origin for that request.

## 7. Query String Cache Test

**Procedure:**

1. Open the CloudFront homepage.
2. Append `?test=cachemiss1` to the URL.
3. Inspect the `X-Cache` response header.

**Observed response:**

```text
x-cache: Hit from cloudfront
```

**Status:** PASS

**Learning:** The managed `CachingOptimized` policy does not include query strings in its cache key by default. Adding this parameter did not force a Cache Miss.

## 8. Cache Miss Test

**Procedure:**

1. Create a CloudFront invalidation for `/index.html`.
2. Wait until the invalidation status is Completed.
3. Request `/index.html` through CloudFront.
4. Inspect the `X-Cache` response header.

**Observed response:**

```text
x-cache: Miss from cloudfront
```

**Status:** PASS

**Learning:** A Cache Miss can occur even when the S3 object's content has not changed.

## 9. Missing Object Test

**Procedure:**

1. Request a nonexistent object path through CloudFront.
2. Inspect the response and `X-Cache` header.

**Observed response:**

```text
x-cache: Error from cloudfront
```

**Learning:** An error response does not, by itself, establish whether CloudFront experienced a Cache Hit or Cache Miss. Error responses can also be cached.

## 10. Cache Invalidation Test

**Procedure:**

1. Upload an updated `index.html` file to S3.
2. Observe that CloudFront continues displaying the previously cached version.
3. Create an invalidation for `/*`.
4. Wait for the invalidation to complete.
5. Refresh the website.

**Actual result:** CloudFront displayed the updated Version 2 website.

**Status:** PASS

**Learning:** Invalidation makes cached responses unusable so subsequent requests can retrieve current content.

## 11. TTL Configuration Verification

**Procedure:**

1. Open CloudFront → Cache policies.
2. Select the AWS-managed `CachingOptimized` policy.
3. Inspect the TTL settings.

**Observed configuration:**

| Setting | Value |
|---|---|
| Minimum TTL | 1 second |
| Default TTL | 86,400 seconds (24 hours) |
| Maximum TTL | 31,536,000 seconds (365 days) |

**Status:** PASS

These values define caching limits and the default freshness duration when applicable origin cache headers are absent.

Actual TTL expiration was not tested.

## 12. Age Header Test

**Procedure:**

1. Open the website and inspect the response headers.
2. Observe the `Age` value for a Cache Hit.
3. Wait approximately 20–30 seconds.
4. Request the same URL again.
5. Compare the `Age` values.

**Actual result:** The `Age` header increased.

**Status:** PASS

**Learning:** The `Age` header reports the approximate age of a cached response. It is not the same as TTL.

## 13. Final Validation

The tests confirmed that:

- The website was accessible through CloudFront over HTTPS.
- The S3 origin remained private.
- CloudFront OAC and the S3 bucket policy provided restricted origin access.
- Cache Hits and Cache Misses were observed using HTTP response headers.
- Cache invalidation successfully exposed updated website content.
- TTL configuration and cached response age were inspected.

The project was successfully deployed and validated using the AWS Management Console and Chrome Developer Tools.
