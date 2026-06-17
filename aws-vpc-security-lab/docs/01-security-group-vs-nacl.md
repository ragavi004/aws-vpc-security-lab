# Security Group vs NACL, and whether you even need a NACL

The question I kept asking myself during this lab was: if a Security Group already lets me control traffic, why does AWS even bother giving me NACLs? Couldn't I just use Security Groups for everything?

For a small setup with one instance, honestly, I mostly could. But once I dug into it I found a few things a NACL does that a Security Group simply can't, and that's what justifies having both.

## A NACL can deny. A Security Group can't.

This was the big realisation. A Security Group is allow-only — there is no deny option anywhere in it. You can say "let port 8000 in," but you can't say "block this one IP."

So if some address, say `3.4.5.6`, is hammering my server, a Security Group leaves me stuck. A NACL doesn't:

```
Rule 100  DENY   source 3.4.5.6
Rule 200  ALLOW  all traffic
```

Blocking a known-bad IP or range is something only a NACL can do. That alone is reason enough for it to exist.

## A NACL covers the whole subnet, and the instance owner can't get around it

A Security Group is attached to a specific instance, and whoever owns that instance can edit it however they like — including doing something careless like allowing all traffic from anywhere just to make their life easier.

A NACL sits one level up, on the subnet. If I deny port 8000 there, it applies to every instance in that subnet, and no individual instance owner can override it from their own Security Group. I actually saw this in experiment 3 of the lab: the Security Group was allowing port 8000, but the NACL deny still won.

So the way I think about it now:

- NACL = a guardrail the whole subnet has to live with.
- Security Group = a per-instance permission slip.

## Two layers means one mistake doesn't sink you

This is the defense-in-depth idea. If someone fat-fingers a Security Group rule (human error is the usual cause of these things), the NACL is still standing as a second check. Two independent gates are just harder to get past than one.

## So when do I actually reach for each?

| | Security Group | NACL |
|---|---|---|
| Everyday work | This is the main tool | Usually left at allow-all |
| Who tends to manage it | App / DevOps engineer | Network or senior engineer |
| Blocking a bad IP | Can't | This is the main reason to use it |
| Subnet-wide rule | No, per-instance only | Yes |
| Fine-grained allow rules | Best for this | More awkward |

In practice the Security Group does most of the day-to-day work because it's stateful and simpler. The NACL mostly sits quietly in the background and comes out when I need to deny something across a whole subnet.

The short version of what clicked for me: it's not Security Group versus NACL. It's the Security Group for normal allow-listing, and the NACL as the subnet-wide backstop that has a deny button the Security Group never had.
