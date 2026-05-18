# Set 34

| S.No. | Question                                                                                                                  |
| ----- | ------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [Design Instagram’s post + comment system usingMongoDB](#question-1-design-instagrams-post--comment-system-using-mongodb) |

## Question 1. Design Instagram’s post + comment system using MongoDB

Designing Instagram’s **post + comment system in MongoDB** is a classic **schema design + scalability interview question**. The key is to balance **read performance, write scalability, and document size limits (16MB)** while supporting high engagement.

### 1. Core Concept (Interview Definition)

In MongoDB, Instagram-like systems are designed using a combination of:

- **Embedded documents** (for fast reads, fewer queries)
- **Referenced documents** (for scalability, large datasets like comments/likes)
- **Indexing + pagination strategies** for performance at scale

We design based on access patterns:

- Feed reads (very frequent)
- Post detail view
- Comment loading (high volume, paginated)
- Write-heavy interactions (likes/comments)

### 2. Recommended Schema Design

#### A. Posts Collection (Main entity)

We keep **post data + lightweight metadata embedded**, but NOT full comments.

```javascript
db.posts.insertOne({
  _id: ObjectId(),
  userId: ObjectId("..."),
  imageUrl: "https://cdn.instagram.com/p1.jpg",
  caption: "Enjoying MongoDB scaling 🚀",

  likesCount: 1200,
  commentsCount: 340,

  createdAt: ISODate("2026-05-14T10:00:00Z"),

  // optional lightweight recent comments preview
  topComments: [
    {
      commentId: ObjectId("..."),
      userId: ObjectId("..."),
      text: "Amazing post!",
      createdAt: ISODate(),
    },
  ],
});
```

#### Why this works:

- Fast feed rendering (no joins)
- Avoids embedding thousands of comments
- Keeps document size small

#### B. Comments Collection (Separate, scalable design)

Comments are **high-volume → must be separate collection**

```javascript
db.comments.insertOne({
  _id: ObjectId(),
  postId: ObjectId("..."),
  userId: ObjectId("..."),
  text: "Great post!",
  parentCommentId: null, // for threaded replies
  createdAt: ISODate(),
});
```

#### Key design choice:

- `postId` = foreign key (manual reference)
- Supports **pagination + infinite scroll**

#### C. Users Collection (simplified)

```javascript
db.users.insertOne({
  _id: ObjectId(),
  username: "alice",
  profilePic: "url",
});
```

### 3. Indexing Strategy (VERY IMPORTANT for interviews)

#### Posts indexes

```javascript
db.posts.createIndex({ userId: 1, createdAt: -1 });
db.posts.createIndex({ createdAt: -1 });
```

##### Why

- Feed sorting (latest posts)
- Profile-based post retrieval

#### Comments indexes

```javascript
db.comments.createIndex({ postId: 1, createdAt: -1 });
db.comments.createIndex({ postId: 1, _id: 1 });
```

##### Why

- Fast fetch of comments per post
- Efficient pagination using `_id` or `createdAt`

### 4. Fetching Data Patterns

#### A. Get Instagram Feed (Posts)

```javascript
db.posts.find({}).sort({ createdAt: -1 }).limit(20);
```

Paginated using `createdAt` or `_id`

#### B. Get Comments for a Post (Paginated)

```javascript
db.comments
  .find({ postId: ObjectId("...") })
  .sort({ createdAt: -1 })
  .skip(0)
  .limit(20);
```

##### Better approach (scale optimization):

Use **range-based pagination (avoid skip)**

```javascript
db.comments
  .find({
    postId: ObjectId("..."),
    _id: { $lt: ObjectId("lastSeenId") },
  })
  .sort({ _id: -1 })
  .limit(20);
```

#### C. Add a Comment (Atomic update pattern)

```javascript
db.comments.insertOne({
  postId: ObjectId("..."),
  userId: ObjectId("..."),
  text: "Nice!",
  createdAt: new Date(),
});

db.posts.updateOne({ _id: ObjectId("...") }, { $inc: { commentsCount: 1 } });
```

##### Why split?

- Prevents large array growth in posts
- Keeps write operations scalable

### 5. Embedded vs Referenced Decision

| Data Type     | Strategy                                | Reason                |
| ------------- | --------------------------------------- | --------------------- |
| Post metadata | Embedded                                | Frequently accessed   |
| Comments      | Referenced                              | High volume           |
| Likes         | Usually counters or separate collection | Avoid array explosion |
| Top comments  | Embedded (limited)                      | UI optimization       |

---

### 6. Scalability Enhancements

#### A. Sharding Strategy

##### Comments collection (ideal candidate for sharding)

Shard key:

```javascript
{
  postId: 1;
}
```

Ensures comments of same post are grouped

##### Posts collection shard key:

```javascript
{ userId: 1, createdAt: -1 }
```

#### B. Caching Layer (real systems)

- Redis cache for:
  - feed posts
  - trending posts
  - top comments

#### C. Change Streams (real-time updates)

Used for:

- Live comment updates
- Notifications system

```javascript
db.comments.watch();
```

### 7. Common Pitfalls

❌ Embedding all comments inside posts → document explosion
❌ Using skip for deep pagination → slow queries
❌ No indexing on `postId` → full collection scans
❌ Not separating counters (likes/comments) → write bottlenecks

### 8. Final Architecture Summary

- **posts** → core content + counters + small embedded preview
- **comments** → separate scalable collection
- **users** → profile data
- strong indexing + pagination strategy
- optional caching + sharding for scale

### 9. Follow-up Interview Questions

#### 1. How would you implement “infinite scroll feed” efficiently in MongoDB?

Use cursor-based pagination (`createdAt` or `_id`) instead of skip/limit.

#### 2. How would you scale comments for a viral post with millions of comments?

Shard by `postId`, paginate using `_id`, cache top comments, and optionally store “hot comments” separately.
