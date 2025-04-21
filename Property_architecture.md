# Property Listing Feed Architecture (Microservices)

## Overview

This document describes the architecture for a property listing feed in a microservices-based system. The system is designed to support paginated property discovery with multimedia content submitted by landlords.

---

## Services

### 1. Property Management Service
- Handles listing creation, updates, and deletion
- Stores metadata (e.g., title, price, location)
- Stores references (keys/paths) to multimedia files

### 2. Media Service
- Handles media upload, validation, storage
- Stores files in S3 or similar object storage
- Generates thumbnails and optional encodings
- Provides URLs:
  - Public CDN-backed URLs for direct access
  - Pre-signed URLs if media is private

### 3. Property Feed (Discovery) Service
- Accepts filter and pagination parameters
- Queries listings from a read-optimized backend (e.g., Elasticsearch)
- Embeds media URLs directly in response
- Supports:
  - Filters: location, price, property type, amenities, etc.
  - Pagination: offset-based or cursor-based
  - Sorting: by date, price, location

### 4. Client Application (Web/Mobile)
- Sends paginated requests to Feed Service
- Renders media using provided URLs

---

## Request Flow

### Client Request:
- Sends request to `PropertyFeedService` with filters and pagination

### Feed Service:
1. Queries listings from search backend (Elasticsearch)
2. Retrieves metadata + stored media URLs (no MediaService call needed if media is public)
3. Returns listings with embedded media URLs

---

## Media URL Strategy

### For Public Access (Preferred for Feed):
- Use deterministic naming: `properties/{property_id}/image.jpg`
- Host files via CDN (e.g., CloudFront)
- Store URLs or object keys at listing creation time
- No need to contact Media Service during feed requests

### For Private Access:
- Use pre-signed URLs with expiration
- Media Service must be called to generate signed links

---

## Default Feed Behavior
- If no filters provided:
  - Location defaults to client-IP-derived city
  - First 100 listings are fetched
- Sorting defaults to date listed

---

## Optional: Media Post-Processing
- Media uploads may trigger async processing:
  - Resize images, generate video thumbnails
  - Use queue (e.g., SQS, Kafka) for background jobs
