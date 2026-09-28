# LOCUS Data Schema

## CulturalObject

Each cultural record must separate verified source information from generated scene information.

Suggested shape:

```ts
interface CulturalObject {
  id: string;
  type: "painting" | "literature" | "history" | "film" | "architecture";
  title: string;
  creator?: string;
  year?: number | string;

  location: {
    lat: number;
    lng: number;
    city?: string;
    country?: string;
  };

  geoRelation:
    | "depicted"
    | "set_here"
    | "written_here"
    | "inspired_by"
    | "creator_lived_here"
    | "historical_event"
    | "fictionalized"
    | "approximate";

  geoConfidence: number;
  culturalSignificance: number;
  visualPotential: number;

  sources: SourceReference[];

  rights: {
    status: "public_domain" | "copyrighted" | "unknown";
    verified: boolean;
  };

  scene?: SceneDefinition;
}
```

## CulturalMatch

```ts
interface CulturalMatch {
  objectId: string;
  distanceMeters: number;
  proximityScore: number;
  culturalScore: number;
  geoConfidenceScore: number;
  visualScore: number;
  personalRelevanceScore?: number;
  finalScore: number;
}
```

## Data Integrity Rules

- Never fabricate coordinates.
- Never fabricate cultural provenance.
- Keep source facts separate from generated descriptions.
- Generated content must never overwrite source records.
- Missing certainty must be represented explicitly.
