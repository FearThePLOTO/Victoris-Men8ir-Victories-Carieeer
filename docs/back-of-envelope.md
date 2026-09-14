# Carieeer - Back of envelope

Diagram: ../diagrams/05-back-of-envelope.excalidraw

Targets from our NFR: 1,000,000 users at a time, 400ms search, 100ms profile-details, 200ms CRUD, 99% up.

## Assumptions

- 1,000,000 registered users, about 200,000 active on a busy day
- 50,000 open positions
- 500,000 job searches per day, 100,000 profile reads per day
- 20,000 applies per day
- Sizes: profile 8KB, position 6KB, application 3KB, notification 1KB, resume 300KB in storage
- Reads far outnumber writes, about 90 to 10

## Requests per second

```
job search:     500,000 / 86400 = about 6 per sec avg, about 20 peak
profile reads:  100,000 / 86400 = about 1 per sec avg, about 5 peak
job details:    200,000 / 86400 = about 2 per sec avg, about 10 peak
applies:         20,000 / 86400 = under 1 per sec avg, about 2 peak
```

Peak edge is under 50 requests per sec. Two small API servers plus Redis handle this with room to spare. Latency is the hard part.

## Storage

```
profiles:      1,000,000 x 8KB = 8 GB
positions:        50,000 x 6KB = 0.3 GB
applications:    5,000,000 x 3KB = 15 GB over a few years
notifications kept 90 days, small
resumes in storage: 500,000 x 300KB = 150 GB
DB total well under 50 GB active. Fits on one Postgres node with replica.
Redis: profiles plus job details plus matching lists, about 5 GB. One Redis node is enough.
```

## Bandwidth

```
search: 500,000 x 20KB = 10 GB per day
details and lists: similar size
push: small
```

## What this means

1. Cache profile and job details to hit 100ms and 200ms. DB alone is close but Redis gives margin.
2. Search index stays small at 50k open positions. One worker keeps it fresh.
3. Matching batch scores only active users with overlapping skills. Never all users times all jobs.
4. First bottleneck will be search query shape, second will be Redis evictions if matching lists grow. Both are fixed with indexes and TTL, not more servers.
