# Sample Lambda Authorizer for AWS API Gateway
### This sample is based on https://github.com/mcguinness/node-lambda-oauth2-jwt-authorizer by Karl McGuinness. Karl's original README can be found on his github repo at [https://github.com/mcguinness/node-lambda-oauth2-jwt-authorizer](https://github.com/mcguinness/node-lambda-oauth2-jwt-authorizer). A modified version, including changes made for this sample, is included below.

# OAuth 2.0 Bearer JWT Authorizer for AWS API Gateway

This project is sample implementation of an AWS Lambda custom authorizer for [AWS API Gateway](https://aws.amazon.com/api-gateway/) that works with a JWT bearer token (`id_token` or `access_token`) issued by an OAuth 2.0 Authorization Server.  It can be used to secure access to APIs managed by [AWS API Gateway](https://aws.amazon.com/api-gateway/).

## Configuration

### Environment Variables (.env)

Update the `ISSUER` and `AUDIENCE` variables in the `.env` file

```
ISSUER=https://example.oktapreview.com/oauth2/aus8o56xh1qncrlwT0h7
AUDIENCE=https://api.example.com
```

It is critical that the `issuer` and `audience` claims for JWT bearer tokens are [properly validated using best practices](http://www.cloudidentity.com/blog/2014/03/03/principles-of-token-validation/).  You can obtain these values from your OAuth 2.0 Authorization Server configuration.

The `audience` value should uniquely identify your AWS API Gateway deployment.  You should assign unique audiences for each API Gateway authorizer instance so that a token intended for one gateway is not valid for another.

### Signature Keys (keys.json)

Update `keys.json` with the JSON Web Key Set (JWKS) format for your issuer.   You can usually obtain the JWKS for your issuer by fetching the `jwks_uri` published in your issuer's metadata such as `${issuer}/.well-known/openid-configuration`.

> The authorizer only supports RSA signature keys

> Ensure that your issuer uses a pinned key for token signatures and does not automatically rotate signing keys.  The authorizer currently does not support persistence of cached keys (e.g. dynamo) obtained via metadata discovery.

### Scopes

This sample currently enforces scope-based access to API resources using the `scp` claim in the JWT.  The `api:read` scope is required for `GET` requests and `api:write` scope for `POST`, `PUT`, `PATCH`, or `DELETE` requests.

Update `index.js` with your authorization requirements and return the resulting AWS IAM Policy for the request.

# Deployment

### Install Dependencies

Run `npm install` to download all of the authorizer's dependent modules. This is a prerequisite for deployment as AWS Lambda requires these files to be included in the uploaded bundle.

### Create Bundle

You can create the bundle using `npm run zip`. This creates a oauth2-jwt-authorizer.zip deployment package in the `dist` folder with all the source, configuration and node modules AWS Lambda needs.


### Test your endpoint remotely

#### With Postman

You can use Postman to test the REST API

* Method: < matching the Method in API Gateway >
* URL `https://<api-id>.execute-api.<region>.amazonaws.com/<stage>/<resource>`
 * The base URL you can see in the Stages section of the API
 * Append the Resource name to get the full URL
* Header - add an Authorization key
 * Authorization : Bearer <token>

#### With curl from the command line

    $ curl -X POST <url> -H 'Authorization: Bearer <token>'

#### In (modern) browsers console with fetch

    fetch( '<url>', { method: 'POST', headers: { Authorization : 'Bearer <token>' }}).then(response => { console.log( response );});
