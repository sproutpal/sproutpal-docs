# Encyclopedia data model

- **Status:** Draft
- **Related:** [concept](../01-product/concept.md), [ADR-0001](../03-architecture/decisions/0001-knowledge-base-mongodb.md), [ADR-0002](../03-architecture/decisions/0002-content-formats.md)

This spec defines the encyclopedia entries of the knowledge base: which kinds of entries exist, their fields, and how they refer to each other. Entries are JSON documents in MongoDB (ADR-0001); text fields are plain text and relationships are reference fields (ADR-0002).

The first version covers fruit trees only.

## Entry types

Each entry type has its **own MongoDB collection**. Entries are not embedded in each other: a variety is a separate document that refers to its species, so every entry has its own version history, change log and page.

| Type | Collection | Example | Status |
| --- | --- | --- | --- |
| Species | `species` | Apple, Pear, Cherry | To be defined |
| Variety | `varieties` | 'Jonagold', 'Bosc kobak' | **Draft below** |
| Rootstock | `rootstocks` | M9, MM106, Quince A | To be defined |
| Pest / disease | `diseases` | Apple scab, Codling moth | To be defined |

## Common conventions

These apply to every entry type.

### Identifiers

- Every entry has a stable identifier of the form `<type>:<slug>`, for example `species:apple`, `rootstock:m9`, `disease:apple-scab`.
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

## Variety (`variety`)

> **Draft for discussion.** Field list to be confirmed.

### Fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `species` | reference | yes | The species, for example `species:apple`. Must match the species part of the identifier. |
| `origin` | localized text | no | Where and by whom it was bred, and when. |
| `ripening` | object | yes | Ripening time, see below. |
| `fruit` | object | no | Size, shape and color of the fruit. |
| `taste` | object | no | Structured taste profile, see below. |
| `storage` | localized text | no | How long and how the fruit keeps. |
| `tree` | object | no | Vigor, growth habit and bearing, see below. |
| `frostHardiness` | enum | no | `low` / `medium` / `high`. |
| `pollination` | object | yes | Self-fertility and pollinators, see below. |
| `rootstocks` | reference[] | no | Suitable rootstocks. |
| `diseaseResistance` | object[] | no | Resistance per disease, see below. |
| `description` | localized text | no | Short plain-text description. |

### Ripening

| Field | Type | Description |
| --- | --- | --- |
| `season` | enum | `summer` / `autumn` / `winter`. |
| `from` | string | Start of harvest, as month and part of month, for example `09-mid`. |
| `to` | string | End of harvest, same format. |

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

### Pollination

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
  "ripening": { "season": "autumn", "from": "09-late", "to": "10-early" },
  "taste": { "sweetness": "high", "acidity": "medium", "aroma": ["spicy"], "texture": ["juicy", "crisp"] },
  "pollination": {
    "selfFertile": "no",
    "poorPollinator": true,
    "pollinators": [{ "id": "variety:apple-idared", "name": { "en": "Idared", "hu": "Idared" } }]
  },
  "rootstocks": [{ "id": "rootstock:m9", "name": { "en": "M9", "hu": "M9" } }],
  "diseaseResistance": [
    { "disease": { "id": "disease:apple-scab", "name": { "en": "Apple scab", "hu": "Varasodás" } }, "level": "susceptible" }
  ],
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

## Open questions

- Does each species need its own variety fields (for example S-allele groups for cherries), or is the common list enough?
- Should the fixed lists (aroma, texture, habit) be defined in this spec or in a separate vocabulary document?
- Should the `changeNote` also be a localized text, since it is shown to users?
