#sd  
https://interviewing.io/guides/system-design-interview

## Alternative 

![[System_design_topics.jpg]]

## How to approach a system design interview

- Not an engineering problem (=**when you can get a great deal of data and you’re sure there is one best solution**).- There is not a single “best” solution to a system design problem. There are no predetermined outcomes. The less code you write in a system design interview, the better. Instead, collaborate with your interviewer, try crazy stuff until design is right
- Not a coding problem: **The difference between coding and system design is the difference between retrieving and creating.** If coding is science, SD is more of an art. You are not solving a problem, instead we create a map to help someone else find the solution. Consider you are the tech lead and the interviewer is the junior engineer that will implement your design. He will have a lot of question
- Back and forth exercise with the interviewer: When in doubt, always ask for clarification

## Soft tips
- No **right** way to design a system
- Rules of thumb
	- If the interviewer interrupts, it means you are probably going off track. Ask for clarification
	- Better to cover everything broadly than to explain every thing in detail
	- For any design decision, explain why
- What to say
	- If you don't know something say it. If you know a bit, say you are not familiar but .. and then proceed with explanation
	- if you want to push back: accept explanation from interviewer and clarify your initial thoughts on why you thought it was a good idea. 
	- Handwave stuff for the sake of time: "i am going to skip the details on this for now..."
	- with cold interviewers, use statements and sentences such as "Correct me if i am wrong but i would go with". With warm interviewers, use more questions. 
	- Do not ask questions for the sake of it but check with the interviewer at each milestone


## Fundamental concepts
- [[API]]
- [[Databases (SQL vs NoSQL)]]
- [[Scalability|Scaling]]
- [[CAP Theorem]]
- [[Authentication]]
- [[Load balancer]]
- [[Cache]]
- [[Message queues]]
- [[Index|Indexing]]
- [[Replication]]
- [[Consistent hashing]]

## 3 steps framework

Inputs: Problem statement given by your interviewer.

Notes: 
- Interviewer intentionally chooses to withhold information so we need to ask questions
- if your interviewer says something that seems random, you’ll surely need to use it.
### Overview

**Functional requirements**
1. Identify the main objects and their relations.
2. What information do these objects hold? Are they mutable?
3. Think about access patterns. “Given object X, return all related objects Y.” Consider the cross product of all related objects.
4. List all the requirements you’ve identified and validate with your interviewer.

**Non functional requirements**
1. Performance: Which access patterns, if any, require good performance?
2. Availability: What’s the cost of downtime for this system?
3. Security: Is there any workflow that requires special security considerations (e.g., code execution)?

**Data**
1. **Data Types:** Start by identifying the main business objects that you need to store.
2. **API:** How are these going to be accessed?
3. **Scale:** Is the system read-heavy or write-heavy?

**Design**
1. **Data storage.** We already know from previous steps “what” we are storing. Now the question is where are we storing it?
2. **Microservices.** How do we store our data? How do we retrieve it to the API? Think of these as the middlemen between storage and the API.
### Requirements

**Outputs**: List of functional and non-functional 



#### Functional requirements

Core product features and use cases that the system needs to support

Treat the system as a black box. No thinking about design, implementation, or pretty much anything technical. The sole goal of this first step is to specify what needs to be built. Not how. Not the scale. Focus on the “what.”

##### Identify the main business objects and their relations
For ex tweets and accounts, what are their relation.
Then deep dive into each object: Does it contain media? 

##### Think about the possible access patterns for these objects
Access patterns are probably the single most influential part of design because they determine how data will be stored.

The general shape of an access pattern requirement is:
- Given object A, get all related object B (note: get all does not necessarily mean all at once)

##### Consider mutability
Always ask if objects are mutable (can objects be edited, deleted?)

#### Non Functional requirements

The most common non-functional requirements you should consider in a system design interview are:

- Performance
- Availability
- Security

Here the goal is more to say: for this type of system we don't need to focus on consistency because we don't really care about up to date data OR we don't need to focus on availability because a little of down time is ok. This part allows us to relax expectations on some parts and simplify the design

##### Performance
Performance is pretty straightforward. It’s the system’s ability to respond quickly to user requests. Better performance may come at the cost of consistency or just an overall more complex solution.

Note: It makes the most sense when we have synchronous user-facing workflows. That is, the user is expecting an immediate response from the system. In addition, we want to optimize for the synchronous workflows that are accessed the most frequently.

Take a look at the access patterns you identified in your functional requirements. Which ones are user-facing, expected to happen synchronously, and accessed frequently?

##### Availability
Availability refers to how much downtime the service can tolerate.
A good question to know if it is important is What is the cost of downtime?

Also, if consistency is very important (banks for ex or anything which requires complex [[Transaction|transactions]]), we may be willing to take a hit on availability

Tip: don't mention number of 9s so that your are not interrogated on it

##### Security

Here we don't talk about the expected baseline (HTTPS, OAuth, encryption, etc). Instead we want to focus on the components that require extra security. For ex if we execute user code, then it needs to run in isolation. 

### Data types, access patterns, and scale




**Outputs:**
- List of Data Types we need to store.  
- Access patterns for these data types.  
- Scale of the data and requests the system needs to serve.


#### What data types does the system need to store?

2 types of data we might need to store:

- **Structured data.** Think business objects, like accounts, tweets, likes.
- **Media and blobs.** Think images, videos, or any type of large binary data such as TAR or ZIP files.

#### What does the API look like?
Think of HTTPS request, even in the rare cases you need websocket. It is better to think in matter of request/response cycle

API should be easy to create from the requirements

```txt
  getTweets:
    GET /{accountId}/tweets?nextPageToken={token}
    returns: Paged list of tweets sorted by creation time desc.
    
  getFeed:
    GET /{accountId}/feed?nextPageToken={token}
    returns: Paged list of tweets for the given users feed.
```
#### What volume of requests do we need to support?

First ask yourself whether this system is read-heavy or write-heavy. 
To do so look at the API and figure out which endpoints will be called more. NOTE: if you don't know, it is fine to ask the interviewer

Note: After this step:
- we have the requirements
- we have the api spec
- we could do back of the envelope calculation (ask interviewer if it is needed or if it should be skipped)

### Design

**Outputs:** 
- Data storage.  
- Microservices.

good design is 70%+ requirements and planning. Which is why we don't dive into it

Tip: Want speed? Use a cache. Want availability? Put in some redundancy.

![[what_is_design.png]]

#### Data storage

##### Blob storage
- Did we identify media => blob storage. Note: usually you couple blob storage with CDN but it should be discussed in the microservice part
##### Database
###### Relational vs non relational
Just pick one, and make your rationale clear for why you chose it. Better if you include a drawback of making the pick you made. Always be prepared to change your mind if interviewer gives you a tip. 

Is it important for data to have structured relationship?
- Yes: SQL
- No: Do you need strong consistency? Strong ACID guarantee?
	- Yes: SQL
	- No: NoSQL
###### Entities to store
Time to zoom more on the identities we defined and use our API spec. 
Now, what data must be in each entity. What index should we use. 
Also mention that a getAll actually uses paging

Then when we have a solution, time to assess it. Is it good enough?

Follow these steps iteratively: Identify → Implement → Assess → Improve.

Assess your solution before offering to improve. For ex, we assumed a read heavy system and the current solution does not seem to satisfy the non functional requirements. I assume we'd like to optimize it. What do you think?

**stating your rationale followed by a subtle “what do you think?” or “let me know if you think I’m approaching this the wrong way” is the perfect balance between being independent but also collaborative.**

When looking to optimize performance for a read-heavy access pattern that requires several queries, consider storing the final result in a separate table and keeping it up to date.

#### Microservices

Once we have our storage layer somewhat defined, the last step is connecting our API to the storage layer. There are a few decisions that often arise at this stage:

- Caching
- Load balancing
- Queuing systems
##### Caching
Are there any access patterns that would benefit from caching the results in-memory? It is not always yes

Consider using caching when all three of these are true:
- Computing the result is costly
- Once computed, the result tends to not change very often (or at all)
- The objects we are caching are read often

Caching also had drawbacks as we can have inconsistencies with the storage layer

##### Load balancing

Load balancing helps us scale horizontally and maintain high availability. While horizontal scaling is desired in most systems, it is again advisable to consider whether it is strictly necessary given the requirements you’ve identified.

Load balancing is easier when our API servers are stateless because requests can be routed to any node. This will give us two key benefits:

1. **Horizontal scaling.** We can add more API servers to handle more load.
2. **High Availability.** Whenever we need to upgrade or restart our API servers, we can perform a rolling restart.