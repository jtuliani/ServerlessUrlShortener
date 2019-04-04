# ServerlessUrlShortener
This project is for a simple, highly scalable and performant 'bitly'-like URL shortener, implemented entirely using serverless technologies in Microsoft Azure

## Set Up Instructions

So far, the set up instructions are entirely manual. I created these to give a talk that creates the entire solution from scratch. They' weren't written for public distribution, so they might be a bit brief. I'm sharing them here since I've had a number of requests for the source following that talk.

Apologies if they're hard to understand. Over time, I plan to improve and automate.

### Function App / DNS
Create function app 'urii'. Use a new resource group admsusRG; North Europe; defaults

Go to func, platform features, register domain name urii.org (or choose your own!)
Assign root domain only

### Redirect Function
Create function 'Redirect', HTTP trigger, authorization anonymous
Show it working from in browser experience
Show logs
Show it working (e.g. http://admsus.azurewebsites.net/api/Redirect?name=Jonathan)

Click 'Integrate', New Input, Table Storage
Install extension
parameter lookupTable; table name lookup
Storage account connection: new, choose storage account created with function

Storage explorer
Create lookup table in storage account
Create row PK = ab, RK = c, uri = http://www.jet.com

Replace code with code from 'Redirect'
Test from browser: set HTTP GET, query string shortcode = abc

THIS SECTION NOW OBSOLETE?
500 Internal Server Error!
But logs look OK?
https://stackoverflow.com/questions/54291473/azure-functions-redirectresult-causing-a-http500-error
https://github.com/Azure/azure-functions-host/issues/3986
https://www.yammer.com/azureadvisors/#/threads/show?threadId=1228117048
Go to platform features > Application settings
Show connection string for table binding
Change FUNCTION_EXTENSION_VERSION to 2.0.12246.0

Run the function - NOW IT WORKS!

Explain URL is long, explain proxy functions
Create 'RedirectProxy': template {n}, GET only, URL http://admsus.azurewebsites.net/api/redirect?shortcode={n}
Show it works: http://admsus.org/abc

### CreateShortcut Function
Create new HTTP Trigger function CreateShortcut
Add table input binding same as before
If doesn't offer table input, used advanced editor and copy from Redirect function
Copy code from 'CreateShortcut', walk through and explain
Set up Application setting DomainName, value 'admsus.org'
Show it works (create new, existing, and redirect)

Show data in storage explorer

### KeepAwake Function
Explain sleepiness / cold start
Create KeepAwake timer function, schedule 0 0/2 * * * *
Copy code from 'KeepAwake'

### Web Site
Copy 'index.html' locally, open in browser
Edit domain name
Show it doesn't work - Debug tools - CORS restriction
Change function app CORS settings to '*' (only) and now it works

Explain blob-based website hosting
Upgrade account kind (from 'Configuration') to v2
Enable static website hosting (index.html, error.html)
Upload HTML files as blobs (may have to edit to use urii.org domain name or whatever domain you chose in the JavaScript)
Create DNS entry CNAME to point to DNS name of website (copy from static website page)
Enable custom domain in blob settings (just add www.urii.org)

Show http://urii.org shows Azure Functions holding page
Create proxy function 'RedirectRoot' to remedy:
Path '/'
Response override: 301 Moved Permanently, header 'Location' 'http://www.urii.org'

App now working end to end !!
