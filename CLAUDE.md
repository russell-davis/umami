# Umami Fork - Timezone Fix

Fork of [umami-software/umami](https://github.com/umami-software/umami) with a timezone bug fix for the Traffic heatmap.

## The Fix

**File:** `src/queries/sql/getWeeklyTraffic.ts:17`

```typescript
// Upstream (bug): hardcodes UTC
const timezone = 'utc';

// This fork (fixed): uses timezone from API request
const { timezone = 'utc' } = filters;
```

The PostgreSQL query was ignoring the timezone parameter passed from the frontend, causing the Traffic heatmap to display hours in UTC instead of the user's local timezone.

## Pre-Build Verification

**ALWAYS run before building a new image:**

```bash
grep -n "const { timezone = 'utc' } = filters" src/queries/sql/getWeeklyTraffic.ts
```

Expected output should show line 17. If this returns nothing, the fix was lost during a merge.

## Building

```bash
docker build -t ghcr.io/russell-davis/umami:latest .

# Push to registry
docker push ghcr.io/russell-davis/umami:latest
```

## Merging Upstream Updates

```bash
# Add upstream remote (one-time)
git remote add upstream https://github.com/umami-software/umami.git

# Fetch and merge
git fetch upstream
git merge upstream/master

# Verify fix survived the merge
grep -n "const { timezone = 'utc' } = filters" src/queries/sql/getWeeklyTraffic.ts

# If fix is missing, reapply it at line 17 of src/queries/sql/getWeeklyTraffic.ts
```

## Deployment

Update homelab compose file (`~/Work/homelab/umami.yml`):

```yaml
services:
  umami:
    image: ghcr.io/russell-davis/umami:latest
```

Deploy: `docker --context devtop compose -f umami.yml up -d`
