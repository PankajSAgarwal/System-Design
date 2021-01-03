# Design a Twitter

## 1. What is Twitter

Twitter is an online social networking service where users post and read short 140-character messages called "tweets." Registered users can post and read tweets, but those who are not registered can only read them. Users access Twitter through their website interface, SMS, or mobile app.

## 2.Requirements and Goals of the System

We will be designing a simpler version of Twitter with the following requirements:

**Functional Requirements**

1. Users should be able to post new tweets.
2. A user should be able to follow other users.
3. Users should be able to mark tweets as favorites.
4. The service should be able to create and display a user’s timeline consisting of top tweets from all the people the user follows.
5. Tweets can contain photos and videos.

**Non-functional Requirements**

1. Our service needs to be highly available.
2. Acceptable latency of the system is 200ms for timeline generation.
3. Consistency can take a hit (in the interest of availability); if a user doesn’t see a tweet for a while, it should be fine.

**Extended Requirements**

1. Searching for tweets.
2. Replying to a tweet.
3. Trending topics – current hot topics/searches.
4. Tagging other users.
5. Tweet Notification.
6. Who to follow? Suggestions?
7. Moments.

**3. Capacity Estimation and Constraints #**

Let’s assume we have one billion total users with 200 million daily active users (DAU). Also assume we have 100 million new tweets every day and on average each user follows 200 people.

_Total Users = 1 billion_
_DAU = 200 million_
_No of tweets every day = 100 million_
_Avg number of people each user follows = 200 people_

**How many favorites per day? If, on average, each user favorites five tweets per day we will have:**

_200M users \* 5 favourites => 1B favorites_

**How many total tweet-views will our system generate?** Let’s assume on average a user visits their timeline two times a day and visits five other people’s pages. On each page if a user sees 20 tweets, then our system will generate 28B/day total tweet-views:

_200M DAU \* ((2 + 5) \* 20 tweets) => 28B/day_

**Storage Estimates** Let’s say each tweet has 140 characters and we need two bytes to store a character without compression. Let’s assume we need 30 bytes to store metadata with each tweet (like ID, timestamp, user ID, etc.). Total storage we would need:

_100M \* (280 + 30) bytes => 30GB/day_
