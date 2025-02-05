# PID/mDL OID4VCI Draft 12 - version 0.4 (dynamic registration (AUTH))

Generates the nonce and credential of the PID/mDL in different formats (mdoc and SD-JWT) through a series of POST and GET requests.

Available *country* codes, for testing:

PID:
+ FC (Form Country) - a form, with the necessary PID/mDL attributes, will be presented to the user (EUDI Wallet holder). The user will insert the values, that will not be verified;
+ CW (country W) - the user will be redirected to the eIDAS node of country W (user:xavi, password:creus);
+ CZ (Czech Republic) - the user will be redirected to the eIDAS node of Czech Republic ;
+ EE (Estonia) - the user will be redirected to the Identity Provider of Estonia (https://e-gov.github.io/TARA-Doku/Testing#21-quick-reference-of-testing-accounts);
+ PT (Portugal) - the user will be redirected to the Identity Provider of Portugal .

mDL:
+ FC (Form Country) - a form, with the necessary PID/mDL attributes, will be presented to the user (EUDI Wallet holder). The user will insert the values, that will not be verified;
+ PT (Portugal) - the user will be redirected to the Identity Provider of Portugal .

![image](https://github.com/devisefutures/eudiw-issuer/assets/61158161/375ff784-d828-4917-9a65-4cebcd055ba8)


## 1. Get Metadata 
It starts by making a GET request for metadata.

<https://issuer.eudiw.dev/oidc/.well-known/openid-configuration>

## 2. Dynamic Registration Request
Next, a POST request is made to perform Dynamic Registration.

for this request, the following fields need to be included:

+ *application_type* - “native”     
+ *response_types* - [“code”]
+ *redirect_uris*  - Rfc6749 see section 3.1.2
+ *grant_types* - ["authorization_code"]
+ *subject_type* - Rfc6749 see section 2.1
+ *id_token_signed_response_alg* - “ES256”
+ *userinfo_signed_response_alg* - “ES256”
+ *request_object_signing_alg* - “ES256”
+ *token_endpoint_auth_method* - “public”
+ *token_endpoint_auth_signing_alg* - “ES256”
+ *default_max_age* - 86400
+ *response_modes* - [“query”,”fragment”,”form_post”]

at the end of this request, a JSON response is sent, containing the following data:

+ *client_id* - Rfc6749 see section 2.2
+ *client_secret* - Rfc6749 see section 2.3.1
+ *coderegistration_access_token*
+ *client_secret_expires_at*
+ *registration_client_uri*

## 3. Push Authorization Request

Now it's necessary to save both the *"client_id"* and *"client_secret"* from the previous request (see section 2) to be used in the GET request for Push Authorization.

Both the *"client_id"* and *"client_secret"* will be used for authentication, and the following data is also required:

+ *response_type* - “code”
+ *state*
+ *client_id* - Rfc6749 see section 2.2
+ *code_challenge* - RFC 7636  see section 4.2
+ *code_challenge_method* - RFC 7636  see section 4.2
+ *scope* - “org.iso.18013.5.1.mDL openid” or “eu.europa.ec.eudiw.pid.1 openid”

At the end of this request, you receive a JSON that contains:

+ *expires_in* - RFC 9126 see section 2.2
+ *request_uri* - RFC 9126 see section 2.2

## 4. Authorization Request

It is necessary to save the *"request_uri"* from the previous request (see section 3) to be used in the GET request for Authorization.

In addition to the *"request_uri"* the following data is required:

+ *redirect_uri* - URL Encoded URI 
+ *response_type* - Required. Value must be “code”
+ *nonce* - String
+ *scope* - Optional. The scope of the access request RFC6749 3.3
+ *state* - Recommended. Opaque valued used by the client to maintain state between requests
+ *client_id* - Required. The client identifier as described in RFC6749 2.2
+ *request_uri* - RFC 9126 see section 2.2

In the end, a redirect is made to a website where the user needs to select one of the countries for which authentication is required to obtain user data from the PID. 
At the end of this authentication, the follow parameters are obtained:

+ *state* - Required. Same value received from client
+ *scope*
+ *code* - Required. Authorization code generated by authorization server 
+ *iss*
+ *client_id*

## 5. Token request

The "code" and "state" (see section 4) need to be saved to make the POST request for the Token.

The attributes required to make the request are:

+ *grant_type* - Required. Value must be set to “authorization_code”
+ *code* - Required. The authorization code received from server
+ *redirect_uri* - Required. Must be identical value to authorization requrest redirect_uri
+ *client_id* - Required.
+ *state* - Recommended. 
+ *code_verifier* - RFC 7636 see section 4.1

At the end of this request, you get a JSON with:

+ *token_type*
+ *scope*
+ *access_token*
+ *expires_in*
+ *id_token*

## 6. Credential Request

Finally, the last required request is the POST request for Credentials.

To make this request, you need to save the *"access_token"* from the previous request (see section 5), and additionally, the following attributes are required:

+ *format* - Required. Format of credential to be issued. (vc+sd-jwt / mso_mdoc)
+ *doctype* - For mso_mdoc format
+ *proof* - Optional  JWT

In the end, you receive a JSON with:

+ *c_nonce* - Required. JSON string containing a nonce
+ *c_nonce_expires_in* - Required. Json integer denoting lifetime of c_nonce
+ *error*
+ *error_description*
+ *credential* - Optional. Contains issued Credential
+ *format* - Required. Json string denoting format
