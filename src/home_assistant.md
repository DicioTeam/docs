# Home Assistant Skill

Control and query Home Assistant entities with voice commands.

## Setup

You need a running Home Assistant instance and a Long-Lived Access Token.

### Getting an access token

1. Open your Home Assistant web UI
2. Click your profile (bottom left)
3. Scroll to **Long-Lived Access Tokens** → **Create Token**
4. Copy the token — it's only shown once

### Configuring Dicio

In Dicio, go to **Settings → Skills → Home Assistant** and enter:

- **Base URL**: your HA address, e.g. `http://192.168.1.100:8123`
- **Access Token**: the token you just created

### Adding entity mappings

Entity mappings link the names you speak to HA entity IDs. Tap **Add Mapping** and fill in:

- **Friendly Name**: what you'll say (e.g. "kitchen light")
- **Entity ID**: the HA entity ID (e.g. `light.kitchen`)

You can find entity IDs in HA under **Developer Tools → States**.

You can also tap **Pick** to browse entities directly from your HA instance.

## Voice commands

### Control (on/off/toggle)

> "Turn kitchen light on"
> "Switch the bedroom fan off"
> "Toggle living room light"

Works with lights, switches, fans, covers (open/close), locks (lock/unlock), and media players.

### Status

> "What is the garage door status?"
> "Check front door"

### Person location

> "Where is the person Mark?"
> "What is Sarah's location?"

### Media source selection

> "Set kitchen radio to BBC Radio 2"
> "Tune kitchen radio to Heart"

The skill fuzzy-matches source names against the entity's available source list.

## Supported entity types

| Domain | On action | Off action |
|--------|-----------|------------|
| `light.*` | turn_on | turn_off |
| `switch.*` | turn_on | turn_off |
| `fan.*` | turn_on | turn_off |
| `cover.*` | open_cover | close_cover |
| `lock.*` | unlock | lock |
| `media_player.*` | turn_on / select_source | turn_off |
| `person.*` | — (status only) | — |

## Troubleshooting

- **Connection failed**: check the URL is reachable and HA is running
- **Auth failed**: create a fresh token and re-enter it
- **Entity not mapped**: add a mapping for the name you're saying
- **Entity not found**: verify the entity ID exists in HA Developer Tools
