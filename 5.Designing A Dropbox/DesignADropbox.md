# Design a Dropbox

## <b> Goal </b>

Let's design a file hosting service like Dropbox or Google Drive. Cloud file storage enables users to store their data on remote servers. Usually, these servers are maintained by cloud storage providers and made available to users over a network (typically through the Internet). Users pay for their cloud data storage on a monthly basis.

<em>Similar Services: OneDrive, Google Drive</em>

<em>Difficulty Level: Medium</em>

<b>1. Why Cloud Storage?</b>

#

Cloud file storage services have become very popular recently as they simplify the storage and exchange of digital resources among multiple devices. The shift from using single personal computers to using multiple devices with different platforms and operating systems such as smartphones and tablets each with portable access from various geographical locations at any time, is believed to be accountable for the huge popularity of cloud storage services. Following are some of the top benefits of such services:

<b>Availability:</b> The motto of cloud storage services is to have data availability anywhere, anytime. Users can access their files/photos from any device whenever and wherever they like.

<b>Reliability and Durability:</b> Another benefit of cloud storage is that it offers 100% reliability and durability of data. Cloud storage ensures that users will never lose their data by keeping multiple copies of the data stored on different geographically located servers.

<b>Scalability:</b> Users will never have to worry about getting out of storage space. With cloud storage you have unlimited storage as long as you are ready to pay for it.

If you haven’t used dropbox.com before, we would highly recommend creating an account there and uploading/editing a file and also going through the different options their service offers. This will help you a lot in understanding this chapter.

<b>2. Requirements and Goals of the System</b>

#

💡 You should always clarify requirements at the beginning of the interview. Be sure to ask questions to find the exact scope of the system that the interviewer has in mind.
What do we wish to achieve from a Cloud Storage system? Here are the top-level requirements for our system:

1. Users should be able to upload and download their files/photos from any device.
2. Users should be able to share files or folders with other users.
3. Our service should support automatic synchronization between devices, i.e., after updating a file on one device, it should get synchronized on all devices.
4. The system should support storing large files up to a GB.
5. ACID-ity is required. Atomicity, Consistency, Isolation and Durability of all file operations should be guaranteed.
6. Our system should support offline editing. Users should be able to add/delete/modify files while offline, and as soon as they come online, all their changes should be synced to the remote servers and other online devices.

<b>Extended Requirements</b>

The system should support snapshotting of the data, so that users can go back to any version of the files.

<b>3. Some Design Considerations</b>

#

• We should expect huge read and write volumes.
• Read to write ratio is expected to be nearly the same.
• Internally, files can be stored in small parts or chunks (say 4MB); this can provide a lot of benefits i.e. all failed operations shall only be retried for smaller parts of a file. If a user fails to upload a file, then only the failing chunk will be retried.
• We can reduce the amount of data exchange by transferring updated chunks only.
• By removing duplicate chunks, we can save storage space and bandwidth usage.
• Keeping a local copy of the metadata (file name, size, etc.) with the client can save us a lot of round trips to the server.
• For small changes, clients can intelligently upload the diffs instead of the whole chunk.

<b>4. Capacity Estimation and Constraints</b>

• Let’s assume that we have 500M total users, and 100M daily active users (DAU).
• Let’s assume that on average each user connects from three different devices.
• On average if a user has 200 files/photos, we will have 100 billion total files.
• Let’s assume that average file size is 100KB, this would give us ten petabytes of total storage.
100B \* 100KB => 10PB
•Let’s also assume that we will have one million active connections per minute.

Total Users <br/><br/>500 million users
Files per user \t \t 200
File size (avg) \t \t 100KB
Total Files \t \t 100 billion
Total Storage \t \t 10PB

<b>5. High Level Design</b>

#

<p>
The user will specify a folder as the workspace on their device. Any file/photo/folder placed in this folder will be uploaded to the cloud, and whenever a file is modified or deleted, it will be reflected in the same way in the cloud storage. The user can specify similar workspaces on all their devices and any modification done on one device will be propagated to all other devices to have the same view of the workspace everywhere.
</p>

<p>
At a high level, we need to store files and their metadata information like File Name, File Size, Directory, etc., and who this file is shared with. So, we need some servers that can help the clients to upload/download files to Cloud Storage and some servers that can facilitate updating metadata about files and users. We also need some mechanism to notify all clients whenever an update happens so they can synchronize their files.
</p>

<p>
As shown in the diagram below, Block servers will work with the clients to upload/download files from cloud storage and Metadata servers will keep metadata of files updated in a SQL or NoSQL database. Synchronization servers will handle the workflow of notifying all clients about different changes for synchronization.
</p>
