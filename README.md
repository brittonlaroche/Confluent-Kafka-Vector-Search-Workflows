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

Lets consider the question "I need to buy some formal dress shoes now, where can I go?" This question needs the shoe size from the user_profile topic or ODS. It needs the store hours to compare to the current time, and its needs the store location.  Finally it needs a vector search against the product collection based on the user shoe size and closest store id.

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



