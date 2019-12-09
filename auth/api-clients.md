# API Clients & Key

**API Clients** are consumers of the app's APIs. For example, you may have an iOS, Android, and web app, all using the same set of backend APIs. In this case, you'll have one Skygear Apps, with three API clients for each platforms.

Each API client is associated with a unique **API Key** and a set of configurations. API consumers are required to provide a valid API key when calling app APIs. 

Using a separate API clients for different platforms allows you to identify which platform the request comes from by the API key.

In the above example, you may want to configure the API clients as follows:

| Name | Session Transport | API Key |
| :--- | :--- | :--- |
| iOS | Header | &lt;iOS API Key&gt; |
| Android | Header | &lt;Android API Key&gt; |
| Web | Cookies | &lt;Web API Key&gt; |

### Session Transports

After logging in, the client would receive an access token, representing the user session. Session transport configures where to store this access token.

* For web platforms, it is a common practice to put access token in cookies, so that browser would store and transmit it securely.
* For native platforms \(e.g. iOS & Android\), it is a common practice to put access token in HTTP Authorization header.

