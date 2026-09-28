# LOCUS Data Schema — V2

LOCUS must keep verified cultural facts, geographic provenance, rights information, synthetic test data, and AI-generated interpretation explicitly separable.

The schema below is the authoritative MVP domain model direction. Implementation types should remain aligned with it.

---

## 1. Shared Provenance Types

```ts
type SourceType =
  | "wikidata"
  | "europeana"
  | "openstreetmap"
  | "museum_dataset"
  | "library_archive"
  | "geocoding_provider"
  | "device_gps"
  | "manual_verified"
  | "other";

interface SourceReference {
  sourceId: string;
  sourceType: SourceType;

  sourceUrl?: string | null;
  externalId?: string | null;

  retrievedAt?: string | null; // ISO-8601
  confidence?: number | null;  // 0..1 when meaningful

  notes?: string | null;
}
```

Rules:

- `sourceId` must be stable within LOCUS.
- `confidence` must not be invented when the upstream source does not provide or justify one.
- Missing provenance must be represented explicitly rather than guessed.

---

## 2. Geographic Provenance

```ts
type CoordinateSourceType =
  | "device_gps"
  | "geocoding_provider"
  | "verified_dataset"
  | "manual_verified";

type CoordinatePrecision =
  | "exact"
  | "building"
  | "street"
  | "neighbourhood"
  | "area"
  | "approximate"
  | "unknown";

interface GeoLocation {
  lat: number;
  lng: number;

  city?: string | null;
  country?: string | null;

  coordinateSource: CoordinateSourceType;
  coordinateSourceId?: string | null;

  precision: CoordinatePrecision;
  accuracyMeters?: number | null;

  verified: boolean;
}
```

Rules:

- LLM output must never be used as the authoritative source of latitude or longitude.
- Coordinates must originate from device GPS, an approved geocoder, a verified dataset, or a manually verified entry.
- If precision is unknown, use `"unknown"`.
- Approximate cultural locations must not be represented as exact locations.

---

## 3. Rights Metadata

```ts
type RightsStatus =
  | "public_domain"
  | "licensed"
  | "copyrighted"
  | "unknown";

interface RightsMetadata {
  status: RightsStatus;

  license?: string | null;
  sourceUrl?: string | null;

  attributionRequired?: boolean | null;
  attributionText?: string | null;

  verified: boolean;
  verifiedAt?: string | null; // ISO-8601
}
```

Rules:

- Public online availability does not imply reuse permission.
- Unknown rights remain `"unknown"`; do not infer a license.
- Production media with unknown reuse rights must not be treated as cleared.
- Required attribution must remain attached to the asset.

---

## 4. CulturalObject

```ts
type CulturalObjectType =
  | "painting"
  | "literature"
  | "history"
  | "film"
  | "architecture";

type GeoRelation =
  | "depicted"
  | "set_here"
  | "written_here"
  | "inspired_by"
  | "creator_lived_here"
  | "historical_event"
  | "fictionalized"
  | "approximate";

interface CulturalObject {
  id: string;

  type: CulturalObjectType;

  title: string;
  creator?: string | null;
  year?: number | string | null;

  location: GeoLocation;

  geoRelation: GeoRelation;
  geoConfidence: number | null;

  culturalSignificance: number | null;
  visualPotential: number | null;

  sources: SourceReference[];

  rights: RightsMetadata;

  /**
   * true only for fixtures, tests, demos, or explicitly synthetic
   * local mock datasets.
   */
  isSynthetic: boolean;

  scene?: SceneDefinition | null;
}
```

### CulturalObject rules

- Verified facts must remain source-backed.
- `geoConfidence`, `culturalSignificance`, and `visualPotential` must document their origin when they become production ranking features.
- Missing values remain `null`; do not fabricate scores.
- Synthetic objects must use `isSynthetic: true`.
- Synthetic objects must not be silently mixed into verified production datasets.

---

## 5. Generated Content Provenance

```ts
interface GenerationProvenance {
  generated: true;

  generator: string;
  generatorVersion?: string | null;

  generatedAt: string; // ISO-8601

  basedOnSourceIds: string[];
  basedOnCulturalObjectIds: string[];

  seed?: number | string | null;
}
```

Rules:

- Generated content must always be marked `generated: true`.
- Generated content must identify the verified source IDs or cultural object IDs used as input.
- Generated content must never overwrite verified factual fields.
- If a deterministic or seeded generation path is used, preserve the seed when available.

---

## 6. Scene Types

```ts
interface SceneMedia {
  id: string;

  kind: "image" | "video" | "audio";

  url: string;

  generated: boolean;
  generation?: GenerationProvenance | null;

  rights: RightsMetadata;

  sourceIds: string[];
}

interface SceneDefinition {
  id: string;

  culturalObjectId: string;

  title: string;

  observerMode: true;

  era?: string | null;
  weather?: string | null;
  timeOfDay?: string | null;

  generated: boolean;
  generation?: GenerationProvenance | null;

  sourceIds: string[];

  media?: SceneMedia[];
}
```

### Scene rules

- The product end user remains **The Observer**.
- Scene interpretation may be generated, but source facts remain separate.
- A generated scene must not imply that invented visual details are verified historical facts.
- Source media and generated media must remain distinguishable in metadata.

---

## 7. CulturalMatch

```ts
interface CulturalMatch {
  objectId: string;

  distanceMeters: number;

  proximityScore: number;
  culturalScore: number;
  geoConfidenceScore: number;
  visualScore: number;
  personalRelevanceScore?: number | null;

  finalScore: number;

  rankingVersion: string;
}
```

### Ranking rules

The initial conceptual weighting remains:

- 30% proximity
- 25% cultural significance
- 20% geographic confidence
- 15% visual potential
- 10% personal relevance

Implementation must not assume these weights are permanent product policy.

Distance must be computed deterministically using a documented geodesic method.

For equal `finalScore`, use the default stable tie-break order:

1. higher geographic confidence
2. higher cultural significance
3. shorter distance
4. stable `objectId`

If a ranking experiment uses randomness, it must use an explicit seed.

---

## 8. Example Synthetic Fixture

The following is illustrative test/demo data only.

```ts
const exampleSyntheticObject: CulturalObject = {
  id: "fixture_london_001",
  type: "painting",
  title: "Example Cultural Object",
  creator: null,
  year: null,

  location: {
    lat: 51.5007,
    lng: -0.1246,
    city: "London",
    country: "UK",
    coordinateSource: "manual_verified",
    coordinateSourceId: "fixture-only",
    precision: "approximate",
    accuracyMeters: null,
    verified: false
  },

  geoRelation: "approximate",
  geoConfidence: null,
  culturalSignificance: null,
  visualPotential: null,

  sources: [
    {
      sourceId: "fixture:example",
      sourceType: "other",
      sourceUrl: null,
      externalId: null,
      retrievedAt: null,
      confidence: null,
      notes: "Synthetic fixture for tests and demos only."
    }
  ],

  rights: {
    status: "unknown",
    license: null,
    sourceUrl: null,
    attributionRequired: null,
    attributionText: null,
    verified: false,
    verifiedAt: null
  },

  isSynthetic: true,
  scene: null
};
```

This fixture must never be treated as verified cultural data.

---

## 9. Data Integrity Rules

- Never fabricate coordinates.
- Never fabricate cultural provenance.
- Never convert generated interpretation into verified source facts.
- Never overwrite verified source data with generated content.
- Keep source facts and generated scene data structurally separate.
- Represent missing certainty explicitly with `null`, `unknown`, or another documented state.
- Preserve provenance for externally sourced facts.
- Preserve coordinate source and precision.
- Preserve rights status and attribution requirements.
- Mark fixtures and demo data with `isSynthetic: true`.
- Mark generated content with `generated: true`.
- Do not use an LLM as the authoritative source of geographic coordinates.
