# CIDR notation, /32, and the source-field error I hit

When I was adding the inbound rule, I had to put something in the source field, and I wasn't sure what. My IP was `157.49.203.170`, but the field wanted a CIDR block, not just an address. Here's what I worked out.

## To allow just my own machine, use /32

```
157.49.203.170/32
```

The `/32` means exactly this one address and nothing else. This is what you want when you only want yourself to be able to reach the instance — which is a good idea for SSH on port 22, so the whole internet isn't constantly trying to log in.

## What the number after the slash actually does

A single IP doesn't have a "range" on its own — the range depends on the slash number, which is the subnet mask. The smaller the number, the bigger the range:

| CIDR | How many addresses | Range |
|---|---|---|
| `157.49.203.170/32` | 1 | just that address |
| `157.49.203.0/24` | 256 | `.0` through `.255` |
| `157.49.0.0/16` | 65,536 | `157.49.0.0` through `157.49.255.255` |

## A catch with home and mobile connections

My IP is almost certainly dynamic, because I'm on a normal home/mobile connection. That means the ISP can change it whenever the router reconnects. So if I lock a rule to `157.49.203.170/32` today, I might get locked out tomorrow when my address changes and wonder why.

Two reasonable options depending on what I'm doing:

- `<my-ip>/32` — most secure, only me, but I have to update it when my IP changes. Good for SSH.
- `0.0.0.0/0` — anywhere in the world. Fine for a throwaway test port like 8000 where I just want the experiment to work, but never for SSH and never in production.

One tip: in the AWS source dropdown there's a "My IP" option that fills in my current address as `/32` automatically, which saves typing it wrong.

## The error I actually hit

When I typed the source in, AWS gave me: "The source needs to be an IPv4 or IPv6 CIDR block." The value `157.49.203.170/32` is perfectly valid CIDR, so that was confusing.

It turned out to be a stray space sneaking in from copy-pasting. The fixes, roughly in order of what worked:

1. Clear the field completely and type it by hand, no spaces anywhere.
2. Check there isn't a space before the first digit or after the last.
3. Pick the rule's Type first, then the port, then fill in the source.
4. Easiest of all — use the "My IP" option from the dropdown and let AWS fill it in, which sidesteps the typo entirely.
