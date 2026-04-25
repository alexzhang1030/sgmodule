# Surge Modules

Reusable Surge modules for macOS networking workflows.

## VPN Split DNS Router

`vpn-split-dns-router.sgmodule` routes one internal DNS suffix through macOS system DNS, then lets private VPN routes handle the resolved addresses.

This is useful for Tailscale, ZeroTier, WireGuard, and corporate VPN setups where internal domains resolve through the system resolver.

### Install

Use this Module URL in Surge:

```text
https://raw.githubusercontent.com/alexzhang1030/sgmodule/main/vpn-split-dns-router.sgmodule
```

After installing, set the module argument:

```ini
suffix=sr
```

Replace `sr` with your internal DNS suffix, such as `corp.example.com` or `tailnet.ts.net`.

### What It Adds

```ini
[Host]
{{{suffix}}} = server:system
*.{{{suffix}}} = server:system

[Rule]
DOMAIN-SUFFIX,{{{suffix}}},DIRECT
IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
IP-CIDR,192.168.0.0/16,DIRECT,no-resolve
IP-CIDR,100.64.0.0/10,DIRECT,no-resolve
IP-CIDR6,fc00::/7,DIRECT,no-resolve
```

### Requirements

The macOS system resolver must resolve the internal suffix.

```bash
dscacheutil -q host -a name service.sr
```

The resolved address must have a VPN route.

```bash
route -n get 10.10.18.25
```

### Verify Through Surge

```bash
/Applications/Surge.app/Contents/Applications/surge-cli reload
/Applications/Surge.app/Contents/Applications/surge-cli flush dns
curl --proxy http://127.0.0.1:6152 http://service.sr
```

Replace `service.sr` with a host under your internal suffix.
