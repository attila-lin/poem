Define an OpenAPI Security Scheme.

# Macro parameters

| Attribute          | Description                                                                                                                                                                                                                                                                       | Type       | Optional |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|----------|
| rename             | Rename the security scheme.                                                                                                                                                                                                                                                       | string     | Y        |
| ty                 | The type of the security scheme. (api_key, basic, bearer, oauth2, openid_connect)                                                                                                                                                                                                 | string     | N        |
| key_in             | `api_key` The location of the API key. Valid values are "query", "header" or "cookie". (query, header, cookie)                                                                                                                                                                    | string     | Y        |
| key_name           | `api_key` The name of the header, query or cookie parameter to be used..                                                                                                                                                                                                          | string     | Y        |
| bearer_format      | `bearer` A hint to the client to identify how the bearer token is formatted. Bearer tokens are usually generated by an authorization server, so this information is primarily for documentation purposes.                                                                         | string     | Y        |
| flows              | `oauth2` An object containing configuration information for the flow types supported.                                                                                                                                                                                             | OAuthFlows | Y        |
| openid_connect_url | OpenId Connect URL to discover OAuth2 configuration values.                                                                                                                                                                                                                       | string     | Y        |
| checker            | Specify a function to check the original authentication information and convert it to the return type of this function. This function must return `Option<T>` or `poem::Result<T>`, with `None` meaning a General Authorization error and an `Err` reflecting the error supplied. | string     | Y        |

# OAuthFlows

| Attribute          | description                                              | Type      | Optional |
|--------------------|----------------------------------------------------------|-----------|----------|
| implicit           | Configuration for the OAuth Implicit flow                | OAuthFlow | Y        |
| password           | Configuration for the OAuth Resource Owner Password flow | OAuthFlow | Y        |
| client_credentials | Configuration for the OAuth Client Credentials flow      | OAuthFlow | Y        |
| authorization_code | Configuration for the OAuth Authorization Code flow      | OAuthFlow | Y        |

# OAuthFlow

| Attribute         | description                                                                                  | Type        | Optional |
|-------------------|----------------------------------------------------------------------------------------------|-------------|----------|
| authorization_url | `implicit` `authorization_code` The authorization URL to be used for this flow.              | string      | Y        |
| token_url         | `password` `client_credentials` `authorization_code` The token URL to be used for this flow. | string      | Y        |
| refresh_url       | The URL to be used for obtaining refresh tokens.                                             | string      | Y        |
| scopes            | The available scopes for the OAuth2 security scheme.                                         | OAuthScopes | Y        |

# Multiple Authentication Methods

When `SecurityScheme` macro is used with an enumerated type, it is used to define multiple authentication methods.

```rust
use poem_openapi::{OpenApi, SecurityScheme};
use poem_openapi::payload::PlainText;
use poem_openapi::auth::{ApiKey, Basic};

#[derive(SecurityScheme)]
#[oai(ty = "basic")]
struct MySecurityScheme1(Basic);

#[derive(SecurityScheme)]
#[oai(ty = "api_key", key_name = "X-API-Key", key_in = "header")]
struct MySecurityScheme2(ApiKey);

#[derive(SecurityScheme)]
enum MySecurityScheme {
    MySecurityScheme1(MySecurityScheme1),
    MySecurityScheme2(MySecurityScheme2),
}

struct MyApi;

#[OpenApi]
impl MyApi {
    #[oai(path = "/test", method = "get")]
    async fn test(&self, auth: MySecurityScheme) -> PlainText<String> {
        match auth {
            MySecurityScheme::MySecurityScheme1(auth) => {
                PlainText(format!("basic: {}", auth.0.username))
            }
            MySecurityScheme::MySecurityScheme2(auth) => {
                PlainText(format!("api-key: {}", auth.0.key))
            }
        }
    }
}
```