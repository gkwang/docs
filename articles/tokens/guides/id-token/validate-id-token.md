---
title: Validate an ID Token
description: How to validate an ID Token.
toc: true
topics:
  - tokens
  - id-tokens
contentType:
  - how-to
useCase:
  - invoke-api
  - secure-api
---
# Validate an ID Token

To validate an ID Token, your application needs to:

1. Check that the ID Token is correctly formatted
2. Verify the ID Token's signature
3. Verify the claims found inside the ID Token

::: note
Most JWT libraries will take care of the token validation for you automatically, so be sure to reference the [Libraries for Token Signing/Verification section of JWT.io](https://jwt.io/#libraries-io) to find a JWT library for your platform and programming language.
:::

## Check that the ID Token is correctly formatted

Auth0 will generate the ID Token in <dfn data-key="json-web-token">JSON Web Token (JWT)</dfn> [format](/jwt#what-is-the-json-web-token-structure-). Before doing anything else, your app should parse the ID Token to make sure it conforms to the structure of a JWT.

A correctly formatted, or well-formed, JWT consists of three concatenated base64url-encoded strings, separated by dots (`.`):

* The **Header** contains metadata about the type of token and the cryptographic algorithms used to secure its contents.
* The **Body**, also called the payload, contains identity claims about a user. Additional custom claims can also be added to the payload, although in cases where the JWT will be returned via a URL, care must be taken to keep the JWT within browser size limitations for URLs.
* The **Signature** is used to validate that the token is trustworthy and has not been tampered with.

To see for yourself what is inside a JWT, use the [JWT.io Debugger](https://jwt.io/#debugger). It will allow you to quickly check that a JWT is well formed and manually inspect the values of the various claims.

## Verify the signature

The last part of a JWT is the signature, which is used to verify that the token was signed by the sender and not altered in any way. You will need to base64url-decode the signature to do this.

### Check the signing algorithm

Check that the algorithm (`alg`) specified in the header of the decoded token matches the one you [selected when you registered your Application with Auth0](/tokens/guides/update-signing-algorithm-application). 

### Confirm that the token is correctly signed using the proper key

Since the ID Token is always a JWT, the signature is created using its header and payload, a secret, and the selected hashing algorithm such as HMAC SHA256 or RSA. Thus, the method of verifying the signature depends on the hashing algorithm:

- For `RS256` (the most secure algorithm), the tenant's [JSON Web Key Set (JWKS)](/jwks) is used. Your tenant's JWKS is `https://${account.namespace}/.well-known/jwks.json`.
- For `HS256`, the API's __Signing Secret__ is used. You can find this information in the [Dashboard > APIs > Settings > Token Settings](${manage_url}/#/apis). This field is only displayed for APIs that use `HS256`.

## Verify the claims

Check the standard claims contained in the decoded token's payload:

* **Token expiration** (`exp`, Unix timestamp): The expiration date/time must be after the current date/time.
* **Token issuer** (`iss`): The issuing authority inside the token must match the issuing authority identified in your Auth0 tenant's discovery document, which exists at `https://${account.namespace}/.well-known/openid-configuration`.
* **Token audience** (`aud`): The token audience identifies the recipient for which the token is intended. The value must match the Client ID of your Auth0 Application.
