# HearthRate API Documentation

## Overview

HearthRate provides a RESTful API for managing real estate showings, tours, leads, and decision decks. All endpoints are prefixed with `/api/v1/`.

## Authentication

### JWT Authentication

Most endpoints require JWT authentication. Include the access token in the Authorization header:

```http
Authorization: Bearer <access_token>
```

### Token Endpoints

| Endpoint              | Method | Description                        |
| --------------------- | ------ | ---------------------------------- |
| `/api/v1/register/`   | POST   | Register a new agent account       |
| `/api/v1/login/`      | POST   | Obtain access and refresh tokens   |
| `/api/v1/refresh/`    | POST   | Refresh an expired access token    |
| `/api/v1/google/`     | POST   | Authenticate via Google OAuth      |
| `/api/v1/microsoft/`  | POST   | Authenticate via Microsoft OAuth   |
| `/api/v1/apple/`      | POST   | Authenticate via Apple Sign In     |

### Client Token Authentication

Clients (buyers) access tour lobbies and decks via magic links with access tokens:

```
/tour/{id}/lobby?token={client_access_token}
/deck/{id}/view?token={client_access_token}
```

---

## User Settings & Profile API

Manage user profiles, subscriptions, and account data.

### Profile Endpoints

| Endpoint                            | Method | Description                           |
| ----------------------------------- | ------ | ------------------------------------- |
| `/api/v1/profile/`                  | GET    | Retrieve authenticated user's profile |
| `/api/v1/profile/`                  | PATCH  | Update user profile                   |
| `/api/v1/profile/upload-photo/`     | POST   | Upload profile picture                |
| `/api/v1/data-export/`              | GET    | Export all user data (GDPR/CCPA)      |
| `/api/v1/account/delete/`           | POST   | Request account deletion              |
| `/api/v1/subscription/change-plan/` | POST   | Change subscription tier              |

### Get User Profile

```http
GET /api/v1/profile/
Authorization: Bearer <access_token>
```

**Response:**

```json
{
	"id": "user-uuid",
	"email": "agent@example.com",
	"first_name": "Jane",
	"last_name": "Smith",
	"phone": "+1234567890",
	"bio": "Realtor specializing in downtown condos",
	"license_number": "RE123456",
	"brokerage_name": "Smith Realty",
	"profile_picture": "https://hearthrate.app/profiles/user-uuid.jpg",
	"timezone": "US/Eastern",
	"created_at": "2024-01-15T10:00:00Z"
}
```

### Update User Profile

```http
PATCH /api/v1/profile/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "first_name": "Jane",
  "last_name": "Smith",
  "phone": "+1234567890",
  "bio": "Realtor specializing in luxury homes",
  "license_number": "RE123456",
  "brokerage_name": "Smith Luxury Realty"
}
```

**Response:** Updated user object (same as GET)

### Upload Profile Photo

```http
POST /api/v1/profile/upload-photo/
Authorization: Bearer <access_token>
Content-Type: multipart/form-data

photo: <image file>
```

**Constraints:**

-   Max file size: 5MB
-   Allowed formats: JPEG, PNG, WebP

**Response:**

```json
{
	"url": "https://hearthrate.app/profiles/user-uuid.jpg"
}
```

### Export User Data

Export all user data in JSON format for GDPR/CCPA compliance.

```http
GET /api/v1/data-export/
Authorization: Bearer <access_token>
```

**Response:** JSON file download containing:

-   User profile
-   Events
-   Leads
-   Tours
-   Decision decks
-   Subscription information

### Request Account Deletion

```http
POST /api/v1/account/delete/
Authorization: Bearer <access_token>
```

**Response:**

```json
{
	"message": "Deletion request submitted. Check your email to confirm.",
	"request_id": "deletion-request-uuid"
}
```

### Change Subscription Plan

```http
POST /api/v1/subscription/change-plan/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "tier": "pro"
}
```

**Available tiers:**

-   `free` - Free tier (5 tours, 5 decks)
-   `pro` - Pro tier ($20/mo, unlimited)
-   `team-5` - Team 5 ($60/mo, 5 seats)
-   `team-10` - Team 10 ($100/mo, 10 seats)
-   `team-25` - Team 25 ($200/mo, 25 seats)
-   `team-50` - Team 50 ($350/mo, 50 seats)

**Response:**

```json
{
	"id": "subscription-uuid",
	"tier": {
		"name": "Pro",
		"slug": "pro",
		"price_monthly": "20.00",
		"features": {
			"max_tours": 999,
			"max_decks": 999,
			"ai_scoring": true,
			"offline_mode": true
		}
	},
	"status": "active",
	"current_period_end": "2024-04-15"
}
```

---

## Contact Form API

Public endpoint for website contact form submissions (no authentication required).

### Contact Form Endpoint

| Endpoint          | Method | Description                    |
| ----------------- | ------ | ------------------------------ |
| `/api/v1/contact/` | POST   | Submit contact form inquiry    |

### Submit Contact Form

```http
POST /api/v1/contact/
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "company": "ABC Realty",
  "subject": "Product Demo Request",
  "message": "I'm interested in learning more about HearthRate..."
}
```

**Request Fields:**

-   `name` (string, required) - Contact's full name
-   `email` (string, required) - Contact's email address
-   `phone` (string, optional) - Contact's phone number
-   `company` (string, optional) - Company or brokerage name
-   `subject` (string, optional) - Subject of inquiry
-   `message` (string, required) - Message content

**Response:**

```json
{
	"message": "Thank you! We'll be in touch soon.",
	"id": "submission-uuid"
}
```

---

## Theme Management API

Manage color themes for Pro subscribers. Themes allow brand customization across agent dashboards and client views.

### Theme Endpoints

| Endpoint                  | Method | Description                              |
| ------------------------- | ------ | ---------------------------------------- |
| `/api/v1/themes/`         | GET    | List all active themes                   |
| `/api/v1/themes/{id}/`    | GET    | Get specific theme details               |
| `/api/v1/themes/active/`  | GET    | Get user's currently active theme        |
| `/api/v1/themes/select/`  | POST   | Select and apply a theme (Pro required)  |

### List Themes

```http
GET /api/v1/themes/
Authorization: Bearer <access_token>
```

**Response:**

```json
[
	{
		"id": "theme-uuid",
		"name": "Coral Sunset",
		"slug": "coral-sunset",
		"colors": {
			"primary": "#2B211A",
			"accent": "#DF6C57",
			"secondary": "#6AA7A7",
			"background": "#F6F5F3",
			"surface": "#FFFFFF"
		},
		"preview_image": "https://hearthrate.app/media/theme_previews/coral.jpg",
		"description": "Warm, inviting colors perfect for residential real estate"
	}
]
```

### Get Active Theme

Returns the user's currently active theme. Theme selection follows this precedence:
1. User's selected theme (if set)
2. Organization's selected theme (if user in org and org has theme)
3. Default theme (`is_default=True`)

```http
GET /api/v1/themes/active/
Authorization: Bearer <access_token>
```

**Response:** Theme object (same structure as list response)

### Select Theme

Select and apply a theme. Requires Pro subscription or higher.

```http
POST /api/v1/themes/select/
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "theme_id": "theme-uuid",
  "apply_to": "user"
}
```

**Request Fields:**

-   `theme_id` (string, optional) - Theme UUID to apply, or `null` to reset to default
-   `apply_to` (string, optional) - `"user"` or `"organization"` (default: `"user"`)

**Response:**

```json
{
	"status": "User theme updated",
	"theme": {
		"id": "theme-uuid",
		"name": "Coral Sunset",
		"colors": { ... }
	}
}
```

**Note:** Theme selection requires Pro subscription or higher.

---

## Tours API

Tours represent scheduled property visits with multiplayer support for households.

### Tour Endpoints

| Endpoint                                | Method | Description                                |
| --------------------------------------- | ------ | ------------------------------------------ |
| `/api/v1/tours/`                        | GET    | List all tours for the authenticated agent |
| `/api/v1/tours/`                        | POST   | Create a new tour                          |
| `/api/v1/tours/{id}/`                   | GET    | Retrieve tour details                      |
| `/api/v1/tours/{id}/`                   | PATCH  | Update tour details                        |
| `/api/v1/tours/{id}/`                   | DELETE | Delete a tour                              |
| `/api/v1/tours/{id}/mark_running_late/` | POST   | Update ETA for running late                |
| `/api/v1/tours/{id}/leaderboard/`       | GET    | Get live property rankings                 |
| `/api/v1/tours/{id}/lobby_url/`         | GET    | Get shareable lobby URL                    |
| `/api/v1/tours/{id}/send_invites/`      | POST   | Send invites to group members              |
| `/api/v1/tours/{id}/analytics/`         | GET    | Get tour statistics                        |
| `/api/v1/tours/sync/`                   | POST   | Sync offline operations                    |

### Create Tour

```http
POST /api/v1/tours/
Content-Type: application/json

{
  "client_group": "uuid-of-client-group",
  "title": "Weekend Property Tour",
  "tour_date": "2024-03-15T10:00:00Z"
}
```

**Response:**

```json
{
	"id": "tour-uuid",
	"title": "Weekend Property Tour",
	"tour_date": "2024-03-15T10:00:00Z",
	"status": "draft",
	"lobby_token": "lobby-uuid",
	"lobby_url": "https://hearthrate.app/tour/tour-uuid/lobby?token=lobby-uuid",
	"stops": [],
	"created_at": "2024-03-10T14:30:00Z"
}
```

### Mark Running Late

```http
POST /api/v1/tours/{id}/mark_running_late/
Content-Type: application/json

{
  "minutes": 15
}
```

**Response:**

```json
{
	"status": "success",
	"running_late_minutes": 15,
	"notified_at": "2024-03-15T10:05:00Z"
}
```

### Get Leaderboard

```http
GET /api/v1/tours/{id}/leaderboard/
```

**Response:**

```json
[
	{
		"rank": 1,
		"property_id": "property-uuid",
		"property_address": "123 Main Street",
		"current_group_rating": 4.5,
		"feedback_count": 3,
		"unanimous": true
	},
	{
		"rank": 2,
		"property_id": "property-uuid-2",
		"property_address": "456 Oak Avenue",
		"current_group_rating": 4.0,
		"feedback_count": 3,
		"unanimous": false
	}
]
```

### Tour Stop Endpoints (Nested)

| Endpoint                                 | Method | Description            |
| ---------------------------------------- | ------ | ---------------------- |
| `/api/v1/tours/{tour_id}/stops/`         | GET    | List stops for a tour  |
| `/api/v1/tours/{tour_id}/stops/`         | POST   | Add a property to tour |
| `/api/v1/tours/{tour_id}/stops/{id}/`    | GET    | Get stop details       |
| `/api/v1/tours/{tour_id}/stops/{id}/`    | DELETE | Remove stop from tour  |
| `/api/v1/tours/{tour_id}/stops/reorder/` | POST   | Reorder stops          |

### Reorder Stops

```http
POST /api/v1/tours/{tour_id}/stops/reorder/
Content-Type: application/json

{
  "order": ["stop-uuid-3", "stop-uuid-1", "stop-uuid-2"]
}
```

### Tour Feedback Endpoints (Nested)

| Endpoint                                                 | Method | Description              |
| -------------------------------------------------------- | ------ | ------------------------ |
| `/api/v1/tours/{tour_id}/stops/{stop_id}/feedback/`      | GET    | List feedback for a stop |
| `/api/v1/tours/{tour_id}/stops/{stop_id}/feedback/`      | POST   | Submit feedback          |
| `/api/v1/tours/{tour_id}/stops/{stop_id}/feedback/{id}/` | PATCH  | Update feedback          |

### Submit Feedback

```http
POST /api/v1/tours/{tour_id}/stops/{stop_id}/feedback/
Content-Type: application/json

{
  "rating_vibe": 5,
  "rating_kitchen": 4,
  "rating_living": 5,
  "rating_master": 4,
  "rating_exterior": 3,
  "notes": "Pro: Great natural light\nCon: Small backyard"
}
```

### Tour Sync Endpoint

For offline-first support, sync multiple operations in a single request:

```http
POST /api/v1/tours/sync/
Content-Type: application/json

{
  "operations": [
    {
      "type": "feedback_create",
      "offline_uuid": "client-generated-uuid",
      "timestamp": "2024-03-15T10:30:00Z",
      "data": {
        "tour_stop": "stop-uuid",
        "client": "client-uuid",
        "rating_vibe": 4
      }
    },
    {
      "type": "running_late",
      "offline_uuid": "client-generated-uuid-2",
      "timestamp": "2024-03-15T10:25:00Z",
      "tour_id": "tour-uuid",
      "data": {
        "minutes": 10
      }
    }
  ]
}
```

**Supported Operation Types:**

-   `feedback_create` - Create new feedback
-   `feedback_update` - Update existing feedback
-   `tour_update` - Update tour details
-   `stop_reorder` - Reorder tour stops
-   `running_late` - Update running late status

**Response:**

```json
{
	"processed": 2,
	"results": [
		{
			"offline_uuid": "client-uuid",
			"status": "created",
			"server_id": "feedback-uuid"
		},
		{
			"offline_uuid": "client-uuid-2",
			"status": "updated",
			"server_id": "tour-uuid"
		}
	]
}
```

---

## Decks API

Decision Decks are comparison boards for narrowed-down properties with financial modeling.

### Deck Endpoints

| Endpoint                         | Method | Description               |
| -------------------------------- | ------ | ------------------------- |
| `/api/v1/decks/`                 | GET    | List all decks            |
| `/api/v1/decks/`                 | POST   | Create a new deck         |
| `/api/v1/decks/{id}/`            | GET    | Get deck details          |
| `/api/v1/decks/{id}/`            | PATCH  | Update deck               |
| `/api/v1/decks/{id}/`            | DELETE | Delete deck               |
| `/api/v1/decks/{id}/matrix/`     | GET    | Get comparison matrix     |
| `/api/v1/decks/{id}/share_link/` | GET    | Get shareable client link |
| `/api/v1/decks/{id}/export_pdf/` | GET    | Export as PDF             |
| `/api/v1/decks/{id}/view/`       | GET    | Client view (magic link)  |
| `/api/v1/decks/sync/`            | POST   | Sync offline operations   |

### Create Deck

```http
POST /api/v1/decks/
Content-Type: application/json

{
  "client_group": "group-uuid",
  "title": "Final Three Properties"
}
```

### Get Comparison Matrix

```http
GET /api/v1/decks/{id}/matrix/
```

**Response:**

```json
{
	"deck_id": "deck-uuid",
	"title": "Final Three Properties",
	"items": [
		{
			"id": "item-uuid",
			"rank": 1,
			"property": {
				"id": "property-uuid",
				"address": "123 Main Street",
				"list_price": 450000,
				"bedrooms": 3,
				"bathrooms": 2.5
			},
			"tour_feedback": {
				"avg_rating": 4.5,
				"unanimous": true
			},
			"has_conflict": false
		}
	],
	"comparison_criteria": ["price", "location", "size", "condition"]
}
```

### Deck Item Endpoints (Nested)

| Endpoint                                | Method | Description          |
| --------------------------------------- | ------ | -------------------- |
| `/api/v1/decks/{deck_id}/items/`        | GET    | List items in deck   |
| `/api/v1/decks/{deck_id}/items/`        | POST   | Add property to deck |
| `/api/v1/decks/{deck_id}/items/{id}/`   | DELETE | Remove from deck     |
| `/api/v1/decks/{deck_id}/items/rerank/` | POST   | Update rankings      |

### Rerank Items

```http
POST /api/v1/decks/{deck_id}/items/rerank/
Content-Type: application/json

{
  "ranks": [
    {"id": "item-uuid-1", "rank": 1},
    {"id": "item-uuid-2", "rank": 2},
    {"id": "item-uuid-3", "rank": 3}
  ]
}
```

### Offer Scenario Endpoints (Nested)

| Endpoint                                                     | Method | Description       |
| ------------------------------------------------------------ | ------ | ----------------- |
| `/api/v1/decks/{deck_id}/items/{item_id}/scenarios/`         | GET    | List scenarios    |
| `/api/v1/decks/{deck_id}/items/{item_id}/scenarios/`         | POST   | Create scenario   |
| `/api/v1/decks/{deck_id}/items/{item_id}/scenarios/{id}/`    | PATCH  | Update scenario   |
| `/api/v1/decks/{deck_id}/items/{item_id}/scenarios/compare/` | GET    | Compare scenarios |

### Create Offer Scenario

```http
POST /api/v1/decks/{deck_id}/items/{item_id}/scenarios/
Content-Type: application/json

{
  "name": "Standard Offer",
  "offer_price": 440000,
  "down_payment_percent": 20,
  "interest_rate": 6.5,
  "loan_term_years": 30,
  "property_tax_rate": 2.1,
  "insurance_annual": 1800,
  "hoa_monthly": 150
}
```

**Response:**

```json
{
	"id": "scenario-uuid",
	"name": "Standard Offer",
	"offer_price": 440000,
	"down_payment": 88000,
	"loan_amount": 352000,
	"monthly_payment": 2856,
	"monthly_breakdown": {
		"principal_interest": 2225,
		"property_tax": 770,
		"insurance": 150,
		"hoa": 150,
		"pmi": 0
	},
	"is_affordable": true,
	"housing_ratio": 22.8
}
```

### Deck Sync Endpoint

```http
POST /api/v1/decks/sync/
Content-Type: application/json

{
  "operations": [
    {
      "type": "deck_update",
      "offline_uuid": "uuid",
      "timestamp": "2024-03-15T10:00:00Z",
      "deck_id": "deck-uuid",
      "data": {"title": "Updated Title"}
    },
    {
      "type": "item_create",
      "offline_uuid": "uuid-2",
      "timestamp": "2024-03-15T10:01:00Z",
      "deck_id": "deck-uuid",
      "data": {"property_id": "property-uuid", "rank": 3}
    },
    {
      "type": "scenario_create",
      "offline_uuid": "uuid-3",
      "timestamp": "2024-03-15T10:02:00Z",
      "deck_id": "deck-uuid",
      "item_id": "item-uuid",
      "data": {
        "name": "Aggressive Offer",
        "offer_price": 425000,
        "down_payment_percent": 25
      }
    }
  ]
}
```

**Supported Operation Types:**

-   `deck_create` - Create new deck
-   `deck_update` - Update deck
-   `item_create` - Add item to deck
-   `item_update` - Update deck item
-   `item_rerank` - Reorder items
-   `scenario_create` - Create offer scenario
-   `scenario_update` - Update scenario

---

## Affordability Calculator

Public endpoint for mortgage calculations (no authentication required).

```http
POST /api/v1/decks/calculator/affordability/
Content-Type: application/json

{
  "home_price": 450000,
  "down_payment_percent": 20,
  "interest_rate": 6.5,
  "loan_term_years": 30,
  "annual_income": 150000,
  "property_tax_rate": 2.1,
  "insurance_annual": 1800,
  "hoa_monthly": 150
}
```

**Response:**

```json
{
	"loan_amount": 360000,
	"down_payment": 90000,
	"monthly_payment": 2856,
	"monthly_breakdown": {
		"principal_interest": 2275,
		"property_tax": 787,
		"insurance": 150,
		"hoa": 150,
		"pmi": 0
	},
	"closing_costs_estimate": 13500,
	"cash_needed": 103500,
	"housing_ratio": 22.8,
	"is_affordable": true,
	"max_affordable_price": 525000
}
```

---

## Events API

Events represent open houses and lead capture opportunities.

### Event Endpoints

| Endpoint                             | Method | Description                                         |
| ------------------------------------ | ------ | --------------------------------------------------- |
| `/api/v1/events/`                    | GET    | List all events                                     |
| `/api/v1/events/`                    | POST   | Create event                                        |
| `/api/v1/events/{id}/`               | GET    | Get event details                                   |
| `/api/v1/events/perpetual/`          | GET    | Get user's perpetual event (always-on lead capture) |
| `/api/v1/events/{id}/qr_code/`       | GET    | Get QR code for kiosk                               |
| `/api/v1/events/{id}/seller_report/` | GET    | Generate seller report PDF                          |

### Get Perpetual Event

Each user has one "perpetual" event - an always-on lead capture event that never expires. This endpoint retrieves it (or auto-creates if missing).

```http
GET /api/v1/events/perpetual/
Authorization: Bearer <access_token>
```

**Response:**

```json
{
	"id": "event-uuid",
	"event_type": "PERPETUAL",
	"title": "Jane's Lead Capture",
	"active": true,
	"qr_code_uuid": "qr-uuid",
	"magic_link": "https://hearthrate.app/kiosk?event=qr-uuid",
	"created_at": "2024-01-15T10:00:00Z"
}
```

**Use Case:** The "Start Kiosk" button on the dashboard uses this endpoint to open the user's perpetual event in a new tab.

---

## Leads API

Leads represent potential buyers from events.

### Lead Endpoints

| Endpoint                      | Method | Description                    |
| ----------------------------- | ------ | ------------------------------ |
| `/api/v1/leads/`              | GET    | List all leads                 |
| `/api/v1/leads/`              | POST   | Create lead (kiosk submission) |
| `/api/v1/leads/{id}/`         | GET    | Get lead details               |
| `/api/v1/leads/{id}/promote/` | POST   | Promote to client              |
| `/api/v1/leads/sync_offline/` | POST   | Sync offline leads             |

### Lead Scoring

Leads are automatically scored using a dual-bucket system:

**Deal Score** (0-100): Likelihood to make offer on this property
**Lead Score** (0-100): Likelihood to become a client

---

## Dashboard API

| Endpoint                   | Method | Description              |
| -------------------------- | ------ | ------------------------ |
| `/api/v1/dashboard/stats/` | GET    | Get dashboard statistics |

**Response:**

```json
{
	"active_leads": { "value": 42, "trend": "+15%" },
	"event_signins": { "value": 156, "trend": "+8%" },
	"offers_generated": { "value": 12, "trend": "+3" },
	"recent_activity": [
		{
			"id": "lead-uuid",
			"first_name": "John",
			"last_name": "Doe",
			"status_deal": "HOT",
			"created_at": "2024-03-15T10:00:00Z"
		}
	]
}
```

---

## Error Responses

All errors follow a consistent format:

```json
{
	"error": "Human-readable error message",
	"code": "error_code",
	"details": {}
}
```

---

## Rate Limiting

-   Anonymous: 500 requests/hour
-   Authenticated: 2000 requests/hour
-   Login attempts: 15 requests/minute

---

## Webhooks

Configure webhooks to receive real-time notifications:

| Event                | Description              |
| -------------------- | ------------------------ |
| `lead.created`       | New lead submitted       |
| `lead.promoted`      | Lead converted to client |
| `tour.completed`     | Tour marked complete     |
| `feedback.submitted` | New feedback received    |

See `/api/v1/webhooks/` for webhook management.
