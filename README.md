# AWS-Vehicle-Submission-Form-and-Verification

Overview

A web-based submission form that allows sellers of free vehicles listed on Facebook Marketplace to submit photos of their vehicle, title, and VIN for verification. Upon successful verification, the seller enters a pickup location and selects an appointment time from an inline calendar. The boss receives an email notification with the pickup details and a vehicle photo.

------------------------------------------------------------------------------------------------------------------------------------------------------------------

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
- Amazon SES — sends pickup confirmation email with vehicle photo to boss when a slot is claimed
- CloudFront — serves the form over HTTPS with a custom domain
- Route 53 — manages DNS routing for the custom domain

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Criteria for Approval

A submission is approved if both of the following are met:

- The VIN on the VIN plate photo matches the VIN on the title photo (verified by Textract)
- Wheels are detected in both vehicle photos above the 90% confidence threshold (verified by Rekognition)

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Photo Requirements

Sellers are required to submit four photos:

- Two side profile photos of the vehicle, one per side, clearly showing both wheels on that side
- One clear photo of the vehicle title
- One clear photo of the VIN plate on the vehicle

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Scheduling

After verification passes, sellers enter a pickup location and book an appointment via an inline calendar embedded directly on the form. No name or email is required from the seller.

Availability logic:

- Boss defines a weekly recurring template — currently Saturdays and Sundays 9 AM to 9 PM in 3 hour blocks Central Time
- Slots are generated dynamically for a rolling 21 day window
- Overrides can block dates, replace hours, or add non-template availability via the vehicle-pickup-overrides DynamoDB table
- Slots are claimed via conditional DynamoDB writes preventing double booking
- Claimed slots are deleted automatically after the pickup date via DynamoDB TTL

Override format in DynamoDB:

Each override item requires:

- override_date — date string in YYYY-MM-DD format (partition key)
- type — one of block, replace, or add
- hours — list of integers representing hours in 24 hour format, required for replace and add types
- ttl — Unix timestamp for automatic deletion

------------------------------------------------------------------------------------------------------------------------------------------------------------------

API Routes

- POST /presign — verifies Turnstile token, returns presigned S3 upload URLs
- POST /verify — runs Textract VIN matching and Rekognition wheel detection
- GET /slots — returns available slots for the next 21 days
- POST /claim — claims a slot, stores location, sends SES notification
- OPTIONS /* — handled in Lambda for CORS preflight requests

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Security

- Cloudflare Turnstile (Managed mode) handles bot protection on the form
- File upload size capped at 10MB per photo
- Vehicle photos capped at 2, title and VIN at 1 each
- S3 uploads bucket is fully private — accessible only via Lambda
- Presigned URLs used for direct browser to S3 uploads, bypassing API Gateway size limits
- Presigned URLs expire after 5 minutes
- All AWS services follow least privilege IAM roles

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Cost Estimate
Approximately $0.005 per successful submission and schedule. Textract dominates at ~$0.0045. Fixed monthly costs for CloudFront and Route 53 are approximately $0.50 to $1.00. CloudWatch alarms add approximately $0.20 to $0.30 per month.

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Environment Variables

Lambda requires the following environment variables:

- UPLOADS_BUCKET — name of the S3 uploads bucket
- TURNSTILE_SECRET — Cloudflare Turnstile secret key
- RECEIVER_EMAIL — email address to receive pickup notifications
- SENDER_EMAIL — verified SES sender address

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Testing

Local testing performed using AWS SAM CLI with Docker. Test events located in events/ folder.

Validated:

- /presign route — Turnstile verification, presigned URL generation, S3 key assignment
- /verify route — Textract VIN matching, Rekognition wheel detection, pass and fail paths
- /slots route — dynamic slot generation from weekly template
- /claim route — conditional DynamoDB write, location storage, SES notification with photo
- Failure responses — clean descriptive reasons returned for each failure condition
- Full end to end live test passed

Outstanding tests for pre-production:

- Mismatched VIN test
- No wheels test
- Unreadable VIN test
- Large file test
- Non-vehicle image test
- Invalid S3 keys test
- Empty body test
- Performance consistency test

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Current Status

- S3 form bucket created and configured
- HTML form built, tested, and uploaded
- S3 uploads bucket created and configured
- Lambda function deployed
- API Gateway configured
- Local end-to-end test passed
- DynamoDB tables created
- Lambda scheduling routes added
- SES configured and verified
- Inline calendar built into index.html
- Location field added to scheduling flow
- Live end-to-end test passed

To Be Done

- CloudFront configured
- Route 53 and custom domain configured
- CloudFormation template written
- Terraform configuration written

------------------------------------------------------------------------------------------------------------------------------------------------------------------

Notes

- Cloudflare Turnstile domain list must be updated when custom domain is added
- Rekognition confidence threshold to be validated with edge case testing before production
- Outstanding SAM tests should be completed before production launch
- All times stored and displayed in Central Time
- SES currently in sandbox mode — production access request required before launch
- To block a date, add an item to vehicle-pickup-overrides with type: block
- To reset test slots, delete items from vehicle-pickup-slots in DynamoDB Console
