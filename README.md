# AWS-Vehicle-Submission-Form-and-Verification

Overview

A web-based submission form that allows sellers of free vehicles listed on Facebook Marketplace to submit photos of their vehicle, title, and VIN for verification. Upon successful verification, the seller is presented with an inline scheduling calendar to book a pickup appointment.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Architecture

This system uses the following services:

- S3 (form bucket) — hosts the static HTML submission form
- S3 (uploads bucket) — stores submitted photos with a 7-day automatic deletion policy
- API Gateway — receives form submissions and enforces rate limiting
- Lambda — orchestrates the full verification and scheduling flow
- Amazon Textract — extracts and compares VIN text from VIN photo and title photo
- Amazon Rekognition — detects wheels in vehicle photos and returns a confidence score
- DynamoDB (vehicle-pickup-slots) — stores claimed pickup slots with automatic TTL deletion after pickup date
- DynamoDB (vehicle-pickup-overrides) — stores schedule overrides that add, replace, or block dates outside the standard weekly template
- Amazon SES — sends pickup confirmation email to boss when a slot is claimed
- CloudFront — serves the form over HTTPS with a custom domain
- Route 53 — manages DNS routing for the custom domain

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Criteria for Approval

A submission is approved if both of the following are met:

- The VIN on the VIN plate photo matches the VIN on the title photo (verified by Textract)
- Wheels are detected in both vehicle photos above the 90% confidence threshold (verified by Rekognition)

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Photo Requirements

Sellers are required to submit four photos:

- Two side profile photos of the vehicle, one per side, clearly showing both wheels on that side
- One clear photo of the vehicle title
- One clear photo of the VIN plate on the vehicle

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Scheduling

After verification passes, sellers book a pickup appointment via an inline calendar embedded directly on the form. No name or email is required from the seller.

Availability logic:

- Boss defines a weekly recurring template — currently Saturdays and Sundays 9 AM to 9 PM in 3 hour blocks Central Time
- Slots are generated dynamically for a rolling 21 day window
- Overrides can block dates, replace hours, or add non-template availability
- Slots are claimed via conditional DynamoDB writes preventing double booking
- Claimed slots are deleted automatically after the pickup date via DynamoDB TTL

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Security

- Cloudflare Turnstile (Managed mode) handles bot protection on the form
- File upload size capped at 10MB per photo
- Vehicle photos capped at 2, title and VIN at 1 each
- S3 uploads bucket is fully private — accessible only via Lambda
- Presigned URLs used for direct browser to S3 uploads, bypassing API Gateway size limits
- Presigned URLs expire after 5 minutes

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Cost Estimate

Approximately $0.005 per submission. Textract dominates at ~$0.0045. Rekognition adds ~$0.0002. DynamoDB and SES add negligible cost at this volume. Fixed monthly costs for CloudFront and Route 53 are approximately $0.50 to $1.00. CloudWatch alarms add approximately $0.20 to $0.30 per month.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Testing

Local testing performed using AWS SAM CLI with Docker. Test events located in events/ folder.
Validated:

- /presign route — Turnstile verification, presigned URL generation, S3 key assignment
- /verify route — Textract VIN matching, Rekognition wheel detection, pass and fail paths
- Failure responses — clean descriptive reasons returned for each failure condition

Outstanding tests for pre-production:

- Mismatched VIN test
- No wheels test
- Unreadable VIN test
- Large file test
- Non-vehicle image test
- Invalid S3 keys test
- Empty body test
- Performance consistency test

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Current Status

- S3 form bucket created and configured
- HTML form built, tested, and uploaded
- S3 uploads bucket created and configured
- Lambda function deployed
- API Gateway configured
- Local end-to-end test passed

To Be Done

- DynamoDB tables created
- Lambda scheduling routes added
- SES configured and verified
- Inline calendar built into index.html
- Live end-to-end test passed
- CloudFront configured
- Route 53 and custom domain configured
- CloudFormation template written
- Terraform configuration written

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Notes

- TURNSTILE_SECRET stored as Lambda environment variable
- UPLOADS_BUCKET stored as Lambda environment variable
- Boss notification email stored as Lambda environment variable BOSS_EMAIL
- Cloudflare Turnstile domain list must be updated when custom domain is added
- Rekognition confidence threshold to be validated with edge case testing before production
- Outstanding SAM tests should be completed before production launch
- All times stored and displayed in Central Time
