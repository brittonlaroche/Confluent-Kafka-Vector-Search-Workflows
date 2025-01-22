# Step 3.  Workflows

Workflows are often a second thought on most Generative AI applications.  Its critical to capture all the different possibilities and places that a GenAI app will need to get data.  Not all of it comes from the Vector Database, and often there are better, easier more effective and effcient ways to get things done. 
   
![Workflows General Architecture](/files/img/workflowsGeneralPattern2.png)  

## 4 Steps to Building a GenAI Application
There are 4 steps to building a GenAI Application and I have included a github for each step.    
The githubs (some still work in progress) are indexed here:   
Step 1. Data Augmenatation: [Confluent-Kafka-Vector-Encoding](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Encoding)   
Step 2. Inference: [Confluent-Kafka-Vector-Search-Prompt-Inference](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Search-Prompt-Inference)   
Step 3. Workflows: [Confluent-Kafka-Vector-Search-Workflows](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Search-Workflows)   
Step 4. Post Processing: [Confluent-Kafka-Vector-Search-Post-Processing](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Search-Post-Processing)   
   
## Workflows Github Description
This github explores the third step in Building a RAG Enabaled Gen AI application.  Workflows is where we decide what to do with the user question. Whats the most efficent cost effective way to get the right data? We need a reasonng agent to look at the question.  There are a few Steps to follow:   

   1. Obtain the user content (question or statement) from the GenAI app in a topic through a Kafka Consumer  
   2. Call a reasoning agent to determine what datasources are required and what is the most efficient way to obtain the data.   
   3. Place the reasoning agent reponse with the datasource type with the users orginal question, and a refined set of queries into a new topic "user_questions_processed".
   4. Process the data in the "user_questions_processed" topic for each different datasource (vector database, kafka topic, ODS, etc..)
   5. After obtaining the data from each data source place it in the "user_prompts" topic
   6. Prompt the LLM with the the user's prompts through the Model function and place the response in the LLM Response topic
   7. Place the final repsonse in a topic after post processing to be consumed by the GenAI app (We will skip this step as step 6 will be enough for work flows)  

This github is a continuation of a previous github that populated a vector database.  Be sure to check it out as we are using the data generated in step 1 for vector searches in this github example. [Confluent-Kafka-Vector-Search-Prompt-Inference](https://github.com/brittonlaroche/Confluent-Kafka-Vector-Search-Prompt-Inference)   
   
Workflows are a critical part to solving a users question by getting the right answers from the right places.  Maybe we need to call the LLM maybe we don't, maybe we need to do a vector search maybe we do not.  Maybe we can save on cost if we route the questions more effectively or cache the results to common questions.  Focusing on this retail example lets consider the following questions.

"What time does the store on Stonebrook Pkwy open tomorrow?"   
"How many reward points do I have?"   
"Show me your cheapest mens formal dress shirts."   
"I need to buy some formal dress shoes now, where can I go?"   
   
In some cases we need to get information from the product vector database.  In some cases we need to get user profile data. In some cases we need to get information on the physical location and hours of operation of retail store.  Some times we need to combine all three types of data into a vector search.
   
### Implementation Detail
![Workflows Genreral Architecture](/files/img/workflowsImplementation.png)  

Lets consider the question "I need to buy some formal dress shoes in my size, where can I go right now thats close to my address?" This question needs the shoe size from the user_profile topic or ODS. It needs the store hours to compare to the current time, and its needs the store location.  Finally it needs a vector search against the product collection based on the user shoe size and closest store id.

Lets take a look at our user profile data.

```json
{
   "email": "bob@example.com",
   "address": "1966 Natural Bridge Dr, Frisco TX 75036",
   "first_name": "Bob",
   "last_name": "Miko",
   "sex": "Male",
   "shirt_size": "Large",
   "shoe_size": "Medium",
   "pants_size": "Medium",
   "hat_size": "Large",
   "age_group":"Adult",
   "reward_points": 38790
}
```

Lets take a look at our store operations data:  
   
```json
 {
    "store_id": 1,
    "address": {
     "street": "15220 Montfort Dr",
     "city": "Dallas",
     "state": "TX",
     "zip": "75248"
   },
   "hoursOfOperation": [
     {
       "day": "Monday",
       "open": "06:00",
       "close": "23:00"
     },
     {
       "day": "Tuesday",
       "open": "06:00",
       "close": "23:00"
     },
     {
       "day": "Wednesday",
       "open": "06:00",
       "close": "23:00"
     },
     {
       "day": "Thursday",
       "open": "06:00",
       "close": "23:00"
     },
     {
       "day": "Friday",
       "open": "06:00",
       "close": "23:00"
     },
     {
       "day": "Saturday",
       "open": "06:00",
       "close": "23:00"
     },
     {
       "day": "Sunday",
       "open": "06:00",
       "close": "23:00"
     }
   ]
 }
```

Sending all the user_profile data and all of the store_operations data to the LLM for each user question would not be advisable.  It would eat up all our tokens and context very quickly.  What we need to do is send the schema and maybe an example of a single store to the LLM and ask it to determine the type of data search we need, User, Store, or Product Vector and what fields we need based on the user question. For example it might respond with the following recommednations.

```json
{"type": "user_profile", "fields": ["shoe_size", "age group", "address"]}
{"type": "store_operations", "fields": [hoursOfOperation,address], "Value":"Saturday 1800"}
{"type": "product_vector", "search_query": "I need to buy some formal dress shoes"}
```

We would prompt the LLM with the products and stores retrieved from the LLM and the users shoe size and address. Or we could glean that information and pass it to the Vector search for example only search for medium size shoes. The more work we can do on our end before passing it to the LLM the better as it reduces cost.  We can take prompts like shoe size and matching store_id(s) and pass that into the vector search. For example we can take the vector search_query and append the results from its user_prompt. 

```json
{"type": "product_vector", "search": "I need to buy some formal dress shoes adult medium size store_ids: 37,38,42"}
```
    
When we pull back the product recomendations we have already made the vector search more efficient.  If we pass back products with an error for example the wrong size the LLM can notice and possibly remove them. 

How do we merge this data?  How do we know its all ready to query?  Again this is all part of the workflows process. For our purposes we will use Flink SQL and query the user_prompts topic based of the values in the user_questions processed topic.  This github will walk us through teh process.  Certianly there are different ways to do this, but its good to see this method as a learning exercise.

    
