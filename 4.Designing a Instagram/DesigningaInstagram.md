# Design a Instagram

## <b> Goal </b>

Let's design a photo-sharing service like Instagram, where users can upload photos to share them with other users.

<em>Similar Services: Flickr, Picasa</em>

<em>Difficulty Level: Medium</em>

<b>1. What is Instagram?</b>

#

Instagram is a social networking service that enables its users to upload and share their photos and videos with other users. Instagram users can choose to share information either publicly or privately. Anything shared publicly can be seen by any other user, whereas privately shared content can only be accessed by the specified set of people. Instagram also enables its users to share through many other social networking platforms, such as Facebook, Twitter, Flickr, and Tumblr.

We plan to design a simpler version of Instagram for this design problem, where a user can share photos and follow other users. The ‘News Feed’ for each user will consist of top photos of all the people the user follows.

<b>2. Requirements and Goals of the System</b>

#

We’ll focus on the following set of requirements while designing Instagram:

<em>Functional Requirements</em>

a. Users should be able to upload/download/view photos.
b. Users can perform searches based on photo/video titles.
c. Users can follow other users.
d. The system should generate and display a user’s News Feed consisting of top photos from all the people the user follows.

<em>Non-functional Requirements</em>

a. Our service needs to be highly available.
b. The acceptable latency of the system is 200ms for News Feed generation.
c. Consistency can take a hit (in the interest of availability) if a user doesn’t see a photo for a while; it should be fine.
d. The system should be highly reliable; any uploaded photo or video should never be lost.

<b><em>Not in scope:</em></b>

Adding tags to photos, searching photos on tags, commenting on photos, tagging users to photos, who to follow, etc.

<b>3. Some Design Considerations</b>

#

The system would be read-heavy, so we will focus on building a system that can retrieve photos quickly.

a. Practically, users can upload as many photos as they like; therefore, efficient management of storage should be a crucial factor in designing this system.
b. Low latency is expected while viewing photos.
c. Data should be 100% reliable. If a user uploads a photo, the system will guarantee that it will never be lost.

<b>4. Capacity Estimation and Constraints</b>

#

• Let’s assume we have 500M total users, with 1M daily active users.
• 2M new photos every day, 23 new photos every second.
• Average photo file size => 200KB
• Total space required for 1 day of photos

2M \* 200KB => 400 GB

• Total space required for 10 years:

400GB \* 365 (days a year) \* 10 (years) ~= 1425TB

<b>5. High Level System Design</b>

#

At a high-level, we need to support two scenarios, one to upload photos and the other to view/search photos. Our service would need some <em><b>object storage</b></em> servers to store photos and some database servers to store metadata information about the photos.
