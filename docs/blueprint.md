# Group Message Broadcaster — Bot specification

**Archetype:** workflow

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that allows an owner to manage destination groups and broadcast messages to all configured groups with delivery confirmation feedback.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group administrators

## Success criteria

- Messages sent to bot are forwarded to all configured groups with per-group success/failure reports

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu
- **/add** (command, actor: user, command: /add) — Add a new destination group by forwarding a message from it or providing an invite link
- **/remove** (command, actor: user, command: /remove) — Remove a destination group by selecting from list or providing ID/name
- **/list** (command, actor: user, command: /list) — View current list of destination groups
- **/status** (command, actor: user, command: /status) — Show recent delivery log entries
- **Message submission** (message, actor: user, command: /any message) — Send any text/media message to be broadcast to all destination groups

## Flows

### Add destination
_Trigger:_ /add

1. Owner forwards message from target group or provides invite link
2. Bot resolves and stores group ID with optional friendly name

_Data touched:_ Destination groups

### Broadcast message
_Trigger:_ User sends message to bot

1. Bot forwards message to all destination groups
2. Collects per-group delivery status
3. Sends summary report to owner

_Data touched:_ Messages, Delivery log

### Remove destination
_Trigger:_ /remove

1. Owner selects group from list or provides ID/name
2. Bot removes group from destination list

_Data touched:_ Destination groups

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Owner** _(retention: persistent)_ — Single administrator user
  - fields: telegram_user_id
- **Destination groups** _(retention: persistent)_ — Telegram groups configured for broadcasting
  - fields: group_id, friendly_name
- **Messages** _(retention: session)_ — Content sent by owner for broadcasting
  - fields: message_id, content_type, timestamp
- **Delivery log** _(retention: persistent)_ — Recent broadcast delivery status
  - fields: timestamp, message_id, destination_group, status

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /add
- /remove
- /list
- /status

## Notifications

- Per-group delivery success/failure reports in owner chat
- Recent delivery log status summaries

## Permissions & privacy

- Only owner can manage destination groups
- Messages are forwarded as bot account only
- Group IDs are stored securely

## Edge cases

- Invalid group ID resolution
- Unsupported message types in destinations
- Owner trying to add non-group chats

## Required tests

- End-to-end broadcast flow with message forwarding and delivery confirmation
- Group management commands with validation
- Error handling for failed deliveries

## Assumptions

- Owner will provide valid group IDs through message forwarding or invite links
- Bot has 'post messages' permission in all destination groups
- Delivery status reporting works across all Telegram message types
