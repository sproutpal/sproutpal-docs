# Encyclopedia data model

- **Status:** Draft
- **Related:** [concept](../01-product/concept.md), [ADR-0001](../03-architecture/decisions/0001-knowledge-base-mongodb.md), [ADR-0002](../03-architecture/decisions/0002-content-formats.md)

This spec defines the encyclopedia entries of the knowledge base: which kinds of entries exist, their fields, and how they refer to each other. Entries are JSON documents in MongoDB (ADR-0001); text fields are plain text and relationships are reference fields (ADR-0002).

The first version covers fruit trees only.

## Entry types

Each entry type has its **own MongoDB collection**. Entries are not embedded in each other: a variety is a separate document that refers to its species, so every entry has its own version history, change log and page.

| Type | Collection | Example | Status |
| --- | --- | --- | --- |
| Species | `species` | Apple, Pear, Cherry | **Draft below** |
| Variety | `varieties` | 'Jonagold', 'Bosc kobak' | **Draft below** |
| Rootstock | `rootstocks` | M9, MM106, Quince A | **Draft below** |
| Pest / disease | `diseases` | Apple scab, Codling moth | **Draft below** |

## Common conventions

These apply to every entry type.

### Identifiers

- Every entry has a stable identifier of the form `<type>:<slug>`, for example `species:apple`, `rootstock:m9`, `disease:apple-scab`, `pest:codling-moth`. Course topics use `topic:<slug>`.
- **Varieties include their species** in the slug, because varieties of different species can share a name: `variety:<species>-<variety>`, for example `variety:apple-jonagold`, `variety:pear-bosc-kobak`.
- The slug is lowercase ASCII, words separated by `-`, without Hungarian accents.
- The identifier is stored as the MongoDB **`_id`** of the document. MongoDB keeps `_id` unique and indexed, so no separate `id` field is needed.
- An identifier never changes, even if the entry is renamed.

### Languages

The **data model is in English**: field names, identifiers and enum values. **Content texts** (names, descriptions and other text fields) are stored per language.

A translatable text is a **localized text** object, keyed by language code:

```json
{ "hu": "Alma", "en": "Apple" }
```

- `hu` is **required**: the first version is Hungarian, and the sources are Hungarian.
- `en` and other languages are optional and are added when the website gets that language. Adding a language does not change the model.

**Enum values** (for example `high`, `susceptible`) are stored in English only, as keys. Their display texts come from the **frontend's translation files** (one file per language, for example with i18next), together with the other UI texts such as field labels. They are not stored in the entries.

### References

A reference to another entry or a course topic is an object:

```json
{ "id": "rootstock:m9", "name": { "en": "M9", "hu": "M9" } }
```

- `id`: the target's identifier (its `_id`).
- `name`: the target's display name as a localized text, stored so the reference can be shown even if the target is missing. It is updated when the target is renamed.

### Common fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `_id` | string | yes | Stable identifier, see above. |
| `name` | localized text | yes | Display name. |
| `synonyms` | localized text[] | no | Other names the entry is known by, for search. |
| `relatedTopics` | reference[] | no | Course topics that explain related fundamentals. |
| `version` | integer | yes | Increases by 1 with every change. |
| `changeNote` | string | yes | What changed in this version. |
| `updatedAt` | date | yes | Time of the last change. |

## Species (`species`)

> **Draft for discussion.** Field list to be confirmed.

A species holds only what is truly common to all its varieties: general requirements such as light, soil and water, and when to plant.

What depends on something else is stored there, and **every variety has its own values filled in** (no defaults inherited from the species):

- **Variety:** frost hardiness, crown forms, vigor, self-fertility, harvest time.
- **Rootstock:** tree size and planting distance (to be defined in the rootstock section).

Lists from other entries (for example the harvest seasons of a species' varieties, the rootstocks used for it, or the pests and diseases affecting it) are produced by queries, not stored on the species. The rootstock and the pest or disease store which species they belong to.

### Fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `scientificName` | string | yes | Latin name, for example `Malus domestica`. Not translated. |
| `family` | string | no | Plant family, for example `Rosaceae`. |
| `light` | enum | yes | `full-sun` / `partial-shade` / `shade`. |
| `soil` | object | no | Soil requirements, see below. |
| `water` | enum | no | Water need: `low` / `medium` / `high`. |
| `lifespan` | string | no | Typical productive lifespan, for example `20–30 years`. |
| `plantingSeasons` | enum[] | no | When to plant: `autumn` / `spring`. |
| `description` | localized text | no | Short plain-text description. |

### Soil

| Field | Type | Description |
| --- | --- | --- |
| `types` | enum[] | Suitable soil types: `sandy` / `loam` / `clay` / `chalky`. |
| `ph` | object | Suitable pH range, `{ "min": 6.0, "max": 7.0 }`. |
| `drainage` | enum | Need for drainage: `low` / `medium` / `high`. |

### Example

```json
{
  "_id": "species:apple",
  "name": { "hu": "Alma", "en": "Apple" },
  "scientificName": "Malus domestica",
  "family": "Rosaceae",
  "light": "full-sun",
  "soil": { "types": ["loam"], "ph": { "min": 6.0, "max": 7.0 }, "drainage": "high" },
  "water": "medium",
  "plantingSeasons": ["autumn", "spring"],
  "version": 1,
  "changeNote": "First version",
  "updatedAt": "2026-11-18T00:00:00Z"
}
```

> The example values are placeholders for illustrating the structure, not verified data.

## Variety (`variety`)

> **Draft for discussion.** Field list to be confirmed.

### Fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `species` | reference | yes | The species, for example `species:apple`. Must match the species part of the identifier. |
| `origin` | localized text | no | Where and by whom it was bred, and when. |
| `ripening` | object | yes | Picking and eating ripeness, see below. |
| `usage` | enum[] | no | `fresh` / `cooking` / `juice` / `drying`. |
| `fruit` | object | no | Size, shape and color of the fruit. |
| `taste` | object | no | Structured taste profile, see below. |
| `storage` | localized text | no | How long and how the fruit keeps. |
| `tree` | object | no | Vigor, growth habit and bearing, see below. |
| `frostHardiness` | enum | yes | `low` / `medium` / `high`. |
| `pollination` | object | yes | Self-fertility and pollinators, see below. |
| `rootstocks` | reference[] | no | Suitable rootstocks. |
| `crownForms` | enum[] | yes | Suitable crown forms, see below. |
| `diseaseResistance` | object[] | no | Resistance per disease, see below. |
| `description` | localized text | no | Short plain-text description. |

### Ripening

For apples and pears, picking ripeness and eating ripeness differ: a winter apple is picked in October but is best to eat in December or January.

| Field | Type | Description |
| --- | --- | --- |
| `season` | enum | `summer` / `autumn` / `winter`. |
| `picking` | period | When the fruit is picked. |
| `eating` | period | When the fruit is best to eat. Omitted if it is the same as `picking`. |

A **period** is `{ "from": ..., "to": ... }`, each value a month and a part of the month, for example `09-mid` (`early` / `mid` / `late`).

### Taste

| Field | Type | Description |
| --- | --- | --- |
| `sweetness` | enum | `low` / `medium` / `high`. |
| `acidity` | enum | `low` / `medium` / `high`. |
| `aroma` | string[] | Terms from a fixed list, for example `spicy`, `honey`, `wine-like`. |
| `texture` | string[] | Terms from a fixed list, for example `juicy`, `crisp`, `mealy`. |

### Tree

| Field | Type | Description |
| --- | --- | --- |
| `vigor` | enum | `weak` / `medium` / `strong`. |
| `habit` | enum | Growth habit, for example `upright` / `spreading` / `weeping`. |
| `bearingStart` | string | When the tree starts bearing, for example `early`, or a number of years. |
| `yield` | enum | `low` / `medium` / `high`. |
| `alternateBearing` | boolean | Whether it tends to bear heavily only every second year. |

### Crown forms

`crownForms` lists the crown forms the variety suits, as enum values, for example `spindle` (orsó), `open-center` (katlan), `central-leader` (sudaras), `palmette`. The crown forms themselves are explained in the course; the variety page links to the course topic through `relatedTopics`.

### Pollination

Flowering time is not stored; pollination is described by the list of good pollinator varieties instead.

| Field | Type | Description |
| --- | --- | --- |
| `selfFertile` | enum | `yes` / `partly` / `no`. |
| `group` | string | Pollination or flowering group, as used for the species. |
| `pollinators` | reference[] | Good pollinator varieties. |
| `poorPollinator` | boolean | True for triploid varieties that do not pollinate others well. |

### Disease resistance

Each item:

| Field | Type | Description |
| --- | --- | --- |
| `disease` | reference | The disease or pest. |
| `level` | enum | `susceptible` / `moderate` / `resistant`. |

### Example

```json
{
  "_id": "variety:apple-jonagold",
  "name": { "en": "Jonagold", "hu": "Jonagold" },
  "species": { "id": "species:apple", "name": { "en": "Apple", "hu": "Alma" } },
  "ripening": {
    "season": "autumn",
    "picking": { "from": "09-late", "to": "10-early" },
    "eating": { "from": "10-mid", "to": "01-late" }
  },
  "usage": ["fresh", "cooking"],
  "taste": { "sweetness": "high", "acidity": "medium", "aroma": ["spicy"], "texture": ["juicy", "crisp"] },
  "pollination": {
    "selfFertile": "no",
    "poorPollinator": true,
    "pollinators": [{ "id": "variety:apple-idared", "name": { "en": "Idared", "hu": "Idared" } }]
  },
  "rootstocks": [{ "id": "rootstock:m9", "name": { "en": "M9", "hu": "M9" } }],
  "crownForms": ["spindle"],
  "frostHardiness": "medium",
  "diseaseResistance": [
    { "disease": { "id": "disease:apple-scab", "name": { "en": "Apple scab", "hu": "Varasodás" } }, "level": "susceptible" }
  ],
  "version": 1,
  "changeNote": "First version",
  "updatedAt": "2026-11-18T00:00:00Z"
}
```

> The example values are placeholders for illustrating the structure, not verified data.

## Rootstock (`rootstock`)

> **Draft for discussion.** Field list to be confirmed.

The rootstock is the root system a variety is grafted onto. It largely decides how big the tree grows, how far apart trees are planted, how soon they bear fruit and which soils they tolerate.

Rootstocks can also be used across species (for example quince for pears), so the identifier does not include a species: `rootstock:m9`, `rootstock:quince-a`.

### Fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `species` | reference[] | yes | Species the rootstock is used for, for example apple; quince rootstocks for pear. |
| `origin` | localized text | no | Where it was developed, for example East Malling, England. |
| `vigor` | enum | yes | `very-weak` / `weak` / `medium` / `strong` / `very-strong`. |
| `treeSize` | object | yes | Typical mature tree size on this rootstock, see below. |
| `spacing` | object | yes | Planting distance, see below. |
| `bearingStart` | enum | no | How soon trees start bearing: `early` / `medium` / `late`. |
| `support` | enum | yes | Need for a stake or trellis: `none` / `recommended` / `permanent`. |
| `soilTolerance` | object | no | Which soil conditions it tolerates, see below. |
| `frostHardiness` | enum | no | Frost hardiness of the roots: `low` / `medium` / `high`. |
| `lifespan` | string | no | Typical productive lifespan of trees on it, for example `15–20 years`. |
| `suckering` | enum | no | Tendency to produce root suckers: `low` / `medium` / `high`. |
| `crownForms` | enum[] | no | Crown forms it suits (same values as for varieties). |
| `diseaseResistance` | object[] | no | Same structure as for varieties: `{ disease, level }`. |
| `incompatibleVarieties` | reference[] | no | Varieties that cannot be grafted directly onto it (for example some pears on quince, which need an interstem). |
| `description` | localized text | no | Short plain-text description. |

### Tree size

| Field | Type | Description |
| --- | --- | --- |
| `height` | range | Height in meters, `{ "min": 2, "max": 3 }`. |
| `width` | range | Crown width in meters, same format. |
| `percentOfSeedling` | range | Size compared to a tree on a seedling rootstock, in percent, for example `{ "min": 25, "max": 35 }`. |

### Spacing

| Field | Type | Description |
| --- | --- | --- |
| `inRow` | range | Distance between trees in a row, in meters. |
| `betweenRows` | range | Distance between rows, in meters. |

### Soil tolerance

| Field | Type | Description |
| --- | --- | --- |
| `types` | enum[] | Soil types it tolerates: `sandy` / `loam` / `clay` / `chalky`. |
| `drought` | enum | Drought tolerance: `low` / `medium` / `high`. |
| `wetness` | enum | Tolerance of wet, poorly drained soil: `low` / `medium` / `high`. |

### Example

```json
{
  "_id": "rootstock:m9",
  "name": { "hu": "M9", "en": "M9" },
  "species": [{ "id": "species:apple", "name": { "hu": "Alma", "en": "Apple" } }],
  "origin": { "hu": "East Malling, Anglia", "en": "East Malling, England" },
  "vigor": "weak",
  "treeSize": {
    "height": { "min": 2, "max": 3 },
    "width": { "min": 1.5, "max": 2.5 },
    "percentOfSeedling": { "min": 25, "max": 35 }
  },
  "spacing": { "inRow": { "min": 0.8, "max": 1.5 }, "betweenRows": { "min": 3, "max": 4 } },
  "bearingStart": "early",
  "support": "permanent",
  "soilTolerance": { "types": ["loam"], "drought": "low", "wetness": "low" },
  "crownForms": ["spindle"],
  "version": 1,
  "changeNote": "First version",
  "updatedAt": "2026-11-18T00:00:00Z"
}
```

> The example values are placeholders for illustrating the structure, not verified data.

## Pest and disease (`disease`)

> **Draft for discussion.** Field list to be confirmed.

Pests and diseases share one collection, because they are looked up and linked the same way. The `category` field tells them apart, and the identifier prefix follows it: `pest:codling-moth`, `disease:apple-scab`.

The longer text fields are fixed for every entry and contain plain text (ADR-0002). Relationships are reference fields.

### Fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `category` | enum | yes | `pest` / `fungal` / `bacterial` / `viral` / `physiological`. |
| `scientificName` | string | no | Latin name of the pathogen or pest, for example `Venturia inaequalis`. Not translated. |
| `affectedSpecies` | reference[] | yes | Species it affects. |
| `activePeriod` | period[] | no | When it is active or when symptoms appear, same period format as for ripening. |
| `severity` | enum | no | Typical damage: `low` / `medium` / `high`. |
| `symptoms` | localized text | yes | How to recognize it. |
| `cause` | localized text | no | What causes it and how it spreads or develops. |
| `prevention` | localized text | yes | How to prevent it. |
| `treatment` | localized text | no | What to do if it appears. |
| `organicOptions` | boolean | no | Whether organic methods are available. |
| `description` | localized text | no | Short plain-text summary. |

Which varieties and rootstocks are resistant or susceptible is not stored here: it is in their `diseaseResistance` field, and the disease page lists them by query.

### Example

```json
{
  "_id": "disease:apple-scab",
  "name": { "hu": "Varasodás", "en": "Apple scab" },
  "category": "fungal",
  "scientificName": "Venturia inaequalis",
  "affectedSpecies": [{ "id": "species:apple", "name": { "hu": "Alma", "en": "Apple" } }],
  "activePeriod": [{ "from": "04-early", "to": "07-late" }],
  "severity": "high",
  "symptoms": { "hu": "…" },
  "prevention": { "hu": "…" },
  "organicOptions": true,
  "relatedTopics": [{ "id": "topic:plant-protection-basics", "name": { "hu": "A növényvédelem alapjai" } }],
  "version": 1,
  "changeNote": "First version",
  "updatedAt": "2026-11-18T00:00:00Z"
}
```

> The example values are placeholders for illustrating the structure, not verified data.

## Lists and indexes

Lists in the other direction are produced by queries, not stored (ADR-0001). For example, the varieties of a species:

```js
db.varieties.find({ "species.id": "species:apple" })
```

Fields that are queried often get an index, so these queries stay fast:

| Collection | Indexed fields | Used for |
| --- | --- | --- |
| all | `_id` (automatic) | Looking up an entry by identifier |
| `varieties` | `species.id` | Varieties of a species |
| `varieties` | `pollination.pollinators.id` | Varieties a given variety pollinates |
| `varieties` | `rootstocks.id` | Varieties suitable for a rootstock |
| `varieties` | `diseaseResistance.disease.id` | Varieties affected by or resistant to a disease |
| `rootstocks` | `species.id` | Rootstocks used for a species |
| `rootstocks` | `diseaseResistance.disease.id` | Rootstocks resistant to or affected by a disease |
| `diseases` | `affectedSpecies.id` | Pests and diseases affecting a species |

## Open questions

- The final list of crown form values, aligned with the crown form course topic.
- Does each species need its own variety fields (for example S-allele groups for cherries), or is the common list enough?
- Should the fixed lists (aroma, texture, habit) be defined in this spec or in a separate vocabulary document?
- Should the `changeNote` also be a localized text, since it is shown to users?
