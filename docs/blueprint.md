# Notification Subscription Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for managing notification subscriptions and delivering categorized alerts to private chats. Users can subscribe/unsubscribe, configure notification categories, and receive messages with optional action buttons. Admin receives subscription events and delivery failures via private messages.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual Telegram users seeking real-time notifications

## Success criteria

- Users can manage subscriptions via buttons/commands
- Notifications are delivered to private chats with configured categories
- Admin receives subscription events and delivery failures

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with subscription status and options
- **Subscribe** (button, actor: user, callback: subscribe:start) — Enable default notification categories
- **Unsubscribe** (button, actor: user, callback: unsubscribe:start) — Disable all notifications
- **Manage Categories** (button, actor: user, callback: categories:manage) — Configure specific notification categories
- **/help** (command, actor: user, command: /help) — Show command list and support information

## Flows

### Onboarding
_Trigger:_ /start

1. Display greeting and subscription status
2. Show category explanation
3. Offer subscription options

_Data touched:_ User

### Subscription Management
_Trigger:_ subscribe:start

1. Confirm subscription
2. Update Subscription entity
3. Send confirmation message

_Data touched:_ Subscription

### Notification Delivery
_Trigger:_ system:notification

1. Check user's active categories
2. Format message with title/body/timestamp
3. Send to private chat

_Data touched:_ Notification

### Admin Alerts
_Trigger:_ system:admin_event

1. Format subscription event
2. Send to admin chat

_Data touched:_ Subscription

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with subscription preferences
  - fields: chat_id, language_preference, opt_in_timestamp
- **Subscription** _(retention: persistent)_ — User notification preferences and delivery status
  - fields: enabled_categories, delivery_settings, failure_count
- **Notification** _(retention: session)_ — Categorized message payload for delivery
  - fields: category, title, body, action_url

## Integrations

- **Telegram** (required) — Private message delivery and user interaction
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure admin notification channel
- View subscription events
- Manage notification categories

## Notifications

- Admin notifications for new subscriptions/unsubscriptions
- User notifications with optional action buttons

## Permissions & privacy

- User data stored securely with opt-in consent
- Unsubscribe permanently stops notifications but retains preferences
- Admin chat receives event notifications

## Edge cases

- User not in subscription database
- Message delivery failures to private chats
- Invalid category configuration attempts

## Required tests

- Verify subscription toggle affects notifications
- Confirm admin receives event alerts
- Test message formatting with action buttons

## Assumptions

- Default categories include Alerts, News, Reminders, Transactions
- Admin notifications sent to owner's private chat by default
- Russian is default language for user messages
