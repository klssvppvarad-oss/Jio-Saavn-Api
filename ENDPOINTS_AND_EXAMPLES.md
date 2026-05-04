# JioSaavn API - Endpoints Reference & Code Examples

## Complete API Endpoints Table

| # | Feature | Route | Method | Query Params | Response | Notes |
|---|---------|-------|--------|--------------|----------|-------|
| 1 | Home | `/` | GET | - | HTML page | Landing page |
| 2 | Search All | `/search/all` | GET | `query` | All categories | Autocomplete |
| 3 | Search Songs | `/search/songs` | GET | `query`, `page`, `limit` | Songs list | With download links |
| 4 | Search Albums | `/search/albums` | GET | `query`, `page`, `limit` | Albums list | API v4 |
| 5 | Search Playlists | `/search/playlists` | GET | `query`, `page`, `limit` | Playlists list | - |
| 6 | Search Artists | `/search/artists` | GET | `query`, `page`, `limit` | Artists list | - |
| 7 | Song Details | `/songs` | GET | `id` OR `link` | Song object | Get by ID or link |
| 8 | Album Details | `/albums` | GET | `id` OR `link` | Album object | Get by ID or link |
| 9 | Artist Details | `/artists` | GET | `id` OR `link` | Artist object | API v4, get by ID or link |
| 10 | Artist Songs | `/artists/:artistId/songs` | GET | `page`, `category`, `sort` | Paginated songs | Artist's songs |
| 11 | Artist Albums | `/artists/:artistId/albums` | GET | `page`, `category`, `sort` | Paginated albums | Artist's albums, API v4 |
| 12 | Artist Top Songs | `/artists/:artistId/recommendations/:songId` | GET | `language` | Songs array | Top songs for artist |
| 13 | Playlist Details | `/playlists` | GET | `id` | Playlist object | - |
| 14 | Song Lyrics | `/lyrics` | GET | `id` | Lyrics object | API v4 |
| 15 | Browse Modules | `/modules` | GET | `language` | Modules object | Trending, charts, etc. |

---

## cURL Examples

### 1. Search Songs
```bash
curl "http://localhost:3000/search/songs?query=Imagine&page=1&limit=10"
```

### 2. Get Song Details by ID
```bash
curl "http://localhost:3000/songs?id=123456789"
```

### 3. Get Song Details by Link
```bash
curl "http://localhost:3000/songs?link=https://www.jiosaavn.com/song/imagine/..."
```

### 4. Get Album by ID
```bash
curl "http://localhost:3000/albums?id=123456789"
```

### 5. Get Artist Details
```bash
curl "http://localhost:3000/artists?id=987654321"
```

### 6. Get Artist Songs
```bash
curl "http://localhost:3000/artists/987654321/songs?page=1&category=featured&sort=1"
```

### 7. Get Artist Albums
```bash
curl "http://localhost:3000/artists/987654321/albums?page=1&category=&sort=1"
```

### 8. Get Song Lyrics
```bash
curl "http://localhost:3000/lyrics?id=123456789"
```

### 9. Get Playlist
```bash
curl "http://localhost:3000/playlists?id=1261197649"
```

### 10. Browse Modules
```bash
curl "http://localhost:3000/modules?language=english"
```

### 11. Search All
```bash
curl "http://localhost:3000/search/all?query=Imagine"
```

### 12. Search Albums
```bash
curl "http://localhost:3000/search/albums?query=Abbey%20Road&page=1&limit=10"
```

### 13. Search Artists
```bash
curl "http://localhost:3000/search/artists?query=Beatles&page=1&limit=10"
```

---

## JavaScript/Fetch Examples

### 1. Search Songs
```javascript
fetch('http://localhost:3000/search/songs?query=Imagine&page=1&limit=10')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

### 2. Get Song by ID
```javascript
fetch('http://localhost:3000/songs?id=123456789')
  .then(res => res.json())
  .then(data => {
    if (data.status === 'SUCCESS') {
      console.log('Song:', data.data);
      console.log('Download URLs:', data.data[0].downloadUrl);
    }
  });
```

### 3. Get Artist with All Details
```javascript
const fetchArtistWithDetails = async (artistId) => {
  try {
    // Get artist details
    const artistRes = await fetch(`http://localhost:3000/artists?id=${artistId}`);
    const artistData = await artistRes.json();
    
    // Get artist top songs
    const songsRes = await fetch(
      `http://localhost:3000/artists/${artistId}/songs?page=1&category=all&sort=1`
    );
    const songsData = await songsRes.json();
    
    // Get artist albums
    const albumsRes = await fetch(
      `http://localhost:3000/artists/${artistId}/albums?page=1&category=&sort=1`
    );
    const albumsData = await albumsRes.json();
    
    return {
      artist: artistData.data,
      songs: songsData.data.songs,
      albums: albumsData.data.albums
    };
  } catch (error) {
    console.error('Error fetching artist:', error);
  }
};
```

### 4. Search and Get First Result Details
```javascript
const searchAndGetDetails = async (query) => {
  try {
    // Search all categories
    const searchRes = await fetch(`http://localhost:3000/search/all?query=${query}`);
    const searchData = await searchRes.json();
    
    // Get first song details
    if (searchData.data.songs.results.length > 0) {
      const songId = searchData.data.songs.results[0].id;
      const detailsRes = await fetch(`http://localhost:3000/songs?id=${songId}`);
      const details = await detailsRes.json();
      
      return details.data[0];
    }
  } catch (error) {
    console.error('Error:', error);
  }
};
```

### 5. Get Playlist with All Songs
```javascript
fetch('http://localhost:3000/playlists?id=1261197649')
  .then(res => res.json())
  .then(data => {
    if (data.status === 'SUCCESS') {
      const playlist = data.data;
      console.log('Playlist:', playlist.title);
      console.log('Total songs:', playlist.songCount);
      console.log('Songs:', playlist.songs);
    }
  });
```

### 6. Get Song Lyrics
```javascript
fetch('http://localhost:3000/lyrics?id=123456789')
  .then(res => res.json())
  .then(data => {
    if (data.status === 'SUCCESS') {
      console.log('Lyrics:', data.data.lyrics);
    } else {
      console.log('Lyrics not available');
    }
  });
```

---

## Node.js/Axios Examples

### 1. Setup
```javascript
const axios = require('axios');

const api = axios.create({
  baseURL: 'http://localhost:3000'
});
```

### 2. Search with Error Handling
```javascript
const searchSongs = async (query, page = 1, limit = 10) => {
  try {
    const response = await api.get('/search/songs', {
      params: { query, page, limit }
    });
    
    if (response.data.status === 'SUCCESS') {
      return response.data.data;
    } else {
      throw new Error(response.data.message || 'Search failed');
    }
  } catch (error) {
    console.error('Search error:', error.message);
    throw error;
  }
};
```

### 3. Get Comprehensive Artist Info
```javascript
const getArtistComprehensive = async (artistId) => {
  try {
    const [artist, songs, albums] = await Promise.all([
      api.get('/artists', { params: { id: artistId } }),
      api.get(`/artists/${artistId}/songs`, { 
        params: { page: 1, category: 'all', sort: 1 } 
      }),
      api.get(`/artists/${artistId}/albums`, { 
        params: { page: 1, sort: 1 } 
      })
    ]);
    
    return {
      artist: artist.data.data,
      songs: songs.data.data?.songs || [],
      albums: albums.data.data?.albums || []
    };
  } catch (error) {
    console.error('Failed to fetch artist:', error);
  }
};
```

### 4. Download Song Information
```javascript
const getSongForDownload = async (songId) => {
  try {
    const response = await api.get('/songs', { params: { id: songId } });
    
    if (response.data.status === 'SUCCESS') {
      const song = response.data.data[0];
      return {
        title: song.title,
        artist: song.primaryArtists.map(a => a.name).join(', '),
        url_128: song.downloadUrl.quality_128,
        url_320: song.downloadUrl.quality_320,
        image: song.image.quality_500
      };
    }
  } catch (error) {
    console.error('Error fetching song:', error);
  }
};
```

---

## Python Examples

### 1. Basic Setup
```python
import requests

BASE_URL = 'http://localhost:3000'

def search_songs(query, page=1, limit=10):
    response = requests.get(f'{BASE_URL}/search/songs', params={
        'query': query,
        'page': page,
        'limit': limit
    })
    return response.json()

def get_song(song_id):
    response = requests.get(f'{BASE_URL}/songs', params={'id': song_id})
    return response.json()
```

### 2. Advanced: Get Full Artist Profile
```python
import requests
from concurrent.futures import ThreadPoolExecutor

BASE_URL = 'http://localhost:3000'

def get_artist_profile(artist_id):
    with ThreadPoolExecutor(max_workers=3) as executor:
        artist_future = executor.submit(lambda: requests.get(
            f'{BASE_URL}/artists', 
            params={'id': artist_id}
        ).json())
        
        songs_future = executor.submit(lambda: requests.get(
            f'{BASE_URL}/artists/{artist_id}/songs',
            params={'page': 1, 'category': 'all', 'sort': 1}
        ).json())
        
        albums_future = executor.submit(lambda: requests.get(
            f'{BASE_URL}/artists/{artist_id}/albums',
            params={'page': 1, 'sort': 1}
        ).json())
        
        return {
            'artist': artist_future.result()['data'],
            'songs': songs_future.result()['data']['songs'],
            'albums': albums_future.result()['data']['albums']
        }
```

### 3. Paginate Through Results
```python
def search_all_pages(query, limit=10):
    all_results = []
    page = 1
    
    while True:
        response = requests.get(f'{BASE_URL}/search/songs', params={
            'query': query,
            'page': page,
            'limit': limit
        })
        
        data = response.json()
        if data['status'] != 'SUCCESS':
            break
            
        songs = data['data']['results']
        if not songs:
            break
            
        all_results.extend(songs)
        page += 1
    
    return all_results
```

---

## Response Examples

### Song Response
```json
{
  "status": "SUCCESS",
  "message": null,
  "data": [
    {
      "id": "123456789",
      "title": "Imagine",
      "image": {
        "quality_50": "https://...",
        "quality_150": "https://...",
        "quality_500": "https://..."
      },
      "album": {
        "id": "987654321",
        "name": "Imagine (Album)",
        "url": "https://..."
      },
      "primaryArtists": [
        {
          "id": "111111",
          "name": "John Lennon",
          "role": "Singer",
          "image": { "quality_50": "...", ... }
        }
      ],
      "featuredArtists": [],
      "downloadUrl": {
        "quality_96": "https://...",
        "quality_128": "https://...",
        "quality_320": "https://..."
      },
      "duration": 243,
      "year": 1971,
      "releaseDate": "1971-09-09",
      "playCount": 1000000,
      "language": "english",
      "url": "https://...",
      "explicitContent": false
    }
  ]
}
```

### Artist Response
```json
{
  "status": "SUCCESS",
  "message": null,
  "data": {
    "id": "111111",
    "name": "John Lennon",
    "url": "https://...",
    "image": { "quality_50": "...", ... },
    "followerCount": 5000000,
    "followerCountText": "5M+",
    "description": "British musician...",
    "topSongs": [ /* array of songs */ ],
    "topAlbums": [ /* array of albums */ ],
    "similarArtists": [ /* array of artists */ ]
  }
}
```

### Playlist Response
```json
{
  "status": "SUCCESS",
  "message": null,
  "data": {
    "id": "1261197649",
    "title": "Hits of 90s",
    "userId": "12345678",
    "followerCount": 50000,
    "songCount": 50,
    "image": { "quality_50": "...", ... },
    "url": "https://...",
    "explicitContent": false,
    "description": "Best songs from the 90s",
    "lastUpdated": "2024-01-15",
    "songs": [
      {
        "id": "123456789",
        "title": "Song Title",
        // ... full song object
      },
      // ... more songs
    ]
  }
}
```

### Error Response
```json
{
  "status": "FAILED",
  "message": "song not found",
  "data": null
}
```

### Search Results Response
```json
{
  "status": "SUCCESS",
  "message": null,
  "data": {
    "results": [
      {
        "id": "123456789",
        "title": "Imagine",
        "image": { "quality_50": "...", ... },
        "url": "https://...",
        "album": "Imagine",
        "type": "song",
        "description": "1971 • John Lennon",
        "position": 1,
        "primaryArtists": "John Lennon",
        "singers": "John Lennon"
      },
      // ... more results
    ],
    "totalCount": 100
  }
}
```

---

## Testing Checklist

- [ ] Test search with empty query
- [ ] Test pagination (page 1, last page, beyond total)
- [ ] Test with invalid song/album IDs
- [ ] Test with unavailable lyrics
- [ ] Test rate limiting (if enabled)
- [ ] Test concurrent requests
- [ ] Test timeout (request taking >9.5s)
- [ ] Test different languages
- [ ] Test with special characters in query
- [ ] Test different sort/category combinations

---

## Common Issues & Solutions

### Issue: "song not found" (404)
**Solution:** Verify the song ID is correct. Song IDs are numeric strings.

### Issue: No lyrics returned
**Solution:** Not all songs have lyrics. Check the endpoint response for null or missing lyrics field.

### Issue: Empty download URLs
**Solution:** Some songs may not have download links. Check if downloadUrl exists and has quality options.

### Issue: Timeout errors (503)
**Solution:** The API call took too long (>9.5s). Try breaking down large requests or checking network connectivity.

### Issue: Rate limit exceeded (429)
**Solution:** If rate limiting is enabled, wait before making more requests. Check ENABLE_RATE_LIMIT configuration.

### Issue: No results for artist details
**Solution:** For artist endpoints without v4 API specified in service, use the `/artists?id=` endpoint instead of link-based queries.

---

## Performance Tips

1. **Pagination:** Always use pagination for search to avoid large responses
2. **Caching:** Cache frequently accessed data (song details, artist info)
3. **Concurrent Requests:** Use Promise.all() or ThreadPoolExecutor for parallel API calls
4. **Language Selection:** Specify language to get locale-specific results
5. **Image Optimization:** Download only the image quality you need (quality_50, quality_150, quality_500)

