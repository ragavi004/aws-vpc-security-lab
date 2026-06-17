# How NACL rule ordering works when two rules seem to apply

The thing that confused me about NACLs was the rule numbers. Security Groups don't have them, so why do NACLs? And what happens if two rules both seem to apply to the same traffic — which one wins?

## NACLs read rules in order and stop at the first match

A NACL goes through its rules from the lowest number upward, and the moment it finds one that matches the traffic, it applies that rule's decision — allow or deny — and ignores everything below it.

The reason the numbers exist at all is that a NACL can both allow and deny, which means you can write two rules that contradict each other. AWS needs a way to decide which one wins, and the rule number is that tiebreaker. Lower number means higher priority, means it gets checked first.

This is the opposite of a Security Group, which has no ordering — it looks at all its rules together, and if any of them allows the traffic, it's allowed. Security Groups can't deny, so they never have a conflict to resolve in the first place.

## "Both rules apply" — actually, they don't

This was my misunderstanding. The first matching rule wins and evaluation stops right there. The second rule never even gets read. I tested this both ways in the lab.

Allow first, deny second:

```
Rule 100  ALLOW  all traffic
Rule 200  DENY   port 8000
```

A request to port 8000 hits rule 100 first, which matches ("all traffic"), so it's allowed. Rule 200 is never reached, so the deny does nothing.

Now flip the numbers:

```
Rule 100  DENY   port 8000
Rule 110  ALLOW  all traffic
```

Same request, but now rule 100 matches first and denies it. Rule 110 never gets a look. Same two rules, opposite result, and all I changed was the numbering.

## The detail I almost missed: "matches" is doing a lot of work

A rule only counts if the traffic actually fits its conditions. In that second example, rule 100 only denies port 8000. So if a request came in for port 22 instead, rule 100 wouldn't match it, and the NACL would fall through to rule 110 and allow it.

So it's not "the lowest rule always wins." It's "the lowest-numbered rule that actually matches this traffic wins." A low-numbered rule that doesn't apply just gets skipped.

## The asterisk rule at the bottom

Every NACL ends with a rule shown as `*` that you can't edit. It's the catch-all: if nothing above matched, deny. It's there as a safety net so that anything you didn't explicitly account for gets rejected rather than slipping through.

## One habit I picked up

People number their rules with gaps — 100, 200, 300 instead of 1, 2, 3 — so that later you can slot a higher-priority rule in between (a new rule 150 above 200, say) without having to renumber everything. Makes sense once you understand that the number is the priority.
