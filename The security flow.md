# The security flow

*This article is part of a set of tutorial about Open Banking UK, although, they are all written to be read as a standalone article. If you are interested in learning about Open Banking UK from the beginning, you may want to start by our intro article.*

*If you wish to execute the request at the same than the article, please note that it assumes you are already onboard. If it is not your case, please follow the article X*

If you are not interested yet about the security aspect of Open Banking, you may want to skip that article and go directly to the next one.
In this article, we will cover the security theory behind Open Banking, which will help you understand where the apparent complexity comes from.

## The goal

What we want to achieve, is having three different parties, communicating to each others, in a secured way.
Basically, as a **developer**, you want to access the **user** data stored in his **bank**. 

The main security difference with APIs you have used so far, is probably due to the above. An API key would not work here, just because it's not your own data you are accessing. What we want is to get a token, that contains inside somehow the fact that the user agreed to share his data.

We are going to decompose the flow in six parts:

- Authentication and consent with the developer
- Consent preparation
- Authentication and consent with the bank
- Token exchange
- Data access
- Data rendering

For simplicity, we are going to take the account and transactions APIs example. Note that the flow is very similar for payments and confirmation of funds.

## Authentication and consent with the developer

The flow starts with Alice and you. Alice wants to consume your amazing service and for that, you and Alice agree on the Alice data you will request access.
You got full liberty on how you want to achieve this. You just need to make sure you authenticated Alice correctly and you details to Alice all the type of her data you will request access to the bank.
We won't describe further this step, you got full control of this user experience. I let you imagine what you can do here.

## Consent preparation

The first step is to prepare the consent access. Only you and the bank is involved in that process. This consist of creating a consent object into the bank. Inside this consent, you will put the permissions you wish to access and some optional informations like the expiration of the consent.

The bank is protecting their endpoints with OAuth 2, even the consent creation endpoint, meaning you will need an access token to create the consent object.
There are different grant flows in OAuth 2, we are going to see two of them today. The one that interest us for the consent creation is the client credential grant flow.
If you are familiar with this standard, as a developer, you are the OAuth 2 client and you want to create a resources that you will own. For that, you need to use the client credential grant flow.
This flow will not be covered in details in this tutorial, we will concentrate on showing how to use it.

To create the access token:

```
POST /oauth2/access_token HTTP/1.1
Host: matls.as.aspsp.ob.forgerock.financial
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&scope=accounts
```

As a response:

```
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJjdHMiOiJPQVVUSDJfR1JBTlRfU0VUIiwiYXVkaXRUcmFja2luZ0lkIjoiMDU3N2I2MTYtMzYyMS00YjA3LWIxNGEtMzMyYTBkZWZiNTMyLTExMDAxNyIsImlzcyI6Imh0dHBzOi8vYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbC9vYXV0aDIiLCJ0b2tlbk5hbWUiOiJhY2Nlc3NfdG9rZW4iLCJ0b2tlbl90eXBlIjoiQmVhcmVyIiwiYXV0aEdyYW50SWQiOiJaMWIyTnR3Z2YzY091XzBwUlB0U2dpZm53LWMuSEJaaTlaYW1UbWo5cTM3SU0zTXJ1S3k4YjNZIiwiYXVkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwibmJmIjoxNTczNDA1MjgwLCJncmFudF90eXBlIjoiY2xpZW50X2NyZWRlbnRpYWxzIiwic2NvcGUiOlsiYWNjb3VudHMiXSwiYXV0aF90aW1lIjoxNTczNDA1MjgwLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3MzQ5MTY4MCwiaWF0IjoxNTczNDA1MjgwLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiWjFiMk50d2dmM2NPdV8wcFJQdFNnaWZudy1jLkJJOFlNUkZyQzdkUFlDVFM3TmFVWERKY0lnayJ9.i7ZD4i8b53h_tD2yFjURZdmpL6HmVU3rfJGS-x30GuTiYbwYgGcVitEHeoXFd3D4MIwwo_4SWUDfb68TCOuTjQ",
    "expires_in": 86399,
    "token_type": "Bearer",
    "scope": "accounts"
}
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Accounts flow` > `Access preparation` > `‌Client credential`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#21dcd60b-4a78-4523-8a29-ac2677fbb37c

It uses the `tls_client_auth` auth method to authenticate yourself to the bank, via your client certificate. It is why only you can get this access token.

You can now use it to create the account access consent:


```
POST /open-banking/v3.1.1/aisp/account-access-consents HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJjdHMiOiJPQVVUSDJfR1JBTlRfU0VUIiwiYXVkaXRUcmFja2luZ0lkIjoiMDU3N2I2MTYtMzYyMS00YjA3LWIxNGEtMzMyYTBkZWZiNTMyLTExMDAxNyIsImlzcyI6Imh0dHBzOi8vYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbC9vYXV0aDIiLCJ0b2tlbk5hbWUiOiJhY2Nlc3NfdG9rZW4iLCJ0b2tlbl90eXBlIjoiQmVhcmVyIiwiYXV0aEdyYW50SWQiOiJaMWIyTnR3Z2YzY091XzBwUlB0U2dpZm53LWMuSEJaaTlaYW1UbWo5cTM3SU0zTXJ1S3k4YjNZIiwiYXVkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwibmJmIjoxNTczNDA1MjgwLCJncmFudF90eXBlIjoiY2xpZW50X2NyZWRlbnRpYWxzIiwic2NvcGUiOlsiYWNjb3VudHMiXSwiYXV0aF90aW1lIjoxNTczNDA1MjgwLCJyZWFsbSI6Ii9vcGVuYmFua2luZyIsImV4cCI6MTU3MzQ5MTY4MCwiaWF0IjoxNTczNDA1MjgwLCJleHBpcmVzX2luIjo4NjQwMCwianRpIjoiWjFiMk50d2dmM2NPdV8wcFJQdFNnaWZudy1jLkJJOFlNUkZyQzdkUFlDVFM3TmFVWERKY0lnayJ9.i7ZD4i8b53h_tD2yFjURZdmpL6HmVU3rfJGS-x30GuTiYbwYgGcVitEHeoXFd3D4MIwwo_4SWUDfb68TCOuTjQ
Content-Type: application/json
x-idempotency-key: 81f8bd0b-4b58-407e-910c-f0034e64cc8e
x-fapi-financial-id: 0015800001041REAAY
x-fapi-customer-last-logged-time: Sun, 10 Sep 2017 19:43:31 UTC
x-fapi-customer-ip-address: 104.25.212.99
x-fapi-interaction-id: 93bac548-d2de-4546-b106-880a5018460d
Accept: application/json

{
  "Data": {
    "Permissions": [
      "ReadAccountsDetail",
      "ReadBalances",
      "ReadBeneficiariesDetail",
      "ReadDirectDebits",
      "ReadProducts",
      "ReadStandingOrdersDetail",
      "ReadTransactionsCredits",
      "ReadTransactionsDebits",
      "ReadTransactionsDetail",
      "ReadOffers",
      "ReadPAN",
      "ReadParty",
      "ReadPartyPSU",
      "ReadScheduledPaymentsDetail",
      "ReadStatementsDetail"
    ]
  },
  "Risk": {}
}
```

As a response:

```
{
    "Data": {
        "ConsentId": "AAC_616395ad-efb2-4875-85d9-b63a34586f2e",
        "CreationDateTime": "2019-11-10T17:01:24.871Z",
        "Status": "AwaitingAuthorisation",
        "Permissions": [
            "ReadAccountsDetail",
            "ReadBalances",
            "ReadBeneficiariesDetail",
            "ReadDirectDebits",
            "ReadProducts",
            "ReadStandingOrdersDetail",
            "ReadTransactionsCredits",
            "ReadTransactionsDebits",
            "ReadTransactionsDetail",
            "ReadOffers",
            "ReadPAN",
            "ReadParty",
            "ReadPartyPSU",
            "ReadScheduledPaymentsDetail",
            "ReadStatementsDetail"
        ]
    },
    "Risk": {}
}
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Accounts flow` > `Access preparation` > `‌create account request`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#60789eff-af9e-4f61-8577-5278544630fb

The bank will return to you the consent object you created. It's why you will see the same claims that you requested.
Although, the bank added a few new one in the response, that are going to be important for the rest of the tutorial:

* `ConsentId`: an ID for our consent. It's an important value as we will need to attach it to all the subsequent requests of the flow. It's best you store it somewhere safe and easily accessible.
* `Status`: the status of the consent. As you just created it, it's expected to have the status `AwaitingAuthorisation`, which means the consent is waiting for the user authorisation now. This is what we are going to do in the next step

## Authentication and consent with the bank

Now, you need to get this consent agreed by the user. You may be surprise to notice that actually, your consent is not bind to any user particular yet.
I personally find it quite nice that the consent is generic and you don't need to share any form of identification of the user to the bank. It means that as a user, I don't need to share my bank user identity with the developer, which is nice.
Time to use the second OIDC grant flow: for getting the authorisation of the user, we are going to use the hybrid flow.
*If you are familiar with OAuth 2 and don't know the hybrid flow yet: the hybrid is very similar to the authorisation code grant flow, it just returns an ID token with the authorisation code.*
If you don't know what is OIDC, it stands for OpenID connect. It's a standard dedicated to identity, like the things that happen when you login to a website using your facebook or google account for example.
OIDC is like a layer on top of OAuth 2. For Open Banking, We are not really going to use the identity layer added by OIDC, but just use it for additional security check it provides. We will come back on that.
The hybrid flow is based on a redirection model. This means you are going to redirect the user to the bank, with some informations attached to it and once the bank has finish its part, it redirects the user back to you.
The challenge is to attach a context to this redirection, so the bank can load the right consent, but you are not in touch with the bank directly unfortunately.
The trick is to add a GET parameter to the URL to which the user will be redirected. This parameter will contains the instruction for the bank.
We all played with the GET parameters and we know that as a user , it's super easy to change them.
In our case, for security reason, we want to prevent the user to do that. Therefore on top of putting the information in a GET parameter, we are going to sign it. This way, if the user changes some values, the signature will be compromised and the bank would be able to tell something wrong happened.

The information you send as a GET parameter, is called the `request parameter`. The format is standardise by OIDC, it's a JSON with the following claims:

```
{
  "aud": "https://as.aspsp.ob.forgerock.financial/oauth2",
  "scope": "openid accounts",
  "iss": "678c4cc1-12e4-4c82-84fd-73cbf609535a",
  "claims": {
    "id_token": {
      "acr": {
        "value": "urn:openbanking:psd2:sca",
        "essential": true
      },
      "openbanking_intent_id": {
        "value": "AAC_616395ad-efb2-4875-85d9-b63a34586f2e",
        "essential": true
      }
    },
    "userinfo": {
      "openbanking_intent_id": {
        "value": "AAC_616395ad-efb2-4875-85d9-b63a34586f2e",
        "essential": true
      }
    }
  },
  "response_type": "code id_token",
  "redirect_uri": "https://www.google.com",
  "state": "10d260bf-a7d9-444a-92d9-7b7a5f088208",
  "exp": 1573405577.313,
  "nonce": "10d260bf-a7d9-444a-92d9-7b7a5f088208",
  "client_id": "678c4cc1-12e4-4c82-84fd-73cbf609535a"
}
```

Lets details the main claims:

- `aud`: like when you write a letter, you are going to say who is the audience of this message. In this case, it is the bank. To find what is the official audience value corresponding to the bank, remember about the AS discovery. In there, the bank told us its `issuer` value. That this value we are going to use.
- `iss`: This message is signed by us, so we are going to put our official name. The standard is helping us there and telling us to use our OIDC client ID.
- `response_type`: It's a way in OIDC to specify the kind of response you want and therefore which kind of grant flow you are expecting to happen. By saying `code id_token`, we are requesting the hybrid flow.
- `scope`: It's the kind of scope we are going to request authorisation from the user. `openid` scope is a way to tell the bank we want to use OIDC, and `accounts` to tell the scope of this request is to access the accounts and transactions APIs.
- `redirect_uri`: It's the callback URI where we expect the user to return, once the bank has completed its part with the user. We use google in our example to avoid hosting a service but for in a real scenario, you would put a dedicated URI from your web applications.
- `state` and `nonce` serves the same purpose for us here: having a reference to the request. It's important as once the user is redirected to the bank, you will be waiting for a callback. We are zooming on one request but imagine when you will do a lot of requests in parallel. As it's an asynchronous process, you will need a way to bind the callback to the initial request you made. As `state` and `nonce` really serve the same purpose, I would personally recommend putting the same value in both. Depending of how you persisted the consent ID, you probably already have an ID that could play the role of state. If not, a common way is to store the current state of your application before doing the redirection, and use the state id as a value for `state`.
- `claims`: the claims is a way for Open Banking to add custom values without breaking the OIDC standard. It may sounds a bit weird for you, as a developer, to have to format it that way. You can ignore that technical details for now and replace `AAC_616395ad-efb2-4875-85d9-b63a34586f2e` your consent ID.


What you need to do with this request parameter, is to sign it. For that, we are going to use the JWS format.
If you remember the registration, we had to sign a JWS as well. For this tutorial, you can use also the JWKMS provided by ForgeRock:

```
POST /api/crypto/signClaims HTTP/1.1
Host: jwkms.ob.forgerock.financial
Content-Type: application/json
issuerId: 678c4cc1-12e4-4c82-84fd-73cbf609535a

{
  "aud": "https://as.aspsp.ob.forgerock.financial/oauth2",
  "scope": "openid accounts",
  "iss": "678c4cc1-12e4-4c82-84fd-73cbf609535a",
  "claims": {
    "id_token": {
      "acr": {
        "value": "urn:openbanking:psd2:sca",
        "essential": true
      },
      "openbanking_intent_id": {
        "value": "AAC_616395ad-efb2-4875-85d9-b63a34586f2e",
        "essential": true
      }
    },
    "userinfo": {
      "openbanking_intent_id": {
        "value": "AAC_616395ad-efb2-4875-85d9-b63a34586f2e",
        "essential": true
      }
    }
  },
  "response_type": "code id_token",
  "redirect_uri": "https://www.google.com",
  "state": "10d260bf-a7d9-444a-92d9-7b7a5f088208",
  "exp": 1573405577.313,
  "nonce": "10d260bf-a7d9-444a-92d9-7b7a5f088208",
  "client_id": "678c4cc1-12e4-4c82-84fd-73cbf609535a"
}
```

As a result:

```
eyJraWQiOiIyYTNmM2I3ZWQ3MTc0M2FiM2UzNzJkYTk2Y2FiZmJiMzYzZmM5MmQ3IiwiYWxnIjoiUFMyNTYifQ.eyJhdWQiOiJodHRwczpcL1wvYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbFwvb2F1dGgyIiwic2NvcGUiOiJvcGVuaWQgYWNjb3VudHMiLCJpc3MiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJjbGFpbXMiOnsiaWRfdG9rZW4iOnsiYWNyIjp7InZhbHVlIjoidXJuOm9wZW5iYW5raW5nOnBzZDI6c2NhIiwiZXNzZW50aWFsIjp0cnVlfSwib3BlbmJhbmtpbmdfaW50ZW50X2lkIjp7InZhbHVlIjoiQUFDXzYxNjM5NWFkLWVmYjItNDg3NS04NWQ5LWI2M2EzNDU4NmYyZSIsImVzc2VudGlhbCI6dHJ1ZX19LCJ1c2VyaW5mbyI6eyJvcGVuYmFua2luZ19pbnRlbnRfaWQiOnsidmFsdWUiOiJBQUNfNjE2Mzk1YWQtZWZiMi00ODc1LTg1ZDktYjYzYTM0NTg2ZjJlIiwiZXNzZW50aWFsIjp0cnVlfX19LCJyZXNwb25zZV90eXBlIjoiY29kZSBpZF90b2tlbiIsInJlZGlyZWN0X3VyaSI6Imh0dHBzOlwvXC93d3cuZ29vZ2xlLmNvbSIsInN0YXRlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiZXhwIjoxNTczNDExODU5LCJub25jZSI6IjEwZDI2MGJmLWE3ZDktNDQ0YS05MmQ5LTdiN2E1ZjA4ODIwOCIsImlhdCI6MTU3MzQxMDk2NSwiY2xpZW50X2lkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwianRpIjoiOGEwYTI0MGMtN2ZhNS00MjdlLWIyZjQtZTkwMTE1ZGI4MDcyIn0.TMvSWvEWnbqngrQfU-KOrW9-iUdMBegYN8fNxVy7TNazr5zgY0tmcFYYcyrb0eIGVlLlmm4_gBlEDt5h57Q8LJlrkrKQFNlELaCMeLZAhR19d5eGJB3XdzGH8-lJYeLwl96V1K-lvVvTQ_L8qaFWY8NQeRGbC5fsiYBlL9vZdbXVPHqpQRLXmL-1N2n452VJYiyKT0UGO1K64QBY91zacdMnRp_8zyGiHddRH7FOp2OsUfiR_ldNaVeICcERh6AYVvxTVoCairSM8DECd6B1n06s5HC-F1Zm4KqAqqSiT42tQ9z_gAPDwPEGBAkS-rH6oHY004Rkx8dMiji7NRjktg
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Accounts flow` > `Auth & Consent` > `‌Generate request parameter`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#daf4e107-1f99-4edd-8b22-5156fbe0cf55

Now that you got the request parameter ready, you need to build the redirection url for the user needs to follow.
For that, you need to know which url from the bank to use. This information can be find in the response of the bank AS discovery endpoint.

```
    "authorization_endpoint": "https://as.aspsp.ob.forgerock.financial/oauth2/authorize",
```

The OIDC standards requires you to build the URI a certain way, for security reason mainly. Here is the template you can use:

```
https://as.aspsp.ob.forgerock.financial/oauth2/authorize?response_type=code%20id_token&client_id=678c4cc1-12e4-4c82-84fd-73cbf609535a&state=10d260bf-a7d9-444a-92d9-7b7a5f088208&redirect_uri=https://www.google.com&nonce=10d260bf-a7d9-444a-92d9-7b7a5f088208&scope=openid%20accounts&request=eyJraWQiOiIyYTNmM2I3ZWQ3MTc0M2FiM2UzNzJkYTk2Y2FiZmJiMzYzZmM5MmQ3IiwiYWxnIjoiUFMyNTYifQ.eyJhdWQiOiJodHRwczpcL1wvYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbFwvb2F1dGgyIiwic2NvcGUiOiJvcGVuaWQgYWNjb3VudHMiLCJpc3MiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJjbGFpbXMiOnsiaWRfdG9rZW4iOnsiYWNyIjp7InZhbHVlIjoidXJuOm9wZW5iYW5raW5nOnBzZDI6c2NhIiwiZXNzZW50aWFsIjp0cnVlfSwib3BlbmJhbmtpbmdfaW50ZW50X2lkIjp7InZhbHVlIjoiQUFDXzYxNjM5NWFkLWVmYjItNDg3NS04NWQ5LWI2M2EzNDU4NmYyZSIsImVzc2VudGlhbCI6dHJ1ZX19LCJ1c2VyaW5mbyI6eyJvcGVuYmFua2luZ19pbnRlbnRfaWQiOnsidmFsdWUiOiJBQUNfNjE2Mzk1YWQtZWZiMi00ODc1LTg1ZDktYjYzYTM0NTg2ZjJlIiwiZXNzZW50aWFsIjp0cnVlfX19LCJyZXNwb25zZV90eXBlIjoiY29kZSBpZF90b2tlbiIsInJlZGlyZWN0X3VyaSI6Imh0dHBzOlwvXC93d3cuZ29vZ2xlLmNvbSIsInN0YXRlIjoiMTBkMjYwYmYtYTdkOS00NDRhLTkyZDktN2I3YTVmMDg4MjA4IiwiZXhwIjoxNTczNDExODU5LCJub25jZSI6IjEwZDI2MGJmLWE3ZDktNDQ0YS05MmQ5LTdiN2E1ZjA4ODIwOCIsImlhdCI6MTU3MzQxMDk2NSwiY2xpZW50X2lkIjoiNjc4YzRjYzEtMTJlNC00YzgyLTg0ZmQtNzNjYmY2MDk1MzVhIiwianRpIjoiOGEwYTI0MGMtN2ZhNS00MjdlLWIyZjQtZTkwMTE1ZGI4MDcyIn0.TMvSWvEWnbqngrQfU-KOrW9-iUdMBegYN8fNxVy7TNazr5zgY0tmcFYYcyrb0eIGVlLlmm4_gBlEDt5h57Q8LJlrkrKQFNlELaCMeLZAhR19d5eGJB3XdzGH8-lJYeLwl96V1K-lvVvTQ_L8qaFWY8NQeRGbC5fsiYBlL9vZdbXVPHqpQRLXmL-1N2n452VJYiyKT0UGO1K64QBY91zacdMnRp_8zyGiHddRH7FOp2OsUfiR_ldNaVeICcERh6AYVvxTVoCairSM8DECd6B1n06s5HC-F1Zm4KqAqqSiT42tQ9z_gAPDwPEGBAkS-rH6oHY004Rkx8dMiji7NRjktg
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Accounts flow` > `Auth & Consent` > `Hybrid flow`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#dbc709ae-618f-458f-91b5-563d84c76acf
This one is a bit different than the other Postman example. You just need to build the URI, not execute it in Postman. We still use a Postman request, as a way to help you preparing the request to send to Alice. You need to click on 'code' on the top right, then select 'curl' and copy the URL. 

You need now to redirect the user to this URI with a simple 302.

From now, the user is authenticating and authorising the consent to the bank. Unfortunately you can't tell the status of this flow at this point, it's only happening between the user and the bank. All you can do is wait for the user to come back to your application.
If everything goes well, the bank is going to redirect the user to your redirect URI.
The bank is going use the same trick to send you information, via the parameters.
The only difference is that for security reason, the information are send not via the GET parameter but the fragment of the request:
Https://your-redirect-uri.com/my-path#foo=bar
In the example, foo->bar is the information send to you. It is good to know that the fragment is not send to the server side, it is only accessible via the client side like JS. You will need to write a bit of JS to compute those parameters. The easiest things to do is to send those fragment parameters to your backend server via a POST. You may wonder what the goal really if at the end, you send it to the backend anyway. The difference is on the http method used. A GET request, which is a 302, has the GET parameters accessible to browser history, proxy audit logs etc, whereas a POST request has its payload encrypted and signed, and not accessible by any other party then the backend. I agree it's a bit painful to have to go through this fragment and send it to the backend, unfortunately you can't workaround it. 
For our tutorial, we will do that manually so don't worry about writing this JS script yet.

Now it's time to follow the redirect you created earlier into a new incognito web browser tab. This way, you will simulate the journey of the user Alice. 
It's really important to understand that you are changing your role here: you are not the developer anymore, you become Alice that has been redirected to her bank.

1. You should be redirected to the bank login page. 
2. Use the credential `Alice` / `changeit`
3. You should now be redirect to the consent page
4. Approve the consent request
5. You should be redirected to the redirect URI with some fragment information in it

Once you get call back via the redirect URI with some information in the fragment, you are back to your previous role: the developer.

In our tutorial so far we used google as a redirect URI. It allows us to not have to host a service and still been able to use Postman.
For me, the redirection looks like:

```
https://www.google.com/#code=2SSqrV318h_c50BGDA6Osfj5-8Y.GnvacU7_cLQa9oraGwGjNIbX7jQ&id_token=eyJraWQiOiIyYzk3ZDdmOWQyYjRkNTE5OTI4MDM2MGVkZTMzZTYzZDQ4MTUwOTRkIiwiYWxnIjoiUFMyNTYifQ.eyJzdWIiOiJBQUNfNTJiNjk2M2EtYjhhYy00Y2M3LTk0YjgtZmI1ODU5OGVkODhmIiwiYXVkaXRUcmFja2luZ0lkIjoiMTQ0MzY0ZmUtYzE5Mi00Mzc2LWFjNjctMWNjZjI1NzdjNDBiLTMyMzg1IiwiaXNzIjoiaHR0cHM6XC9cL2FzLmFzcHNwLm9iLmZvcmdlcm9jay5maW5hbmNpYWxcL29hdXRoMiIsInRva2VuTmFtZSI6ImlkX3Rva2VuIiwibm9uY2UiOiIxMGQyNjBiZi1hN2Q5LTQ0NGEtOTJkOS03YjdhNWYwODgyMDgiLCJhY3IiOiJ1cm46b3BlbmJhbmtpbmc6cHNkMjpzY2EiLCJhdWQiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJjX2hhc2giOiJ1NmxLS1lsZmFUYmtHUEpQVG04YTNBIiwib3BlbmJhbmtpbmdfaW50ZW50X2lkIjoiQUFDXzUyYjY5NjNhLWI4YWMtNGNjNy05NGI4LWZiNTg1OThlZDg4ZiIsInNfaGFzaCI6IlhOU3VSWnNKVW9aY3Y3WTkwaDRaX1EiLCJhenAiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJhdXRoX3RpbWUiOjE1NzM1NTE0MDEsInJlYWxtIjoiXC9vcGVuYmFua2luZyIsImV4cCI6MTU3MzYzNzgxNywidG9rZW5UeXBlIjoiSldUVG9rZW4iLCJpYXQiOjE1NzM1NTE0MTcsImp0aSI6ImYxZmQxODMzLWQ1NGEtNDZkZi1iM2ZlLTAyYWE2ZWI0ZjEyMSJ9.aQnKLF8iPX_2G1ceRQ2-b8iNkqsv0g493jkuVRqa-Rwar8Lm6Xm7FLeS8bgROu_WLsUE9mIy2hRunXaZh3Nuubc3gQYbF0Mxqjuf1cvAAyi9ZSoaqDOCnS8UR_ofEvgALSqUV6ian-qfl70vlEGOdmydSRaKoLRug5YuVDJT4rpc2U3T7rHCAKq1W2VqcYVPaYs0ZPkqvZnpljr9gPjlQWmihExqK63JknmfxvBrt51lhJ6V6m9x7TfUwqEmDQALRDZJf61Y9q3lzveRBrci-B_oLp0tOjciq4DYq-ElW1oCWT5ME3tVFGQ0VTo2q4paZnDlslVu2VfI46XmxH294w&state=10d260bf-a7d9-444a-92d9-7b7a5f088208
```

Not that easy to parse manually, I admit that. Fortunately parsing with JS would be much easier. It's the fact we are doing it manually that makes it tricky to do.

You can notice the following parameters:

- `code`: `2SSqrV318h_c50BGDA6Osfj5-8Y.GnvacU7_cLQa9oraGwGjNIbX7jQ`
- `id_token`: `eyJraWQiOiIyYzk3ZDdmOWQyYjRkNTE5OTI4MDM2MGVkZTMzZTYzZDQ4MTUwOTRkIiwiYWxnIjoiUFMyNTYifQ.eyJzdWIiOiJBQUNfNTJiNjk2M2EtYjhhYy00Y2M3LTk0YjgtZmI1ODU5OGVkODhmIiwiYXVkaXRUcmFja2luZ0lkIjoiMTQ0MzY0ZmUtYzE5Mi00Mzc2LWFjNjctMWNjZjI1NzdjNDBiLTMyMzg1IiwiaXNzIjoiaHR0cHM6XC9cL2FzLmFzcHNwLm9iLmZvcmdlcm9jay5maW5hbmNpYWxcL29hdXRoMiIsInRva2VuTmFtZSI6ImlkX3Rva2VuIiwibm9uY2UiOiIxMGQyNjBiZi1hN2Q5LTQ0NGEtOTJkOS03YjdhNWYwODgyMDgiLCJhY3IiOiJ1cm46b3BlbmJhbmtpbmc6cHNkMjpzY2EiLCJhdWQiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJjX2hhc2giOiJ1NmxLS1lsZmFUYmtHUEpQVG04YTNBIiwib3BlbmJhbmtpbmdfaW50ZW50X2lkIjoiQUFDXzUyYjY5NjNhLWI4YWMtNGNjNy05NGI4LWZiNTg1OThlZDg4ZiIsInNfaGFzaCI6IlhOU3VSWnNKVW9aY3Y3WTkwaDRaX1EiLCJhenAiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJhdXRoX3RpbWUiOjE1NzM1NTE0MDEsInJlYWxtIjoiXC9vcGVuYmFua2luZyIsImV4cCI6MTU3MzYzNzgxNywidG9rZW5UeXBlIjoiSldUVG9rZW4iLCJpYXQiOjE1NzM1NTE0MTcsImp0aSI6ImYxZmQxODMzLWQ1NGEtNDZkZi1iM2ZlLTAyYWE2ZWI0ZjEyMSJ9.aQnKLF8iPX_2G1ceRQ2-b8iNkqsv0g493jkuVRqa-Rwar8Lm6Xm7FLeS8bgROu_WLsUE9mIy2hRunXaZh3Nuubc3gQYbF0Mxqjuf1cvAAyi9ZSoaqDOCnS8UR_ofEvgALSqUV6ian-qfl70vlEGOdmydSRaKoLRug5YuVDJT4rpc2U3T7rHCAKq1W2VqcYVPaYs0ZPkqvZnpljr9gPjlQWmihExqK63JknmfxvBrt51lhJ6V6m9x7TfUwqEmDQALRDZJf61Y9q3lzveRBrci-B_oLp0tOjciq4DYq-ElW1oCWT5ME3tVFGQ0VTo2q4paZnDlslVu2VfI46XmxH294w`
- `state`: `10d260bf-a7d9-444a-92d9-7b7a5f088208`

The `code` is the authorisation code that you need to exchange in order to get an access token. It has a relatively short period of life, just the time for you to parse it and exchange it.
You may wonder why you don't get directly the access token. The reason is that it goes through the user web browser and the user would be able to intercept it otherwise. By using a code as an intermediate step, the flow makes sure that only you get the access token. This is warranty by the fact you need to authenticate in order to exchange the code.

The `state` is useful for you, as it will allow you to recover the state of the initial request. Remember that this redirection has happen un-synchronously from the point of the developer and therefore, you need a way to link the context to this response.

The ID token here is not used for identity, as you would usually expect in OIDC. It is used more as detached signature. What this means is that we used the fact that the ID token is signed by the bank, to verify the information has not been corrupted on the way in.

### Verifying the parameters received
For this tutorial, you can skip this section. It's something you need to do if you want to be bullet proof in your implementation.
If you read the ID token, you will find the following payload.
Note: You need to verify the ID token signature against the bank public keys. You can skip this for now but remember it's important to do it from a security point of view.

```
{
  "sub": "AAC_52b6963a-b8ac-4cc7-94b8-fb58598ed88f",
  "auditTrackingId": "144364fe-c192-4376-ac67-1ccf2577c40b-32385",
  "iss": "https://as.aspsp.ob.forgerock.financial/oauth2",
  "tokenName": "id_token",
  "nonce": "10d260bf-a7d9-444a-92d9-7b7a5f088208",
  "acr": "urn:openbanking:psd2:sca",
  "aud": "678c4cc1-12e4-4c82-84fd-73cbf609535a",
  "c_hash": "u6lKKYlfaTbkGPJPTm8a3A",
  "openbanking_intent_id": "AAC_52b6963a-b8ac-4cc7-94b8-fb58598ed88f",
  "s_hash": "XNSuRZsJUoZcv7Y90h4Z_Q",
  "azp": "678c4cc1-12e4-4c82-84fd-73cbf609535a",
  "auth_time": 1573551401,
  "realm": "/openbanking",
  "exp": 1573637817,
  "tokenType": "JWTToken",
  "iat": 1573551417,
  "jti": "f1fd1833-d54a-46df-b3fe-02aa6eb4f121"
}
```

You can find the `nonce` and the `intent ID` for example. The one that are important to verify really is the `c_hash` and `s_hash`.
Those two claims allows you to verify respectively the `code` and `state` value, the one you received as part of the fragment.
You need to do that as they are not signed (so the user could have trick you and edit them manually), so by using the ID token and verifying the hashes matches the values you received in the fragment, you can make sure they have not been corrupted and you can trust them.
To compute the hash of the state for example and then compare it with the s_hash, you can do something like: (example in JAVA)

```
 public static String computeHash(String token) {
        try {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");
            final byte[] code_bytes = digest.digest(token.getBytes(StandardCharsets.UTF_8));
            final byte[] half_code_bytes = Arrays.copyOfRange(code_bytes, 0, code_bytes.length / 2);
            return new String(Base64.getUrlEncoder().encode(half_code_bytes));

        } catch (NoSuchAlgorithmException ex) {
            log.error("Failed to compute hash for token: '{}' with exception.", token, ex);
        }
        return null;
    }
```

It's also good to check the `acr` and the other claims, as a sanity check.


## Exchange the code

You got the authorisation code, you are nearly there. You need now to exchange it, which you will see is trivial. All you need to do, is calling:

```
POST /oauth2/access_token HTTP/1.1
Host: matls.as.aspsp.ob.forgerock.financial
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=2SSqrV318h_c50BGDA6Osfj5-8Y.GnvacU7_cLQa9oraGwGjNIbX7jQ&redirect_uri=https%3A%2F%2Fwww.google.com
  ```
  
As a response:

```
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJkZW1vIiwiY3RzIjoiT0FVVEgyX0dSQU5UX1NFVCIsImF1dGhfbGV2ZWwiOjAsImF1ZGl0VHJhY2tpbmdJZCI6IjE0NDM2NGZlLWMxOTItNDM3Ni1hYzY3LTFjY2YyNTc3YzQwYi0zNTEzNCIsImlzcyI6Imh0dHBzOi8vYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbC9vYXV0aDIiLCJ0b2tlbk5hbWUiOiJhY2Nlc3NfdG9rZW4iLCJ0b2tlbl90eXBlIjoiQmVhcmVyIiwiYXV0aEdyYW50SWQiOiJQSFdDTGxOa3R5UzVyZGhxZVVrTVp2T0g5VUkuQk5VQVNZaERaUWQwT2hNRlJUcjlVNGNPQmxFIiwibm9uY2UiOiIxMGQyNjBiZi1hN2Q5LTQ0NGEtOTJkOS03YjdhNWYwODgyMDgiLCJhdWQiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJuYmYiOjE1NzM1NTM3NjcsImdyYW50X3R5cGUiOiJhdXRob3JpemF0aW9uX2NvZGUiLCJzY29wZSI6WyJvcGVuaWQiLCJhY2NvdW50cyJdLCJhdXRoX3RpbWUiOjE1NzM1NTM3NTEsImNsYWltcyI6IntcImlkX3Rva2VuXCI6e1wiYWNyXCI6e1widmFsdWVcIjpcInVybjpvcGVuYmFua2luZzpwc2QyOnNjYVwiLFwiZXNzZW50aWFsXCI6dHJ1ZX0sXCJvcGVuYmFua2luZ19pbnRlbnRfaWRcIjp7XCJ2YWx1ZVwiOlwiQUFDXzcxZjJiNjU2LTEzOGItNGZkYi1iOGU1LTM0MWI0M2NlMTcxYVwiLFwiZXNzZW50aWFsXCI6dHJ1ZX19LFwidXNlcmluZm9cIjp7XCJvcGVuYmFua2luZ19pbnRlbnRfaWRcIjp7XCJ2YWx1ZVwiOlwiQUFDXzcxZjJiNjU2LTEzOGItNGZkYi1iOGU1LTM0MWI0M2NlMTcxYVwiLFwiZXNzZW50aWFsXCI6dHJ1ZX19fSIsInJlYWxtIjoiL29wZW5iYW5raW5nIiwiZXhwIjoxNTczNjQwMTY3LCJpYXQiOjE1NzM1NTM3NjcsImV4cGlyZXNfaW4iOjg2NDAwLCJqdGkiOiJQSFdDTGxOa3R5UzVyZGhxZVVrTVp2T0g5VUkuMzY3a3kwTDg2UXc1VUdCYXpGbFZXNjBfd2lnIn0.zLbqO4JelumoxVUqU2GTrqImGoqdy8tJORveNOkgg8L5rrkByJ2vX6fS8qmBA2zv1CWlzKImBY-xOu0ZVDqGEg",
    "expires_in": 86399,
    "id_token": "eyJraWQiOiIyYzk3ZDdmOWQyYjRkNTE5OTI4MDM2MGVkZTMzZTYzZDQ4MTUwOTRkIiwiYWxnIjoiUFMyNTYifQ.eyJhdF9oYXNoIjoiYUxNVGp0WUE1M1lOeFlxd2poWEJxdyIsInN1YiI6IkFBQ183MWYyYjY1Ni0xMzhiLTRmZGItYjhlNS0zNDFiNDNjZTE3MWEiLCJhdWRpdFRyYWNraW5nSWQiOiIxNDQzNjRmZS1jMTkyLTQzNzYtYWM2Ny0xY2NmMjU3N2M0MGItMzUxMzUiLCJpc3MiOiJodHRwczpcL1wvYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbFwvb2F1dGgyIiwidG9rZW5OYW1lIjoiaWRfdG9rZW4iLCJub25jZSI6IjEwZDI2MGJmLWE3ZDktNDQ0YS05MmQ5LTdiN2E1ZjA4ODIwOCIsImFjciI6InVybjpvcGVuYmFua2luZzpwc2QyOnNjYSIsImF1ZCI6IjY3OGM0Y2MxLTEyZTQtNGM4Mi04NGZkLTczY2JmNjA5NTM1YSIsImNfaGFzaCI6IkhHMmVabjFCRXUtNUVIbU4zeVFJSlEiLCJvcGVuYmFua2luZ19pbnRlbnRfaWQiOiJBQUNfNzFmMmI2NTYtMTM4Yi00ZmRiLWI4ZTUtMzQxYjQzY2UxNzFhIiwic19oYXNoIjoiWE5TdVJac0pVb1pjdjdZOTBoNFpfUSIsImF6cCI6IjY3OGM0Y2MxLTEyZTQtNGM4Mi04NGZkLTczY2JmNjA5NTM1YSIsImF1dGhfdGltZSI6MTU3MzU1Mzc1MSwicmVhbG0iOiJcL29wZW5iYW5raW5nIiwiZXhwIjoxNTczNjQwMTY3LCJ0b2tlblR5cGUiOiJKV1RUb2tlbiIsImlhdCI6MTU3MzU1Mzc2NywianRpIjoiZDlhOTFmMTEtMGYyMy00MmEwLWE2MWItOTYyZjcyOTg2YjgwIn0.XulDBzzvUonTKaUyUu8ui3m2gvex5AwrmLhsMHPnmKQfE_bFTmQnrOCUVMo9vTDBHlWB-Zi4YUukMolGT7h39k9ICHt-YKS3ZGnp7ypM0R6pWkQCpSIIGSUBEv9TTJrgae9n1qdwZXtL2_0_L9chWwQirvNo7zVQ7PYghT0Pu1HQON5bVaxvlzjQY5ER8idATDIjjsYA9CSYaWRP-MHvS-6EUutT_oEchKDKQkK6VpR2a7MuL29XHnMicXdzBmooEI82I25yF1_lX0mfGR6NcKtCX1MnzeYz-A890uxzgwAZ-f2OSnKKRtRb37z9qZ1pHcz1lNbfCV6pgJToKcY62w",
    "token_type": "Bearer",
    "scope": "openid accounts",
    "refresh_token": "eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJkZW1vIiwiY3RzIjoiT0FVVEgyX0dSQU5UX1NFVCIsImF1dGhfbGV2ZWwiOjAsImF1ZGl0VHJhY2tpbmdJZCI6IjE0NDM2NGZlLWMxOTItNDM3Ni1hYzY3LTFjY2YyNTc3YzQwYi0zNTEzMyIsImlzcyI6Imh0dHBzOi8vYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbC9vYXV0aDIiLCJ0b2tlbk5hbWUiOiJyZWZyZXNoX3Rva2VuIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1dGhHcmFudElkIjoiUEhXQ0xsTmt0eVM1cmRocWVVa01adk9IOVVJLkJOVUFTWWhEWlFkME9oTUZSVHI5VTRjT0JsRSIsImF1ZCI6IjY3OGM0Y2MxLTEyZTQtNGM4Mi04NGZkLTczY2JmNjA5NTM1YSIsImFjciI6InVybjpvcGVuYmFua2luZzpwc2QyOnNjYSIsIm5iZiI6MTU3MzU1Mzc2NywiZ3JhbnRfdHlwZSI6ImF1dGhvcml6YXRpb25fY29kZSIsInNjb3BlIjpbIm9wZW5pZCIsImFjY291bnRzIl0sImF1dGhfdGltZSI6MTU3MzU1Mzc1MSwiY2xhaW1zIjoie1wiaWRfdG9rZW5cIjp7XCJhY3JcIjp7XCJ2YWx1ZVwiOlwidXJuOm9wZW5iYW5raW5nOnBzZDI6c2NhXCIsXCJlc3NlbnRpYWxcIjp0cnVlfSxcIm9wZW5iYW5raW5nX2ludGVudF9pZFwiOntcInZhbHVlXCI6XCJBQUNfNzFmMmI2NTYtMTM4Yi00ZmRiLWI4ZTUtMzQxYjQzY2UxNzFhXCIsXCJlc3NlbnRpYWxcIjp0cnVlfX0sXCJ1c2VyaW5mb1wiOntcIm9wZW5iYW5raW5nX2ludGVudF9pZFwiOntcInZhbHVlXCI6XCJBQUNfNzFmMmI2NTYtMTM4Yi00ZmRiLWI4ZTUtMzQxYjQzY2UxNzFhXCIsXCJlc3NlbnRpYWxcIjp0cnVlfX19IiwicmVhbG0iOiIvb3BlbmJhbmtpbmciLCJleHAiOjE1ODEzMjk3NjcsImlhdCI6MTU3MzU1Mzc2NywiZXhwaXJlc19pbiI6Nzc3NjAwMCwianRpIjoiUEhXQ0xsTmt0eVM1cmRocWVVa01adk9IOVVJLmU0TlJkeHRpcDNDY09GaHp4YlJ4SGFQYm9fOCJ9.4R5h7Oat3SfQ0lCysm1D1-kgqWQnYfe_ow56PmE4BlxfUrugJvPRDtr9BaL72TV0qxGufC87kyZwA4kzjVEOGw"
}
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Accounts flow` > `Token exchange` > `Exchange code`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#37f94052-2042-4978-9b56-8a7ee9edbfd7

For now, the claim that interest us is the `access_token`. It is a token you need to store securely. It has a short time life. This is why you receive as well a refresh_token that allows you to regenerate an access token from it.
We will see in a future article how to use the refresh token. For now, let's use our access token!

## Getting the accounts

It was quite a hard work to get to that access token. Now that you got it, you can use to consume all the accounts and transactions APIs.

Here is an example of how you can use this access token to read the user accounts:

```
GET /open-banking/v3.1.1/aisp/accounts HTTP/1.1
Host: matls.rs.aspsp.ob.forgerock.financial
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJ6aXAiOiJOT05FIiwia2lkIjoiRTE5N1kzMVFLT05mSk42aTdrQlkyMzFneUFvPSIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJkZW1vIiwiY3RzIjoiT0FVVEgyX0dSQU5UX1NFVCIsImF1dGhfbGV2ZWwiOjAsImF1ZGl0VHJhY2tpbmdJZCI6IjE0NDM2NGZlLWMxOTItNDM3Ni1hYzY3LTFjY2YyNTc3YzQwYi0zNTEzNCIsImlzcyI6Imh0dHBzOi8vYXMuYXNwc3Aub2IuZm9yZ2Vyb2NrLmZpbmFuY2lhbC9vYXV0aDIiLCJ0b2tlbk5hbWUiOiJhY2Nlc3NfdG9rZW4iLCJ0b2tlbl90eXBlIjoiQmVhcmVyIiwiYXV0aEdyYW50SWQiOiJQSFdDTGxOa3R5UzVyZGhxZVVrTVp2T0g5VUkuQk5VQVNZaERaUWQwT2hNRlJUcjlVNGNPQmxFIiwibm9uY2UiOiIxMGQyNjBiZi1hN2Q5LTQ0NGEtOTJkOS03YjdhNWYwODgyMDgiLCJhdWQiOiI2NzhjNGNjMS0xMmU0LTRjODItODRmZC03M2NiZjYwOTUzNWEiLCJuYmYiOjE1NzM1NTM3NjcsImdyYW50X3R5cGUiOiJhdXRob3JpemF0aW9uX2NvZGUiLCJzY29wZSI6WyJvcGVuaWQiLCJhY2NvdW50cyJdLCJhdXRoX3RpbWUiOjE1NzM1NTM3NTEsImNsYWltcyI6IntcImlkX3Rva2VuXCI6e1wiYWNyXCI6e1widmFsdWVcIjpcInVybjpvcGVuYmFua2luZzpwc2QyOnNjYVwiLFwiZXNzZW50aWFsXCI6dHJ1ZX0sXCJvcGVuYmFua2luZ19pbnRlbnRfaWRcIjp7XCJ2YWx1ZVwiOlwiQUFDXzcxZjJiNjU2LTEzOGItNGZkYi1iOGU1LTM0MWI0M2NlMTcxYVwiLFwiZXNzZW50aWFsXCI6dHJ1ZX19LFwidXNlcmluZm9cIjp7XCJvcGVuYmFua2luZ19pbnRlbnRfaWRcIjp7XCJ2YWx1ZVwiOlwiQUFDXzcxZjJiNjU2LTEzOGItNGZkYi1iOGU1LTM0MWI0M2NlMTcxYVwiLFwiZXNzZW50aWFsXCI6dHJ1ZX19fSIsInJlYWxtIjoiL29wZW5iYW5raW5nIiwiZXhwIjoxNTczNjQwMTY3LCJpYXQiOjE1NzM1NTM3NjcsImV4cGlyZXNfaW4iOjg2NDAwLCJqdGkiOiJQSFdDTGxOa3R5UzVyZGhxZVVrTVp2T0g5VUkuMzY3a3kwTDg2UXc1VUdCYXpGbFZXNjBfd2lnIn0.zLbqO4JelumoxVUqU2GTrqImGoqdy8tJORveNOkgg8L5rrkByJ2vX6fS8qmBA2zv1CWlzKImBY-xOu0ZVDqGEg
Content-Type: application/json
x-idempotency-key: 77e91683-3079-42b9-b084-48aa09a45d9e
x-fapi-financial-id: 0015800001041REAAY
x-fapi-customer-last-logged-time: Sun, 10 Sep 2017 19:43:31 UTC
x-fapi-customer-ip-address: 104.25.212.99
x-fapi-interaction-id: 93bac548-d2de-4546-b106-880a5018460d
Accept: application/json
```

As a result:

```
{
    "Data": {
        "Account": [
            {
                "AccountId": "66e96cd7-d23c-474b-bbd9-1d22d7146511",
                "Currency": "GBP",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "UK Bills",
                "Account": [
                    {
                        "SchemeName": "SortCodeAccountNumber",
                        "Identification": "868418245860",
                        "Name": "demo",
                        "SecondaryIdentification": "20590817"
                    }
                ]
            },
            {
                "AccountId": "d4883a2a-4c62-4d1e-b0c1-3ace5e4a3a8a",
                "Currency": "GBP",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "Household",
                "Account": [
                    {
                        "SchemeName": "SortCodeAccountNumber",
                        "Identification": "7490678739998",
                        "Name": "demo"
                    }
                ]
            },
            {
                "AccountId": "f90a6ae3-4442-4759-b731-66b62d05a39f",
                "Currency": "EUR",
                "AccountType": "Personal",
                "AccountSubType": "CurrentAccount",
                "Nickname": "FR Bills",
                "Account": [
                    {
                        "SchemeName": "SortCodeAccountNumber",
                        "Identification": "28308720990959",
                        "Name": "demo",
                        "SecondaryIdentification": "39896879"
                    }
                ]
            }
        ]
    },
    "Links": {
        "Self": "https://matls.rs.aspsp.ob.forgerock.financial/open-banking/v3.1.1/aisp/accounts"
    },
    "Meta": {
        "TotalPages": 1
    }
}
```

**Postman: ** You can find this request in the ForgeRock Postman collection under `Accounts flow` > `Data access` > `Account API V3.1` > `Get accounts`. The link from the online documentation directly to this request is https://postman.ob.forgerock.financial/?version=latest#388b6997-a527-46d8-a131-1c2743efc639

You can use this access token to other endpoints the same way.
We will see in the next article how to use it to retrieve transactions.

## Data rendering

Once you got access to the data, you can do what ever you like with it. For the purpose of our example, in our case we will render the financial data to Alice.


## Conclusion

As you saw, having an authorisation flow involving three parties is not trivial. You need to get all the parties communicating in a 1-1 relationship before been able to consume the APIs.
We used an account access as an example, the good thing of Open Banking UK is this flow is common for every kind of access. In fact, the concept can be re-used for any kind of APIs. Once you implemented the flow in a generic way, you will be able to re-use it multiple times.
It goes even further than Open Banking UK. You will see that this flow is also very similar in a lot of standards around the world.
