# Instabug Task #

# System requirements #
We should create a RESTful API for a system that creates (applications → chats → messages) with the following requirements:

## Functional Requirements ##
+ We should be able to create applications.
    + Should have a unique token generated by the system.
    + chats_count: which represents the current number of chats in it.
+ We should be able to add chats by using the application token.
    + Every chat should be having a Chat_id
        + Must be unique.
        + Starting from 1.
     + When we create or delete a chat, chats_count should be updated in the application table.
+ We should be able to add messages in the specific chat, by using Chat_id and application token.
    + Every message should have a message_id
         + Must be unique.
         + Starting from 1.
    + When we create or delete a message, messages_count should be updated in the chat table.
+ We would like to do a partial search on the messages.

## non-functional requirements ##
+ We should maintain the referential integrity.
+ We should take care of race conditions.
+ We should use Elasticsearch for the partial search.
+ We should consider using a message queue to do some work in the background.
+ We should add indices in our databases to get a better query performance.
+ We may consider using a cache.
+ We should write specs for our API.
    


## Database (Models) ##
![dbSchema](https://user-images.githubusercontent.com/77211992/151088625-8aaa96b0-76b7-40ab-84e2-32c26be94d02.svg)


### Indices ###
+ there is a default index on each key/ unique column.
+ I didn't need to make any new index my self, because of the way I do my query, but I coinsidered:
    + Message model → (message_id, Application_id, Chat_id) to retrieve a message fast
        + But decided not to make this one, since our write query will be much much more than our read.
### Race condition ###
+ I used transaction and pessimistic locks too:
    + handle race conditions.
    + make sure that I updated the chats_count/ messages_count. (I believe that there is something in ActiveRecord that can handle that automatically, not sure)
### Cache ###
+ I used Redis cache: But not as I wished to do
    + I didn't implement the eviction strategy, due to lack of time + first time ever to use it.
    + I wished if I can implement LRU or LFU cache (I can implement both in C++).
    + Used it to cache the messages_count and chats_count. (didn't know what else to cache)
    + There is a bug with the chats_count_cache which is always off by one, so I'm just not using it for now but works fine for messages_count (hopefully!).
 
## Controllers ##
+ ApplicationsController: Handling everything related to the application
+ ChatsController: Handling everything related to the chat.
+ MessagesController: Handling everything related to the message.
## Routes ##
+ Application routes:
    + PUT methods:
        + `/application/create/:name` Creating a new application.
    + GET methods:
        + `/application/index` Shows all applications.
        + `/application/show/:application_token` Shows a specific application.
    + DELETE methods:
        + `/application/delete/:application_token` Delete a specific application with all of its chats and messages.
+ Chat routes:
    + PUT methods:
        + `application/:application_token/chat/create` Creating a new chat in a specific application.
    + GET methods:
        + `/application/:application_token/chat/index` Shows all chats in a specific application.
        + `/application/:application_token/chat/show/:Chat_id` Shows a specific chat using Chat_ID.
    + DELETE methods: 
        + `/application/:application_token/chat/delete/:Chat_id` Delete a specific chat with is messages.
+ Message routes:
    + PUT methods: 
        + `/application/:application_token/chat/:Chat_id/message/create/:body` Creating a new message.
    + GET methods: 
        + `/application/:application_token/chat/:Chat_id/message/index` Shows all messages for a specific chat.
        + `/application/:application_token/chat/:Chat_id/message/show/:message_id` Shows a specific message in a specific chat.
    + POST methods: 
        + `/application/:application_token/chat/:Chat_id/message/update/:message_id/:body` Updating a specific message.
    + DELETE methods: 
        + `/application/:application_token/chat/:Chat_id/message/delete/:message_id` Delete a specific message.

## Specs ##
I made specs for each controller I have, not the best-automated test cases. But I already did my best to test it properly “manually”.
## What I couldn't finish yet ##
### Elasticsearch ###
+ I believe I finally have the proper setup (I had a bug before), but I couldn't understand it properly + finish it on time.
+ I watched the first 2 lectures on their official YouTube channel and used Kibana for a little.
+ I was just following this [blog](https://iridakos.com/programming/2017/12/03/elasticsearch-and-rails-tutorial).
### Message Queue ###
+ Sadly, I couldn't finish this as well, but planning to study it for a side project inshaa Allah.
+ I believe I only know a little about message queues and THEORETICAL knowledge only.
### Modules and namespaces ###
I will start to understand namespace and modules more, so I can keep my models/controllers small and organized.
### Exception handling ###
I used the default exception, I will look more about writing APIs best practices.
