# 2023-training-project

## Introduction

You won't be creating a game changing product with these requirements, but you will touch on a few new libraries and technologies.

## The Project: Receipt Tracker

You have been requested to create a tracking system for invoices that will be later used downstream in expense systems.

## Technology

You will be using the following technologies:

### NextJS
You will be using this for both the front end and back end.

### ApolloJS
You will be using this both on the front end and back end to query for data.
The back end will require you to setup;
- Queries: for simple get style requests
- Mutations: for receipt upload
- Subscriptions: for watching for changes and pushing them via web sockets to the customer

### MongoDB 
You will be using this as your database. You can register for a free instance here at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/lp/try4?utm_source=bing&utm_campaign=search_bs_pl_evergreen_atlas_core_prosp-brand_gic-null_emea-gb_ps-all_desktop_eng_lead&utm_term=mongodb%20atlas&utm_medium=cpc_paid_search&utm_ad=e&utm_ad_campaign_id=415204562&adgroup=1217159841720713&msclkid=149276a3da6f15aab53c1386c3a892ad)

NOTE: You can subscribe to changes in your collection then push them via a graphql subscription. Details [here](https://www.mongodb.com/docs/drivers/node/current/usage-examples/changeStream/)

### OpenAI
You will be using GPT4 to parse receipts for data. API details can be found [here](https://platform.openai.com/docs/models/gpt-4).
You will need to add $10 of credit to enable access to the GPT4 model (the free credit only applies to GPT3.5). The older model doesn't support reading images.

### Zustand
You will be using Zustand to manage local state within the app (i.e. authentication, ui state etc.)

### Tailwind
You will be using tailwind for themeing. Where possible avoid custom CSS.

### Heroku
You will use Heroku to host your application, NOT vercel.
Vercel serves your app as serverless functions. This means, after a request is completed, the instance it closed. This will not allow for websockets and graphql subscriptions.

## Requirements

### Login

There will be a simple login / registration flow.

- When a user clicks login,
  - If the user has an account, send an email to the users email with a "magic link" back to the site with a token.
  - When they click the link, the front end then uses the token to authenticate them
  - If the user has no account, then email them with an error email, informing them they don't have an account, give them a link that deeplinks to the registration screen.
- When the user clicks register,
  - Ask for an email and name, then send them an email with a "magic link" that will login them in on the site with a token.
 
### Homepage

There will be a simple UI with a table that lists receipts they've previously uploaded.
On each row include;
- Receipt ID (as from the database)
- Receipt date
- Receipt vendor / supplier
- Receipt amount
- Receipt status [Processing / Processed]

You will use a graphql QUERY on init to fetch the receipts.
The response will be paginated.
After initial loading, SUBSCRIBE to the query to dynamically update the table when the data for the user changes.

There will be a single CTA "Upload Receipt". This will open the Receipt upload screen

### Receipt upload screen

Prompt the user for an receipt to upload.
When they have confirmed the image is valid then;
- Store the image (for now just use mongo as a storage mechanism via [gridfs](https://www.mongodb.com/docs/manual/core/gridfs/))
- Create a new entry in your database
- Close the modal

If the subscription is properly setup, the user will see the new document on the homepage with the status "Processing"

### Receipt processor

This will be an async processor of newly upload receipts.
It will only process one job at a time.
It will take a receipt image then;
- submit it to GPT4 and request it analyzes the document and returns a JSON response structured to include;
 - Receipt Date
 - Receipt vendor / supplier
 - Receipt amount
 - Receipt line items [for each capture the name of the product/service, the quantity and the subtotal of the line item)
 - The sub total for the reciept
 - The tax applied to the subtotal
 - The total amount paid
 - [BONUS] which payment method was used to pay for the purchase
- Store this returned data to the database, then set the record to processed

Again if the subscription is properly setup, the user will see the updated status on the document.

[BONUS] trigger a toaster like message that on click takes them to the receipt preview page

### Receipt preview page

This will show;
- The image of the receipt
- The status along with all the data captured from the [receipt processor](#receipt-processor)

There should be an edit button, which will convert all pieces of information previously read only to be editable.
The user should be allowed to save and update the document for mistakes made by the AI.
