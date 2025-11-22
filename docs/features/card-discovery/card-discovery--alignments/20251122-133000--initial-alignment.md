# Feature Alignment - Card Discovery - Pull TLA Slice

**Date**: 2025-11-22

## Goal
Create a reliable service to pull card data from the Scryfall API, specifically focusing on the "Avatar: The Last Airbender" (TLA) set as a vertical slice. This service is the foundation for the Scryscraper product.

## Priorities (Product Perspective)

1.  **Card Discoverability (TLA)**
    - We need to know what cards exist in the set.
    - Priority: High.
    - Note: Include extras and multilingual cards (set flags to true), but verify payload size.

2.  **Low-Res Thumbnails (Base Cards)**
    - Users need to see the cards.
    - Priority: Medium-High.
    - Focus on TLA cards first.

3.  **Low-Res Thumbnails (Alternative Arts)**
    - Enhances the visual experience.
    - Priority: Medium.

4.  **High-Res Images**
    - Lowest priority.
    - We can likely rely on Scryfall's CDN for high-res images to save bandwidth and storage.

## Technical Constraints & Decisions
- **Language**: TypeScript
- **Package Manager**: pnpm
- **Infrastructure**: Docker (Node.js + PostgreSQL)
- **Data Persistence**: PostgreSQL with volume mount.
- **Rate Limiting**: Strict adherence to Scryfall's 50ms/request and 24h cache policy.

## Success Criteria for MVP Slice
- Service can run locally via Docker.
- Service successfully pulls all card metadata for the TLA set from Scryfall.
- Data is persisted in PostgreSQL.
- Rate limits are respected.
