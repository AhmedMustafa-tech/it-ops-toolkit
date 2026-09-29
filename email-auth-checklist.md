# Email authentication checklist (SPF, DKIM, DMARC, MX)

Use this when you move bulk or automated mail to a **dedicated subdomain** (e.g. `mail.example.com`) to protect your main domain's reputation.

## Why a subdomain?

If newsletters, reminders and your people's everyday email all send from `example.com`, one bad bulk campaign can get the **whole domain** blocklisted. That includes mail to customers, partners and regulators. Give every bulk sender its own subdomain.

## Checks (run from any terminal)

```bash
D=mail.example.com

# SPF: exactly ONE record, starting with v=spf1
dig +short TXT $D | grep spf1

# DKIM: selector comes from your sending platform (e.g. mte1, s1, google)
dig +short TXT mte1._domainkey.$D
dig +short CNAME mte1._domainkey.$D

# DMARC: start with p=none + reporting, tighten later
dig +short TXT _dmarc.$D

# MX: only needed if the subdomain must RECEIVE mail (e.g. replies)
dig +short MX $D
```

## The classic mistake: replies bounce after the move

SPF and DKIM make a subdomain able to **send**, not **receive**. If customers reply to `support@mail.example.com` and there's no **MX record**, replies bounce with "domain not found".

Two fixes:

1. Keep **Reply-To** on the address people already use (`support@example.com`) while **From** moves to the subdomain. Most CRMs and ESPs support a separate Reply-To.
2. Or add MX records for the subdomain and route inbound mail to the existing mailbox.

## Before you call it done

- [ ] Every bulk sender identified (check your data warehouse or ESP logs, not just the obvious ones)
- [ ] SPF, DKIM and DMARC pass on a real test message (look at the headers: `spf=pass dkim=pass dmarc=pass`)
- [ ] Replies land where people expect
- [ ] Bounce/complaint monitoring is on for the new subdomain
- [ ] An observation period passes clean before you close the incident

> 💡 A clean complaint and bounce report doesn't prove the list is clean: **spam traps never complain or bounce.** Check recipient list quality too.
