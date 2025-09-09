✅ Begin planning and how to implement by creating a plan document in the docs folder for pricing pan integration. Free plan will be allowed to create 25 clients, projects, quotes, invoice, and 5 user accounts. Inventory should not be accessible to free tier. Paid Plan will allow a account to do any functionality with no limitations. 

✅ Begin planning and how to implement by creating a plan document in the docs folder for strip integration so that a user can pay for their subcription.

✅ Begin planning and how to implement by creating a plan document in the docs folder for setting up the taxes based on Country and State/Province so that the tax are apply on quote and invoices based on that selection. 

✅ Update Settings page and seperate the Admin Panel into multiple tabs for each of the section like Company Info, Integration, Billing and Package, and Users 

✅ Improve the User creation flow in the Setting page. We need to be able to create new users and assign a password.

✅ Fix the Remove User and Deactivation functionality. The system properly implements:
  - **Deactivate/Activate Toggle**: Uses ACTIVE ↔ INACTIVE status (Block/CheckCircle button)
  - **Remove User**: Uses SUSPENDED status for permanent removal (trash button) 
  - **User List**: Shows only ACTIVE and INACTIVE users, hides SUSPENDED (removed) users
  - Both preserve data integrity by keeping user records and their related projects, quotes, etc.

Need a Account Page for when a user logs into the service but their account is part of multiple orgs so they can pick the account they are logging into after a successful authentication. 

Limit the pages based on user role permissions. Admin will be able do all task and changes on any pages. User role will be able to make all the changes on the page but will only be able to see the Admin Panel but can not edit any of the settings. They should all look disabled. Viewer will be able to view all the page but can make no edits to any project or settings, they also can not see the settings page on the nav or get to the page. 

Implement the Pricing Plan Integration based on the pricing-plan-integration.md in the docs folders.

Implement the Stripe Integration based on the stripe-integration.md in the docs folders.

Implement the Tax Configuration based on the tax-configuration.md in the docs folders.