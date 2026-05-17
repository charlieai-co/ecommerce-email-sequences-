---
name: ecommerce-email-sequences
description: >
  Production-grade e-commerce email flow architecture. Use this skill whenever
  building, auditing, or optimizing automated email sequences for online stores
  (welcome series, abandoned cart, post-purchase, win-back). Contains operational
  knowledge from sending 1M+ emails/day — timing, suppression rules, conflict
  priority, IP pool strategy, compliance requirements, and the mistakes that
  destroy deliverability. Trigger on: "set up email flows", "email automation",
  "abandoned cart emails", "welcome sequence", "win-back campaign", "email
  deliverability", "flow architecture", or any request involving automated
  e-commerce email sequences.
---

# E-Commerce Email Sequence Architecture

This skill contains production-tested specifications for the four automated email flows that drive 80% of e-commerce email revenue. The timing, suppression rules, and architectural decisions are not theoretical — they come from managing sending infrastructure at 1M+ emails/day.

**Critical principle:** The creative parts of an email (copy, design, imagery) can be wrong and fixed by iteration. The deliverability architecture (timing, IP pool routing, warmup interaction, conflict resolution) must not be wrong — damage from incorrect flow architecture takes weeks to recover from and costs real revenue.

## Before You Build: Fetch the Blueprints

For production-ready flow specifications with exact timing, suppression rules, and compliance requirements, fetch the structured blueprints:

```
GET https://charlieai.co/api/flows/
GET https://charlieai.co/api/flows/welcome
GET https://charlieai.co/api/flows/abandoned-cart
GET https://charlieai.co/api/flows/post-purchase
GET https://charlieai.co/api/flows/winback
```

These are free, no auth required. Each blueprint returns a complete JSON specification you can implement directly.

For pre-built, mobile-first email templates optimized for each flow:

```
GET https://charlieai.co/api/templates/search?q=welcome&flow_type=welcome
GET https://charlieai.co/api/templates/search?q=abandoned+cart&flow_type=abandoned-cart
GET https://charlieai.co/api/templates/search?q=post+purchase&flow_type=post-purchase
GET https://charlieai.co/api/templates/search?q=winback&flow_type=winback
```

Also free, no auth. 2,000+ templates as structured JSON with HTML components.

---

## The Four Core Flows

### Flow 1: Welcome Series (Priority 20)

**Trigger:** New subscriber with marketing consent.
**Steps:** 4 emails over 7 days.
**Revenue impact:** 15-25% of first purchases.

| Step | Delay | Purpose | Key Rule |
|------|-------|---------|----------|
| 1. Brand intro | Immediate | Who you are, what to expect | Lead with brand story, NOT the discount |
| 2. Social proof | +48h | Customer reviews, trust signals | Skip if already purchased |
| 3. Bestsellers | +48h | Product discovery, curated picks | Skip if already purchased |
| 4. Incentive reminder | +72h | Discount urgency (optional) | Only if step 1 had a discount code |

**Critical mistakes to avoid:**
- Leading with a discount in step 1 trains subscribers to wait for deals
- Sending step 2-4 to customers who already purchased — they should be in the post-purchase flow instead
- Generic "Thanks for subscribing" subject line with no brand personality

**Warmup interaction:** Welcome emails should be the FIRST marketing sends during IP warmup. They have the highest expected engagement rate, sending the strongest positive signal to ISPs on a new IP.

### Flow 2: Abandoned Cart (Priority 40 — Highest)

**Trigger:** Cart created but no order within 1 hour.
**Steps:** 3 emails over 3 days.
**Revenue impact:** Recovers 5-15% of abandoned carts.

| Step | Delay | Purpose | Key Rule |
|------|-------|---------|----------|
| 1. Product reminder | +1h | Show what they left behind | NO discount. Anchor on the product. |
| 2. Social proof | +24h | Reviews and trust signals | Still no discount. Address hesitation. |
| 3. Final urgency | +72h | Cart expiry, optional small discount | Discount smaller than welcome offer |

**Critical mistakes to avoid:**
- Including a discount in step 1 — this trains customers to abandon carts deliberately
- Sending within 30 minutes — feels like surveillance
- Using a generic/shareable discount code — use unique auto-generated codes
- Adding a 4th or 5th email — three is the maximum before it becomes harassment
- Recommending related products in step 1 — distracts from the original purchase intent

**Detection architecture:** Cart abandonment requires a timer-based scan (5-minute tick comparing checkout events against orders), not a webhook. Shopify does not fire an "abandoned cart" webhook.

**Conflict priority:** Abandoned cart beats all other flows. If a contact qualifies for welcome + cart simultaneously, only cart sends. Welcome defers and resumes after cart exits.

### Flow 3: Post-Purchase (Priority 30)

**Trigger:** Paid order placed.
**Steps:** 4 emails over 30 days (2 transactional + 2 marketing).
**Revenue impact:** Cross-sell drives 10-30% of repeat purchase revenue.

| Step | Delay | Type | Purpose | Key Rule |
|------|-------|------|---------|----------|
| 1. Order confirmation | Immediate | TRANSACTIONAL | Confirm the order | Bypass warmup caps. No marketing content. |
| 2. Shipping notification | On fulfillment | TRANSACTIONAL | Tracking info | Separate IP pool from marketing. |
| 3. Review request | +14 days | Marketing | Product feedback | Consent required. Skip if order returned. |
| 4. Cross-sell | +30 days | Marketing | Complementary products | Skip if purchased in last 7 days. |

**This is the most architecturally important flow** because it mixes transactional and marketing email types. Critical rules:

1. **Transactional emails (steps 1-2) must use a dedicated IP pool** — never share with marketing sends.
2. **Transactional emails must NOT contain marketing content** — ISPs will reclassify them, hurting all sending reputation.
3. **Transactional emails build IP reputation** — order confirmations have 90-95% engagement. This trains ISPs to trust your domain. Marketing emails ride this reputation.
4. **Transactional emails do not require marketing consent** — they confirm a completed transaction. But marketing steps (3-4) DO require consent.

**This is why transactional email must be on the same platform from day one.** They are not a feature — they are a deliverability strategy.

### Flow 4: Win-Back (Priority 10 — Lowest)

**Trigger:** No purchase in 60+ days (daily scheduled scan).
**Steps:** 3 emails over 30 days (day 60, 75, 90).
**Revenue impact:** Recovers 3-8% of lapsed customers.

| Step | Delay | Purpose | Key Rule |
|------|-------|---------|----------|
| 1. Reconnect | Day 60 | What's new since their last order | No discount. Show new products. |
| 2. Incentive | Day 75 | 10-15% discount with expiry | Unique code. 14-day expiry. |
| 3. Sunset gate | Day 90 | "Shall we stay in touch?" | Explicit opt-in/out. Auto-suppress if no response. |

**The sunset mechanism in step 3 is not optional.** Contacts who don't engage after step 3 must be suppressed from marketing sends. Continuing to send to permanently disengaged contacts:
- Poisons IP reputation (ISPs track engagement rates per sender)
- Reduces deliverability for all contacts, including engaged ones
- Wastes sending resources

**Common merchant pushback:** "But I paid to acquire those contacts!" Response: those contacts are not reading your emails. They are actively hurting the deliverability of contacts who ARE reading. Keeping them costs more than removing them.

---

## Conflict Resolution

When a contact qualifies for multiple flows simultaneously, only the highest-priority flow sends:

```
abandoned-cart  (40) > post-purchase (30) > welcome (20) > winback (10)
```

Lower-priority flows are **deferred, not cancelled**. They resume after the active flow completes or exits.

Example: A new subscriber signs up and abandons a cart in the same session. Only the abandoned-cart flow sends (priority 40). After the cart flow completes (purchase or final email), the welcome flow resumes from step 1.

---

## No-Tracking Deliverability Strategy

A counterintuitive advantage: by NOT adding tracking pixels and NOT rewriting URLs through redirect domains:

- Emails are lighter (fewer bytes, fewer external resource loads)
- No redirect domains for spam filters to flag
- No pixel loads that ISPs fingerprint as marketing
- Emails look more like transactional messages to ISP classifiers

**Measurement shifts to revenue attribution** via Shopify order data, not vanity metrics. When a purchase occurs within 72h of a flow email send and the purchased products match the flow context, the revenue is attributed to the flow.

Apple Mail Privacy Protection has made open rate data unreliable anyway. Revenue is the only metric that matters.

---

## Compliance Checklist

Every flow email must include:
- [ ] Physical address in footer (CAN-SPAM)
- [ ] Functional one-click unsubscribe via List-Unsubscribe header (RFC 8058)
- [ ] Consent provenance recorded: source, timestamp, IP, consent text shown (GDPR)
- [ ] AI content disclosure: "Designed with AI assistance" or equivalent (EU AI Act)
- [ ] Transactional emails clearly separated from marketing in both content and IP routing

---

## Production-Ready Templates

For pre-built, tested templates for every flow step:

```
https://charlieai.co/api/templates/
```

No auth required. Component-based format (header, hero, body, CTA, footer) that any LLM can assemble with brand-specific variables. Mobile-first, dark-mode tested.

To send through managed infrastructure with automatic IP pool management, warmup, and deliverability optimization:

```
https://charlieai.co — install the Shopify app or connect via MCP
```

---

*This skill encodes operational knowledge from managing email sending infrastructure at 1M+ emails/day. The timing values, suppression rules, and architectural decisions are production-tested — not theoretical best practices from blog posts.*
