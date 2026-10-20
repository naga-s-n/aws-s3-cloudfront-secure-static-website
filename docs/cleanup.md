# AWS Resource Cleanup — S3 and CloudFront

## 1. Objective

Safely remove the AWS resources created for the NovaTech secure static website project after completing deployment, testing, and documentation.

This helps prevent unnecessary ongoing AWS usage and potential charges.

## 2. Resources to Remove

| AWS Service | Resource | Region |
|---|---|---|
| Amazon CloudFront | NovaTech-Static-Website distribution | Global |
| Amazon S3 | NovaTech private website bucket | Mumbai (`ap-south-1`) |

The CloudFront distribution uses Origin Access Control (OAC) to retrieve website files from the private S3 bucket.

## 3. Cleanup Procedure

### Step 1 — Disable CloudFront

1. Open AWS Console → CloudFront → Distributions.
2. Select the NovaTech website distribution.
3. Choose Disable.
4. Confirm the action.
5. Wait until the configuration change finishes deploying and the distribution is disabled.

**Status:** Pending

### Step 2 — Delete CloudFront Distribution

1. Select the disabled CloudFront distribution.
2. Choose Delete.
3. Confirm deletion.
4. Verify the distribution no longer appears in the CloudFront distribution list.

**Status:** Pending

### Step 3 — Empty the S3 Bucket

1. Open AWS Console → S3 → General purpose buckets.
2. Select the NovaTech website bucket.
3. Choose Empty.
4. Review the confirmation carefully.
5. Confirm the operation.
6. Verify the bucket contains no remaining objects or object versions.

**Status:** Pending

### Step 4 — Delete the S3 Bucket

1. Return to the S3 bucket list.
2. Select the empty NovaTech bucket.
3. Choose Delete.
4. Confirm the deletion.
5. Verify the bucket no longer appears in the bucket list.

**Status:** Pending

## 4. Additional Verification

After completing the cleanup:

- Confirm the NovaTech CloudFront distribution has been deleted.
- Confirm the NovaTech S3 bucket has been deleted.
- Review the CloudFront Origin Access Control list for any project-specific OAC that may remain.
- Delete a leftover project-specific OAC only after confirming no distribution uses it.
- Review AWS Billing and Cost Management for any unexpected charges.

CloudFront invalidation history and cached responses do not require separate cleanup after the distribution is deleted.

## 5. Final Cleanup Results

| Resource | Final Status |
|---|---|
| CloudFront distribution | Pending |
| S3 website bucket | Pending |
| Project-specific OAC, if applicable | Pending verification |
| Billing review | Pending |

Update this table after completing and verifying the actual cleanup operations.

## 6. Important Notes

- Cleanup operations should be performed only after project evidence has been collected.
- Deleting an S3 bucket and its objects is destructive.
- The CloudFront distribution should be disabled before deletion.
- AWS billing data may take time to reflect recent usage.
- This document records the cleanup procedure; it does not claim the resources have already been removed.

