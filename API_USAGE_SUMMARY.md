# JioSaavn API - Complete API Usage Summary

## Project Overview
This is an unofficial JioSaavn API wrapper built with Express.js and TypeScript. It wraps the internal JioSaavn API endpoints to provide a cleaner interface.

**Base URL:** `https://www.jiosaavn.com/api.php`

**HTTP Client:** `got-cjs` (HTTP requests library)

---

## Architecture Overview

### Project Structure
```
src/
├── app.ts                 # Express app setup
├── server.ts              # Server entry point with all routes
├── constants.ts           # Global constants
├── configs/               # Configuration (dev and production)
├── controllers/           # Request handlers
├── services/              # Business logic & API calls
│   ├── api.service.ts      # Base HTTP client
│   ├── payload.service.ts  # Data transformation/mapping
│   └── [feature].service.ts
├── routes/                # Express routes
├── interfaces/            # TypeScript interfaces
├── middlewares/           # Express middlewares
├── helpers/               # Validation helpers
├── exceptions/            # Custom exceptions
└── utils/                 # Utility functions
```

### Request Flow
Route → Controller → Service → API Service → JioSaavn API

---

## HTTP Client Configuration

### Base HTTP Client (api.service.ts)
```typescript
HTTP Library: got.js
Base URL: https://www.jiosaavn.com/api.php

Default Query Parameters:
- _format: json
- _marker: 0
- ctx: web6dot0

Default Headers:
- cookie: L=english; gdpr_acceptance=true; DL=english

Response Type: JSON
```

---

## API Endpoints & Usage

### 1. HOME ENDPOINT
**Route:** `GET /`

**Controller:** HomeController
- Returns HTML landing page with documentation links

---

### 2. SEARCH ENDPOINTS
**Base Route:** `GET /search/*`
**Base Service:** SearchService

#### 2.1 Search All (Autocomplete)
**Route:** `GET /search/all?query=<search_query>`
- **Endpoint:** `autocomplete.get`
- **API Version:** v3 (API v4 doesn't provide positions)
- **Query Parameters:** `query` (search term)
- **Returns:** Top results across all categories (songs, albums, artists, playlists)

#### 2.2 Search Songs
**Route:** `GET /search/songs?query=<search_query>&page=<page>&limit=<limit>`
- **Endpoint:** `search.getResults`
- **API Version:** v3 (doesn't contain media_preview_url in v4)
- **Query Parameters:** 
  - `q` (search query)
  - `p` (page number)
  - `n` (limit/results per page)
- **Returns:** Song search results with download links

#### 2.3 Search Albums
**Route:** `GET /search/albums?query=<search_query>&page=<page>&limit=<limit>`
- **Endpoint:** `search.getAlbumResults`
- **API Version:** v4
- **Query Parameters:** 
  - `q` (search query)
  - `p` (page number)
  - `n` (limit/results per page)
- **Returns:** Album search results

#### 2.4 Search Playlists
**Route:** `GET /search/playlists?query=<search_query>&page=<page>&limit=<limit>`
- **Endpoint:** `search.getPlaylistResults`
- **API Version:** v3
- **Query Parameters:** 
  - `q` (search query)
  - `p` (page number)
  - `n` (limit/results per page)
- **Returns:** Playlist search results

#### 2.5 Search Artists
**Route:** `GET /search/artists?query=<search_query>&page=<page>&limit=<limit>`
- **Endpoint:** `search.getArtistResults`
- **API Version:** v3
- **Query Parameters:** 
  - `q` (search query)
  - `p` (page number)
  - `n` (limit/results per page)
- **Returns:** Artist search results

---

### 3. SONGS ENDPOINTS
**Base Route:** `GET /songs`
**Base Service:** SongsService

#### 3.1 Get Song Details by ID
**Route:** `GET /songs?id=<song_id>`
- **Endpoint:** `song.getDetails`
- **API Version:** v3
- **Query Parameters:** 
  - `pids` (song IDs, comma-separated)
- **Returns:** Song details with metadata, artists, images, download links

#### 3.2 Get Song Details by Link
**Route:** `GET /songs?link=<song_link>`
- **Endpoint:** `webapi.get`
- **API Version:** v3
- **Query Parameters:** 
  - `token` (song link)
  - `type` = 'song'
- **Returns:** Song details from link

---

### 4. ALBUMS ENDPOINTS
**Base Route:** `GET /albums`
**Base Service:** AlbumsService

#### 4.1 Get Album Details by ID
**Route:** `GET /albums?id=<album_id>`
- **Endpoint:** `content.getAlbumDetails`
- **API Version:** v3
- **Query Parameters:** 
  - `albumid` (album ID)
- **Returns:** Album details with songs, artists, metadata

#### 4.2 Get Album Details by Link
**Route:** `GET /albums?link=<album_link>`
- **Endpoint:** `webapi.get`
- **API Version:** v3
- **Query Parameters:** 
  - `token` (album link)
  - `type` = 'album'
- **Returns:** Album details from link

---

### 5. ARTISTS ENDPOINTS
**Base Route:** `GET /artists/*`
**Base Service:** ArtistsService

#### 5.1 Get Artist Details by ID
**Route:** `GET /artists?id=<artist_id>`
- **Endpoint:** `artist.getArtistPageDetails`
- **API Version:** v4 (no data returned without v4)
- **Query Parameters:** 
  - `artistId` (artist ID)
- **Returns:** Artist details with top songs, albums, metadata

#### 5.2 Get Artist Details by Link
**Route:** `GET /artists?link=<artist_link>`
- **Endpoint:** `webapi.get`
- **API Version:** v4 (no data returned without v4)
- **Query Parameters:** 
  - `token` (artist link)
  - `type` = 'artist'
- **Returns:** Artist details from link

#### 5.3 Get Artist Songs
**Route:** `GET /artists/:artistId/songs?page=<page>&category=<category>&sort=<sort>`
- **Endpoint:** `artist.getArtistMoreSong`
- **API Version:** v3
- **URL Parameters:** 
  - `artistId` (artist ID)
- **Query Parameters:** 
  - `page` (page number, 0-indexed in API)
  - `category` (song category filter)
  - `sort_order` (sort order)
- **Returns:** Paginated list of artist's songs

#### 5.4 Get Artist Albums
**Route:** `GET /artists/:artistId/albums?page=<page>&category=<category>&sort=<sort>`
- **Endpoint:** `artist.getArtistMoreAlbum`
- **API Version:** v4 (no data returned without v4)
- **URL Parameters:** 
  - `artistId` (artist ID)
- **Query Parameters:** 
  - `page` (page number, 0-indexed in API)
  - `category` (album category filter)
  - `sort_order` (sort order)
- **Returns:** Paginated list of artist's albums

#### 5.5 Get Artist Top Songs/Recommendations
**Route:** `GET /artists/:artistId/recommendations/:songId?language=<language>`
- **Endpoint:** `search.artistOtherTopSongs`
- **API Version:** v3
- **URL Parameters:** 
  - `artistId` (artist ID)
  - `songId` (song ID for context)
- **Query Parameters:** 
  - `artist_ids` (artist ID)
  - `song_id` (song ID)
  - `language` (language code)
- **Returns:** Top songs/recommendations for artist based on a song

---

### 6. PLAYLISTS ENDPOINTS
**Base Route:** `GET /playlists`
**Base Service:** PlaylistsService

#### 6.1 Get Playlist Details by ID
**Route:** `GET /playlists?id=<playlist_id>`
- **Endpoint:** `playlist.getDetails`
- **API Version:** v3
- **Query Parameters:** 
  - `listid` (playlist ID)
- **Returns:** Playlist details with all songs, metadata, creator info

---

### 7. LYRICS ENDPOINTS
**Base Route:** `GET /lyrics`
**Base Service:** LyricsService

#### 7.1 Get Song Lyrics
**Route:** `GET /lyrics?id=<song_id>`
- **Endpoint:** `lyrics.getLyrics`
- **API Version:** v4
- **Query Parameters:** 
  - `lyrics_id` (song ID)
- **Returns:** Song lyrics with metadata
- **Error Handling:** Returns 404 if lyrics not found

---

### 8. MODULES ENDPOINTS
**Base Route:** `GET /modules`
**Base Service:** ModulesService

#### 8.1 Get Browse Modules
**Route:** `GET /modules?language=<language>`
- **Endpoint:** `content.getBrowseModules`
- **API Version:** v4
- **Query Parameters:** 
  - `language` (language code, e.g., 'english', 'hindi')
- **Returns:** Browsable modules including:
  - New albums
  - Top playlists
  - Charts
  - Trending songs
  - Trending albums

---

## Data Transformation Pipeline

### PayloadService
All services extend `PayloadService` which handles:

1. **API Response Transformation**
   - Maps raw JioSaavn API responses to clean, consistent formats
   - Handles missing/optional fields gracefully

2. **Image Link Creation** 
   - Generates multiple resolution image URLs from JioSaavn image URLs
   - Utility: `createImageLinks(imageUrl)`

3. **Download Link Creation**
   - Extracts and sanitizes download URLs from encrypted/obfuscated formats
   - Utility: `createDownloadLinks(downloadUrl)`

4. **Lyrics Sanitization**
   - Cleans up lyrics content
   - Utility: `sanitizeLyrics(lyricsText)`

### Key Payload Methods in PayloadService
- `modulesPayload()` - Transforms modules response
- `allSearchPayload()` - Transforms all-search response
- `songSearchPayload()` - Transforms song search results
- `albumSearchPayload()` - Transforms album search results
- `playlistSearchPayload()` - Transforms playlist search results
- `artistSearchPayload()` - Transforms artist search results
- `songPayload()` - Transforms single song
- `albumPayload()` - Transforms single album
- `artistPayload()` - Transforms single artist
- `playlistPayload()` - Transforms single playlist
- `lyricsPayload()` - Transforms lyrics response

---

## Request/Response Format

### Standard Response Format
```json
{
  "status": "SUCCESS" | "FAILED",
  "message": null,
  "data": { /* endpoint-specific data */ }
}
```

### Error Response Format
```json
{
  "status": "FAILED",
  "message": "error message",
  "data": null
}
```

---

## Middleware & Features

### Rate Limiting
- Controlled via `ENABLE_RATE_LIMIT` environment variable
- Implemented in `rateLimiterMiddleware` using LRU cache

### Request Timeout
- Default timeout: 9.5 seconds (Vercel hobby plan limit is 10s)
- Returns 503 status if exceeded
- Implemented in `express-timeout-handler`

### Logging
- Using `winston` logger
- Formats: 
  - Development: `dev` (detailed)
  - Production: `tiny` (minimal)

### CORS
- Enabled for all origins
- Headers configured in `app.ts`

### Request Validation
- Using `celebrate` and `joi` for input validation
- Validators in `validation.helper`

---

## Query Parameter Details

### Common Parameters
- `query` / `q` - Search query string
- `page` - Page number (1-indexed for API, converted to 0-indexed before sending)
- `limit` / `n` - Results per page
- `id` / `artistId` / `albumid` / `listid` / `lyrics_id` - Resource IDs
- `link` / `token` - Resource URL/link
- `language` - Language code (english, hindi, etc.)

### Pagination
- **API expects:** 0-indexed pages
- **Client sends:** 1-indexed pages
- **Conversion:** `page: page - 1` in service

---

## API Versioning Notes

### API v3 (Default)
- Older version with more data
- Contains `media_preview_url`
- Better for most search endpoints

### API v4
- Newer version with some limitations
- No `media_preview_url`
- Required for: artist details, album filtering, lyrics
- Set via query parameter: `api_version: 4`

**Comments in code indicate which endpoints need specific versions:**
- "api v4 does not contain media_preview_url"
- "without api v4 no data is returned"
- "api v4 does not provide positions"

---

## Error Handling

### Custom Exception
`HttpExceptionError` - Thrown when:
- Resource not found (404)
- Invalid request data
- Caught by `errorMiddleware` and returned as standard error response

### Common Error Scenarios
1. **Song/Lyrics not found** - 404 status
2. **Invalid search query** - Validation error
3. **Request timeout** - 503 status (Vercel limit)
4. **Invalid parameters** - Validation error before API call

---

## HTTP Headers & Cookies

### Request Headers
```
cookie: L=english; gdpr_acceptance=true; DL=english
```

### Language Handling
- Language is set from query parameter or defaults to 'english'
- Updated in request headers dynamically via `beforeRequest` hook

### Search Params (All Requests)
- `_format=json` - Response format
- `_marker=0` - Pagination marker
- `ctx=web6dot0` - Context/client type
- `__call=<endpoint>` - JioSaavn endpoint to call

---

## Data Types & Response Examples

### Common Response Fields

**Song Object:**
- `id`, `title`, `image`, `url`, `album`, `year`, `duration`
- `primaryArtists`, `featuredArtists`, `explicitContent`
- `playCount`, `language`, `downloadUrl` (with multiple qualities)

**Album Object:**
- `id`, `title`, `image`, `url`, `artist`, `year`, `releaseDate`
- `songCount`, `primaryArtists`, `explicitContent`
- `playCount`, `language`, `songs` (array)

**Artist Object:**
- `id`, `name`, `url`, `image`, `followerCount`
- `topSongs`, `topAlbums`, `similarArtists`

**Playlist Object:**
- `id`, `title`, `image`, `url`, `userId`, `followerCount`
- `songCount`, `lastUpdated`, `songs` (array)

---

## Configuration

### Environment Variables
- `NODE_ENV` - development | production
- `PORT` - Server port (default 3000 for dev, 80 for prod)
- `ENABLE_RATE_LIMIT` - true | false

### Config Files
- `src/configs/dev.config.ts` - Development configuration
- `src/configs/production.config.ts` - Production configuration

Both configs define:
- Base URL
- All endpoint names
- Server settings
- Log settings
- Rate limiting

---

## Key Technologies

- **HTTP Client:** got-cjs
- **Server Framework:** Express.js
- **Language:** TypeScript
- **Validation:** celebrate + joi
- **Logging:** winston
- **Rate Limiting:** lru-cache
- **Encryption:** node-forge (for secure handling)
- **CORS:** cors middleware
- **Timeout Handling:** express-timeout-handler

---

## Summary

This JioSaavn API wrapper provides a clean interface to:
1. **Search** across songs, albums, artists, playlists
2. **Get Details** for songs, albums, artists, playlists
3. **Browse** curated modules with trending content
4. **Get Lyrics** for songs
5. **Discover** artist recommendations and more songs/albums

All requests are routed through a payload transformation layer that:
- Cleans up raw JioSaavn API responses
- Generates proper image URLs
- Extracts download links
- Handles errors gracefully
- Supports pagination
- Respects rate limits

The API uses a mix of v3 and v4 endpoints depending on required data availability, with careful documentation of version-specific limitations.
