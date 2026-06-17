# Stateful vs stateless, and why a Security Group's source field still can't block an IP

Two things tripped me up here, so I'm writing both down.

## "Wait, a Security Group has a source field too — doesn't that let me block an IP?"

This was my first wrong assumption. The source field looks like it should let you control individual addresses, and it does, but only in one direction: it can narrow who is *allowed*. It can never block anyone.

So I can write things like:

- Allow port 8000 from `1.2.3.4` — only that address gets in.
- Allow port 22 from `10.0.0.0/16` — only my internal network can SSH.

But I can't write "block port 8000 from `3.4.5.6`," because there's no deny in a Security Group at all.

The way it finally made sense to me: a Security Group source is a guest list — only these people may come in. A NACL deny is a banned list — this specific person may never come in.

And here's where it actually matters. Say I want the whole internet to reach my site except one attacker. With a Security Group I'm stuck: to let "everyone" in I set the source to `0.0.0.0/0`, and that includes the attacker. There's no way to carve out an exception, because carving out an exception means denying, and I can't deny. A NACL can: deny the attacker, allow everyone else.

## What stateful and stateless actually mean

This is about whether the layer remembers a conversation. Network traffic goes both ways — a request comes in, and a response has to go back out. The question is whether the firewall connects those two halves.

### Stateful — it remembers (Security Group)

If I allow a request in, the Security Group automatically allows the matching response back out. I don't have to write an outbound rule for the reply; it just knows the reply belongs to a request it already approved.

With my port 8000 server:

- The request comes in on 8000, my inbound rule allows it.
- The response goes back out, and it's allowed automatically because the Security Group remembers the request that started it.

### Stateless — it remembers nothing (NACL)

A NACL treats every packet as a stranger. The incoming request and the outgoing response are judged by separate rules. It doesn't connect them.

Same server, but through a NACL:

- The request comes in on 8000 and needs an inbound allow rule.
- The response going back out needs its own outbound allow rule, or it gets blocked.

The part that catches people out: the response doesn't leave on port 8000. It leaves on a random high-numbered port (the ephemeral range, roughly 1024 to 65535). So a strict NACL has to also allow that outbound range, otherwise the request gets in but the reply can't get out, and the connection just hangs.

### Side by side

| | Security Group (stateful) | NACL (stateless) |
|---|---|---|
| Remembers connections? | Yes | No |
| Return traffic | Allowed automatically | Needs its own rule |
| What I configure | One direction | Both directions |
| Ephemeral ports | Handled for me | I have to allow them myself |

The practical upshot is that the stateful Security Group is just easier and safer to manage — I only think about "who can connect to me" and the replies take care of themselves. That's a big part of why Security Groups do the everyday work and NACLs usually sit at allow-all until I specifically need to deny something.
