# AWS-Vehicle-Submission-Form-and-Verification

Overview
A web-based submission form that allows sellers of free vehicles listed on Facebook Marketplace to submit photos of their vehicle, title, and VIN for verification. Upon successful verification, the seller is presented with a scheduling link to arrange a pickup appointment.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Architecture
This system uses the following services:

- S3 (form bucket) — hosts the static HTML submission form

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Criteria for Approval
A submission is approved if both of the following are met:

- The seller has the title for the vehicle (VIN on title matching VIN on vehicle)
- The vehicle has wheels

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

Current Status

- S3 form bucket created and configured
- HTML form built and uploaded
