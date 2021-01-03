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

What would our storage needs be for five years? How much storage we would need for users’ data, follows, favorites? We will leave this for the exercise.

Not all tweets will have media, let’s assume that on average every fifth tweet has a photo and every tenth has a video. Let’s also assume on average a photo is 200KB and a video is 2MB. This will lead us to have 24TB of new media every day.

_(100M/5 photos * 200KB) + (100M/10 videos * 2MB) ~= 24TB/day_

**Bandwidth Estimates** Since total ingress is 24TB per day, this would translate into 290MB/sec.

Remember that we have 28B tweet views per day. We must show the photo of every tweet (if it has a photo), but let’s assume that the users watch every 3rd video they see in their timeline. So, total egress will be:

_(28B \* 280 bytes) / 86400s of text => 93MB/s_

_\+ (28B/5 \* 200KB ) / 86400s of photos => 13GB/s_
_\+ (28B/10/3 \* 2MB ) / 86400s of Videos => 22GB/s_

_Total ~= 35GB/s_

**4. System APIs**

_💡 Once we've finalized the requirements, it's always a good idea to define the system APIs. This should explicitly state what is expected from the system._

We can have SOAP or REST APIs to expose the functionality of our service. Following could be the definition of the API for posting a new tweet:

_tweet(api_dev_key, tweet_data, tweet_location, user_location, media_ids)_

**Parameters:**

_api_dev_key_ (string): The API developer key of a registered account. This will be used to, among other things, throttle users based on their allocated quota.
_tweet_data_ (string): The text of the tweet, typically up to 140 characters.
_tweet_location_ (string): Optional location (longitude, latitude) this Tweet refers to.
_user_location_ (string): Optional location (longitude, latitude) of the user adding the tweet.
_media_ids_ (number[]): Optional list of media_ids to be associated with the Tweet. (all the media photo, video, etc. need to be uploaded separately).

**Returns:** (string)
A successful post will return the URL to access that tweet. Otherwise, an appropriate HTTP error is returned.

**5. High Level System Design #**

We need a system that can efficiently store all the new tweets,
_100M/86400s => 1150_ tweets per second and read _28B/86400s => 325K_ tweets per second. It is clear from the requirements that this will be a read-heavy system.

At a high level, we need multiple application servers to serve all these requests with load balancers in front of them for traffic distributions. On the backend, we need an efficient database that can store all the new tweets and can support a huge number of reads. We also need some file storage to store photos and videos.

_Refer to HLD diagram_

Although our expected daily write load is 100 million and read load is 28 billion tweets. This means on average our system will receive around 1160 new tweets and 325K read requests per second. This traffic will be distributed unevenly throughout the day, though, at peak time we should expect at least a few thousand write requests and around 1M read requests per second. We should keep this in mind while designing the architecture of our system.

**6. Database Schema #**

We need to store data about users, their tweets, their favorite tweets, and people they follow.

_Refer to schema diagram_

For choosing between SQL and NoSQL databases to store the above schema, please see ‘Database schema’ under Designing Instagram.

**7. Data Sharding #**

Since we have a huge number of new tweets every day and our read load is extremely high too, we need to distribute our data onto multiple machines such that we can read/write it efficiently. We have many options to shard our data; let’s go through them one by one:

**Sharding based on UserID:** We can try storing all the data of a user on one server. While storing, we can pass the UserID to our hash function that will map the user to a database server where we will store all of the user’s tweets, favorites, follows, etc. While querying for tweets/follows/favorites of a user, we can ask our hash function where can we find the data of a user and then read it from there. This approach has a couple of issues:

What if a user becomes hot? There could be a lot of queries on the server holding the user. This high load will affect the performance of our service.
Over time some users can end up storing a lot of tweets or having a lot of follows compared to others. Maintaining a uniform distribution of growing user data is quite difficult.
To recover from these situations either we have to repartition/redistribute our data or use consistent hashing.

**Sharding based on TweetID:** Our hash function will map each TweetID to a random server where we will store that Tweet. To search for tweets, we have to query all servers, and each server will return a set of tweets. A centralized server will aggregate these results to return them to the user. Let’s look into timeline generation example; here are the number of steps our system has to perform to generate a user’s timeline:

1. Our application (app) server will find all the people the user follows.
   App server will send the query to all database servers to find tweets from these people.
2. Each database server will find the tweets for each user, sort them by recency and return the top tweets.
3. App server will merge all the results and sort them again to return the top results to the user.
4. This approach solves the problem of hot users, but, in contrast to sharding by UserID, we have to query all database partitions to find tweets of a user, which can result in higher latencies.

We can further improve our performance by introducing cache to store hot tweets in front of the database servers.

**Sharding based on Tweet creation time:** Storing tweets based on creation time will give us the advantage of fetching all the top tweets quickly and we only have to query a very small set of servers. The problem here is that the traffic load will not be distributed, e.g., while writing, all new tweets will be going to one server and the remaining servers will be sitting idle. Similarly, while reading, the server holding the latest data will have a very high load as compared to servers holding old data.

**What if we can combine sharding by TweetID and Tweet creation time?** If we don’t store tweet creation time separately and use TweetID to reflect that, we can get benefits of both the approaches. This way it will be quite quick to find the latest Tweets. For this, we must make each TweetID universally unique in our system and each TweetID should contain a timestamp too.

We can use epoch time for this. Let’s say our TweetID will have two parts: the first part will be representing epoch seconds and the second part will be an auto-incrementing sequence. So, to make a new TweetID, we can take the current epoch time and append an auto-incrementing number to it. We can figure out the shard number from this TweetID and store it there.

What could be the size of our TweetID? Let’s say our epoch time starts today, how many bits we would need to store the number of seconds for the next 50 years?

_86400 sec/day \* 365 (days a year) \* 50 (years) => 1.6B_

We would need 31 bits to store this number. Since on average we are expecting 1150 new tweets per second, we can allocate 17 bits to store auto incremented sequence; this will make our TweetID 48 bits long. So, every second we can store (2^17 => 130K) new tweets. We can reset our auto incrementing sequence every second. For fault tolerance and better performance, we can have two database servers to generate auto-incrementing keys for us, one generating even numbered keys and the other generating odd numbered keys.

If we assume our current epoch seconds are “1483228800,” our TweetID will look like this:

1483228800 000001
1483228800 000002
1483228800 000003
1483228800 000004
…

If we make our TweetID 64bits (8 bytes) long, we can easily store tweets for the next 100 years and also store them for mili-seconds granularity.

In the above approach, we still have to query all the servers for timeline generation, but our reads (and writes) will be substantially quicker.

Since we don’t have any secondary index (on creation time) this will reduce our write latency.
While reading, we don’t need to filter on creation-time as our primary key has epoch time included in it.
