Setting up Authentication
Transformer Lab supports several authentication methods. Enable one or more of the following providers by setting environment variables in the Transformer Lab .env file.

Email Authentication
Google OAuth
GitHub OAuth
OIDC / OpenID Connect (Any IdP)
Email Authentication
Email-based authentication is enabled by default. To control it explicitly:

EMAIL_AUTH_ENABLED="true"

If you enable email authentication, you must also set up SMTP so that the server can send confirmation emails during registration.

Google OAuth
To obtain a client ID and secret, create an OAuth 2.0 Client ID in the Google Cloud Console.

Set Application type to "Web Application".
Authorized JavaScript origins should be the exact host name you would use in your browser (including protocol and port, if required). e.g. http://lab.mydomain.com:8338
Authorized redirect URIs should be the exact server name with /auth/google/callback appended. e.g. http://lab.mydomain.com:8338/auth/google/callback
Make sure to record your client secret, as you will not be able to access this later.
Then set this in your .env file:

GOOGLE_OAUTH_ENABLED="true"
GOOGLE_OAUTH_CLIENT_ID="your-google-oauth-client-id.apps.googleusercontent.com"
GOOGLE_OAUTH_CLIENT_SECRET="your-google-oauth-client-secret"


GitHub OAuth
To obtain a client ID and secret, create an OAuth app under GitHub → Settings → Developer settings → OAuth Apps.

Then set this in your .env file:

GITHUB_OAUTH_ENABLED="true"
GITHUB_OAUTH_CLIENT_ID="your_github_client_id"
GITHUB_OAUTH_CLIENT_SECRET="your_github_client_secret"

OIDC / OpenID Connect (Any IdP)
You can add one or more generic OIDC providers (e.g., Okta, Keycloak, Auth0, Azure AD, or any OpenID Connect–compliant identity provider).

For each provider, set the following variables, replacing N with an index (0, 1, 2, …):

OIDC_N_DISCOVERY_URL – The IdP's OpenID discovery endpoint (e.g., https://your-idp.example.com/.well-known/openid-configuration).
OIDC_N_CLIENT_ID – OAuth 2.0 client ID registered with the IdP.
OIDC_N_CLIENT_SECRET – OAuth 2.0 client secret registered with the IdP.
OIDC_N_NAME (optional) – Label shown on the login button (e.g., "Company SSO"). Defaults to "OpenID #1", "OpenID #2", etc.
Example for a single provider:

OIDC_0_DISCOVERY_URL="https://your-idp.example.com/.well-known/openid-configuration"
OIDC_0_CLIENT_ID="your-client-id"
OIDC_0_CLIENT_SECRET="your-client-secret"
OIDC_0_NAME="Company SSO"


In your IdP's app configuration, set the redirect / callback URI to:

<API_BASE_URL>/auth/oidc-0/callback

For additional providers, increment the index (oidc-1, oidc-2, and so on). The login page displays a "Continue with <name>" button for each configured provider.