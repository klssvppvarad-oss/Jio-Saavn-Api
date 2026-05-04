# JioSaavn API - Quick Reference Guide

## Endpoint Summary Table

| Endpoint | Route | HTTP Method | Parameters | API Version |
|----------|-------|-------------|-----------|-------------|
| **Search All** | `/search/all` | GET | `query` | v3 |
| **Search Songs** | `/search/songs` | GET | `query`, `page`, `limit` | v3 |
| **Search Albums** | `/search/albums` | GET | `query`, `page`, `limit` | v4 |
| **Search Playlists** | `/search/playlists` | GET | `query`, `page`, `limit` | v3 |
| **Search Artists** | `/search/artists` | GET | `query`, `page`, `limit` | v3 |
| **Song Details (ID)** | `/songs` | GET | `id` | v3 |
| **Song Details (Link)** | `/songs` | GET | `link` | v3 |
| **Album Details (ID)** | `/albums` | GET | `id` | v3 |
| **Album Details (Link)** | `/albums` | GET | `link` | v3 |
| **Artist Details (ID)** | `/artists` | GET | `id` | v4 |
| **Artist Details (Link)** | `/artists` | GET | `link` | v4 |
| **Artist Songs** | `/artists/:artistId/songs` | GET | `page`, `category`, `sort` | v3 |
| **Artist Albums** | `/artists/:artistId/albums` | GET | `page`, `category`, `sort` | v4 |
| **Artist Top Songs** | `/artists/:artistId/recommendations/:songId` | GET | `language` | v3 |
| **Playlist Details** | `/playlists` | GET | `id` | v3 |
| **Song Lyrics** | `/lyrics` | GET | `id` | v4 |
| **Browse Modules** | `/modules` | GET | `language` | v4 |

---

## API Endpoints Used (Internal JioSaavn API)

### Search Endpoints
```
autocomplete.get                    # /search/all
search.getResults                   # /search/songs
search.getAlbumResults             # /search/albums
search.getPlaylistResults          # /search/playlists
search.getArtistResults            # /search/artists
```

### Song Endpoints
```
song.getDetails                    # Song details by ID
webapi.get (type=song)            # Song details by link
```

### Album Endpoints
```
content.getAlbumDetails           # Album details by ID
webapi.get (type=album)           # Album details by link
```

### Artist Endpoints
```
artist.getArtistPageDetails       # Artist details (v4 required)
artist.getArtistMoreSong          # Artist's songs list
artist.getArtistMoreAlbum         # Artist's albums list
search.artistOtherTopSongs        # Top songs for artist
webapi.get (type=artist)          # Artist details by link
```

### Playlist Endpoints
```
playlist.getDetails               # Playlist details
```

### Other Endpoints
```
lyrics.getLyrics                  # Song lyrics
content.getBrowseModules          # Browse modules (trending, charts, etc)
```

---

## Common Query Parameters

### Pagination
```
page: 1-indexed (converted to 0-indexed in service)
limit (n): Results per page
```

### Search
```
query: Search term
q: Alternative search parameter
```

### Resource IDs
```
id: Resource ID (generic)
pids: Song IDs (comma-separated for songs endpoint)
albumid: Album ID
listid: Playlist ID
lyrics_id: Song ID for lyrics
artistId: Artist ID
```

### Links
```
link: Resource link/URL
token: Link token (when using webapi.get)
type: Resource type (song, album, artist, playlist)
```

### Other
```
language: Language code (english, hindi, tamil, etc.)
category: Filter category
sort: Sort order
```

---

## Data Transformation Utilities

### Location: `src/utils/link.ts`

```typescript
createImageLinks(imageUrl)      # Generates multiple resolution image URLs
createDownloadLinks(url)        # Extracts download links
sanitizeLyrics(lyrics)          # Cleans up lyrics content
```

---

## Service Architecture

### Base Service Hierarchy
```
ApiService
    ↓
PayloadService (adds data transformation methods)
    ↓
├── SearchService
├── SongsService
├── AlbumsService
├── ArtistsService
├── PlaylistsService
├── LyricsService
└── ModulesService
```

### ApiService
- Initializes HTTP client (got.js)
- Sets up default headers, query params, cookies
- Provides `http<T>()` method for making API calls

### PayloadService
- Extends ApiService
- Transforms raw API responses into clean DTOs
- Handles image/download link generation
- Provides payload methods for each entity type

### Feature Services
- Extend PayloadService
- Implement specific business logic
- Call API and transform responses

---

## Request/Response Pattern

### Example: Get Song Details

```typescript
// Route Handler
GET /songs?id=123456789

// Controller (SongsController)
const result = await this.songsService.detailsById(id)

// Service Method (SongsService)
async detailsById(ids: string) {
  const response = await this.http<{ songs: SongRequest[] }>(
    this.endpoints.songs.id,  // 'song.getDetails'
    false,                     // isVersion4
    { pids: ids }              // query params
  )
  return response.songs.map(song => this.songPayload(song))
}

// ApiService.http()
// Makes request to: https://www.jiosaavn.com/api.php?__call=song.getDetails&pids=123456789&_format=json&_marker=0&ctx=web6dot0

// Response
{
  "status": "SUCCESS",
  "message": null,
  "data": [
    {
      "id": "123456789",
      "title": "Song Title",
      "image": { "quality_50": "...", "quality_150": "...", ... },
      "album": { "id": "...", "name": "..." },
      "primaryArtists": [...],
      "downloadUrl": { "quality_96": "...", "quality_128": "...", ... },
      ...
    }
  ]
}
```

---

## Error Handling

### Validation Errors
- Input validated in routes using `celebrate` + `joi`
- Returns 400 Bad Request

### Resource Not Found
- Thrown from service with `HttpExceptionError(404, 'message')`
- Returns 404 Not Found

### Timeout
- Middleware returns 503 after 9.5 seconds
- Prevents exceeding Vercel's 10s limit

### API Errors
- Any error bubbles to `errorMiddleware`
- Returns standard error response format

---

## Rate Limiting

### Configuration
```
enableRateLimit: true/false (env: ENABLE_RATE_LIMIT)
```

### Implementation
- Uses LRU cache strategy
- Tracks requests per IP
- Returns 429 Too Many Requests when limit exceeded

---

## Language Support

### Supported Languages
JioSaavn supports multiple languages. Common ones:
- `english`
- `hindi`
- `tamil`
- `telugu`
- `kannada`
- `marathi`
- `gujarati`
- `punjabi`
- `bengali`
- (and others)

### Usage
```
/modules?language=hindi
/artists/:artistId/recommendations/:songId?language=english
```

---

## File Structure for Adding New Endpoint

### To add a new feature endpoint:

1. **Create Route** (`src/routes/feature.route.ts`)
```typescript
export class FeatureRoute implements Route {
  public path = '/features'
  public router = Router()
  public featureController = new FeatureController()
  
  private initializeRoutes() {
    this.router.get(`${this.path}`, this.featureController.method)
  }
}
```

2. **Create Controller** (`src/controllers/feature.controller.ts`)
```typescript
export class FeatureController {
  private featureService: FeatureService
  
  public method: RequestHandler = async (req, res, next) => {
    try {
      const result = await this.featureService.method()
      res.json({ status: 'SUCCESS', message: null, data: result })
    } catch (error) {
      next(error)
    }
  }
}
```

3. **Create Service** (`src/services/feature.service.ts`)
```typescript
export class FeatureService extends PayloadService {
  public method = async () => {
    const response = await this.http(
      this.endpoints.feature,
      false,
      { /* params */ }
    )
    return this.featurePayload(response)
  }
}
```

4. **Create Interface** (`src/interfaces/feature.interface.ts`)
```typescript
export interface FeatureRequest { /* raw API response */ }
export interface FeatureResponse { /* transformed response */ }
```

5. **Add to server** (`src/server.ts`)
```typescript
import { FeatureRoute } from './routes/feature.route'
// ...
new FeatureRoute()
```

6. **Add endpoint to config** (`src/configs/*.config.ts`)
```typescript
endpoint: {
  feature: 'feature.endpointName'
}
```

---

## Development Commands

```bash
# Start development server (with hot reload)
npm run dev

# Build TypeScript
npm run build

# Start production server
npm start
```

---

## Response Status Codes

- `200 OK` - Successful request
- `400 Bad Request` - Invalid input parameters
- `404 Not Found` - Resource not found
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Request timeout (>9.5s)

---

## Key Constants

### Response Status Values
```
status: 'SUCCESS' | 'FAILED'
```

### Cookie Values
```
L: Language (defaults to 'english')
gdpr_acceptance: 'true'
DL: Display Language (defaults to 'english')
```

### Context
```
ctx: 'web6dot0' (JioSaavn web version)
```

---

## Important Notes

1. **API Version Handling**
   - Some endpoints require `api_version: 4` query param
   - Some endpoints need v3 for complete data
   - Check service comments for specific version requirements

2. **Pagination**
   - Client sends 1-indexed pages
   - Service converts to 0-indexed before API call
   - `page - 1` conversion happens in service

3. **Download Links**
   - Multiple quality options available
   - Transformed from encrypted format
   - Utility: `createDownloadLinks()`

4. **Image URLs**
   - Multiple resolution options
   - Transformed to structured format
   - Utility: `createImageLinks()`

5. **Language Handling**
   - Set in query params
   - Updated in request headers dynamically
   - Defaults to 'english'

6. **Timeout Handling**
   - Hard limit: 9.5 seconds
   - Prevents Vercel timeout (10s limit)
   - Returns 503 if exceeded

---

## Useful Links

- **Documentation:** https://docs.saavn.me
- **GitHub:** https://github.com/sumitkolhe/jiosaavn-api
- **Issues/Features:** https://github.com/sumitkolhe/jiosaavn-api/issues

