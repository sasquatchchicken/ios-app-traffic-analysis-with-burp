# Intercepting iOS app traffic with Burp Suite

## Dependencies
```
Laptop running Burp Suite
iPhone connected to same Wi-Fi as laptop
Burp's CA certificate installed on iPhone
If app is using certificate pinning you will need to bypass it.
```
## Setup
```
1. Configure Burp Proxy Listener
On laptop Burp Suite:
Proxy → Proxy settings → Proxy Listeners
Listener should be on All Interfaces 0.0.0.0 or your specific local IP not 127.0.0.1
ipconfig for windows
ifconfig for macos/linux

2. Set iPhone to use Burp as proxy
Settings → Wi-Fi → [your network]
Tap the  ⓘ icon→scroll down to HTTP Proxy → Configure Proxy → set to Manual
Server: [your laptop IP]
Port: [what you set up listener to]

3. Install Burp Certificate on iPhone
On iPhone browser go to http://burpsuite while burp is running on laptop
Download the certificate
Install it
Settings → General → VPN & Device Management → Downloaded Profile → Portswigger CA
After install
Settings → General → About → Certificate Trust Settings → Portswigger CA
Enable full trust for Burp Certificate.

4. Start app and watch traffic in Burp
Open the app you choose
In BurpSuite Proxy → HTTP history
Now you should see all outbound requests: login, commands, telemetry, etc.
```
## what you can learn
```
Setting up Burp Suite as an iPhone HTTPS proxy
Installing Burp’s CA cert on iOS for full MITM capabilities
Inspecting and replaying captured requests
Detecting insecure API behavior
Curl & Postman replay for API fuzzing and abuse simulation
```
## Example of a replay simultation with curl
```
curl -v -X POST "https://login.coinbase.com/api/two-factor/v1/active-statuses" -H "Authorization: Bearer <insert_your_token_here>" -H "Content-Type: application/json"
```
## Output
```
Host: login.coinbase.com
> User-Agent: curl/8.4.0
> Accept: */*
> Authorization: Bearer <insert_your_token_here>
> Content-Type: application/json
>
* schannel: remote party requests renegotiation
* schannel: renegotiating SSL/TLS connection
* schannel: SSL/TLS connection renegotiated
< HTTP/1.1 403 Forbidden
< Date: Tue, 20 May 2025 14:06:16 GMT
< Content-Type: text/plain; charset=utf-8
< Content-Length: 42
< Connection: keep-alive
< access-control-allow-headers: Authorization, Content-Type, Accept, Second-Factor-Proof-Token, Client-Id, Access-Token, X-Cb-Project-Name, X-Cb-Is-Logged-In, X-Cb-Platform, X-Cb-Session-Uuid, X-Cb-Pagekey, X-Cb-UJS, Fingerprint-Tokens, X-Cb-Device-Id, X-Cb-Version-Name, identity-version, project-id, proof-token, two-factor-client-id, X-CB-Traffic-Type, tmp-amp-id
< access-control-allow-methods: GET,POST,DELETE,PUT
< access-control-allow-private-network: true
< access-control-expose-headers:
< access-control-max-age: 7200
< Cache-Control: no-store
< strict-transport-security: max-age=31536000; includeSubDomains; preload
< trace-id: <TRACE_ID>
< vary: Origin
< x-content-type-options: nosniff
< x-dns-prefetch-control: off
< x-download-options: noopen
< x-frame-options: SAMEORIGIN
< x-xss-protection: 1; mode=block
< cf-cache-status: DYNAMIC
< Set-Cookie: <COOKIE_HERE>; path=/; expires=Tue, 20-May-25 14:36:16 GMT; domain=.coinbase.com; HttpOnly; Secure; SameSite=None
< Report-To: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v4?<not_for_your_eyes>"}],"group":"cf-nel","max_age":604800}
< NEL: {"success_fraction":0.01,"report_to":"cf-nel","max_age":604800}
< Server: cloudflare
< CF-RAY: 942c64c8eec993b9-EWR
<
{ [42 bytes data]
```
**We can see from the output above the replay was forbiddenadn login was not granted.**
**I also tried this with my banking app trying to replay a login with curl and I was immediately logged out adn session was terminated.**

From a red team perspective this is essential during 
```
Mobile API enumeration
Session hijacking or token abuse scenarios
Testing 2FA bypass or token replay
Fuzzing endpoints for IDOR, role escalation or rate limiting
```
**Used with authorization this maps to TTPs under**
```
MITRE ATT&CK T1056.001 - input capture
OWASP 10 - M1 insecure data storage, M5 insufficient cryptography
```
**https://attack.mitre.org/techniques/T1056/**

**https://owasp.org/www-project-mobile-top-10/**

From a black hat perspective these same steps could be abused as well.

**This exists for defensive validation and ethical research.  Always ensure proper authorization, disclosure, and scope.**

