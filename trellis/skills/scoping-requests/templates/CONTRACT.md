# Contract — <item-slug>

## Handshake

- **name**: <proposed identifier>
- **consumer**: `<consumer node · item>`
- **provider**: `<provider node>` (coordinator-classified)
- **inputs**: <arguments / props / params>
- **output**: <shape the consumer requires — single source of truth for shape>
- **placement**: <intended concern/layer + composition notes>
- **status**: proposed | satisfied | countered | create

## Source

- code: <resolved host path, set by Self-Document once built — empty while status is `proposed`>
- node: `<concern> · <layer>`

## Provides

- name: <identifier>
- inputs: <arguments / props / params>
- output: <shape actually delivered — must match the ratified Handshake output once satisfied>
- placement: <who composes this>

## Consumes

| name | from node | shape | status |
|------|-----------|-------|--------|

## History

| date | change | ratified with |
|------|--------|----------------|
| <YYYY-MM-DD> | initial contract | — |
