# HearthRate Pricing Structure

## Overview

HearthRate uses a tiered pricing model designed to serve individual agents and scale to teams and brokerages. The pricing follows a declining per-seat cost model to incentivize larger team adoption while maintaining accessibility for solo agents.

## Pricing Tiers

### Free Plan

-   **Price:** $0/month
-   **Seats:** N/A (individual)
-   **Target Audience:** Agents trying out lead sign-in functionality
-   **Features:**
    -   Unlimited open house and live events
    -   QR code sign-in
    -   Basic lead capture
    -   Lead export (CSV/PDF)

### Pro Plan

-   **Price:** $20/month
-   **Seats:** 1 agent
-   **Per-Seat Cost:** $20
-   **Target Audience:** Solo agents and independent professionals
-   **Features:**
    -   Everything in Free
    -   Automatic lead scoring
    -   Email/SMS notifications
    -   Multiplayer tour mode
    -   Live property leaderboard
    -   Decision Deck comparisons
    -   Offer scenario modeling
    -   Offline mode
    -   Client magic links
    -   Priority support

### Team Plans

Team pricing follows a **volume discount model** where the per-seat cost decreases as team size increases.

| Tier    | Seats | Monthly Price | Per-Seat Cost | Savings vs Pro |
| ------- | ----- | ------------- | ------------- | -------------- |
| Team 5  | 5     | $60           | $12/seat      | 40%            |
| Team 10 | 10    | $100          | $10/seat      | 50%            |
| Team 25 | 25    | $200          | $8/seat       | 60%            |
| Team 50 | 50    | $350          | $7/seat       | 65%            |

**Additional Team Features:**

-   Everything in Pro (per agent)
-   Round-robin lead distribution
-   First-come-first-served mode
-   Team analytics dashboard
-   Co-hosted events
-   Admin controls

### Enterprise Plan

-   **Price:** Custom pricing (contact sales)
-   **Seats:** Unlimited
-   **Target Audience:** Large brokerages and franchises
-   **Features:**
    -   Everything in Team
    -   Custom integrations
    -   Company branding (white-label options)
    -   Dedicated support
    -   Custom SLAs

## Pricing Algorithm

The per-seat cost follows a **declining marginal cost curve** based on team size:

```
Per-Seat Cost = Base Price × Discount Factor(seats)

Discount Schedule:
- 1 seat:  100% of base ($20)
- 5 seats: 60% of base ($12)
- 10 seats: 50% of base ($10)
- 25 seats: 40% of base ($8)
- 50 seats: 35% of base ($7)
- 50+ seats: Custom pricing
```

### Volume Discount Curve

```
Seats → Price
1 → $20  (baseline)
5 → $60  (-40% per seat)
10 → $100 (-50% per seat)
25 → $200 (-60% per seat)
50 → $350 (-65% per seat)
```

## Billing Cycles

-   **Monthly:** Default billing cycle
-   **Annual:** 20% discount (available on request)
    -   Pro Annual: $192/year (save $48)
    -   Team 5 Annual: $576/year (save $144)
    -   etc.

## Trial Period

-   **Pro Plan**
-   **Team Plans**
-   **No Credit Card Required:** Users can explore full functionality before committing
-   **After Trial:** Automatic downgrade to Free plan if no payment method added

## Upgrades & Downgrades

### Upgrading

-   Immediate access to new features
-   Prorated credit applied for remainder of billing cycle
-   No service interruption

### Downgrading

-   Takes effect at next billing cycle
-   Data retained for 90 days (read-only after downgrade)
-   Can re-upgrade within 90 days without data loss

## Agent Portability

HearthRate is designed around the **agent as the primary entity**, not the brokerage. This means:

### What Agents Own (Portable Across Organizations)

-   **Tours:** All tour history and data
-   **Clients:** Client relationships and ClientGroups
-   **Decision Decks:** Property comparisons and offer scenarios
-   **Lead History:** Personal lead capture history
-   **Uploaded Files:** Photos, notes, recordings

### What Organizations Own (Non-Portable)

-   **Open House Events:** Events created under org context
-   **Team Analytics:** Aggregate team performance data
-   **Org Settings:** Lead distribution rules, branding

### Moving Between Organizations

When an agent changes brokerages:

1. **Agent retains:**

    - All tour data and client relationships
    - Personal subscription (if on Pro plan)
    - Historical performance data

2. **Process:**

    - Admin removes agent from old organization
    - Agent joins new organization (or stays solo on Pro plan)
    - All personal data travels with agent

3. **Subscription Impact:**
    - **If on Team plan:** Agent loses Team features when removed from org
    - **If on Pro plan:** No change, remains on Pro plan
    - **New org can:** Add agent to their Team plan subscription

This model ensures agents never lose their data history (their "Touring Resume") when changing brokerages, which is critical in an industry with high turnover.

## Backend Implementation

### Database Models

**Subscription Tracking:**

-   `SubscriptionTier`: Defines available plans (Free, Pro, Team-5, Team-10, etc.)
-   `Subscription`: Tracks active subscription per User or Organization
    -   `subscriber_type`: 'user' or 'organization'
    -   `status`: 'active', 'trial', 'canceled', 'past_due'
    -   `current_period_start` / `current_period_end`
    -   `external_subscription_id`: Stripe subscription ID

**Organization Membership:**

-   `OrganizationMembership`: Links User to Organization
    -   `OneToOneField(User)`: Agent can only be in ONE org at a time
    -   `role`: 'owner', 'admin', 'agent'
    -   `is_active`: Allows soft-deactivation without data loss

**Seat Counting Logic:**

```python
def check_seat_availability(organization, subscription):
    """
    Verify org hasn't exceeded seat limit.
    """
    active_members = organization.get_active_members().count()
    max_seats = subscription.tier.max_seats

    if active_members >= max_seats:
        raise ValidationError(f"Organization has reached max seats ({max_seats})")

    return True
```

### Stripe Integration

**Subscription Creation:**

```python
# When team admin upgrades to Team-10 plan
stripe.Subscription.create(
    customer=org.stripe_customer_id,
    items=[{
        'price': 'price_team_10_monthly',  # $100/mo
    }],
    metadata={
        'organization_id': str(org.id),
        'plan': 'team-10'
    }
)
```

**Metered Billing (Future):**
For enterprise plans, consider metered billing for usage-based features:

-   SMS notifications sent
-   API calls to property enrichment services
-   Storage usage for uploaded files

## Support Tiers

| Plan       | Support Level             | Response Time |
| ---------- | ------------------------- | ------------- |
| Free       | Community (docs/forum)    | N/A           |
| Pro        | Email support             | 24-48 hours   |
| Team       | Priority email            | 12-24 hours   |
| Enterprise | Dedicated support + Slack | 4-hour SLA    |

## Feature Flags by Tier

Features are controlled via JSONField in `SubscriptionTier.features`:

```python
{
    "max_events": -1,  # -1 = unlimited
    "max_tours_per_month": -1,
    "max_clients": -1,
    "can_use_offline_mode": True,
    "can_use_decision_decks": True,
    "can_use_offer_scenarios": True,
    "can_use_team_analytics": False,  # Team+ only
    "can_use_lead_distribution": False,  # Team+ only
    "can_cohost_events": False,  # Team+ only
    "email_notifications": True,
    "sms_notifications": True,
    "priority_support": True
}
```

## Future Considerations

### Usage-Based Add-Ons

-   SMS overage: $0.02 per message beyond quota
-   Storage overage: $5 per 10GB beyond included amount
-   Premium property data: $0.05 per property lookup with enriched data

### Partner Integrations

-   CRM integrations (Salesforce, HubSpot): $20/month add-on
-   Transaction management (Dotloop, Skyslope): $30/month add-on
-   MLS direct feeds: Custom pricing per MLS region

### Geographic Expansion

-   International markets may require different pricing due to:
    -   Local payment processing fees
    -   Currency conversion
    -   Regional competitive landscape
