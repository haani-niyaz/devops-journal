# Azure AD

## Getting Started with Azure Active Directory Graph API

{{< hint warning >}}
**AAD**
The following section is pertaining to `graph.windows.net` and not `graph.microsoft.com`.
{{< /hint >}}

Start up installing package deps:

{{< highlight bash >}}
 go get -u github.com/Azure/azure-sdk-for-go/...
{{< / highlight >}}

## User API

### Limit the number of results returned with query parameters

https://docs.microsoft.com/en-us/graph/query-parameters

{{< highlight bash >}}
/users?$select=givenName,surname
{{< / highlight >}}

### Filter with mail, name etc.

{{< highlight bash >}}
https://graph.windows.net/contoso.com/users?api-version=2013-11-08&$filter=accountEnabled eq true and (userPrincipalName eq 'jonlawr@contoso.com' or mail eq 'jonlawr@contoso.com')
{{< / highlight >}}


### References

- [Go SDK](https://github.com/Azure/azure-sdk-for-go)
- [Application types](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-app-types) and see a bit more on background services [here](https://docs.microsoft.com/en-us/graph/auth-v2-service)
- [Go Azure SDK authentication](https://docs.microsoft.com/en-us/azure/developer/go/azure-sdk-authorization)
- [How to use tokens](https://docs.microsoft.com/en-gb/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) 
- Microsoft graph API for Users; see [this](https://docs.microsoft.com/en-us/graph/api/resources/users?view=graph-rest-1.0) and [this](https://docs.microsoft.com/en-us/graph/use-the-api).
- [Go SDK video](https://channel9.msdn.com/Shows/Azure-Friday/Go-on-Azure-Part-5-Build-apps-with-the-Azure-SDK-for-Go)
- [Training](https://docs.microsoft.com/en-us/learn/browse/?products=azure-active-directory)
- [Go SDK Samples](https://github.com/azure-samples/azure-sdk-for-go-samples)


## Getting Started with Microsoft Graph API

### References

- [How consent works](https://docs.microsoft.com/en-us/azure/active-directory/develop/consent-framework)
- [User authentication in code grant flow](https://docs.microsoft.com/en-us/graph/auth-v2-user)
- [When to use OAuth2 auth code grant flow](https://developer.okta.com/blog/2018/04/10/oauth-authorization-code-grant-type#when-to-use-the-authorization-code-flow)




