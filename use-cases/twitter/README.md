## Twitter

There are two major operations in Twitter
- Publishing a tweet: Write operation
- Checking Timeline: Read operation

Close to 5000 tweets are published per second. There are close to 300,000 requests per second for home timeline.

Also, there is a huge difference in the database operation that needs to be performed for 1 write against 1 read.

1 write operation only needs to insert a row into say `tweets` table. At max, there can be few more inserts to log tables. It's easy to achieve 5000 inserts per second for 5000 tweets per second.

1 read for a home timeline needs to fetch the latest tweets for each person the current user follows. Let's assume there are three tables:

user
- id
- user_handle

follow
- id
- follower_id: FK to user_id
- followee_id: FK to user_id

tweets
- id
- text
- user_id: FK to user_id

The currently logged in user might be following 100 users. Hence tweet of all these 100 users need to be fetched to show the timeline of current user. This will involve join between 3 tables.

    select t.text from tweets as t inner join follow as f on t.user_id=f.followee_id inner join user as u on u.id=f.follower_id where u.id=current_user.id

Join between multiple tables is expensive. Imagine doing it for 300000 times per second. This can cause significant challenges.

The solution for it could be keeping the home timelines for users ready well in advance. As write operations in this case are less demanding on the resources compared to read, hence it gives us avenue to perform further operations during write.

Thus we can introduce a cache here. This cache will keep home timelines ready for each user.
As soon as a user posts a tweet, the home timeline for each follower can be populated through a background task. Consider a user has 100 followers on an average. And we the system is processing 5000 tweets per second. Hence, every second (5000\*100) cache entries would have to be updated.
The upside is that home timeline can be read directly from the cache.
