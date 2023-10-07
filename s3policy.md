# IAM User Creation - S3ReadOnlyUser

This document outlines the steps to create an IAM user named "S3ReadOnlyUser" with read-only access to Amazon S3.

## Step 1: Create IAM User
- Navigate to the IAM dashboard -> **Users** -> **ADD Users**
  - **Name:** S3ReadOnlyUser
  - **AccessType:** Select AWS Mangement Console Access 
  - **Console passowrd :** Bangladesh1328!
  - **Require Password Reset:** Uncheck Box
  - **Next - Permissions :** Select Policy - AmazonS3ReadOnlyAccess
  - **Next - Tag :** Next review - AmazonS3ReadOnlyAccess
  - Click on **Create**

3. Click "Users" in the left navigation pane.
4. Click "Add user."
5. Enter "S3ReadOnlyUser" as the user name.
6. Choose "AWS Management Console Access" for access type.
7. Set a strong password (e.g., "Bangladesh1328!").
8. Uncheck the "Require password reset" option.
9. Click "Next: Permissions."

## Step 2: Set Permissions

10. Choose "Add user to group" and create a new group if needed.
11. Create a group (e.g., "S3ReadOnlyGroup") and attach the "AmazonS3ReadOnlyAccess" policy.
12. Click "Next: Tags" (optional).
13. Click "Next: Review."
14. Review user details.
15. Click "Create user."

The IAM user "S3ReadOnlyUser" has been created with read-only access to Amazon S3.

