# 🌿 MongoDB Notes 🌿

This document will just show some points that we should keep in mind while working with [MongoDB](https://www.mongodb.com/)

# 🧠 Keep in mind rules

- Data modeling principles from relational databases will give you hard time if used in [MongoDB](https://www.mongodb.com/)
- Data that are accessed together is stored together.
- The way you model your documents should be governed by how your clients uses the data.
- Use something like [MongoDB Compass](https://www.mongodb.com/products/compass) to see your stroed data, and don't depend on ODMs(object document mappers) to see them, looking at how the data is stored and their types using [MongoDB Compass](https://www.mongodb.com/products/compass) will help you on the long term.

# 💃 How to model

- Think about how your application consumes your data, not the relations between data.
- Embedding data that are related are generally a good idea.

Asking yourself questions like

- What does my app do?
- What gets reads & writes
- what does my data look like when it's being used (how does the client uses the data)

Will help you organize your documents.

# 🩰 Modeling 1:1 relations in MongoDB

In realtional DBs 1:1 relation will look like this

> movie 1:1 director (a movie can be directed by a single director)
> a director can only direct a single movie

Movies table

| id  | title          | runtime |
| --- | -------------- | ------- |
| 1   | true detective | 400     |
| 2   | westworld      | 1200    |
| 3   | Filth          | 300     |

Directors table

| id  | name   | movie_id |
| --- | ------ | -------- |
| 1   | rowadz | 1        |
| 2   | sara   | 2        |
| 3   | koa    | 3        |

To get the directors and their movie you will use inner join to get the data

```SQL
SELECT
  *
FROM
  movies m
JOIN  directors d ON d.movie_id = m.id
```

In [MongoDB](https://www.mongodb.com/), you just nest these documents

```js
[
  {
    _id: ObjectId,
    title: 'true detective',
    runtime: 400,
    director: { name: 'rowadz', director_id: ObjectId },
  },
  {
    _id: ObjectId,
    title: 'westworld',
    runtime: 1200,
    director: { name: 'sara', director_id: ObjectId },
  },
  {
    _id: ObjectId,
    title: 'Filth',
    runtime: 300,
    director: { name: 'koa', director_id: ObjectId },
  },
]
```

# 🍈 Modeling 1:m relations in MongoDB

In realtional DBs 1:m relation will look like this

> director 1:m movie (a movie can be directed by a single director)
> a director can direct many movies

Movies table

| id  | title          | runtime | director_id |
| --- | -------------- | ------- | -------- |
| 1   | true detective | 400     | 1        |
| 2   | westworld      | 1200    | 1        |
| 3   | Filth          | 300     | 3        |

Directors table

| id  | name   |
| --- | ------ |
| 1   | rowadz |
| 2   | sara   |
| 3   | koa    |

To get the directors and their movie you will use inner join to get the data

```SQL
SELECT
  *
FROM
  movies m
JOIN  directors d ON d.id = m.director_id
```

In [MongoDB](https://www.mongodb.com/), you just nest these documents

```js
[
  {
    _id: ObjectId,
    name: 'rowadz',
    movies: [
      { title: 'true detective', runtime: 400, movie_id: ObjectId },
      { title: 'WestWrold', runtime: 1200, movie_id: ObjectId },
    ],
  },
  {
    _id: ObjectId,
    name: 'sara',
  },
  {
    _id: ObjectId,
    name: 'koa',
    movies: [{ title: 'Filth', runtime: 300, movie_id: ObjectId }],
  },
]
```

> `movie_id` in the embedded documents will help you find all the places where the same movie have been referenced

# 🌋 Modeling m:m relations in MongoDB

In realtional DB m:m relation will look like this

> user m:m rooms 
> a user can enter many rooms
> a room can contain many users

To model this in SQL we use 3 tables.

Users table

| id | name   |
|----|--------|
| 1  | rowadz |
| 2  | sara   |
| 3  | koa    |

Rooms table

| id | title        |
|----|--------------|
| 1  | my room      |
| 2  | another room |
| 3  | js room      |

and to model the m:m relation we creta a thrid table called [junction or associative table](https://en.wikipedia.org/wiki/Associative_entity#:~:text=An%20associative%20(or%20junction)%20table,to%20the%20individual%20data%20tables.)

Rooms Users table

| id | user_id | room_id | joined_date |
|----|---------|---------|-------------|
| 1  | 1       | 1       | 01/01/2991  |
| 2  | 2       | 1       | 01/01/2414  |
| 3  | 3       | 1       | 01/01/2941  |
| 4  | 1       | 2       | 05/12/2014  |
| 5  | 3       | 2       | 11/11/2014  |

Now to get related data you just do some inner/left/right joins and that's it.

In [MongoDB](https://www.mongodb.com/), you also nest these related data

```js
[
  {
    _id: ObjectId,
    name: 'rowadz',
    rooms: [
      { room_id: ObjectId(A), joined_data: Date(01 / 01 / 2991) },
      { room_id: ObjectId(B), joined_data: Date(05 / 12 / 2014) },
    ],
  },
  {
    _id: ObjectId,
    name: 'sara',
    rooms: [{ room_id: ObjectId(A), joined_data: Date(01 / 01 / 2414) }],
  },
  {
    _id: ObjectId,
    name: 'koa',
    rooms: [
      { room_id: ObjectId(A), joined_data: Date(01 / 01 / 2941) },
      { room_id: ObjectId(B), joined_data: Date(11 / 11 / 2014) },
    ],
  },
]
```
> The `room_id` will help you find all the references of a specific room

# 🥥 Nesting everything might not be the way

Let's assume that we have a collection of books, and inside each book document we store the information of each user that bought that book.

And now on AVERAGE each book might have 20 users that bought it, except 1 book that have 20 millions users that bought it, so it does not make sence that we return a list that contains 20 millions of object to the clients that consume the data, so what we do is we use the [extended reference pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-extended-reference-pattern) and we only store the top 20 users in the list of users in that book document then we store in the same document an array of ids/ObjectIds that points to the other 20 millions users.

> This list could also be stored in a different collection, and our document can just refernce the other document from another collection that points to the 20 million users.

> each document have a max size of 16MB (???you can increase that???)

> We can use things like [$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/) or [$graphLookup](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/) but doing a lot of them is a sign that your documents are not modeled correctly plus using them a lot in the same aggregation might affect the speed of getting the related data.


The above example might look something like this

```js
[
  {
    book_title: 'ALMOND coffee house',
    users: [
      { name: 'rowad', age: 23, user_id: ObjectId },
      { name: 'sara', age: 25, user_id: ObjectId },
    ],
  },
  {
    book_title: 'Harry potter',
    users: [
      { name: 'rowad', age: 23, user_id: ObjectId },
      { name: 'sara', age: 25, user_id: ObjectId },
      { name: 'koa', age: 26, user_id: ObjectId },
      { name: 'almond', age: 28, user_id: ObjectId },
      { name: 'hapi', age: 85, user_id: ObjectId },
    ],
    // 20 millions or just a ref to another document in another collection that contains this list
    all_users_ref: [ObjectId, ObjectId, ObjectId, ObjectId], 
  },
]

```

# 🧆 Some modeling patterns

- [The Document Versioning Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-document-versioning-pattern)
- [The Outlier Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-outlier-pattern)
- [The Polymorphic Pattern](https://www.mongodb.com/developer/how-to/polymorphic-pattern/)
- [The Preallocation Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-preallocation-pattern)
- [The Subset Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-subset-pattern)
- [The Approximation Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-approximation-pattern)
- [The Computed Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-computed-pattern)


# 🤺 Anti patterns
- [Massive Arrays](https://www.mongodb.com/developer/article/schema-design-anti-pattern-massive-arrays/)
- [Massive number of collections](https://www.mongodb.com/developer/article/schema-design-anti-pattern-massive-number-collections/)
- [Bloated Documents](https://www.mongodb.com/developer/article/schema-design-anti-pattern-bloated-documents/)
- [storing data that are used together in different collections/documents](https://www.mongodb.com/developer/article/schema-design-anti-pattern-separating-data/)


> [A Summary of Schema Design Anti-Patterns and How to Spot Them](https://www.mongodb.com/developer/article/schema-design-anti-pattern-summary/)