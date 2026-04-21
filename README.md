# AWS-Vehicle-Submission-Form-and-Verification

Overview
A web-based submission form that allows sellers of free vehicles listed on Facebook Marketplace to submit photos of their vehicle, title, and VIN for verification. Upon successful verification, the seller is presented with a scheduling link to arrange a pickup appointment.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Architecture
This system uses the following services:

- S3 (form bucket) — hosts the static HTML submission form
- S3 (uploads bucket) — stores submitted photos with a 7-day automatic deletion policy
- API Gateway — receives form submissions and enforces rate limiting
- Lambda — verifies submission criteria using Amazon Textract for OCR
- Amazon Textract — extracts and compares VIN text from vehicle and title photos
- CloudFront — serves the form over HTTPS with a custom domain
- Route 53 — manages DNS routing for the custom domain
- Calendly — handles pickup appointment scheduling (third-party, free tier)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Criteria for Approval
A submission is approved if both of the following are met:

- The seller has the title for the vehicle (VIN on title matching VIN on vehicle)
- The vehicle has wheels

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Security

- Cloudflare Turnstile (Managed mode) handles bot protection on the form
- File upload size capped at 10MB per photo
- Vehicle photos capped at 2, title and VIN at 1 each

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Current Status

- S3 form bucket created and configured
- HTML form built and uploaded
