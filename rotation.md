# Rotation YAML format

Primate loads rotation files named `rotation_<specId>.yaml` or a local `rotation.yaml` using the same format.
The file is a YAML map of **APL lists** (action priority lists). The `main` list is the required entry point.
Each list is a sequence of **steps**. Steps can:

- cast a spell (`spell_id`),
- return early (`return: true`), or
- call another APL list (`call_apl` or `run_apl`).

See [`rotation.yaml`](../rotation.yaml) for a working example.

## Top-level structure

```yaml
sanitychecks:
  - name: Rotation Stop
    return: true
    conditions:
      - type: disabled

main:
  - name: Sanity Checks
    run_apl: sanitychecks
  - name: My Spell
    spell_id: 12345
```

### APL calls

- `call_apl`: expands the named list in place and continues evaluating the current list.
- `run_apl`: expands the named list and **stops** evaluation of the current list afterwards.

## Step fields

| Field | Type | Notes |
| --- | --- | --- |
| `name` | string | Friendly name for logging. |
| `spell_id` | number | Spell to cast. Leave as `0` or omit to skip casting. |
| `return` | boolean | If `true`, stops rotation evaluation (used for sanity checks). |
| `hotkey` | number | Override hotkey index for this step. |
| `modifier` | number | Override modifier index for this step. |
| `rangecheck` | string | One of `target`, `mouseover`, `mobcount8y`, `mobcount40y`, `none`. Defaults to `target`. |
| `castingcheck` | string/number | `none`, `any`, or a spell ID (string or number). |
| `castremains` | number/`none` | Optional cast remains threshold. |
| `channelremains` | number/`none` | Optional channel remains threshold. |
| `conditions` | list/map | Conditions that must all be true for the step. |
| `call_apl` | string | Calls another list (continues current list). |
| `run_apl` | string | Calls another list (stops current list after call). |

## Conditions

Conditions are ANDed together. You can use a list of condition objects **or** a map keyed by
condition type (the map form can be shorter for repeated types).

```yaml
conditions:
  - type: player_power
    op: "<"
    value: 70
  - type: spell_cd
    spell_id: 123456
    op: "=="
    value: 0
```

```yaml
conditions:
  aura_count:
    - spell_id: 456789
      op: ">="
      value: 5
```

### Condition fields

| Field | Type | Notes |
| --- | --- | --- |
| `type` | string | Condition type name (see below). |
| `op` | string | One of `<`, `<=`, `>`, `>=`, `==`, `!=`. Defaults to `>=`. |
| `value` | number | Comparison value. Defaults to `0`. |
| `spell_id` | number | Required for spell/aura conditions. |

### Supported condition types

**Status & toggles**
- `enabled`, `disabled`, `in_combat`, `cds_enabled`, `aoe_enabled`, `in_melee_range`, `in_ranged_range`
- `spec_id`, `one_button_assistant`, `combat_time`

**Items**
- `trinket1_ready`, `trinket2_ready`, `healthpotion_ready`, `healthpotion_alt_ready`,
  `healthstone_ready`, `rune_ready`

**Player**
- `gcd`, `player_hp`, `player_power`, `player_power_secondary`, `player_cast_id`, `player_cast_remains`

**Target**
- `target_exists`, `target_lootable`, `target_friendly`, `target_enemy`, `target_attackable`,
  `target_dead`, `target_alive`, `target_combat`, `target_being_tanked`
- `target_hp`, `target_cast_id`, `target_cast_remains`, `target_interruptible`, `target_cast_important`

**Mouseover**
- `mouseoveR_exists` (note the capital `R`, matches the current parser)
- `mouseover_lootable`, `mouseover_friendly`, `mouseover_enemy`, `mouseover_attackable`,
  `mouseover_dead`, `mouseover_alive`, `mouseover_combat`, `mouseover_being_tanked`
- `mouseover_cast_id`, `mouseover_cast_remains`, `mouseover_interruptible`, `mouseover_cast_important`

**Mob counts**
- `mob_count_8y`, `mob_count_15y`, `mob_count_40y`

**Spells & auras**
- `spell_cd`, `spell_charges`, `spell_charges_remains`, `spell_range`, `last_cast`, `prev_cast`
- `aura_count`, `aura_count_player`, `aura_count_target`, `aura_cd`, `aura_cd_player`, `aura_cd_target`

> Note: the parser also accepts a `target_status` or `status` type, but the engine primarily uses the
> specific flag types listed above.
