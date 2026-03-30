# Disclaimer

ggCON is provided by [GG Host](https://www.gghost.games){target=_blank} as a best-effort service for use with GG Host SCUM dedicated servers.

## Use at Your Own Risk

ggCON is provided **as-is**, without warranty of any kind, express or implied. By installing or using ggCON, you acknowledge and accept the following:

- ggCON interacts directly with the SCUM dedicated server process at a low level. Incorrect configuration or misuse may result in server instability, unexpected behaviour, or data loss.
- GG Host is not responsible for any loss of game data, player data, server downtime, or damages of any kind arising from the use or misuse of ggCON.
- ggCON is provided as a closed-alpha service and may change, be interrupted, or be discontinued at any time without notice.
- Compatibility with future SCUM updates is not guaranteed. SCUM game updates may break functionality without prior warning.

## Security & Attack Surface

!!! danger "Network exposure"
    ggCON opens additional network ports on your server (HTTP and optionally RCON). **This increases your server's attack surface.** By enabling ggCON, you are exposing administrative interfaces that could be targeted by unauthorised access attempts, brute-force attacks, denial-of-service (DDoS) attacks, or other malicious activity.

It is **your responsibility** to secure your server, including but not limited to:

- Setting a strong, unique password
- Restricting access to trusted IPs using `AllowedIPs` and `AllowedCIDRs`
- Keeping your credentials private and not sharing them in public channels
- Binding to `127.0.0.1` (localhost) when external access is not required
- Using the SSL panel proxy or a reverse proxy with TLS when accessing the panel over the internet

**GG Host accepts no liability for unauthorised access, data breaches, DDoS attacks, service disruption, or any other security incidents resulting from the use of ggCON, regardless of whether the server was properly configured.** The decision to expose administrative interfaces on your server is yours alone.

## Licence Validation

ggCON requires periodic communication with the GG Host licence validation service to verify your licence status. If the validation service is unreachable, ggCON will continue to operate using a cached validation for a limited time, after which the mod may disable itself.

## No Guarantees

GG Host makes no guarantees regarding:

- Uptime or availability of the licence validation service
- Continued compatibility with future versions of SCUM or its dependencies
- The accuracy or completeness of data returned by ggCON endpoints
- Response times for bug fixes or feature requests

## Limitation of Liability

To the maximum extent permitted by applicable law, GG Host shall not be liable for any indirect, incidental, special, consequential, or punitive damages, or any loss of profits, revenue, data, or goodwill, arising out of or in connection with your use of ggCON. This includes, without limitation, damages arising from security incidents, unauthorised access, denial-of-service attacks, or data loss.

## Changes to This Disclaimer

GG Host reserves the right to update this disclaimer at any time. Continued use of ggCON following any changes constitutes acceptance of the updated terms.

---

*For questions, contact us via [GG Host](https://www.gghost.games){target=_blank}.*
