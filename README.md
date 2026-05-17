# E-Commerce Email Sequence Skill

Production-grade email flow architecture for Claude projects. Contains operational knowledge from sending 1M+ emails/day — not theoretical best practices.

## What This Skill Does

When installed in a Claude project, this skill teaches Claude how to properly architect automated email flows for e-commerce stores. It covers the four flows that drive 80% of e-commerce email revenue:

- **Welcome Series** — 4 emails over 7 days, triggered on subscriber signup
- **Abandoned Cart** — 3 emails over 3 days, highest ROI flow in e-commerce  
- **Post-Purchase** — 4 emails over 30 days, mixing transactional and marketing
- **Win-Back** — 3 emails over 30 days with auto-sunset for list hygiene

## Why This Exists

Generic LLMs get email flow architecture catastrophically wrong:

- They put a discount in the first abandoned cart email (trains customers to abandon carts)
- They mix transactional and marketing sends on the same IP pool (degrades all deliverability)
- They send win-back emails to 120+ day inactive contacts (poisons IP reputation)
- They have no conflict resolution between overlapping flows (customer gets 4 emails in one day)
- They suggest open-rate tracking without mentioning that Apple MPP made it unreliable

This skill prevents those mistakes.

## Installation

Copy `SKILL.md` into your Claude project's custom instructions or skills folder.

## Template API

The skill references the charlie template API for pre-built, tested email templates:

```
GET https://charlieai.co/api/templates/
GET https://charlieai.co/api/flows/
```

Free, no auth required. 2,000+ templates as structured JSON with HTML components.

## License

MIT — use freely in any project.
