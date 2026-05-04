# JioSaavn API - Architecture & Data Flow Diagrams

## 1. Request Flow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLIENT REQUEST                              │
│                                                                 │
│    GET /songs?id=123456789                                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    EXPRESS APP                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ MIDDLEWARE STACK                                         │  │
│  │ ├─ morgan (logging)                                      │  │
│  │ ├─ cors                                                  │  │
│  │ ├─ express.json()                                        │  │
│  │ ├─ rateLimiterMiddleware                                │  │
│  │ └─ express-timeout-handler (9.5s max)                   │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ROUTE LAYER                                  │
│                                                                 │
│  SongsRoute                                                    │
│  ├─ GET /songs (songsSchema validation)                      │
│  └─ handlers: [SongsController.songDetails]                  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                 CONTROLLER LAYER                                │
│                                                                 │
│  SongsController                                               │
│  ├─ Receives: req (id=123456789)                              │
│  ├─ Creates: SongsService                                      │
│  ├─ Calls: songsService.detailsById('123456789')              │
│  ├─ Returns: SongResponse[]                                    │
│  └─ Sends: { status: 'SUCCESS', data: [...] }                │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  SERVICE LAYER                                  │
│                                                                 │
│  SongsService extends PayloadService                           │
│  ├─ Method: detailsById(ids: string)                          │
│  ├─ Calls: this.http<SongRequest>(                            │
│  │          this.endpoints.songs.id,  // 'song.getDetails'   │
│  │          false,                     // isVersion4          │
│  │          { pids: ids }              // query params        │
│  │        )                                                    │
│  ├─ Transforms: response.songs.map(song => songPayload(song)) │
│  └─ Returns: SongResponse[]                                    │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                 PAYLOAD SERVICE LAYER                           │
│                                                                 │
│  PayloadService (Data Transformation)                         │
│  ├─ songPayload(song: SongRequest): SongResponse              │
│  │  ├─ Extract base fields                                    │
│  │  ├─ createImageLinks(song.image)                           │
│  │  ├─ createDownloadLinks(song.download_url)                 │
│  │  └─ Map artists, metadata                                  │
│  └─ Returns: Cleaned SongResponse object                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                API SERVICE LAYER (HTTP)                         │
│                                                                 │
│  ApiService                                                    │
│  ├─ HTTP Client: got.js                                        │
│  ├─ Base URL: https://www.jiosaavn.com/api.php               │
│  ├─ Method: http<T>(url, isVersion4, query?)                 │
│  │  ├─ Builds search params:                                  │
│  │  │  ├─ __call: url (endpoint name)                         │
│  │  │  ├─ api_version: 4 (if isVersion4)                      │
│  │  │  ├─ ...query params                                     │
│  │  │  ├─ _format: json                                       │
│  │  │  ├─ _marker: 0                                          │
│  │  │  ├─ ctx: web6dot0                                       │
│  │  │  └─ ...                                                 │
│  │  └─ Sets Headers & Cookies                                 │
│  └─ Returns: Promise<T>                                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                EXTERNAL API REQUEST                             │
│                                                                 │
│  HTTP GET https://www.jiosaavn.com/api.php?                   │
│            __call=song.getDetails&                             │
│            pids=123456789&                                     │
│            _format=json&                                       │
│            _marker=0&                                          │
│            ctx=web6dot0&                                       │
│            ...                                                 │
│                                                                 │
│  Headers:                                                      │
│  ├─ cookie: L=english; gdpr_acceptance=true; DL=english      │
│  └─ ...                                                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│            JIOSAAVN SERVER (External API)                       │
│                                                                 │
│  Processes: song.getDetails endpoint                           │
│  Returns: Raw JSON response with song data                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│            API SERVICE RESPONSE PARSING                         │
│                                                                 │
│  ├─ Parse JSON                                                 │
│  ├─ Type as SongRequest[]                                      │
│  └─ Return to PayloadService                                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│          PAYLOAD TRANSFORMATION (PayloadService)                │
│                                                                 │
│  ├─ Transform each SongRequest → SongResponse                  │
│  ├─ Clean and map fields                                       │
│  ├─ Generate image URLs and download links                     │
│  └─ Return: SongResponse[]                                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│         SERVICE → CONTROLLER RESPONSE                           │
│                                                                 │
│  SongResponse[] is returned to SongsController                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│       CONTROLLER RESPONSE FORMATTING                            │
│                                                                 │
│  Wraps in standard response:                                   │
│  {                                                              │
│    "status": "SUCCESS",                                        │
│    "message": null,                                            │
│    "data": SongResponse[]                                      │
│  }                                                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│         ERROR MIDDLEWARE (if error occurs)                      │
│                                                                 │
│  ├─ Catch error from any layer                                │
│  ├─ Format as error response                                   │
│  ├─ Log error                                                  │
│  └─ Send error response to client                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│              CLIENT RESPONSE                                    │
│                                                                 │
│  HTTP 200 OK                                                   │
│  {                                                              │
│    "status": "SUCCESS",                                        │
│    "message": null,                                            │
│    "data": [                                                   │
│      {                                                          │
│        "id": "123456789",                                      │
│        "title": "Song Title",                                  │
│        "image": { "quality_50": "...", ... },                 │
│        "downloadUrl": { "quality_96": "...", ... },           │
│        ...                                                      │
│      }                                                          │
│    ]                                                            │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Service Class Hierarchy

```
┌──────────────────────────────┐
│      ApiService              │
│  (HTTP Client Setup)         │
│                              │
│  - httpClient (got.js)       │
│  - baseURL                   │
│  - endpoints                 │
│  - http<T>() method          │
└────────────┬─────────────────┘
             │
             │ extends
             │
             ▼
┌──────────────────────────────────────────────────────────┐
│         PayloadService                                   │
│    (Data Transformation)                                 │
│                                                          │
│  - modulesPayload()                                      │
│  - allSearchPayload()                                    │
│  - songPayload()                                         │
│  - albumPayload()                                        │
│  - artistPayload()                                       │
│  - playlistPayload()                                     │
│  - lyricsPayload()                                       │
│  - mapArtists()                                          │
│  - And many more...                                      │
└────┬────────┬────────┬────────┬────────┬────────┬────────┘
     │        │        │        │        │        │
     │extends │extends │extends │extends │extends │extends
     │        │        │        │        │        │
     ▼        ▼        ▼        ▼        ▼        ▼        ▼
  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌────────┐
  │Search│ │Songs │ │Albums│ │Artist│ │Playlist│ │Lyrics │ │Modules │
  │Service│ │Service│ │Service│ │Service│ │Service│ │Service│ │Service │
  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └────────┘
```

---

## 3. Data Flow: Search Endpoint Example

```
┌────────────────────────────────────────────────────────────┐
│ CLIENT                                                     │
│ GET /search/songs?query=Imagine&page=1&limit=20           │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ SearchRoute                                                │
│ ├─ Path: /search                                           │
│ └─ Endpoint: GET /songs                                    │
│    └─ Middleware: searchSchema validation                  │
│    └─ Handler: SearchController.searchSongs               │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ SearchController                                           │
│ searchSongs(req, res, next)                               │
│ ├─ Extract: query, page, limit                            │
│ ├─ Call: searchService.songs(                             │
│ │         "Imagine",                                       │
│ │         1,                                               │
│ │         20                                               │
│ │       )                                                  │
│ └─ Send: { status: 'SUCCESS', data: result }              │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ SearchService.songs()                                      │
│                                                            │
│ ├─ Call: this.http<SongSearchRequest>(                    │
│ │         this.endpoints.search.songs,  // 'search...'   │
│ │         false,                         // v3 API        │
│ │         {                                               │
│ │           q: "Imagine",                                 │
│ │           p: 1,                                         │
│ │           n: 20                                         │
│ │         }                                               │
│ │       )                                                 │
│ │                                                         │
│ ├─ Get: raw SongSearchRequest response                   │
│ │                                                         │
│ ├─ Call: this.songSearchPayload(response)                │
│ │                                                         │
│ └─ Return: SongSearchResponse (transformed)               │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ ApiService.http<SongSearchRequest>()                       │
│                                                            │
│ ├─ Build params: {                                        │
│ │   __call: 'search.getResults',                          │
│ │   q: 'Imagine',                                         │
│ │   p: 1,                                                 │
│ │   n: 20,                                                │
│ │   _format: 'json',                                      │
│ │   _marker: 0,                                           │
│ │   ctx: 'web6dot0'                                       │
│ │ }                                                        │
│ │                                                         │
│ ├─ Add headers: {                                         │
│ │   cookie: 'L=english; gdpr_acceptance=true; ...'       │
│ │ }                                                        │
│ │                                                         │
│ ├─ Make HTTP GET request to:                             │
│ │   https://www.jiosaavn.com/api.php?[params]            │
│ │                                                         │
│ └─ Return: Promise<SongSearchRequest>                     │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ JioSaavn External API                                      │
│                                                            │
│ Process: search.getResults                                │
│ Input: q=Imagine, p=1, n=20                               │
│ Output: Raw JSON with search results                      │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ Response received: SongSearchRequest                       │
│                                                            │
│ Structure:                                                 │
│ {                                                          │
│   results: [                                               │
│     {                                                      │
│       id: "123456789",                                     │
│       title: "Imagine",                                    │
│       image: "base_image_url",                             │
│       media_preview_url: "...",                            │
│       download_url: "...encrypted...",                     │
│       // ... raw fields                                    │
│     },                                                     │
│     ...                                                    │
│   ],                                                       │
│   total: "5000"                                            │
│ }                                                          │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ PayloadService.songSearchPayload()                         │
│                                                            │
│ ├─ For each song in results:                              │
│ │  ├─ Call: songPayload(song)                             │
│ │  │  ├─ Extract fields                                   │
│ │  │  ├─ createImageLinks(song.image)                     │
│ │  │  │  └─ Generate URLs for different sizes              │
│ │  │  ├─ createDownloadLinks(song.download_url)           │
│ │  │  │  └─ Parse encrypted download URL                  │
│ │  │  └─ Return: SongResponse                             │
│ │  │                                                      │
│ │  └─ Add to results array                               │
│ │                                                         │
│ ├─ Transform total count                                  │
│ │                                                         │
│ └─ Return: SongSearchResponse {                           │
│      results: [SongResponse[], ...],                       │
│      totalCount: 5000                                      │
│    }                                                       │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ Back to SearchController                                   │
│                                                            │
│ ├─ Receive: SongSearchResponse                            │
│ ├─ Format response:                                        │
│ │  {                                                       │
│ │    "status": "SUCCESS",                                  │
│ │    "message": null,                                      │
│ │    "data": SongSearchResponse                            │
│ │  }                                                       │
│ └─ res.json(response)                                      │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ CLIENT RECEIVES RESPONSE                                   │
│ HTTP 200 OK                                                │
│ {                                                          │
│   "status": "SUCCESS",                                     │
│   "message": null,                                         │
│   "data": {                                                │
│     "results": [                                           │
│       {                                                    │
│         "id": "123456789",                                 │
│         "title": "Imagine",                                │
│         "image": {                                         │
│           "quality_50": "https://...",                     │
│           "quality_150": "https://...",                    │
│           "quality_500": "https://..."                     │
│         },                                                 │
│         "downloadUrl": {                                   │
│           "quality_96": "https://...",                     │
│           "quality_128": "https://...",                    │
│           "quality_320": "https://..."                     │
│         },                                                 │
│         // ... other fields                                │
│       },                                                   │
│       // ... more songs                                    │
│     ],                                                     │
│     "totalCount": 5000                                     │
│   }                                                        │
│ }                                                          │
└────────────────────────────────────────────────────────────┘
```

---

## 4. Route Organization

```
Server (Express App)
│
├─ HomeRoute
│  └─ GET / → HomeController.home() → HTML landing page
│
├─ SearchRoute
│  ├─ GET /search/all → SearchController.searchAll()
│  ├─ GET /search/songs → SearchController.searchSongs()
│  ├─ GET /search/albums → SearchController.searchAlbums()
│  ├─ GET /search/playlists → SearchController.searchPlaylists()
│  └─ GET /search/artists → SearchController.searchArtists()
│
├─ SongsRoute
│  └─ GET /songs → SongsController.songDetails()
│
├─ AlbumsRoute
│  └─ GET /albums → AlbumsController.albumDetails()
│
├─ ArtistsRoute
│  ├─ GET /artists → ArtistsController.artistDetails()
│  ├─ GET /artists/:artistId/albums → ArtistsController.artistAblums()
│  ├─ GET /artists/:artistId/songs → ArtistsController.artistSongs()
│  └─ GET /artists/:artistId/recommendations/:songId → ArtistsController.artistTopSongs()
│
├─ PlaylistsRoute
│  └─ GET /playlists → PlaylistsController.playlistDetails()
│
├─ LyricsRoute
│  └─ GET /lyrics → LyricsController.lyricsDetails()
│
└─ ModulesRoute
   └─ GET /modules → ModulesController.browseModules()
```

---

## 5. API Versioning Decision Tree

```
┌─────────────────────────────────────────┐
│   Need to call JioSaavn Endpoint        │
└────────────────┬────────────────────────┘
                 │
                 ▼
        ┌────────────────┐
        │ Endpoint Type? │
        └────────────────┘
                 │
    ┌────────────┼────────────┬─────────────┐
    │            │            │             │
    ▼            ▼            ▼             ▼
┌────────┐ ┌────────┐ ┌─────────────┐ ┌──────────┐
│ Search │ │ Songs  │ │ Artist      │ │ Lyrics   │
│        │ │        │ │ Modules     │ │          │
└────────┘ └────────┘ │ Album       │ └──────────┘
    │          │      │ (by link)   │      │
    │          │      └─────────────┘      │
    │          │             │             │
  v3        v3       Use v4 if:         Use v4
            OR           │
  No v4     v4        │ Artist Page
  needed    if          │  Details
            want        │ OR Album
            preview   │  Details
            URL        │ OR Artist
                      │  Filtering
                      │
                      ▼
                    Use v4
```

---

## 6. Error Handling Flow

```
┌─────────────────────────────┐
│   Request processed in any  │
│       service layer         │
└────────────────┬────────────┘
                 │
        ┌────────▼─────────┐
        │ Error occurs?    │
        └────────┬─────────┘
                 │
          ┌──────┴──────┐
          │             │
         NO             YES
          │              │
          ▼              ▼
    ┌─────────────┐  ┌──────────────┐
    │ Continue    │  │ Throw error  │
    │ processing  │  │ to catch     │
    └─────────────┘  └──────┬───────┘
          │                 │
          │                 ▼
          │         ┌─────────────────────┐
          │         │ Error propagates to │
          │         │ next(error) in      │
          │         │ controller catch    │
          │         └──────────┬──────────┘
          │                    │
          │                    ▼
          │         ┌──────────────────────┐
          │         │ errorMiddleware      │
          │         │                      │
          │         ├─ Check error type   │
          │         │  & status code      │
          │         ├─ Log error          │
          │         ├─ Format response:   │
          │         │  {                  │
          │         │    status: FAILED   │
          │         │    message: error   │
          │         │    data: null       │
          │         │  }                  │
          │         └──────────┬──────────┘
          │                    │
          └────────┬───────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │ Send to CLIENT       │
        │                      │
        │ HTTP Status Code:    │
        │ - 400 Bad Request    │
        │ - 404 Not Found      │
        │ - 500 Server Error   │
        │ - 503 Timeout        │
        │ - etc.               │
        └──────────────────────┘
```

---

## 7. Configuration & Environment Flow

```
┌──────────────────────────────┐
│   Application Startup        │
│   (server.ts)                │
└───────────────┬──────────────┘
                │
                ▼
        ┌───────────────────┐
        │ getConfig()       │
        └───────┬───────────┘
                │
        ┌───────▼────────┐
        │ Check:         │
        │ NODE_ENV       │
        └───────┬────────┘
                │
    ┌───────────┴──────────────┐
    │                          │
  'prod'?                    'dev'?
    │                          │
    ▼                          ▼
┌─────────────┐        ┌──────────────┐
│production   │        │dev.config.ts │
│.config.ts   │        └──────┬───────┘
│             │               │
│- Port: 80   │         - Port: 3000
│- Host:      │         - Host: localhost
│  0.0.0.0    │         - Log: dev
│- Log:tiny   │         - RateLimit: false
│- RateLimit: │
│  env-based  │    BASE CONFIG USED:
└──────┬──────┘    - baseURL
       │           - endpoints
       │             (all endpoint names)
       └─────┬──────────────────┐
             │                  │
             └─────────────────┐│
                              ││
                              ▼▼
                    ┌──────────────────────┐
                    │ App Initialization   │
                    │                      │
                    ├─ Create Express app  │
                    ├─ Setup middlewares   │
                    ├─ Register routes     │
                    ├─ Setup error handler │
                    │                      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ App listening on     │
                    │ configured port      │
                    └──────────────────────┘
```

This comprehensive set of diagrams shows how the JioSaavn API wrapper processes requests from initial receipt through transformation to final response delivery.
