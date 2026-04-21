# AWS-Vehicle-Submission-Form-and-Verification

Overview
A web-based submission form that allows sellers of free vehicles listed on Facebook Marketplace to submit photos of their vehicle, title, and VIN for verification. Upon successful verification, the seller is presented with a scheduling link to arrange a pickup appointment.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Architecture

This system uses the following services:

- S3 (form bucket) — hosts the static HTML submission form
- S3 (uploads bucket) — stores submitted photos with a 7-day automatic deletion policy
- API Gateway — receives form submissions and enforces rate limiting
- Lambda — orchestrates the full verification flow
- Amazon Textract — extracts and compares VIN text from vehicle and title photos
- Amazon Rekognition — detects wheels in vehicle photos and returns a confidence score
- CloudFront — serves the form over HTTPS with a custom domain
- Route 53 — manages DNS routing for the custom domain
- Calendly — handles pickup appointment scheduling (third-party, free tier)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Criteria for Approval

A submission is approved if both of the following are met:

- The VIN on the vehicle matches the VIN on the title (verified by Textract)
- Wheels are detected in the vehicle photos above the confidence threshold (verified by Rekognition)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Security

- Cloudflare Turnstile (Managed mode) handles bot protection on the form
- File upload size capped at 10MB per photo
- Vehicle photos capped at 2, title and VIN at 1 each
- S3 uploads bucket is fully private — accessible only via Lambda

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Cost Estimate

Approximately $0.005 per submission. Textract dominates at ~$0.0045. Rekognition adds ~$0.0002. Fixed monthly costs for CloudFront and Route 53 are approximately $0.50 to $1.00. CloudWatch alarms add approximately $0.20 to $0.30 per month.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Current Status

- S3 form bucket created and configured
- HTML form built and uploaded
- S3 uploads bucket created and configured

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Notes

- API_ENDPOINT in index.html must be updated once API Gateway is configured
- CALENDLY_URL in index.html must be updated once Calendly is set up
- Turnstile secret key must be stored in Lambda for server side verification
- Cloudflare Turnstile domain list must be updated when custom domain is added
- Rekognition confidence threshold to be validated with edge case testing before production
