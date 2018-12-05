---

copyright:
  years: 2015, 2018
lastupdated: "2018-12-05"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:deprecated: .deprecated}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# Authenticating with IAM tokens
{: #iam}

You use {{site.data.keyword.cloud}} Identity and Access Management (IAM) tokens to make authenticated requests to {{site.data.keyword.ibmwatson}} services without embedding service credentials in every call. IAM authentication uses access tokens for authentication, which you acquire by sending a request with an API key.
{: shortdesc}

{{site.data.keyword.watson}} services that are created in a resource group or are migrated from Cloud Foundry to a resource group use IAM authentication. By default, all new {{site.data.keyword.watson}} services use IAM authentication.
{: note}

## Getting IAM service credentials
{: #getting-credentials-manually}

To access API methods by using IAM service credentials, you must first collect the credentials. You can access the service credentials from the {{site.data.keyword.cloud_notm}} web interface.

1.  Log in to [{{site.data.keyword.cloud_notm}} ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://{DomainName}){: new_window}.
1.  Go to your [resource list ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://{DomainName}/resources){: new_window}.
1.  Select a service instance.
1.  Click **Show Credentials** to see the **API Key** and **Url** for the credentials.
    1.  To see the tokens for the credentials, select **Service credentials** from the navigation menu on the left of the page.
    1.  Select **View credentials** to view the full API key and token information for the credentials.

        ```json
        {
          "apikey": "3df...   ...Y7Pc9",
          "iam_apikey_description": "Auto generated apikey during resource-key operation for...",
          "iam_apikey_name": "auto-generated-apikey-31b336bc-2d6a-41c3-a8b2-e05ec6db19b4",
          "iam_role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager",
          "iam_serviceid_crn": "crn:v1:bluemix:public:iam-identity::a/57d48380...::serviceid:...",
          "url": "https://gateway.watsonplatform.net/{service_name}/api"
        }
        ```

## Updating IAM service credentials
{: #existing-svcs}

You can update service credentials for an existing service instance from the service dashboard.

1.  Log in to [{{site.data.keyword.cloud_notm}} ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://{DomainName}){: new_window}.
1.  Go to your [resource list ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://{DomainName}/resources){: new_window}.
1.  Select **Service credentials** from the navigation menu on the left of the page.
1.  Use the menus and icons to delete existing credentials or to add new credentials.

Make sure that you update the credentials in your applications for any changes.

## Getting an IAM token by using a Watson service API key
{: #iamtoken}

You can access {{site.data.keyword.ibmwatson_notm}} service APIs by using the API keys that were generated for the service instance. You use the API key to generate an IAM access token. You also use this process if you are developing an application that needs to work with other {{site.data.keyword.cloud_notm}} services.

For more information about securely using API keys, see [IAM service API keys](/docs/services/watson/apikey-bp.html).
{: tip}

The following curl command uses the `POST identity/token` method to generate an IAM access token by passing an API key. The `Content-Type` header indicates that the data is URL encoded. The `data-urlencode` parameters pass the URL-encoded data to the method.

```bash
curl -k -X POST \
--header "Content-Type: application/x-www-form-urlencoded" \
--header "Accept: application/json" \
--data-urlencode "grant_type=urn:ibm:params:oauth:grant-type:apikey" \
--data-urlencode "apikey={apikey}" \
"https://iam.bluemix.net/identity/token"
```
{: pre}

The following sample shows the expected response:

```javascript
{
  "access_token": "eyJhbGciOiJIUz......sgrKIi8hdFs",
  "refresh_token": "SPrXw5tBE3......KBQ+luWQVY",
  "token_type": "Bearer",
  "expires_in": 3600,
  "expiration": 1473188353
}
```
{: pre}

## Using a token to authenticate
{: #use_token}

You generate an IAM access token by using the `POST identity/token` method. You can then use the token to make authenticated API calls. An API key is associated with a specific service instance. A token that is generated with an API key can be used only for calls to that service instance.

A typical curl call to a {{site.data.keyword.watson}} service takes the following form, where `{token}` is the IAM access token that you generated:

```bash
curl -X GET \
--header "Authorization: Bearer {token}" \
"https://gateway.watsonplatform.net/discovery/api/v1/environments?version=2017-11-07"
```
{: pre}

Alternatively, you can prefix the token with `Authorization: Bearer ` (for example `Authorization: Bearer eyJhbGciOiJIUz......sgrKIi8hdFs`) and save it to a file. You then use the following command:

```bash
curl -X GET \
--header "$(cat token.txt)" \
"https://gateway.watsonplatform.net/discovery/api/v1/environments?version=2017-11-07"
```
{: pre}

## Renewing the token
{: #renew_token}

The bearer token has a limited lifespan and a new bearer token must be periodically fetched.  Before the bearer token expires use the same api call as above to get a new one.

```bash
curl -k -X POST \
--header "Content-Type: application/x-www-form-urlencoded" \
--header "Accept: application/json" \
--data-urlencode "grant_type=urn:ibm:params:oauth:grant-type:apikey" \
--data-urlencode "apikey={apikey}" \
"https://iam.bluemix.net/identity/token"
```
{: pre}