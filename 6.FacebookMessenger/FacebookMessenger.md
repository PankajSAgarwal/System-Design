# Design a Facebook Messenger

## <b> Goal </b>

Let's design an instant messaging service like Facebook Messenger where users can send text messages to each other through web and mobile interfaces.

<b>1. What is Facebook Messenger?</b>

#

Facebook Messenger is a software application which provides text-based instant messaging services to its users. Messenger users can chat with their Facebook friends both from cell-phones and Facebook’s website.

<b>2. Requirements and Goals of the System</b>

#

Our Messenger should meet the following requirements:

<b>Functional Requirements:</b>

1. Messenger should support one-on-one conversations between users.
2. Messenger should keep track of the online/offline statuses of its users.
3. Messenger should support the persistent storage of chat history.

<b>Non-functional Requirements:</b>

1. Users should have real-time chat experience with minimum latency.
2. Our system should be highly consistent; users should be able to see the same chat history on all their devices.
3. Messenger’s high availability is desirable; we can tolerate lower availability in the interest of consistency.

<b>Extended Requirements:</b>

• Group Chats: Messenger should support multiple people talking to each other in a group.

•Push notifications: Messenger should be able to notify users of new messages when they are offline.

<b>3. Capacity Estimation and Constraints</b>

Let’s assume that we have 500 million daily active users and on average each user sends 40 messages daily; this gives us 20 billion messages per day.

<b>Storage Estimation:</b> Let’s assume that on average a message is 100 bytes, so to store all the messages for one day we would need 2TB of storage.

<em>20 billion messages \* 100 bytes => 2 TB/day</em>

To store five years of chat history, we would need 3.6 petabytes of storage.

<em>2 TB \* 365 days \* 5 years ~= 3.6 PB</em>

Other than the chat messages, we would also need to store users’ information, messages’ metadata (ID, Timestamp, etc.). Not to mention, the above calculation doesn’t take data compression and replication into consideration.

<b>Bandwidth Estimation:</b> If our service is getting 2TB of data every day, this will give us 25MB of incoming data for each second.

<em>2 TB / 86400 sec ~= 25 MB/s</em>

Since each incoming message needs to go out to another user, we will need the same amount of bandwidth 25MB/s for both upload and download.

<b>High level estimates:</b>

<em>Total messages</em> - 20 billion per day
<em>Storage for each day</em> - 2TB
<em>Storage for 5 years</em> - 3.6PB
<em>Incomming data</em> - 25MB/s
<em>Outgoing data</em> 25MB/s

<b>4. High Level Design</b>

#

At a high-level, we will need a chat server that will be the central piece, orchestrating all the communications between users. When a user wants to send a message to another user, they will connect to the chat server and send the message to the server; the server then passes that message to the other user and also stores it in the database.
