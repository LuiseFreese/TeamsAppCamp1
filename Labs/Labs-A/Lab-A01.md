![Teams App Camp](../Assets/code-lab-banner.png)

## Lab A01: Start with Azure Active Directory

In this series of labs, you will port a simple "Northwind Orders" web application to become a full-fledged Microsoft Teams application. To make the app understandable by a wide audience, it is written in vanilla JavaScript with no UI framework, however it does use modern browser capabilities such as web components, CSS variables, and ECMAScript modules. The server side is also in JavaScript, using Express, the most popular web server platform for NodeJS.

There are two options for doing the labs:

* The "A" path is for developers with apps that are already based on Azure Active Directory. The starting app uses Azure Active Directory and the Microsoft Authentication Library (MSAL).

* the "B" path is for developers with apps that use some other identity system. It includes a simple (and not secure!) cookie-based auth system based on the Employees table in the Northwind database. You will use an identity mapping scheme to allow your existing users to log in directly or via Azure AD Single Sign-On.

This is the very first lab in Path A, which begins with an application that already uses Azure AD.

In this lab you will set up the Northwind Orders application, which can be found in the [A01-Start-AAD](../../A01-Start-AAD/) folder. The labs that follow will lead you step by step into extending the web application to be a Microsoft Teams application as well.

* [Lab A01: Setting up the application with Azure AD](./Lab-A01.md) (📍You are here)
* Lab A02: (there is no lab A02; please skip to A03)
* [Lab A03: Creating a Teams app with Azure AD SSO](./Lab-A03.md)
* [Lab A04: Teams styling and themes](./Lab-A04.md)
* [Lab A05: Add a Configurable Tab](./Lab-A05.md)
* [Lab A06: Add a Messaging Extension](./Lab-A06.md)
* [Lab A07: Add a Task Module and Deep Link](./Lab-A07.md)
* [Lab A08: Add support for selling your app in the Microsoft Teams store](./Lab-A08.md)

In this lab you will learn to:

* [Register an application with the Microsoft identity platform](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app)
* How to use the [Microsoft Authentication Library (MSAL)](https://docs.microsoft.com/azure/active-directory/develop/msal-overview)
* How to validate an [Azure AD access token](https://docs.microsoft.com/azure/active-directory/develop/access-tokens) in a NodeJS application

### Features

* View orders associated with the logged-in user (sales representative)
* View products by category
* View product details and orders for a product
* View order details

The application is based on the Northwind Traders Database, which is a sample relational database that originally shipped with Microsoft Access. The Northwind Traders Database is now available as a [demonstration OData service](https://services.odata.org/), which is queried in this lab. This is a read-only data source; some of the later exercises appear to update the data but the changes are only stored in the server memory and will only persist until the server is restarted.

### Exercise 1: Install prerequisites

You can complete these labs on a Windows, Mac, or Linux machine, but you do need the ability to install the prerequisites. If you are not permitted to install applications on your computer, you'll need to find another machine (or virtual machine) to use throughout the workshop.

#### Step 1: Install NodeJS

NodeJS is a program that allows you to run JavaScript on your computer; it uses the open source "V8" engine, which is used in popular web browsers such as Microsoft Edge and Google Chrome. You will need NodeJS to run the web server code used throughout this workshop.

Browse to [https://nodejs.org/en/download/](https://nodejs.org/en/download/) and install the "LTS" (Long Term Support) version for your operating system. This lab has been tested using NodeJS version 14.17.4 and 16.14.0. If you already have another version of NodeJS installed, you may want to set up the [Node Version Manager](https://github.com/nvm-sh/nvm) (or [this variation](https://github.com/coreybutler/nvm-windows) for Microsoft Windows), which allows you to easily switch Node versions on the same computer.

#### Step 2: Install a Code Editor

You can really use any code editor you wish, but we recommend [Visual Studio Code](https://code.visualstudio.com/download).

#### Step 3: Install ngrok

ngrok is a tunneling program that allows you to access your local web server (running in NodeJS in this case) from the Internet. To complete this exercise, download and install ngrok from [here](https://ngrok.com/download).

The free version of ngrok will assign a URL similar to https://something.ngrok.io, where "something" is a random identifier. As long as ngrok is running (leave it going in a command or terminal window), you can browse your web site at that URL. If you start and stop ngrok, or try to keep it running for more than 8 hours, you'll get a new identifier and you'll need to update your app registration, environment variables, etc. The paid version of ngrok allows you to reserve the same URL for use over time, removing the need to update it when you return to the lab.

While ngrok isn't strictly required for developing Microsoft Teams applications, it makes things much easier, especially if Bots are involved (Lab 6 has a bot inside to support Messaging Extensions). If you or your company aren't comfortable with running ngrok (some companies block it on their corporate networks), please check out [this video](https://www.youtube.com/watch?v=A5U-3o-mHD0) which explains the details and work-arounds.

### Exercise 2: Set up your Microsoft 365 Subscription

The initial Northwind Orders application does not require Microsoft 365, but it does use Azure AD. In order to ensure the users are in the same directory as Microsoft Teams, we'll set up the Microsoft 365 tenant now and set up the application in the same Azure AD directory that we'll use throughout the workshop.

#### Step 1: Get a tenant

If you don't yet have a tenant, please join the [Microsoft 365 Developer Program](https://developer.microsoft.com/microsoft-365/dev-program) to get a free one. Your tenant includes 25 [E5 user licenses](https://www.microsoft.com/microsoft-365/enterprise/compare-office-365-plans) and can be renewed as long as you keep developing!

Select "Join now" to begin.
![Signup](../Assets/01-003-JoinM365DevProgram1.png)

Log in with any Microsoft personal or work and school account, enter your information, and select "Next". You will have an opportunity to choose what kind of "sandbox" you want; the "Instant sandbox" is recommended.

![Signup](../Assets/01-004-JoinM365DevProgram2.png)

Follow the wizard and select your administrator username and password, tenant domain name, etc. The domain name you choose is just the left-most portion * for example if you enter "Contoso" your domain will be "Contoso.onmicrosoft.com".

Remember this information as you'll need it throughout the labs! You will log in as <username>@<domain>.onmicrosoft.com with the password your chose. You'll be prompted for your phone number and then the system will set up your subscription.

Eventually you'll be prompted to log into your new tenant. Be sure to use the new administrator credentials you just created, not the ones you used when you signed up for the developer program.

---
😎 DON'T DEVELOP IN PRODUCTION: It may be tempting to build solutions right where you work every day, but there are good reasons to have a dedicated dev tenant - and probably additional staging/test tenants. They're free, and you can safely experiment as a tenant admin without risking your production work.

---
😎 NAVIGATING MANY TENANTS: Consider creating a browser profile for each tenant that will have its own favorites, stored credentials, and cookies so you can easily swtch between tenants as you work.

---
😎 CHANGES ROLL OUT FIRST TO "TARGETED RELEASE" TENANTS. You may want to [enable Targeted Release](https://docs.microsoft.com/microsoft-365/admin/manage/release-options-in-office-365) in your developer tenant and keep production on Standard Release so you have a head start to test out new features.

---

#### Step 2: Enable Teams application uploads

By default, end users can't upload Teams applications directly; instead an administrator needs to upload them into the enterprise app catalog. In this step you will enable direct uploads to make development easier and allow installation directly from the Teams user interface.

  a. Navigate to [https://admin.microsoft.com/](https://admin.microsoft.com/), which is the Microsoft 365 Admin Center.

  b. In the left panel of the admin center, select "Show all" to open up the entire navigation

  ![M365 Admin](../Assets/01-005-M365Admin.png)

  When the panel opens, select Teams to open the Microsoft Teams admin center.

  ![M365 Admin](../Assets/01-006-M365Admin2.png)

  c. In the left of the Microsoft Teams admin center, open the Teams apps accordion 1️⃣ and select Setup Policies 2️⃣. You will see a list of App setup policies. Select the Global (Org-wide default) policy 3️⃣.

  ![Teams admin](../Assets/01-007-TeamsAdmin1.png)

 d. Ensure the first switch, "Upload custom apps" is turned On.

 ![Teams admin](../Assets/01-008-TeamsAdmin2.png)

Be sure to scroll down and select the "Save" button to persist your change.

![Teams admin](../Assets/01-008-TeamsAdmin2b.png)

 We have been working to get this enabled by default on developer tenants, so it may already be set for you. The change can take up to 24 hours to take effect, but usually it's much faster.

### Exercise 3: Assign users as Northwind "Employees"

 The Northwind database contains 9 employees, so up to 9 users in your tenant will be able to use the application. (You'll only need two to complete the labs.)

The Northwind Orders application expects each user's employee ID in Azure Active Directory to match their employee ID in the Northwind database. In this exercise you'll set up some test users accordingly.

#### Step 1: Edit Azure AD users

* Navigate to the Microsoft 365 admin center at https://admin.microsoft.com/ and log in as the administrator of your new dev tenant.

* In the left navigation, select "Show All" to reveal the full list of admin centers, and then select "Azure Active Directory". This will bring you to the [Azure AD admin center](https://aad.portal.azure.com/).

![Navigating to the M365 Admin site](../Assets/01-009-RegisterAADApp-1.png)

* Select "Azure Active Directory" again in the left navigation bar.

![Navigating to the M365 Admin site](../Assets/01-010-RegisterAADApp-2.png)

* This will bring you to the overview of your Azure AD tenant. Note that a "tenant" is a single instance of Azure Active Directory, with its own users, groups, and app registrations. Verify that you're in the developer tenant you just created, and select "Users" in the navigation bar.

![Edit users](../Assets/01-030-EditUsers-1.png)

You can use existing users to run the Northwind Orders application (the names may not match the Northwind database unless you change them, but you'll know what's going on), or create new ones. It's easiest if one of the users is the administrator account you're logged into right now, so you can test the application without logging on and off, but that's up to you. Select on the user to view their user profile, and then select the "Edit" button.

![Edit user's employee ID](../Assets/01-031-EditUser-2.png)

Change the Employee ID to the ID of one of the users in the Northwind datbase, which are:

| Employee ID | Name |
|---|---|
| 1 | Nancy Davolio |
| 2 | Andrew Fuller |
| 3 | Janet Leverling |
| 4 | Margaret Peacock |
| 5 | Steven Buchanan |
| 6 | Michael Suyama |
| 7 | Robert King |
| 8 | Laura Callahan |
| 9 | Anne Dodsworth |

You may also choose to rename the users to match the database.

#### Step 2: Ensure the users are licensed for Microsoft 365

From the same user profile screen, select "Licenses" and ensure the user has an Office 365 license so they can run Microsoft Teams.

![Check license](../Assets/01-032-CheckLicense.png)

> NOTE: When you publish your application in the Microsoft Teams store, you will be responsible for your own license management and licenses for your application will not appear here along with the licenses for Microsoft products. In Lab 08, you will implement this strategy for the Northwind Orders app.

### Exercise 4: Register your application with Azure AD

In order for users to log into your application with Azure AD, you need to register it. In this exercise you will register your application directly in the tenant you created in Exercise 2, however we'll set it up so it can be used from other tenants, such as those of customers who purchase your application in the Microsoft Teams store. To learn more about multitenant applications, see [this video](https://www.youtube.com/watch?v=RjGVOFm39j0&t=7s).

#### Step 1: Start ngrok

Before you can register your application, you will need to start ngrok to obtain the URL for your application. In the command line tool of your choice, navigate to the folder where you've saved **ngrok.exe** and run this command:

~~~shell
ngrok http 3978 -host-header=localhost
~~~

The terminal will display a screen like this; note the https forwarding URL for use in this lab. Save this URL for use throughout the labs.

![ngrok output](../Assets/01-002-ngrok.png)

---
> **NOTE:** [This page](../../docs/ngrokReferences.md) lists all the exercises which involve the ngrok URL so you can easily update it if it changes.
---

#### Step 2: Register your application in Azure Active Directory

* Navigate to the Microsoft 365 admin center at https://admin.microsoft.com/ and log in as the administrator of your new dev tenant.

* In the left navigation, select "Show More" to reveal the full list of admin centers, and then select "Azure Active Directory". This will bring you to the [Azure AD admin center](https://aad.portal.azure.com/).

![Navigating to the M365 Admin site](../Assets/01-009-RegisterAADApp-1.png)

* Select "Azure Active Directory" again in the left navigation bar.

![Navigating to the M365 Admin site](../Assets/01-010-RegisterAADApp-2.png)

* This will bring you to the overview of your Azure AD tenant. Note that a "tenant" is a single instance of Azure Active Directory, with its own users, groups, and app registrations. Verify that you're in the developer tenant you just created, and select "App Registrations" in the navigation bar.

![Opening App Registrations](../Assets/01-011-RegisterAADApp-3.png)

* You will be shown a list of applications (if any) registered in the tenant. Select "+ New Registration" at the top to register a new application.

![Adding a registration](../Assets/01-012-RegisterAADApp-4.png)

You will be presented with the "Register an application" form.

![Register an application form](../Assets/01-013-RegisterAADApp-5.png)

* Enter a name for your application 1️⃣.
* Under "Supported account types" select "Accounts in any organizational directory" 2️⃣. This will allow your application to be used in your customer's tenants.
* Under "Redirect URI", select "Single-page application (SPA)" 3️⃣ and enter the ngrok URL you saved earlier 4️⃣.
* Select the "Register" button 5️⃣

You will be presented with the application overview. There are two values on this screen you need to copy for use later on; those are the Application (client) ID 1️⃣ and the Directory (tenant) ID 2️⃣.

![Application overview screen](../Assets/01-014-RegisterAADApp-6.png)

When you've recorded these values, navigate to "Certificates & secrets" 3️⃣.

![Adding a secret](../Assets//01-015-RegisterAADApp-7.png)

Now you will create a client secret, which is like a password for your application to use when it needs to authenticate with Azure AD.

* Select "+ New client secret" 1️⃣
* Enter a description 2️⃣ and select an expiration date 3️⃣ for your secret
* Select "Add" to add your secret. 4️⃣

The secret will be displayed just this once on the "Certificates and secrets" screen. Copy it now and store it in a safe place.

![Copy the app secret](../Assets/01-016-RegisterAADApp-8.png)

---
😎 MANAGING APP SECRETS IS AN ONGOING RESPONSIBILITY. App secrets have a limited lifetime, and if they expire your application may stop working. You can have multiple secrets, so plan to roll them over as you would with a digital certificate.

---
😎 KEEP YOUR SECRETS SECRET. Give each developer a free developer tenant and register their apps in their tenants so each developer has his or her own app secrets. Limit who has access to app secrets for production. If you're running in Microsoft Azure, a great place to store your secrets is [Azure KeyVault](https://azure.microsoft.com/services/key-vault/). You could deploy an app just like this one and reference store sensitive application settings in Keyvault. See [this article](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references) for more information.

---

#### Step 3: Grant your application permission to call the Microsoft Graph API

The app registration created an identity for your application; now we need to give it permission to call the Microsoft Graph API. The Microsoft Graph is a RESTful API that allows you to access data in Azure AD and Microsoft 365, including Microsoft Teams.

* While still in the app registration, navigate to "API Permissions" 1️⃣ and select "+ Add a permission" 2️⃣.

![Adding a permission](../Assets/01-017-RegisterAADApp-9.png)

On the "Request API permissions" flyout, select "Microsoft Graph". It's hard to miss!

![Adding a permission](../Assets/01-018-RegisterAADApp-10.png)

Notice that the application has one permission already: delegated permission User.Read permission for the Microsoft Graph. This allows the logged in user to read his or her own profile.

The Northwind Orders application uses the Employee ID value in each users's Azure AD profile to locate the user in the Employees table in the Northwind database. The names probably won't match unless you rename them but in a real application the employees and Microsoft 365 users would be the same people.

So the application needs to read the user's employee ID from Azure AD. It could use the delegated User.Read permission that's already there, but to allow elevation of privileges for other calls it will use application permission to read the user's employee ID. For an explanation of application vs. delegated permissions, see [this documentation](https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#permission-types) or watch [this video](https://www.youtube.com/watch?v=SaBbfVgqZHc)

Select "Application permissions" to add the required permission.

![Adding an app permission](../Assets/01-019-RegisterAADApp-10.png)

You will be presented with a long list of objects that the Microsoft Graph can access. Scroll all the way down to the User object, open the twistie 1️⃣, and check the "User.Read.All" permission 2️⃣. Select the "Add Permission" button 3️⃣.

![Adding User.Read.App permission](../Assets/01-020-RegisterAADApp-11.png)

### Step 4: Consent to the permission

You have added the permission but nobody has consented to it. If you return to the permission page for your app, you can see that the new permission has not been granted. 1️⃣ To fix this, select the "Grant admin consent for <tenant>" button and then agree to grant the consent 2️⃣. When this is complete, the message "Granted for <tenant>" should be displayed for each permission.

![Grant consent](../Assets/01-024-RegisterAADApp-15.png)

#### Step 5: Expose an API

The Northwind Orders app is a full stack application, with code running in the web browser and web server. The browser application accesses data by calling a web API on the server side. To allow this, we need to expose an API in our Azure AD application. This will allow the server to validate Azure AD access tokens from the web browser.

Select "Expose an API" 1️⃣ and then "Add a scope"2️⃣. Scopes expose an application's permissions; what you're doing here is adding a permission that your application's browser code can use it when calling the server.

![Expose an API](../Assets/01-021-RegisterAADApp-12.png)

On the "Add a scope" flyout, edit the Application ID URI to include your ngrok URL between the "api://" and the client ID. Select the "Save and continue" button to proceed.

![Set the App URI](../Assets/01-022-RegisterAADApp-13.png)

Now that you've defined the application URI, the "Add a scope" flyout will allow you to set up the new permission scope. Fill in the form as follows:

* Scope name: access_as_user
* Who can consent: Admins only
* Admin consent display name: Access as the logged in user
* Admin consent description: Access Northwind services as the logged in user
* (skip User consent fields)
* Ensure the State is set to "Enabled"
* Select "Add scope"

![Add the scope](../Assets/01-023-RegisterAADApp-14.png)

### Exercise 6: Configure and run the application

#### Step 1: Download the starting application

The starting application is in GitHub at [https://github.com/OfficeDev/TeamsAppCamp1](https://github.com/OfficeDev/TeamsAppCamp1). Select the "Code" button and clone or download the content to your computer.

![Download the lab source code](../Assets/01-001-CloneRepo.png)

The starting code for the "A" path is in the A01-Start-AAD folder. Copy this folder to another location on your computer; this will be your working copy to keep the original source separate. Folders are also provided with the final code for the other labs.

#### Step 2: Install the app's dependencies

Using a command line tool of your choice, navigate to your working directory and type the command:

~~~shell
npm install
~~~

This will install the libraries required to run the server side of your solution.

#### Step 3: Configure the app settings

In a code editor, open the working folder you created in Step 2. Copy the *.env_sample* file to a new file called *.env* and open the new file. It will look like this:

~~~text
COMPANY_NAME=Northwind Traders
PORT=3978

HOSTNAME=something.ngrok.io
TENANT_ID=00000000-0000-0000-0000-000000000000
CLIENT_ID=00000000-0000-0000-0000-000000000000
CLIENT_SECRET=xxxxx
~~~

Fill in the information you've gathered so far, including your ngrok hostname and the information from the app registration.

#### Step 8: Run the application

To run the application, open a command line in your working folder and type:

~~~shell
npm start
~~~

At this point you should be able to browse to your ngrok URL and use the application. Note that due to the ngrok tunnel, you can try your app from anywhere on the Internet.

You will quickly be directed to the Microsoft login page.

Log in using one of the accounts you set up with an employee ID in Exercise 3, and you should be presented with the app's home page. The home page shows the employee name and picture from the Northwind database.

![Home page](../Assets/01-040-Run-1.png)

Select "My Orders" in the top navigation bar to view the employee's orders.

![My Orders page](../Assets/01-041-Run-2.png)

You can select on any order to view the details.

![Viewing an order](../Assets/01-042-Run-3.png)

From here you can select on any product to view its details. Much of the data is hyperlinked in this fashion.

You can also select on "Products" in the top navigation to view a list of product categories.

![View product categories](../Assets/01-043-Run-4.png)

From there you can select into a product category to view a list of products, and then you can select into a product to see its details. The product detail page shows all the orders for the product, which leads to a list of orders, and so you can select your way around the sample data.

Try logging out and logging in; you should be able to view the orders for another user in your developer tenant who has an employee ID set to a Northwind employee ID.

### Known issues

The application does not implement paging for large data sets, so lists of orders etc. are limited to the first 10 results.

While it will work on mobile devices, the application is not responsive and will not look good on these devices. This will be addressed in a future version of the lab.

### References
