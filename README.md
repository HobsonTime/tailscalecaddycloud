# Tailscale through Caddy and Cloudflare
 
Used to quickly deploy a xcaddy container with the cloudflare plugin on Docker. Securly connected through tailscale
 
This configuration is meant to connect [Tailscale](https://tailscale.com/), [Caddy](https://caddyserver.com/), and [Cloudflare](https://www.cloudflare.com/).
 
Allowing for all services you choose to be ran on your NAS, machine, or docker network to be accessed by a domain name through tailscale.
 
> **Note:** These instructions assume you already have a few services running on your device and that tailscale is already installed and properly configured on your machine. If not, some helpful instructions will be below.
 
---
 
## Personal Note
 
No matter how hard I tried I could not find a docker image or github account with a straightforward caddy container that came with the cloudflare plugin AND wasn't bloated with a bunch of other information. I used this configuration on my own NAS, while using Portainer and it works on TrueNas Scale. I have docker containers running in the default apps method that TrueNas Scale provides and this caddy instance is able to reach all of them thanks to the Caddyfile and host network configuration.
 
---

## Background
 
I learned how to do most of this through this [video from tailscale](https://www.youtube.com/watch?v=Vt4PDUXB_fg).
 
It got me 85% of the way there. But unfortunately the data is outdated and I had to scrounged the depths of the internet for hours to figure this all out.
 
---
 
## Key Take Aways
 
- All services are only accessible through Tailscale vpn
- Your services are still not exposed to the open internet
- Each service can be accessed on port 443 (https) with the reverse proxy and domain you purchased
- Cloudflare protection for your domain
- Reverse Proxy safety net and utility of multiple (almost endles) services running on the same domain name/ip adddress
 
### Example of Workflow
 
```
Websearch:  jellyfin.<domainname>.com
Cloudflare: Routes jellyfin.<domain>.com to tailscale servers
Tailscale:  Verifies identify of user, sends request to tailscale host
Caddy:      Receives request for jellyfin.<domainname>.com
Caddy:      Notes jellyfin.<domainname>.com is mapped to http://192.168.23.43:8096
Caddy:      Returns jellyfin.<domainname>.com access
```
 
## Tailscale
 
Ensure tailscale is already installed on your device and that you can access it.
 
If deploying tailscale on this instance you will need to create a `TS_AUTHKEY` on tailscale. To allow the creation of a new node on your tailscale network.
 
Check that out here: [Tailscale Auth Keys](https://tailscale.com/docs/features/access-control/auth-keys)
 
---
 
## Caddy
 
We have to use a xcaddy instance as we are including a cloudflare plugin which is not included on the base caddy model.

Once caddy is installed all you have to do is ensure the caddyfile is up to date. 
You will make constant revisions to it and you just need to ensure caddy reads that new caddyfile. 
You can restart the instance or reload it with a command.[Caddy Documentation](https://caddyserver.com/docs/)
 
Here is an example of what one service in your caddyfile will look. Check out the Caddyfile for the full example.
 
**Caddyfile**
```
immich.<domainname>.com {
    reverse_proxy http://192.168.68.54:30041
    import cloudflare
}
```
 
---
 
## Cloudflare
 
Step 1: Create a cloudflare account

Step 2: Transfer your domain to Cloudflare [Official documentation](https://developers.cloudflare.com/registrar/get-started/transfer-domain-to-cloudflare/)

Step 3: Prepare to set up reverse proxy on Cloudflare [DNS/Reverse proxy documentation](https://developers.cloudflare.com/fundamentals/concepts/how-cloudflare-works/#cloudflare-as-a-reverse-proxy)

Step 4: Set up your A Record. Follow the image format below as of April 2026. [Subdomain Info](https://developers.cloudflare.com/fundamentals/manage-domains/manage-subdomains/)

> **Note:** The IP address is the tailscale node you want your domain to connect to. The `*` tells cloudflare to allow any subdomain you route through it by your reverse proxy. In this case, caddy. Keep TTL while you are testing it. Leave the proxy status off.

![how to configure the proper A record](/images/Cloudflare.png)

Step 5: [API Key Creation](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/) Now, create an API key. Go to your Cloudflare account. Access Profile -> API tokens -> Create Token
> **Creating the token** Choose the correct controls. `Zone:Read, DNS:Edit`. Create a name. Create. Save the key in a very secure place. **ONLY** use it in an .ENV file or when prompted on something like portainer with environment variables.

Step 6: See if it works!

**If on mac or linux**
```
dig <domainname.com>
```
**windows**
```
nslookup <domainname.com>
```
 
### Domain Control
 
**Do I need to buy a domain on cloudflare?**
 
No, you can buy a domain from any dealer such as the ones listed below and then TRANSFER it to cloudflare to control.
 
- [GoDaddy](https://www.godaddy.com/domains)
- [Namecheap](https://www.namecheap.com/)
 
---
 
## Notes
 
### Improvements
 
If you are wanting to run this caddy instance on its own isolated network, give it a shot. I haven't done it yet.
