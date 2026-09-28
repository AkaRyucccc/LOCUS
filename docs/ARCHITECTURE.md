# LOCUS MVP Architecture

## Frontend

- Next.js
- React
- TypeScript
- Mobile-first PWA

## Globe

Preferred MVP direction:

- react-globe.gl / globe.gl
- Three.js underneath

Use for:

- cinematic Earth
- geographic labels
- cultural nodes
- camera fly-to
- rings / location pulse

## City Walk Map

Use MapLibre GL JS.

Use for:

- walk mode
- user position
- nearby memory
- route visualization

## Cultural Data

MVP:

- curated local JSON dataset for London

Later:

- PostgreSQL / PostGIS
- Wikidata
- Europeana
- OpenStreetMap
- museum open datasets

## Matching

Use deterministic code for:

- distance
- radius filtering
- ranking

Do not use an LLM to calculate geographic distance.

Initial score concept:

- 30% proximity
- 25% cultural significance
- 20% geographic confidence
- 15% visual potential
- 10% personal relevance

## AI

MVP AI responsibility:

- convert verified cultural facts into a structured scene specification

MVP must not depend on live video generation.

Use pre-generated media for core scenes.

## Separation of Responsibilities

Verified geographic / cultural data
→ deterministic retrieval
→ deterministic ranking
→ scene composition
→ generated media
→ presentation
