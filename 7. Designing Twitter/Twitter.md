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
