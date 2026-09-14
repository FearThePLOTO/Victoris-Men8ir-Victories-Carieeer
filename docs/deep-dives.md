# Carieeer - Deep dives

## Scaling

At high scale, a single server can bottleneck on CPU, memory, and concurrent requests. Using a load balancer allows us to distribute traffic across multiple stateless app servers.

**Trade-off**: horizontal scaling improves availability and throughput, but adds infrastructure complexity. Keeping the servers stateless also means that shared state cannot depend on a specific server.

## Database and Object Storage

At the beginning, we can use a centralized database because it is easier to maintain consistency between users, jobs, applications, skills, and other related data.

At high scale, the main problem will be the increasing number of database requests, especially reads from the job feed. Instead of immediately introducing a more complicated database architecture, we can first optimize queries and use read replicas to distribute read traffic.

The main trade-off is between consistency and scalability, because replicas can have some replication delay.

For large files such as CVs, resumes, PDFs, and images, we use object storage instead of storing them in the database. This keeps the database smaller and allows file storage to scale independently.

## Cache

The job feed will generate a large number of repeated reads. Sending all of these requests to the database will eventually make the database a bottleneck.

We can cache frequently requested data such as:

- Popular job feeds
- Target roles and their required skills
- Frequently requested job details
- AI-generated matching results

This reduces database load and improves response time.

At high scale, the main problem becomes cache invalidation. For example, if an employer updates a job, the cached version may still contain the old data.

Trade-off: we accept some level of stale data in exchange for faster responses and lower database load. We can use TTLs and invalidate important cached data when it changes.

Another problem is a cache stampede, where many requests hit the database at the same time after a popular cache entry expires. We can handle this by refreshing popular data before expiration or controlling how many requests can rebuild the same cache entry.

## Queue & Workers

AI matching can be expensive, so we should not make the employer wait for the AI model to finish before the job creation request returns.

Instead, the API can respond immediately while the matching happens in the background.
At high scale, the main problem becomes queue backlog. If jobs arrive faster than workers can process them, the queue keeps growing.
We can solve this by adding more workers during high demand and scaling them based on the queue size.

Trade-off: we gain better response time and independent scaling, but the results are no longer necessarily immediate. We also need retry handling when a worker fails.
