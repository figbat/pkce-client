# Demo Background

This is a Node.js native app that runs from the command line. It's as lean as possible to focus on the mechanism of the flow in action. As such, it only has 4 dependencies: `commander` for parsing command line switches, `opn` for a platform-independent way to launch a browser from the command line, `request` to make HTTP requests and `restify` to provide a RESTful API endpoint to listen on.

The Auth Code with PKCE Flow will replace the Implicit Flow over time. This will strengthen SPA apps by reducing the surface area of attack.

[AppAuth-JS](https://github.com/openid/AppAuth-JS) is a high quality JavaScript library that supports the Auth Code with PKCE Flow and hides away a lot of the details of what’s happening during the flow. It can be utilized in a SPA.

## Why not Implicit Flow?

Here's an overview of the Implicit flow :

![implicit](implicit.png)

1. You access the app in your browser and click its login button
2. The app responds with a redirect to the browser
3. The browser follows the redirect to Identity Provider (Okta)
4. Identity Provider (Okta) returns a hosted login form
5. You submit your username and password directly to Identity Provider (Okta)
6. Identity Provider (Okta) authenticates you and sends a redirect with a token in the URL
7. The browser follows the redirect to your Node app, which parses the token out of the URL

This has been a solid approach for SPA apps. It’s a flow that gets your app a token in an untrusted environment without revealing any secrets. But it’s always had a security issue. Notice steps 6 and 7 above involve a redirect back to the Node app in the browser. Redirects are HTTP GETs. As such, the only opportunity to get a token back to the app is include it in the **URL**. _This is problematic from a security standpoint since the token is now sitting in the browser history._

**It’s especially problematic if the token is long lived.** A security professional would say there's an _increase in the attack surface area._ An example of why this is a problem is that many people sync their browser history across multiple devices further expanding the attack surface.

It is also possible to see the token in the browser history

## Command Line

This tool demonstrates the Authorization Code Flow with PKCE. It is a copy of a sample program from OKTA developer portal and requires an account registration and configuration of the IDP (Identity Provider) developer domain associated with the account.

It follows these steps:

1. Creates a random string called the `code verifier`
2. Hashes the `code verifier` creating a value called the `code challenge`
3. Builds an authorization URL which includes:
    a.) OIDC Client ID
    b.) List of request scopes
    c.) Redirect uri
    d.) Randomly generated state value
    e.) The `code challenge`
    f.) Response type set to `code` to indicate that we're using the authorization code flow
4. Launches a browser with the authorization URL, at which point you will authenticate
5. Receives the redirect from the authorization url, which includes a `code`
6. Calls the `token` endpoint with:
    a.) Grant type set to `authorization code`
    b.) Redirect uri that must match the one used in the authorization step
    c.) OIDC Client ID
    d.) The `code`
    e.) The `code verifier` from earlier
7. Displays the tokens returned from the `token` endpoint
8. Uses the returned access token to call the `userinfo` endpoint

## Prerequisites
Install [NodeJS](https://nodejs.org/en/) for your operating system.
Install [Python](https://www.python.org/downloads/) for your operating system.

Add OKTA for user management (Identity Provider)
1. Create yourself a developer account at https://developer.okta.com/signup 
2. Go to the admin console and create an application. Browse to **Applications**. Click **Add Application**. Choose **Native**. Then Select **Next**:
![idp1_app](idp1_app.png)
3. Update the value for Login redirect URIs to: http://localhost:8080/redirect. Select Authorization Code and Refresh Token. Choose **Done**.
![idp2_grant](idp2_grant.png)
4. Note the Application ID generated.
![idp3_id](idp3_id.png)
5. Note the API Server URL under the API tab in OKTA.
![idp4_api](idp4_api.png)

## Install

```
npm install

```
## Usage

```
Usage: pkce-client [options]

Options:
  -c, --client_id <okta client id>               OIDC Client ID (default: "")
  -i, --idp_org <idp org url>                   ex: https://dev-532307.okta.com (default: "")
  -s, --scopes <space separated list of scopes>  Space separated list of scopes (default: "")
  -r, --redirect_uri <redirect uri>              redirect uri (default: "")
  -h, --help                                     output usage information
```
## Run
 The npm pkce script has the following command: \
 node pkce-client \
  --client_id 0oar16ap61JZJIbYp356 \
  --idp_org https://dev-532307.okta.com \
  --scopes "openid profile email offline_access" \
  --redirect_uri http://localhost:8080/redirect 
```
npm run pkce 

```

You'll get output like this:

```
Created Code Verifier (v): 191c_eaa4_01f6_dc74_1374_7859_04d5_fa7b_1729_9611_a897

Created Code Challenge ($): 6hE8f6iL_YJafzSxSiO_zrenB_D_cuy9sDkqutBstHw

About to call Authorize URL: https://dev-532307.okta.com/oauth2/default/v1/authorize?client_id=0oar16ap61JZJIbYp356&response_type=code&scope=openid profile email offline_access&redirect_uri=http://localhost:8080/redirect&state=e5ce_463b_51fa_a453_1105_7f2e_8ea0_019a_2d15_4cab_3ba2&code_challenge_method=S256&code_challenge=6hE8f6iL_YJafzSxSiO_zrenB_D_cuy9sDkqutBstHw

press any key to continue...

Got code (α): pPbru1AcBH1_WoePVW5f

Got state (s): e5ce_463b_51fa_a453_1105_7f2e_8ea0_019a_2d15_4cab_3ba2

press any key to continue...

Calling /token endpoint with:
client_id: 0oar16ap61JZJIbYp356
code_verifier (v): 191c_eaa4_01f6_dc74_1374_7859_04d5_fa7b_1729_9611_a897
code (α): pPbru1AcBH1_WoePVW5f

Form post to be sent to the /token endpoint:
{ grant_type: 'authorization_code',
  redirect_uri: 'http://localhost:8080/redirect',
  client_id: '0oar16ap61JZJIbYp356',
  code: 'pPbru1AcBH1_WoePVW5f',
  code_verifier: '191c_eaa4_01f6_dc74_1374_7859_04d5_fa7b_1729_9611_a897' }

press any key to continue...

Token response:
{ access_token:
   'eyJraWQiOiJNYWhtRXBOeWlWckIxQUdUV0p3YTJvb0VrcHNUdTdYNzNLbXF6V2Yxb3pBIiwiYWxnIjoiUlMyNTYifQ.eyJ2ZXIiOjEsImp0aSI6IkFULllqRWdJb19rZ2h4OVB4NlNGcFZYZTJrM2hKT082Wk43WlVPWWhEanhfXzAuTHppbFFnTVVkY0ZVNVI1d3ZPM2YxakxwM01HT1YyVHZORFl6UEJNdXpxST0iLCJpc3MiOiJodHRwczovL2Rldi01MzIzMDcub2t0YS5jb20vb2F1dGgyL2RlZmF1bHQiLCJhdWQiOiJhcGk6Ly9kZWZhdWx0IiwiaWF0IjoxNTYxNjU1NTU4LCJleHAiOjE1NjE2NTkxNTgsImNpZCI6IjBvYXIxNmFwNjFKWkpJYllwMzU2IiwidWlkIjoiMDB1cjB0MXVvdUlvalkzUzgzNTYiLCJzY3AiOlsiZW1haWwiLCJvZmZsaW5lX2FjY2VzcyIsInByb2ZpbGUiLCJvcGVuaWQiXSwic3ViIjoiYnJpYW5yYWZhbG93c2tpQGdtYWlsLmNvbSJ9.oHDxU_uZNaiWfwPfmYaxCBmg-qfCLhurySG8mA2mHSAmGsApgQ_ajNqGmb-QhSdu82CjycUuf2TkvZX0yZHgptpXjE8krzB_1Z4Uw6b-cjn7lMIJxY-HAsHvGIz_tyC5fOXsJkYf9pqNffcGgfngQ76-KhuQesbxRFMavOlopLrcMe6OzowgycGQfTyAiikLA5Yg2ueC52udMqeUG747oS2z4BTOgWcPey5ZK7ezTT1PAr3cA_GnCOoqJro2meN0yUefZ-QaAGSPM7dKzUpuBWG9CRCSWQ6iN8R5jSCw0q_TmfZWgx2phXdYWbGDN10CyTpl84HJb3gYy-ai6i6Z8g',
  token_type: 'Bearer',
  expires_in: 3600,
  scope: 'email offline_access profile openid',
  refresh_token: 'Nie0ymUUfl628yMjJ86c1ULCXsoKuTxiG9v-DEKbN-c',
  id_token:
   'eyJraWQiOiJNYWhtRXBOeWlWckIxQUdUV0p3YTJvb0VrcHNUdTdYNzNLbXF6V2Yxb3pBIiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiIwMHVyMHQxdW91SW9qWTNTODM1NiIsIm5hbWUiOiJCcmlhbiBSYWZhbG93c2tpIiwiZW1haWwiOiJicmlhbnJhZmFsb3dza2lAZ21haWwuY29tIiwidmVyIjoxLCJpc3MiOiJodHRwczovL2Rldi01MzIzMDcub2t0YS5jb20vb2F1dGgyL2RlZmF1bHQiLCJhdWQiOiIwb2FyMTZhcDYxSlpKSWJZcDM1NiIsImlhdCI6MTU2MTY1NTU1OCwiZXhwIjoxNTYxNjU5MTU4LCJqdGkiOiJJRC5ja1V6enoyV1BWY0xmTUV1b1JCa2dPZkl4YlExN24yd2x1dHVxc0cyZ0FZIiwiYW1yIjpbInB3ZCJdLCJpZHAiOiIwMG9yMHQxcmsyOGZYUUp2RTM1NiIsInByZWZlcnJlZF91c2VybmFtZSI6ImJyaWFucmFmYWxvd3NraUBnbWFpbC5jb20iLCJhdXRoX3RpbWUiOjE1NjE2NTU1NDMsImF0X2hhc2giOiJlZklOSGFBNGxLb2l0T2wxaG1IX1hRIn0.JNHVYUa4nhSH5y9lPXCgwe4MEh-uOJskN25nAuC34nLjeQ7o__x_98o5b6D68V2hwQkTbENGixfht_spSAG6ktDssReaFyW7_JEApx1AG4B0qtnT7ExC2-pOCt5FFFPRQCVYjstcBoaQSKrX4594_YifJg4zDwwoOxPI2fi8uJ1wNHiUfTjE9VXH3DShpImsNrWbV-nkqZdAsZrhcNgqPfPK2enBeGbwHNNl9hHEQoNV8gqCq_3comZql-vJrR1GA21-oT0Uf0JMnaUlGXgURUtHw1bJDyn_QzSrtWcyNUnt9vLlHtanyixgvOVcm0N5yp7b0BfF2v09UpWdm5VP5Q' }

press any key to continue...

Calling /userinfo endpoint with access token

{ sub: '00ur0t1uouIojY3S8356',
  name: 'Brian Rafalowski',
  locale: 'en-US',
  email: 'brianrafalowski@gmail.com',
  preferred_username: 'brianrafalowski@gmail.com',
  given_name: 'Brian',
  family_name: 'Rafalowski',
  zoneinfo: 'America/Los_Angeles',
  updated_at: 1561403556,
  email_verified: true }
```

## Diagrams

Here's an overview of the Authorization Code with PKCE flow:

![pkce](pkce.png)

Note: This image was generated using [mermaid](https://mermaidjs.github.io/). The source is [here](pkce.mmd).
The package is installed locally since there are issues with the global install (npm install -g mermaid)

You can edit and regenrate the image using this command:

```
node_modules\.bin\mmdc -i pkce.mmd -o pkce.png -b transparent -C mmdc.css
node_modules\.bin\mmdc -i pkce.mmd -o pkce.svg -C mmdc.css
``` 
