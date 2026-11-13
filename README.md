[Uploading System Design Notes.md…]()
# system_design_notes# System Design Notes

A running log of my daily system design learning. Each day's notes are appended below the previous day's.

---

## Day 1: What is System Design? (Lecture 1)

**Source:** Padho with Pratyush, "The Best Course On System Design in 2026" (Lecture 1 of a 30-lecture course)

**Note on sources:** These notes are built from the video's title and description (topic list and homework). They are not a line-by-line record of what was said, so the video's own examples and phrasing may differ. Replace or extend any section after pasting the transcript.

### 1. What is system design?

System design is the process of deciding how the parts of a software system (clients, servers, databases, caches, queues, networks) are arranged and how they communicate so that the system meets its requirements at scale.

- Coding answers: "How do I implement this feature correctly?"
- System design answers: "How do I structure the whole system so it keeps working when millions of users, failures and changing requirements hit it?"
- A feature can be perfectly coded and still fail in production because the system around it was badly designed.

**Motivating example from the video's description: IRCTC vs Google Search.** Both are "just websites", yet IRCTC slows down during Tatkal booking while Google Search almost never goes down. The difference is not the programming language or the UI. It is the architecture: how traffic is spread across machines, how data is stored and cached, how failures are absorbed, and how spikes are handled.

- IRCTC at Tatkal time faces a sudden spike of many users competing for the same limited resource (seats) at the same moment. This is hard because every booking must be correct (no double-booking), so it cannot simply serve stale or approximate data.
- Google Search serves mostly read traffic that can be heavily cached, replicated and distributed, and small inconsistencies are acceptable.
- Lesson: the nature of the workload and the requirements decide the design.

### 2. System design vs coding

| Aspect | Coding | System design |
| --- | --- | --- |
| Focus | Logic inside one program | Structure across many components |
| Question | Does the code work? | Does the system work under load, failures and growth? |
| Output | Working code | Architecture, diagrams, trade-off decisions |
| Concerns | Algorithms, data structures, bugs | Scalability, availability, data flow, cost |

Good engineers need both. Coding skill alone does not tell you how WhatsApp, YouTube, Instagram or UPI are built.

### 3. HLD vs LLD

**High Level Design (HLD)** is the big-picture view.

- Major components (load balancer, app servers, databases, cache, queue, CDN)
- How they talk to each other (APIs, protocols)
- Data flow, storage choices, scaling strategy
- Usually drawn as boxes and arrows
- Example: for a URL shortener, HLD decides there is an API layer, a database for mappings, and a cache for hot links.

**Low Level Design (LLD)** is the detailed view inside a component.

- Classes, interfaces, methods, relationships
- Database schema details, design patterns, validation, edge cases
- Example: for the same URL shortener, LLD defines the `UrlService` class, the key-generation method, and the table columns.

|  | HLD | LLD |
| --- | --- | --- |
| Level | Architecture | Code structure |
| Audience | Architects, interviewers, whole team | Developers implementing a module |
| Deliverable | Component and data-flow diagrams | Class diagrams, schemas, method design |
| Question | What components and how do they connect? | How is each component built internally? |

This course is mainly about HLD (the interview focus at product companies for system design rounds), while LLD is a separate round in many companies.

### 4. Requirements: the starting point of every design

Every design starts by clarifying requirements. A design without requirements is a guess.

**Functional requirements (FRs): what the system does.**

- Features and behaviour the user can see
- Example for a messaging app: send a message, create a group, see delivery status
- Example for a URL shortener: create a short link, redirect to the original

**Non-functional requirements (NFRs): how well the system does it.**

- Quality attributes: scalability, latency, availability, reliability, consistency, durability, security, cost, maintainability
- These are what usually make a design hard and what the rest of this lecture covers.

**Key idea:** two systems can have identical functional requirements and completely different architectures because their non-functional requirements differ.

### 5. Non-functional qualities explained

#### 5.1 Scalability

The ability of a system to handle growing load (more users, more data, more requests) by adding resources, without a redesign.

- **Vertical scaling (scale up):** make one machine bigger (more CPU, RAM, disk). Simple, but has a hard ceiling, costs rise steeply, and the machine remains a single point of failure.
- **Horizontal scaling (scale out):** add more machines and split the work. Nearly unlimited growth and better fault tolerance, but adds complexity (load balancing, data partitioning, keeping state consistent).
- Scalability is not the same as raw speed. A system can be fast for 10 users and collapse at 10 million.

#### 5.2 Performance: latency and throughput

- **Latency:** time to complete one request (for example 120 ms from click to response). Lower is better.
- **Throughput:** how many requests the system handles per unit time (for example 50,000 requests per second).
- They are related but different. You can have high throughput with poor latency (batch processing) or low latency with low throughput.
- Users feel latency. Tail latency (the slowest 1% or 0.1% of requests, written p99 or p99.9) matters more than the average because slow outliers hurt real users.

#### 5.3 Availability and the "nines"

Availability is the fraction of time the system is up and able to serve requests.

`Availability = Uptime / (Uptime + Downtime)`

| Availability | Common name | Downtime per year |
| --- | --- | --- |
| 99% | two nines | about 3.65 days |
| 99.9% | three nines | about 8.76 hours |
| 99.99% | four nines | about 52.6 minutes |
| 99.999% | five nines | about 5.26 minutes |

- Each extra nine is roughly ten times harder and more expensive to achieve.
- Availability is typically achieved through redundancy: no single point of failure, replicas, failover, multiple data centers or regions.
- Not every system needs five nines. Pick the target that matches the business impact of downtime.

#### 5.4 Reliability

The system produces correct results consistently over time, even when things go wrong. Available means "it responds"; reliable means "it responds correctly and consistently". A system that is up but returns wrong data is available but not reliable.

#### 5.5 Fault tolerance

The ability to keep operating when components fail (servers crash, disks die, networks drop, a data center goes dark).

- Failures are normal at scale, not exceptional. Design assuming things will fail.
- Techniques: redundancy, replication, health checks, automatic failover, retries with backoff, graceful degradation (serve a reduced experience instead of an error).

#### 5.6 Consistency

Every user sees the same, up-to-date data.

- **Strong consistency:** after a write completes, every read sees it. Needed for money, seat booking, inventory.
- **Eventual consistency:** replicas may briefly disagree but converge over time. Acceptable for likes, view counts, feeds.
- Stronger consistency usually costs latency and availability. (This leads into the CAP theorem later in the course.)

#### 5.7 Durability

Once data is confirmed as saved, it is not lost, even after crashes or power failures.

- Achieved by writing to disk, replication across machines and regions, write-ahead logs, backups.
- Durability is different from availability: data can be safely stored (durable) while the service is temporarily down (unavailable).

#### 5.8 Security: authentication vs authorization

- **Authentication (AuthN):** who are you? Verifying identity (password, OTP, biometrics, tokens).
- **Authorization (AuthZ):** what are you allowed to do? Checking permissions after identity is known (admin vs normal user).
- Order: authenticate first, then authorize.
- Memory trick: AuthN = Name (identity), AuthZ = Zone of access (permissions).
- Security also covers encryption in transit and at rest, protecting against abuse, and safeguarding user data.

#### 5.9 Cost

Every design choice has a price: servers, storage, bandwidth, licenses, and engineering time. The best design is not the most powerful one but the one that meets requirements within budget. Over-engineering is a real failure mode.

#### 5.10 Maintainability

How easily the system can be understood, debugged, changed and operated by a team over years.

- Includes clean modular structure, monitoring and logging, deployability, documentation, and simplicity.
- Most of a system's lifetime cost is in maintaining it, not building it.

### 6. Trade-offs: why no system can be perfect

Qualities pull against each other, so you cannot maximize all of them.

- Strong consistency vs low latency and high availability
- High availability and redundancy vs cost
- Strong security checks vs speed and user convenience
- Rich features and flexibility vs simplicity and maintainability
- Fast writes vs fast reads (indexing helps reads, slows writes)

System design is the skill of choosing which qualities matter most for this particular system, relaxing the rest, and being able to justify the choice. There is rarely one correct answer, only well-reasoned and poorly-reasoned ones.

### 7. Homework answered: which quality matters most?

The video asks: for Instagram likes, a UPI payment and YouTube view counts, which quality matters most and which can be relaxed?

| System | Matters most | Can be relaxed | Why |
| --- | --- | --- | --- |
| UPI payment | Consistency, durability, reliability, security | Latency can be a little higher; cost is secondary | Money must never be lost, duplicated or wrongly debited; correctness beats speed |
| Instagram likes | Availability, low latency, scalability | Strict (immediate) consistency | A like count that is briefly off by a few is harmless; the app must stay fast and responsive |
| YouTube view counts | Scalability, availability, throughput | Exact real-time consistency | Counts are huge and updated constantly; showing slightly stale or approximate numbers is fine, they can be reconciled later |

This is my reasoning from the concepts above, so compare it with the answer given in the video.

### 8. How system design interviews are evaluated

Interviewers at product companies generally look at:

- **Requirement gathering:** do you ask clarifying questions and separate functional from non-functional requirements before drawing anything?
- **Structured approach:** requirements, estimates, high-level design, deep dives, bottlenecks.
- **Trade-off reasoning:** can you explain why you chose one option over another and what you gave up?
- **Scalability and failure thinking:** do you consider growth, bottlenecks and what happens when parts fail?
- **Communication:** can you explain your design clearly and take feedback?
- There is no single right answer. Justified decisions matter more than a memorized architecture.

### 9. Important points

1. System design is about structuring the whole system, not writing code for one feature.
2. HLD = components and data flow; LLD = classes and code-level detail.
3. Always begin with requirements, both functional and non-functional.
4. Non-functional requirements drive most architectural decisions.
5. Scale up = bigger machine; scale out = more machines.
6. Latency = time per request; throughput = requests per time.
7. Each extra "nine" of availability is far costlier than the last.
8. Availability (is it up?) is not reliability (is it correct?) and not durability (is saved data safe?).
9. Strong consistency for money and inventory; eventual consistency for social metrics.
10. Authentication comes before authorization.
11. Every quality has a cost, so trade-offs are unavoidable.

### 10. Points to remember

- IRCTC vs Google is the go-to example that the same kind of website can behave very differently depending on design.
- Failures are normal at scale. Design for failure, not around it.
- Match the design to the requirement: UPI, Instagram likes and YouTube views each need a different balance.
- In an interview, say your trade-offs out loud ("I choose eventual consistency here because...").
- Don't over-engineer. Five nines is wasted on a system that can tolerate an hour of downtime.
- Memory trick for the qualities: **S**calability, **P**erformance, **A**vailability, **R**eliability, **C**onsistency, **D**urability, **S**ecurity, **C**ost, **M**aintainability.

### 11. Day 1 revision

**Quick summary (read this first when revising)** System design means deciding how components fit together so the system meets its requirements at scale. Start with functional (what) and non-functional (how well) requirements. HLD draws the big picture; LLD designs the insides. The key qualities are scalability, performance (latency, throughput), availability (nines), reliability, fault tolerance, consistency, durability, security (authN vs authZ), cost and maintainability. They conflict, so design is about trade-offs chosen to fit the use case.

**One-line definitions**

- Scalability: handle growth by adding resources.
- Latency: time for one request.
- Throughput: requests handled per second.
- Availability: percentage of time the system is up.
- Reliability: correct behavior over time.
- Fault tolerance: keeps working when parts fail.
- Consistency: everyone sees the same data.
- Durability: saved data is never lost.
- Authentication: verify who you are.
- Authorization: decide what you may do.

**Self-test questions**

1. How is system design different from coding?
2. What is the difference between HLD and LLD? Give an example of each.
3. What are functional and non-functional requirements? Give two of each for a chat app.
4. Compare vertical and horizontal scaling. Which has a hard limit?
5. How much downtime per year does 99.9% availability allow?
6. Can a system be available but not reliable? Explain.
7. Why is eventual consistency fine for likes but not for UPI?
8. What is the difference between authentication and authorization?
9. Why can't a system maximize every quality at once? Give two conflicting pairs.
10. Why does IRCTC struggle at Tatkal time while Google Search rarely fails?

**Answers to check yourself (short)**

1. Coding builds one working piece; system design structures many pieces to work under load and failure.
2. HLD: components and data flow (API, DB, cache). LLD: classes, methods and schema inside a component.
3. FR: send message, create group. NFR: low latency, high availability.
4. Vertical = bigger machine, hits a hardware ceiling; horizontal = more machines, scales further.
5. About 8.76 hours.
6. Yes. It can respond (available) while returning wrong data (not reliable).
7. Likes can briefly differ without harm; money must always be exact.
8. AuthN verifies identity; AuthZ checks permissions.
9. Qualities trade off, for example consistency vs latency, and availability vs cost.
10. Tatkal is a spike of competing writes on scarce seats that must stay correct, while Search is mostly cacheable reads that tolerate small inconsistency.

**Tomorrow:** send the Day 2 link and I will append its notes below this section.

---

## Day 2: Working of a System (Lecture 2)

**Source:** Padho with Pratyush, "Working Of A System | System Design Course Ep 2" (Lecture 2 of 30)

**Note on sources:** As with Day 1, YouTube blocked the full page, so these notes are built from the video's title, description and topic list. The explanations go deeper than the description, and where I add detail beyond it I have kept to standard, widely accepted facts. The video's own examples and wording may differ. Paste the transcript to align any section.

### 0. The big question: what happens when you open Instagram?

You tap the icon and your feed appears in about a second. Your phone does not store those photos, so where do they come from, and what happens in that one second? This lecture answers it step by step. Every later topic (load balancers, caching, CDNs, sharding, queues) is a fix for some problem inside this flow, so this is the foundation.

### 1. Client and server

- **Client:** the program or device that asks for something. Examples: your phone's Instagram app, a browser tab, a React front end.
- **Server:** a computer (or program) that listens for requests and sends back responses. It is not magic hardware. It is just a computer that is always on, always connected to the internet, and running software that waits for requests.
- "Client" and "server" are **roles, not machine types**. The same machine can be a client in one interaction and a server in another (a backend server that calls a payment service is the client of that service).
- One server usually serves many clients at once. That is what makes scale a problem.

### 2. Frontend vs backend

|  | Frontend | Backend |
| --- | --- | --- |
| Runs on | The user's device (browser or phone) | Servers |
| Job | Show the UI, take input, call the backend | Business logic, data storage, security, processing |
| Examples | HTML/CSS/JS, React, mobile app screens | Node.js, Java/Spring Boot, databases |
| Trusts user input? | Never fully (users can tamper with it) | Must validate everything |

- The frontend is **untrusted** territory because the user controls their device. All important checks (authentication, payment validation, permissions) must be enforced on the backend.
- System design is mostly about the backend, but a front-end engineer should understand it, because every slow screen or failed request involves it.

### 3. Request and response: the internet in one line

**The internet is, at its core, clients sending requests and servers sending back responses.**

- A **request** says what the client wants: the address (URL), the action (method), optional data, and identity info (such as a token).
- A **response** carries the result: a status code and a body (data).
- The common protocol is **HTTP** (secured as **HTTPS**, which encrypts the traffic).
- Common methods: GET (read), POST (create), PUT/PATCH (update), DELETE (remove).
- Common status codes: 200 OK, 201 Created, 400 bad request, 401 unauthenticated, 403 forbidden, 404 not found, 500 server error.
- HTTP is **stateless**: each request is independent, and the server does not remember the previous one unless you send something (cookie, token) that identifies you. Statelessness is what makes servers easy to scale horizontally, which you will see later.

### 4. Data centers and cloud computing

- A **data center** is a building full of servers, with power, cooling, networking and security.
- Early companies bought and ran their own servers (**on-premises**). This is expensive, slow to expand, and you pay even when idle.
- **Cloud computing** means renting computing resources (servers, storage, databases, networks) over the internet from a provider on a pay-as-you-go basis. Major providers: **AWS, Google Cloud, Microsoft Azure**.
- Benefits: no upfront hardware cost, scale up or down quickly (elasticity), servers in many regions worldwide, managed services (databases, queues, caches) so you do not run everything yourself.
- Cloud providers organize infrastructure into **regions** (a geographic area, such as Mumbai) and **availability zones** (separate data centers within a region). Spreading across zones protects against a single data-center failure, linking back to Day 1's availability and fault tolerance.
- "Cloud" is not a different thing from servers. It is someone else's data center that you rent by the hour.

### 5. IP addresses

Every device on a network needs an address so data can be sent to the right place. That address is the **IP address**.

- **IPv4:** 32 bits, written like `192.168.1.10`. About 4.3 billion possible addresses, which ran short.
- **IPv6:** 128 bits, written in hexadecimal like `2001:db8::1`. A practically unlimited space.
- **Public IP:** reachable from the internet and globally unique. Servers that serve the world need one.
- **Private IP:** used inside a local network (home, office, a company's internal network) and not routable on the public internet. Standard private ranges: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- Many private devices share one public IP using **NAT** (Network Address Translation), done by your home router. That is why every device at home appears to the internet as one address.
- Servers are usually given a stable (static) address, while home devices often get changing (dynamic) ones.
- Memory trick: public = the world can reach you; private = only your local network can.

### 6. DNS: the internet's phone book

Humans remember names (`instagram.com`); computers need IP addresses. **DNS (Domain Name System)** translates domain names into IP addresses.

**Step-by-step lookup for `www.instagram.com`:**

1. **Local caches:** the browser checks its own cache, then the operating system cache. If the IP is there, done.
2. **Recursive resolver:** if not cached, the request goes to a resolver (usually run by your ISP, or a public one like 8.8.8.8 or 1.1.1.1). The resolver does the legwork for you and has its own cache.
3. **Root server:** the resolver asks a root server, which does not know the IP but points to the servers responsible for `.com`.
4. **TLD server:** the top-level-domain server for `.com` does not know the final IP either, but points to the authoritative servers for `instagram.com`.
5. **Authoritative server:** this holds the real record and returns the IP address.
6. The resolver returns the IP to your device and **caches** it. Your browser now connects to that IP.

**TTL (Time To Live):** each DNS record says how long it may be cached (for example 300 seconds). While cached, lookups are instant. Short TTL means changes spread quickly but more lookups; long TTL means fewer lookups but slower propagation of changes.

**Common record types (good to recognize):**

- **A:** domain to IPv4 address
- **AAAA:** domain to IPv6 address
- **CNAME:** one name is an alias for another name
- **NS:** which name servers are authoritative for the domain
- **MX:** mail servers for the domain

**Key points:**

- DNS is a **hierarchy** (root, then TLD, then authoritative) combined with **heavy caching**, which is why it is both scalable and fast.
- DNS can also return different IPs for different users (for example, the nearest server), which is used for load distribution later in the course.
- DNS gives the IP only. Connecting and sending the request is a separate step.

### 7. Packets, routers and latency

- Data is not sent as one big blob. It is broken into small chunks called **packets**. Each packet carries the source and destination IP plus part of the data.
- **Routers** forward packets hop by hop toward the destination. Packets of the same message may take different paths and are **reassembled** at the other end (reliable delivery is handled by TCP, which retransmits lost packets).
- Each hop and each stretch of cable adds delay, so **distance matters**. Light in fiber travels about 200 km per millisecond, so physics alone sets a minimum:
  - A 100 km round trip costs about 1 ms at best.
  - Mumbai to New York is roughly 12,500 km, so the round trip is at least about 125 ms before any processing or router delay. Real numbers are higher.
- This is why a user in India opening an app hosted only in the US feels it is slower. The fix is to **put servers and content closer to users** (regions, CDNs), which the course covers later.
- **Latency** (from Day 1) is the total delay for a request to get its response, made up of propagation delay, hops, processing time and waiting in queues.

### 8. Latency vs bandwidth

Use the highway analogy:

- **Latency** = how long one car takes to get from A to B.
- **Bandwidth** = how many lanes the highway has, so how much traffic can pass at once.

|  | Latency | Bandwidth |
| --- | --- | --- |
| Measures | Time delay | Data capacity per second (for example 100 Mbps) |
| Unit | ms | Mbps / Gbps |
| Improved by | Shorter distance, fewer hops, caching | A bigger pipe |
| Dominates when | Many small requests (API calls, page loads) | Large transfers (video, file downloads) |

- More bandwidth does not fix high latency. A wide road does not make the trip shorter.
- Instagram's feed is a mix: many small API calls (latency-sensitive) and many images (bandwidth-sensitive).

### 9. CPU, RAM and disk

- **CPU:** executes instructions, the "brain".
- **RAM (memory):** very fast, **volatile** (data is lost when power goes off), limited and expensive. Holds what the program is actively using.
- **Disk (SSD/HDD):** much slower, **persistent** (data survives power loss), cheap and large. Holds files and databases permanently.

Approximate speed gap (orders of magnitude, worth remembering):

- RAM access: around 100 nanoseconds
- SSD read: around 100 microseconds (about 1,000 times slower than RAM)
- HDD seek: around 10 milliseconds (about 100,000 times slower than RAM)
- A round trip across a network is slower still.

**Why this matters in system design:** reading from memory is vastly faster than disk, so we keep frequently needed data in memory (**caching**, coming later) and use disk for durable storage. Day 1's durability links here: data must reach disk (or replicas) to survive a crash.

### 10. Databases: why not just use files?

A **database** is software that stores data in an organized way so it can be stored, searched, updated and protected efficiently.

You can store data in files, but at scale files fail because:

- **Slow search:** finding one record means scanning the whole file. Databases use **indexes** to find data fast.
- **Concurrency:** many users reading and writing at the same time can corrupt a file. Databases manage simultaneous access safely.
- **Transactions:** a group of operations must all succeed or all fail (a bank transfer cannot half-happen). Databases provide this (the ACID idea, covered later).
- **Integrity and structure:** databases enforce rules (unique email, required fields, relationships).
- **Durability and recovery:** crash recovery, backups, replication.
- **Security and access control:** who can read or write what.
- **Querying:** a language (such as SQL) to ask complex questions without writing custom code.

Types (SQL vs NoSQL) will be covered in later lectures. For now: the backend stores users, posts, likes and follows in a database, and the feed is built by querying it.

### 11. APIs

An **API (Application Programming Interface)** is a defined way for one piece of software to ask another for something, without knowing how it works inside.

- **Waiter analogy:** you (the client) do not walk into the kitchen. You tell the waiter (the API) what you want from a menu, and the waiter brings it back from the kitchen (the backend and database).
- An API is a **contract**: it defines the endpoints, what request to send, and what response format to expect.
- Typical web APIs are **REST over HTTP**, returning **JSON**. Example: `GET /api/feed?user=123` returns a list of posts as JSON.
- The frontend never touches the database directly. It only talks to the backend through APIs, which keeps data safe and lets the backend change internally without breaking the app.
- The backend may itself call other APIs (payment, maps, notifications).

### 12. Static vs dynamic content

|  | Static content | Dynamic content |
| --- | --- | --- |
| Definition | Same for every user and rarely changes | Generated per request or per user; changes often |
| Examples | Images, videos, logos, CSS, JS files, fonts | Personalized feed, recommendations, comments, like counts |
| Needs computation? | No, just serve the file | Yes, backend logic and database queries |
| Cacheable? | Very easily, and for long periods | Hard, often only briefly or partially |
| Serving at scale | Cheap and easy | Expensive and harder |

**Why it matters (CDN preview):** because static content is identical for everyone, copies can be placed on servers close to users around the world (a **CDN, Content Delivery Network**), which gives low latency and takes load off your main servers. Dynamic content usually has to be generated by the backend.

**Homework answered (my reasoning, check against the video):**

- Static example from YouTube: the video file chunks themselves, thumbnails, the logo, the site's JS and CSS.
- Dynamic example from YouTube: your personalized home page and recommendations, the comment section, like and subscriber counts, watch history.
- **Static is easier to serve to crores of users** because it is the same for everyone, so it can be cached once and copied to many edge locations, needs no database lookup or per-user computation, and a cached copy can answer millions of requests without touching the origin servers. Dynamic content must be computed, often with database reads, for each user.

### 13. End-to-end: what happens when you open Instagram

1. **Tap the icon.** The app (the client and frontend) starts and needs your feed, which it does not have locally.
2. **DNS lookup.** The app resolves the API domain to an IP address (checking caches first, then resolver, root, TLD and authoritative servers if needed).
3. **Connection.** Your device opens a connection to that IP and sets up secure HTTPS (TCP and TLS handshake). Data travels as packets through routers, and latency starts accumulating.
4. **Request.** The app sends an API request such as `GET /feed` with your auth token (who you are).
5. **Backend receives it.** One of many servers in a data center (in the cloud) handles it. (Later lectures: a load balancer chooses which server.)
6. **Authentication and authorization.** The backend checks the token and permissions (Day 1: AuthN then AuthZ).
7. **Data fetch.** The backend checks memory (cache) first, then queries the database for who you follow and their recent posts, and may rank them. Reading from RAM is much faster than from disk.
8. **Response.** The backend returns JSON: post text, counts, and **URLs** of the images and videos, not the photo bytes themselves.
9. **Render.** The app displays the feed structure immediately.
10. **Media loads.** The app fetches the images and videos from those URLs. These are **static content**, usually served from a **CDN** edge location close to you, which is why photos appear quickly.
11. **Result:** about a second later you see your feed.

**Answer to the opening question:** the photos do not live on your phone. They live in storage and on CDN servers, and your phone fetches them using URLs returned by an API call to a backend that read your feed data from a database.

### 14. Important points

1. Client and server are roles. A server is just an always-on computer listening for requests.
2. The internet in one line: request and response.
3. The frontend is untrusted, so enforce security and validation on the backend.
4. HTTP is stateless, which makes horizontal scaling easier.
5. Cloud means renting someone else's data centers (AWS, GCP, Azure), with elasticity and pay-as-you-go.
6. IPv4 is 32-bit (about 4.3 billion addresses), IPv6 is 128-bit. Public IPs are reachable from the internet, private ones are not. NAT shares one public IP among many private devices.
7. DNS order: cache, then resolver, then root, then TLD, then authoritative. TTL controls caching time.
8. Data travels as packets through routers. Distance sets a physical minimum on latency.
9. Latency (time) and bandwidth (capacity) are different. A bigger pipe does not reduce delay.
10. RAM is fast and volatile. Disk is slower and persistent. This gap is why caching exists.
11. Databases beat files for search speed, concurrency, transactions, integrity, security and recovery.
12. An API is a contract between client and backend. The frontend never touches the database directly.
13. Static content is the same for everyone and is easy to cache and push to CDNs. Dynamic content is computed per user and is harder to scale.
14. Feed responses return URLs of media, and the media itself comes separately, usually from a CDN.

### 15. Points to remember

- DNS gives you the IP. It does not fetch the page. Connection and request come after.
- DNS answers are cached at many layers (browser, OS, resolver), which is why DNS is fast, and why DNS changes take time to spread (TTL).
- Moving servers closer to users beats buying more bandwidth when latency is the problem.
- Anything that is the same for all users is a candidate for CDN caching.
- Never trust data from the client. Validate on the server.
- Many later topics are answers to problems seen today: slow distance (CDN, regions), slow disk (cache), too many requests for one server (load balancer, horizontal scaling), slow or overloaded database (replication, sharding).
- Interview habit: describe the full request flow, from DNS to response, before diving into one component.

### 16. Day 2 revision

**Quick summary (read this first when revising)** A client (your app) sends a request to a server (an always-on computer in a data center, often rented from a cloud provider) and gets a response. To find the server, DNS turns a domain name into an IP address through a cached hierarchy (resolver, root, TLD, authoritative). Data moves as packets through routers, and distance creates latency, which is different from bandwidth. The server uses fast but volatile RAM and slower permanent disk, and keeps data in a database rather than files. The frontend talks to the backend only through APIs. Content is static (same for all, easy to cache and serve from a CDN) or dynamic (personalized, computed per request).

**One-line definitions**

- Client: the requester.
- Server: the always-on computer that responds.
- Frontend: what runs on the user's device.
- Backend: logic and data on the servers.
- Data center: a building full of servers.
- Cloud: renting computing resources over the internet.
- IP address: a device's network address.
- Public vs private IP: reachable from the internet vs only inside a local network.
- DNS: translates domain names to IP addresses.
- TTL: how long a DNS answer may be cached.
- Packet: a small chunk of data sent over a network.
- Router: forwards packets toward their destination.
- Latency: delay for one request.
- Bandwidth: data capacity per second.
- RAM: fast, volatile memory.
- Disk: slower, persistent storage.
- Database: organized, safe, searchable storage.
- API: a contract for software to talk to software.
- Static content: same for everyone, rarely changes.
- Dynamic content: generated per request or user.
- CDN: servers near users that serve cached (mostly static) content.

**Self-test questions**

1. What is a server, really? Can one machine be both client and server?
2. Why must security checks be on the backend and not only the frontend?
3. What does "HTTP is stateless" mean, and why does it help scaling?
4. What is cloud computing and why do companies prefer it to owning servers?
5. What is the difference between IPv4 and IPv6? Between public and private IPs?
6. Walk through DNS resolution for `www.instagram.com` from the browser to the authoritative server.
7. What is TTL and what happens if it is too short or too long?
8. Why does distance cause latency? What would you do about it?
9. Latency vs bandwidth: which one does a bigger pipe improve?
10. Why is RAM faster than disk, and why do we still need disk?
11. Give three reasons a database is better than plain files.
12. What is an API and why doesn't the frontend access the database directly?
13. Give one static and one dynamic example from YouTube. Which is easier to serve to crores of users and why?
14. Explain, in order, what happens from tapping the Instagram icon to seeing photos.

**Answers to check yourself (short)**

1. A server is an always-on computer that listens and responds. Yes, client and server are roles, and one machine can play both.
2. The user controls the frontend and can tamper with it, so only backend checks are trustworthy.
3. Each request is independent and the server keeps no memory of earlier ones, so any server can handle any request, which makes adding servers easy.
4. Renting servers, storage and services over the internet. It avoids upfront cost, scales quickly, and offers many regions.
5. IPv4 is 32-bit with about 4.3 billion addresses, IPv6 is 128-bit with a vastly larger space. Public IPs are reachable from the internet, private ones only inside a local network.
6. Browser cache, OS cache, recursive resolver, root server, `.com` TLD server, authoritative server, then the IP is returned and cached.
7. TTL is how long an answer can be cached. Too short means more lookups, too long means slow propagation of changes.
8. Signals take time to travel through cable and routers. Put servers and content closer to users (regions, CDN).
9. Neither fully. Bandwidth improves with a bigger pipe, latency does not. Latency improves with shorter distance, fewer hops and caching.
10. RAM is electronic and quick but loses data without power. Disk is slower but keeps data permanently and is cheaper per byte.
11. Faster search with indexes, safe concurrent access, transactions, integrity rules, security, backup and recovery.
12. An API is a contract that lets software request services without knowing the internals. Direct database access would be unsafe and would tightly couple the app to the data layout.
13. Static: video chunks or thumbnails. Dynamic: personalized home feed or comments. Static is easier because it is identical for everyone, cacheable and servable from a CDN without computation.
14. App starts, DNS lookup, secure connection, request with token, backend authenticates, reads cache or database for the feed, returns JSON with media URLs, app renders, then media loads from a CDN.

**Tomorrow:** send the Day 3 link and I will append its notes below this section.

---

## Day 3: Measuring a System (Lecture 3)

**Source:** Padho with Pratyush, "Measuring A System | System Design Course Lecture 3" (about 35 minutes)

**Note on sources:** YouTube blocked the video content, so these notes are built from the title, description and topic list. I have expanded each topic with standard definitions and worked numbers. The WhatsApp-style sizing example below uses **my own illustrative assumptions**, so the video's exact numbers will differ. Paste the transcript to align them.

### 0. Why numbers decide system design

A college fest website and Hotstar during an India vs Pakistan match are both "websites", yet they need completely different designs. The difference is **scale**, and scale is measured in numbers: how many users, how many requests per second, how much data, how fast the response must be.

- Design decisions follow numbers. 50 requests per second can run on one server. 500,000 requests per second needs load balancers, many servers, caching and sharding.
- **Over-engineering:** building for Google scale when you have 1,000 users wastes money and adds complexity (a fest site does not need Kafka and sharding).
- **Under-engineering:** ignoring scale and crashing when real traffic arrives.
- The skill is **sizing the system correctly before building it**. Numbers tell you which components you actually need.

This builds on Day 1 (scalability, latency, throughput) and Day 2 (latency, bandwidth, RAM vs disk). Today turns those ideas into measurable quantities.

### 1. Latency vs throughput (recap, with numbers)

- **Latency:** time for one request to complete. Unit: ms. Lower is better. Example: an API call takes 80 ms.
- **Throughput:** how much work is completed per unit time. Unit: requests per second (RPS or QPS) or MB per second. Example: the server handles 10,000 requests per second.

**Analogy:** a water pipe. Latency is how long one drop takes to travel from one end to the other. Throughput is how many litres come out per second.

- They are related but independent. A system can have low latency and low throughput (a fast single worker) or high throughput and high latency (batch jobs that process millions of records but each takes minutes).
- **Little's Law (useful bonus):** `concurrent requests in the system = throughput x latency`. If a server gets 1,000 requests per second and each takes 0.2 s, about 200 requests are in flight at any moment. This tells you how many connections or threads you need.

### 2. Bandwidth, and bits vs bytes

- **Bandwidth:** the maximum data that can be transferred per second over a link (from Day 2: the number of lanes on the highway).
- **Bits vs bytes (a classic mistake):**
  - 1 byte = 8 bits.
  - Network speeds are quoted in **bits** per second (Mbps, Gbps, with a small b).
  - File and storage sizes are quoted in **bytes** (KB, MB, GB, with a capital B).
  - So **100 Mbps is about 12.5 MB per second**. Forgetting the factor of 8 gives answers that are off by 8 times.

**Size units to know (decimal approximations used for estimates):**

| Unit | Size | Power of ten |
| --- | --- | --- |
| KB | thousand bytes | 10^3 |
| MB | million bytes | 10^6 |
| GB | billion bytes | 10^9 |
| TB | trillion bytes | 10^12 |
| PB | quadrillion bytes | 10^15 |

**Indian number system:** 1 lakh = 10^5, 1 crore = 10^7, so 50 crore = 500 million = 5 x 10^8.

### 3. QPS: queries per second

**QPS (queries per second)** is the number of requests the system receives (or must handle) each second. It is the single most important sizing number.

`Average QPS = total requests per day / 86,400 seconds`

- For quick estimates round 86,400 up to **10^5 seconds per day**. This keeps the math easy.
- Useful shortcut: **1 million requests per day is about 12 QPS**.

**Average vs peak traffic**

- **Average QPS** is the total spread evenly across the day. Real traffic is never even.
- **Peak QPS** is the highest load, often during events or specific hours. Rule of thumb: **plan for 2x to 5x the average**, or more for spiky events.
- Examples of peaks: IPL and India vs Pakistan on Hotstar, Tatkal booking on IRCTC, Big Billion Days sales, New Year's Eve messages, results day.
- **You must design for peak, not average.** A system sized for the average will fail exactly when it matters most.
- Cost trade-off: provisioning for the absolute peak all the time is wasteful. That is why autoscaling, queues and caching exist (later lectures).

### 4. Read-heavy vs write-heavy systems

QPS should be split into **reads** and **writes** because they stress the system differently.

|  | Read-heavy | Write-heavy |
| --- | --- | --- |
| Examples | Instagram feed, YouTube watching, news sites, product catalog | Chat messaging, logging and analytics, IoT sensor data, ride location updates |
| Typical ratio | Reads outnumber writes, often 10:1 or 100:1 | Writes dominate or are comparable to reads |
| Main techniques | Caching, read replicas, CDN | Sharding, message queues, batching, write-optimized storage |
| Bottleneck | Read capacity and latency | Write throughput and storage growth |

- Knowing the ratio tells you where to spend effort. This is why interviews ask "is this read-heavy or write-heavy?" early.
- Many systems are mixed (WhatsApp writes a message once but it is read by recipients).

### 5. Why average latency misleads: percentiles

Averages hide bad experiences. Percentiles show them.

- **p50 (median):** 50% of requests are faster than this. The typical experience.
- **p95:** 95% of requests are faster than this. 1 in 20 requests is slower.
- **p99:** 99% are faster. 1 in 100 is slower.
- **Tail latency:** the slow end of the distribution (p95, p99, p99.9).

**Example:** 99 requests take 100 ms and 1 request takes 5 seconds. The average is about 149 ms, which looks fine, but one user in 100 waited 5 seconds. The p99 exposes this.

**Why tail latency matters a lot:** one page often makes many backend calls. If each call has a 1% chance of being slow, a page that makes 100 calls hits at least one slow call with probability `1 - 0.99^100`, about **63%**. So rare slowness per call becomes common slowness per page. Large companies therefore set targets on p99, not the average (for example, "p99 under 300 ms").

### 6. Latency numbers every engineer should know

Approximate orders of magnitude (exact values vary by hardware):

| Operation | Approx. time |
| --- | --- |
| RAM access | about 100 nanoseconds |
| SSD random read | about 100 microseconds (about 1,000x slower than RAM) |
| Round trip within the same data center | about 0.5 milliseconds |
| HDD seek | about 10 milliseconds (about 100,000x slower than RAM) |
| Round trip across continents (for example India to US) | about 100 to 200 milliseconds |

Lessons:

- **Memory is far faster than disk, and disk is far faster than going across the network or across the world.** This is the reason for caching and for putting data close to users.
- Avoid unnecessary network hops and disk reads in the hot path.
- A single cross-continent call costs more than millions of RAM accesses.

### 7. How to estimate QPS, storage and bandwidth (back-of-the-envelope method)

A repeatable recipe:

1. **Clarify scale:** daily active users (DAU) and total users.
2. **Actions per user per day** (messages sent, posts viewed, uploads).
3. **Total requests per day** = DAU x actions per user.
4. **Average QPS** = total per day / 10^5 (or 86,400).
5. **Peak QPS** = average x peak factor (2x to 5x).
6. **Storage per day** = number of items per day x size per item. Then multiply by retention period (days or years) and by **replication factor** (commonly 3 copies).
7. **Bandwidth** = QPS x size per request (separately for incoming and outgoing). Remember bits vs bytes.
8. **Servers needed** = peak QPS / QPS one server can handle, then add headroom (for example 2x) for failures and spikes.
9. **Sanity-check** the result. Is it plausible? Does it match a known benchmark?

**Tips:** state your assumptions clearly, use round numbers, write units every time, and do not chase exactness. An answer within the right order of magnitude is the goal.

### 8. Homework answered: Instagram QPS

**Question from the video:** Instagram has 50 crore daily users and each opens 20 posts a day. What is the average QPS, and what should you plan for at peak?

- DAU = 50 crore = 5 x 10^8
- Posts opened per user per day = 20
- Total post views per day = 5 x 10^8 x 20 = **10^10** (1,000 crore, or 10 billion)
- Average QPS = 10^10 / 86,400 = about **115,700**, so roughly **1.2 lakh QPS**. Using the rounded 10^5 seconds per day gives a clean **1 lakh QPS**.
- Peak at 2x to 3x the average: about **2.3 to 3.5 lakh QPS**. A cautious plan might use 3x to 5x for spikes, so **around 3.5 to 6 lakh QPS**.

**Caveats (my reasoning, check against the video):**

- This treats each post view as one request. In reality, one feed request usually returns many posts, and images and videos are separate requests served from a CDN, so the real number of backend requests differs. The method is what matters.
- These are overwhelmingly **reads**, so Instagram is a read-heavy system that relies on heavy caching and CDNs.

### 9. Full worked example: sizing a WhatsApp-like chat app

**These assumptions are mine and are for illustration. Change them and the numbers change.**

**Assumptions**

- DAU = 50 crore (5 x 10^8)
- Each user sends 40 messages per day
- Average text message = 100 bytes
- 5% of messages carry media, averaging 200 KB
- Peak = 3x average
- Data kept for 1 year, replicated 3 times

**Traffic (QPS)**

- Messages per day = 5 x 10^8 x 40 = 2 x 10^10 (20 billion)
- Average write QPS = 2 x 10^10 / 86,400 = about **2.3 lakh QPS**
- Peak write QPS = about **7 lakh QPS**
- Each message is also delivered to at least one recipient, so reads and deliveries are of a similar magnitude. This is a **write-heavy, high-volume** system.

**Storage**

- Text per day = 2 x 10^10 x 100 bytes = 2 x 10^12 bytes = **about 2 TB per day**
- Text per year = 2 TB x 365 = about **730 TB**, and with 3x replication about **2.2 PB**
- Media messages per day = 5% of 2 x 10^10 = 10^9. Media per day = 10^9 x 200 KB = 2 x 10^14 bytes = **about 200 TB per day**
- Conclusion: **media dominates storage by about 100 times**, so media needs separate object storage and a CDN, and text and metadata live in a sharded database.

**Bandwidth**

- Text ingress at average = 2.3 lakh x 100 bytes = about 23 MB/s = about **185 Mbps** (tiny)
- Media ingress at average = 200 TB / 86,400 s = about 2.3 GB/s = about **18.5 Gbps**, and peak is about 3x that
- Conclusion: bandwidth is driven by media, not text.

**Servers (illustrative)**

- If one server can handle about 10,000 QPS, the peak of 7 lakh needs about **70 servers**, and with 2x headroom about **140 servers** for the message path alone. (Real chat systems also hold long-lived connections per user, so connection count is another limit, covered later.)

### 10. From numbers to design decisions

| What the numbers show | Design response (covered in later lectures) |
| --- | --- |
| High QPS beyond one server | Load balancer and horizontal scaling |
| Read-heavy traffic | Caching, read replicas, CDN |
| Write-heavy traffic | Sharding, queues, batching |
| Data larger than one machine can hold | Sharding and partitioning |
| Large static media and global users | Object storage and CDN |
| Strict p99 latency target | Caching, nearby regions, fewer hops |
| Traffic spikes | Autoscaling, queues to absorb bursts |
| Need to survive failures | Replication and redundancy across zones |

**The key idea:** you do not choose components first. You compute the numbers first, and the numbers tell you which components you need.

### 11. Handling numbers in a system design interview

- **Ask for scale first** (users, DAU, read/write ratio). If the interviewer does not give it, state your own assumptions out loud.
- **Use round numbers** and powers of ten. Interviewers want reasoning, not a calculator.
- **Always write units** and check bits vs bytes.
- **Show average and peak** QPS, not just one.
- **Estimate storage with retention and replication.**
- **Sanity-check** the answer and say what it implies for the design.
- **Tie numbers to decisions**, for example "this is about 3 lakh read QPS, so one database is not enough and I will add a cache and read replicas."
- Do not spend too long on arithmetic. A few minutes is enough, then move on to the design.

### 12. Important points

1. Numbers decide the design. Scale is what separates a fest website from Hotstar.
2. Avoid both over-engineering and under-engineering by sizing first.
3. Latency is time per request; throughput is work per second. They are independent.
4. Bandwidth is in bits per second, storage is in bytes. 8 bits = 1 byte, so 100 Mbps is about 12.5 MB/s.
5. Average QPS = requests per day / 86,400 (round to 10^5 for estimates).
6. Always design for peak QPS, usually 2x to 5x the average.
7. Split traffic into reads and writes. Read-heavy needs caching, write-heavy needs sharding and queues.
8. Averages hide bad experiences. Use p50, p95, p99 and watch tail latency.
9. RAM is about 1,000x faster than SSD, and the network across continents is slower still.
10. Storage = items per day x size x retention x replication.
11. Media usually dominates storage and bandwidth over text.
12. Numbers lead directly to design choices: load balancer, sharding, cache, CDN.

### 13. Points to remember

- 1 lakh = 10^5, 1 crore = 10^7. Convert Indian units to powers of ten first.
- 1 million requests per day is about 12 QPS. 10^10 per day is about 1.2 lakh QPS.
- Mbps is bits, MB is bytes. Divide Mbps by 8 for MB/s.
- Tail latency matters because pages make many calls (63% of pages hit a slow call if each of 100 calls is slow 1% of the time).
- Little's Law: concurrent requests = QPS x latency.
- Do not stop at average QPS. Say the peak and what it means.
- Estimate in order: users, actions, QPS, peak, storage, bandwidth, servers.
- An estimate within the right order of magnitude is a good estimate.

### 14. Day 3 revision

**Quick summary (read this first when revising)** Every system design starts with numbers, which prevent over- and under-engineering. Latency is time per request, throughput is work per second, and bandwidth is link capacity in bits per second. QPS (requests per second) is the core sizing number: average = requests per day / 86,400, and you design for peak (2x to 5x the average). Split traffic into reads and writes, since read-heavy systems use caching and replicas while write-heavy systems use sharding and queues. Averages mislead, so use percentiles (p50, p95, p99) and watch tail latency. RAM, SSD, HDD and the network differ by orders of magnitude. To size a system: users, actions, QPS, peak, storage (with retention and replication), bandwidth, servers. Those numbers then point to load balancers, sharding, caches and CDNs.

**Formula sheet**

- Average QPS = requests per day / 86,400 (about 10^5)
- Peak QPS = average QPS x (2 to 5)
- Storage = items per day x size per item x retention days x replication factor
- Bandwidth = QPS x size per request (check bits vs bytes)
- Concurrent requests = QPS x latency (Little's Law)
- Servers = peak QPS / QPS per server, plus headroom
- Bytes to bits: x 8

**One-line definitions**

- QPS: requests per second.
- DAU: daily active users.
- Throughput: work completed per second.
- Latency: time for one request.
- Bandwidth: capacity of a link in bits per second.
- p50 / p95 / p99: the value below which 50% / 95% / 99% of requests fall.
- Tail latency: the slowest requests (the high percentiles).
- Read-heavy: far more reads than writes.
- Write-heavy: writes dominate or are comparable to reads.
- Replication factor: how many copies of data are stored.
- Peak factor: the multiplier from average to peak traffic.

**Self-test questions**

1. Why do numbers decide system design? What are over-engineering and under-engineering?
2. How are latency and throughput different? Give an analogy.
3. A link is 200 Mbps. How many MB per second is that?
4. How do you compute average QPS from requests per day? Why design for peak?
5. What does 1 million requests per day translate to in QPS?
6. Give two read-heavy and two write-heavy systems, and a technique for each kind.
7. Why can average latency be misleading? What are p50, p95 and p99?
8. If a page makes 100 backend calls and each is slow 1% of the time, how often is the page slow?
9. Rank RAM, SSD, HDD and a cross-continent network call by speed.
10. List the steps to estimate storage for a service.
11. Instagram: 50 crore DAU, 20 posts a day. Average QPS and a plan for peak?
12. In the chat example, why does media dominate storage and bandwidth, and what component does that imply?
13. How do numbers lead to the choice of load balancer, sharding and CDN?
14. How should you handle numbers in an interview?

**Answers to check yourself (short)**

1. Different scales need different architectures. Over-engineering builds far more than needed, under-engineering fails under real load.
2. Latency is time per request, throughput is requests per second. A pipe: how long a drop takes vs litres per second.
3. 200 / 8 = 25 MB/s.
4. Requests per day / 86,400. Peak is when the system is stressed, and an average-sized system fails then.
5. About 12 QPS.
6. Read-heavy: Instagram feed, YouTube (caching, replicas, CDN). Write-heavy: chat, logging (sharding, queues, batching).
7. A few very slow requests barely move the average but hurt users. p50 is the median, p95 and p99 are the thresholds under which 95% and 99% of requests finish.
8. About 63% of the time (1 minus 0.99^100).
9. RAM, then SSD, then HDD, then the cross-continent network (slowest).
10. Items per day x size, x retention, x replication factor.
11. 5 x 10^8 x 20 = 10^10 per day, about 1.2 lakh QPS average, and about 3 to 6 lakh QPS at peak.
12. Media files are about 100 times larger than text, so they need object storage and a CDN.
13. High QPS needs a load balancer and horizontal scaling, data too big for one machine needs sharding, and global static media needs a CDN.
14. State assumptions, use round numbers, write units, show average and peak, include retention and replication, sanity-check, then tie the numbers to design choices.

**Tomorrow:** send the Day 4 link and I will append its notes below this section. 
