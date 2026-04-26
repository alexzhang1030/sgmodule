# Surge Modules

Reusable Surge modules for macOS networking workflows.

## Static Host Overrides

`static-host-overrides.sgmodule` maps up to 10 domains to fixed IP addresses through Surge's `[Host]` section.

This is useful for SSH local forwarding, reverse proxies, staging cutovers, and local testing where a real hostname should resolve to `127.0.0.1`, a private IP, or a VPN address.

### Install

Use this Module URL in Surge:

```text
https://raw.githubusercontent.com/alexzhang1030/sgmodule/main/static-host-overrides.sgmodule
```

After installing, set the module arguments you need:

```ini
domain1=foo.internal
ip1=127.0.0.1
domain2=bar.internal
ip2=10.0.0.25
```

This module exposes 10 `domainN` and `ipN` pairs.

Surge renders each pair as a separate editable field because the module defines a concrete default value for every argument.

### What It Adds

```ini
[Host]
{{{domain1}}} = {{{ip1}}}
{{{domain2}}} = {{{ip2}}}
...
{{{domain10}}} = {{{ip10}}}
```

### Example

If you forward local port `443` to an internal HTTPS service:

```bash
ssh -N -L 443:10.0.0.25:443 user@jump
```

Then set:

```ini
domain1=foo.internal
ip1=127.0.0.1
```

Requests to `https://foo.internal` will resolve to your local machine.

## VPN Split DNS Router

`vpn-split-dns-router.sgmodule` routes one internal DNS suffix through a VPN DNS server, then lets private VPN routes handle the resolved addresses.

This is useful for Tailscale, ZeroTier, WireGuard, and corporate VPN setups where internal domains resolve through a VPN DNS server.

### Install

Use this Module URL in Surge:

```text
https://raw.githubusercontent.com/alexzhang1030/sgmodule/main/vpn-split-dns-router.sgmodule
```

After installing, set the module argument:

```ini
suffix=corp
dns-server=10.0.0.53
```

Use bare suffix format: `corp`, `corp.example.com`, or `internal.example.com`.

Set `dns-server` to the resolver that answers the internal suffix. Tailscale users commonly use `100.100.100.100`.

### What It Adds

```ini
[Host]
{{{suffix}}} = server:{{{dns-server}}}
*.{{{suffix}}} = server:{{{dns-server}}}

[Rule]
DOMAIN-SUFFIX,{{{suffix}}},DIRECT
IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
IP-CIDR,192.168.0.0/16,DIRECT,no-resolve
IP-CIDR,100.64.0.0/10,DIRECT,no-resolve
IP-CIDR6,fc00::/7,DIRECT,no-resolve
```

### Requirements

The VPN DNS server must resolve the internal suffix.

```bash
dig +short foo.corp @10.0.0.53
```

The resolved address must have a VPN route.

```bash
route -n get 10.0.0.25
```

### Verify Through Surge

```bash
/Applications/Surge.app/Contents/Applications/surge-cli reload
/Applications/Surge.app/Contents/Applications/surge-cli flush dns
curl --proxy http://127.0.0.1:6152 http://foo.corp
```

Replace `foo.corp` with a host under your internal suffix.
