# DALL★S — Dallas Venue Finder
## Full city coverage · Crowd-ranked · Live updates

---

## Deploy in 10 Minutes (Free)

### Step 1 — GitHub
```bash
git init
git add .
git commit -m "Dallas venue finder"
git remote add origin https://github.com/YOUR_USERNAME/dallas-spots
git push -u origin main
```

### Step 2 — Vercel
1. Go to vercel.com → New Project
2. Import your GitHub repo
3. Deploy → Done. You get a free URL instantly.

### Step 3 — Add Live Data (Optional but powerful)

#### Google Places API — Real busy times + ratings
1. Get key: console.cloud.google.com → Enable "Places API"
2. Add to Vercel: Settings → Environment Variables → `GOOGLE_PLACES_KEY=your_key`
3. Replace static venue data with:
```javascript
// In index.html, replace static VENUES array fetch with:
const res = await fetch(`/api/venues?lat=32.7767&lng=-96.7970`);
const VENUES = await res.json();
```

#### Create `/api/venues.js` on Vercel (serverless function):
```javascript
export default async function handler(req, res) {
  const { lat, lng } = req.query;
  const types = ['bar','restaurant','cafe','night_club'];
  const results = [];
  
  for (const type of types) {
    const r = await fetch(
      `https://maps.googleapis.com/maps/api/place/nearbysearch/json?` +
      `location=${lat},${lng}&radius=25000&type=${type}&key=${process.env.GOOGLE_PLACES_KEY}`
    );
    const data = await r.json();
    results.push(...data.results);
  }
  
  // Sort by rating * user_ratings_total (genuine crowd signal)
  const sorted = results
    .filter(p => p.user_ratings_total > 100)
    .map(p => ({
      name: p.name,
      score: Math.round(p.rating * 20), // Convert 5★ to /100
      mentions: p.user_ratings_total,
      area: p.vicinity,
      type: detectType(p.types),
      lat: p.geometry.location.lat,
      lng: p.geometry.location.lng,
      placeId: p.place_id,
    }))
    .sort((a,b) => (b.score * Math.log(b.mentions)) - (a.score * Math.log(a.mentions)));
  
  res.json(sorted.slice(0, 60));
}
```

#### Reddit Live Updates (real crowd sentiment)
```javascript
// Fetch r/Dallas posts mentioning venue names
const redditFeed = await fetch(
  'https://www.reddit.com/r/Dallas+DFW/search.json?q=restaurant+bar&sort=new&limit=10'
);
```

---

## Ranking Algorithm
Current score = weighted blend of:
- Google rating (converted to /100)
- log(total reviews) — rewards consistency over hype
- Reddit mention frequency (r/Dallas, r/DFW)
- Recency bonus for fresh activity

---

## Adding More Venues
Edit the `VENUES` array in `index.html`. Each venue needs:
```javascript
{
  id: unique_number,
  name: "Venue Name",
  type: "bar" | "pub" | "restaurant" | "cafe",
  area: "Neighborhood, Dallas",
  neighborhood: "Short neighborhood name",
  gs: true,  // true = near Goldman Sachs Downtown
  tags: ["tag1", "tag2"],
  score: 0-100,
  mentions: number,
  hoursNote: "Opening hours string",
  live: { text: "Live update text", time: "X min ago", fresh: true/false },
  reviews: [{ text: "Review text", user: "Username", src: "Source" }]
}
```

---

## Tech Stack
- Pure HTML/CSS/JS — zero build step, zero dependencies
- Vercel static hosting — free, fast CDN globally
- Google Places API — real ratings and business data
- Vercel serverless functions — API proxying with key protection

---

Built for the Goldman Sachs Downtown Dallas crew. Full city coverage.
