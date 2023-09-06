# Collaborating with ChatGPT to generate a data warehouse - tech demo

ChatGPT, Copilot, generative AI, autonomous AI agents, AutoGPT, BabyAGI, … Did everyone suddenly forget variational autoencoders and generative adversarial networks? RNNs? LSTM? BERT? And all the acronyms that were hot just a while ago. Oh, well. Large language models are still using self-supervised learning, embeddings, and transformers under the hood. So, I guess it kind of counts.

But what is new? Where is the business value? If you didn't already read [Collaborating with ChatGPT to generate a data warehouse - how far are we?](), you can find a partial answer there.

> TLDR; Business value is at least in code autocompletion, code generation (text-to-code) and code interpretation (code-to-text), and refactoring (code-to-code).

The purpose of this blog is to offer the technical details of the experiment on how to generate a minimal data warehouse in collaboration with ChatGPT. No autonomous AI agents here, just human - AI teamwork. Let's just dive into it.

## AI makes a plan

Our goal is to generate a minimal data warehouse for an instant messaging system. Let's use [Azure’s OpenAI REST API](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference) and send a prompt using OpenAI's python library that asks ChatGPT to take the role of an expert software engineer and create a step-by-step plan to implement the data warehouse. Then the plan is to take the generated plan and feed it back to ChatGPT step-by-step. Sounds a lot like an autonomous AI agent (AutoGPT), doesn’t it? But let’s keep human-in-the-loop.

<blockquote>

**The initial prompt**

Take the role of an expert software engineer and an expert data engineer. You are really good at planning how software should be implemented. You are also an excellent python programmer who always follows the best practices, produces high quality and readable code, and takes security seriously.

Your task is to implement data warehouse for instant messenger. Operation database has users table, chats table, groups table, and message table.

```
user = {
    id: string
    name: string
    email: string
    passwordHash: string
    createdAt: date
    modifiedAt: date
}

chat = {
    id: string
    name: string
    createdAt: timestamp
    modifiedAt: timestamp
}

group = {
    id: string
    name: string
    userIds: array[string]
    chatIds: array[string]
}

message = {
    id: string
    timestamp: timestamp
    senderId: string
    chatID: string
    content: string
}
```

Please, create a step-by-step plan how to implement MySQL data warehouse from which it is each to query total amount messages in a group per week.

</blockquote>

A pretty standard prompt, if you ignore all the typos: Role, context about the task with pseudo-code, and a command that asks ChatGPT to formulate a step-by-step plan to implement a minimal data warehouse. As an afterthought, I should have asked ChatGPT how to improve my prompt as a first step! I need to add that to my prompting checklist. Anyway, let’s have a look at the response for this not-so-optimal prompt.

<blockquote>

**The response to the initial prompt**

Step 1: Design the schema for the data warehouse

The first step is to design the schema for the data warehouse. This involves creating tables for each of the entities in the operation database. The schema should include the necessary columns for each entity, as well as any additional columns that may be required for reporting purposes.

Step 2: Create the necessary tables in MySQL

Once the schema has been designed, the next step is to create the necessary tables in MySQL. This can be done using SQL commands such as CREATE TABLE.

Step 3: Extract data from the operation database

The next step is to extract data from the operation database and load it into the data warehouse. This can be done using ETL (Extract, Transform, Load) tools such as Apache NiFi or Talend.

Step 4: Create indexes on the necessary columns

To improve query performance, it is important to create indexes on the necessary columns. In this case, indexes should be created on the chatIds column in the group table and the chatID column in the message table.

Step 5: Write SQL queries to retrieve total amount of messages in a group per week

Finally, SQL queries can be written to retrieve the total amount of messages in a group per week. This can be done using the GROUP BY and DATE functions in MySQL. The query should join the group and message tables on the chatIds and chatID columns, respectively, and group the results by week and group.

</blockquote>

Pretty generic response, but looks good enough to me. To be honest, I do not believe that letting AI plan anything more complex than this will be relevant in near future. I might be wrong and I'm sure that the limit is moving to more complex things, but I believe that humans can still contribute a lot to planning - in addition to finding out what is the thing that needs a plan.

But hey, AI did a pretty good job here, didn't it? Let’s try to follow the generated plan and see how it goes.

## AI follows AI generated plan

Prompts usually work better if you ask ChatGPT to think step-by-step and if you provide it more context. So, instead of asking ChatGPT to handle the whole plan in one go, let’s feed each step of the plan one-by-one. Let’s also add more context after each step by adding the responses of previous steps to the prompt.

<blockquote>

**The prompt for Step 1: Design the schema for the data warehouse**

Take the role of an expert software engineer and an expert data engineer. You are really good at planning how software should be implemented. You are also an excellent python programmer who always follows the best practices, produces high quality and readable code, and takes security seriously.

Your task is to implement data warehouse for instant messenger. Operation database has users table, chats table, groups table, and message table.

```
user = {
    id: string
    name: string
    email: string
    passwordHash: string
    createdAt: date
    modifiedAt: date
}

chat = {
    id: string
    name: string
    createdAt: timestamp
    modifiedAt: timestamp
}

group = {
    id: string
    name: string
    userIds: array[string]
    chatIds: array[string]
}

message = {
    id: string
    timestamp: timestamp
    senderId: string
    chatID: string
    content: string
}
```

I have a step-by-step plan how to implement MySQL data warehouse from which it is each to query total amount messages in a group per week.

Step 1: Design the schema for the data warehouse

The first step is to design the schema for the data warehouse. This involves creating tables for each of the entities in the operation database. The schema should include the necessary columns for each entity, as well as any additional columns that may be required for reporting purposes.

Please, design the schema for the data warehouse.

</blockquote>

The same prompt as before, but we added the first step from the generated plan. And added the title of the step with “Please” in front of it to the end.

As a side note, “Please” is not there just to be nice, but to get a better response. In some experiments, it has been shown that generative models perform better if the prompts are friendly and encouraging. Might be something to do with the training data. People tend to be more helpful when you are nice to them. So, be nice to your AI!

<blockquote>

**The response for Step 1: Design the schema for the data warehouse**

Sure, here is the schema for the data warehouse:

```
users_dim table:
- user_id (primary key)
- name
- email
- created_at
- modified_at

chats_dim table:
- chat_id (primary key)
- name
- created_at
- modified_at

groups_dim table:
- group_id (primary key)
- name
- created_at
- modified_at

users_groups_bridge table:
- user_id (foreign key to users_dim)
- group_id (foreign key to groups_dim)

chats_groups_bridge table:
- chat_id (foreign key to chats_dim)
- group_id (foreign key to groups_dim)

messages_fact table:
- message_id (primary key)
- timestamp
- sender_id (foreign key to users_dim)
- chat_id (foreign key to chats_dim)
- content
```

Note: This schema follows the star schema design pattern, which is commonly used for data warehousing. The dimension tables (users_dim, chats_dim, groups_dim) contain descriptive attributes about the entities, while the fact table (messages_fact) contains the measures (total amount of messages). The bridge tables (users_groups_bridge, chats_groups_bridge) are used to model the many-to-many relationships between users and groups, and chats and groups.

</blockquote>

Great, a star schema! The actual SQL for the corresponding tables, please.

<blockquote>

**The prompt for Step 2: Create the necessary tables in MySQL**

Take the role of an expert software engineer and an expert data engineer. You are really good at planning how software should be implemented…

…the rest of the previous prompt plus the above response to it…

Sure, here is the schema for the data warehouse:

```
users_dim table:
- user_id (primary key)
- name
- email
- created_at
- modified_at
```

…the rest of the response and the next step…

Step 2: Create the necessary tables in MySQL

Once the schema has been designed, the next step is to create the necessary tables in MySQL. This can be done using SQL commands such as CREATE TABLE.

Please, create the necessary tables in MySQL.

</blockquote>

As you can see, this is the same thing as for the Step 1. Just add the previous prompt, its response and the next step in the plan to produce a prompt with extended contex and the next task. We should get some SQL now.

<blockquote>

**The response for Step 2: Create the necessary tables in MySQL**

Here are the SQL commands to create the necessary tables in MySQL:

```sql
CREATE TABLE users_dim (
    user_id VARCHAR(255) PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    created_at DATE,
    modified_at DATE
);

CREATE TABLE chats_dim (
    chat_id VARCHAR(255) PRIMARY KEY,
    name VARCHAR(255),
    created_at TIMESTAMP,
    modified_at TIMESTAMP
);

CREATE TABLE groups_dim (
    group_id VARCHAR(255) PRIMARY KEY,
    name VARCHAR(255),
    created_at TIMESTAMP,
    modified_at TIMESTAMP
);

CREATE TABLE users_groups_bridge (
    user_id VARCHAR(255),
    group_id VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users_dim(user_id),
    FOREIGN KEY (group_id) REFERENCES groups_dim(group_id)
);

CREATE TABLE chats_groups_bridge (
    chat_id VARCHAR(255),
    group_id VARCHAR(255),
    FOREIGN KEY (chat_id) REFERENCES chats_dim(chat_id),
    FOREIGN KEY (group_id) REFERENCES groups_dim(group_id)
);

CREATE TABLE messages_fact (
    message_id VARCHAR(255) PRIMARY KEY,
    timestamp TIMESTAMP,
    sender_id VARCHAR(255),
    chat_id VARCHAR(255),
    content TEXT,
    FOREIGN KEY (sender_id) REFERENCES users_dim(user_id),
    FOREIGN KEY (chat_id) REFERENCES chats_dim(chat_id)
);
```

Note: These SQL commands assume that the MySQL database has already been set up and is accessible. The commands should be executed in the order they are presented, as there are foreign key constraints between some of the tables.

</blockquote>

And we have some actual SQL in the response. We have inconsistent use of DATE and TIMESTAMP. Message content is probably irrelevant for us. Not perfect, but not bad! Can you spot any other issues?

Next would be Step 3: Extract data from the operation database. Let’s assume that we have automation for duplicating the operational database to the staging area of the data warehouse. We still need to transform the data from the operational schema to the generated star schema.

<blockquote>

**The prompt for Step 3: Extract data from the operation database**

Take the role of an expert software engineer and an expert data engineer. You are really good at planning how software should be implemented…

…rest of the previous prompts plus responses to them…

Step 3: Extract data from the operation database

The next step is to extract data from the operation database and load it into the data warehouse. This can be done using ETL (Extract, Transform, Load) tools such as Apache NiFi or Talend.

Assume that the data has been duplicated from the operation database to the data warehouse database. Please, write the required transformations to populate data warehouse tables.

</blockquote>

Drum roll…

<blockquote>

**The response for Step 3: Extract data from the operation database**

Transformation 1: Populate users_dim table

To populate the users_dim table, we need to extract data from the users table in the operation database and transform it into the format required by the data warehouse schema. Here is the SQL command to populate the users_dim table:

```sql
INSERT INTO users_dim (user_id, name, email, created_at, modified_at)
SELECT id, name, email, createdAt, modifiedAt
FROM users;
```

Transformation 2: Populate chats_dim table

To populate the chats_dim table, we need to extract data from the chats table in the operation database and transform it into the format required by the data warehouse schema. Here is the SQL command to populate the chats_dim table:

```sql
INSERT INTO chats_dim (chat_id, name, created_at, modified_at)
SELECT id, name, createdAt, modifiedAt
FROM chats;
```

Transformation 3: Populate groups_dim table

To populate the groups_dim table, we need to extract data from the groups table in the operation database and transform it into the format required by the data warehouse schema. Here is the SQL command to populate the groups_dim table:

```sql
INSERT INTO groups_dim (group_id, name, created_at, modified_at)
SELECT id, name, createdAt, modifiedAt
FROM groups;
```

Transformation 4: Populate users_groups_bridge table

To populate the users_groups_bridge table, we need to extract data from the groups table in the operation database and transform it into the format required by the data warehouse schema. Here is the SQL command to populate the users_groups_bridge table:

```sql
INSERT INTO users_groups_bridge (user_id, group_id)
SELECT userId, groupId
FROM groups_users;
```

Transformation 5: Populate chats_groups_bridge table

To populate the chats_groups_bridge table, we need to extract data from the groups table in the operation database and transform it into the format required by the data warehouse schema. Here is the SQL command to populate the chats_groups_bridge table:

```sql
INSERT INTO chats_groups_bridge (chat_id, group_id)
SELECT chatId, groupId
FROM groups_chats;
```

Transformation 6: Populate messages_fact table

To populate the messages_fact table, we need to extract data from the messages table in the operation database and transform it into the format required by the data warehouse schema. Here is the SQL command to populate the messages_fact table:

```sql
INSERT INTO messages_fact (message_id, timestamp, sender_id, chat_id, content)
SELECT id, timestamp, senderId, chatId, content
FROM messages;
```

Note: These SQL commands assume that the data has been duplicated from the operation database to the data warehouse database. The commands should be executed in the order they are presented, as there are foreign key constraints between some of the tables.

</blockquote>

Ok, and the indexes.

<blockquote>

**The response for Step 4: Create indexes on the necessary columns**

Here are the SQL commands to create indexes on the necessary columns:

```sql
CREATE INDEX groups_chatIds_idx ON groups_dim(chatIds);
CREATE INDEX messages_chatId_idx ON messages_fact(chat_id);
```

Note: These SQL commands assume that the necessary tables have already been created in MySQL. The commands should be executed after the tables have been created.

</blockquote>

What the first index is trying to achive? Is it for `groups_dim` or `chats_groups_bridge`. I guess it is for `chats_groups_bridge`. The second one makes sense, right? We will have to do joins for messages based on `chat_id` for sure. I'm pretty sure we would benefit from other indices as well, but let's move on to the query.

<blockquote>

**The prompt for Step 5: Write SQL queries to retrieve total amount of messages in a group per week**

…the previous prompt plus responses to them…

Step 5: Write SQL queries to retrieve total amount of messages in a group per week

Finally, SQL queries can be written to retrieve the total amount of messages in a group per week. This can be done using the GROUP BY and DATE functions in MySQL. The query should join the group and message tables on the chatIds and chatID columns, respectively, and group the results by week and group.

Please, write SQL queries to retrieve total amount of messages in a group per week.

</blockquote>

<blockquote>

**The response for Step 5: Write SQL queries to retrieve total amount of messages in a group per week**

Here is the SQL query to retrieve the total amount of messages in a group per week:

```sql
SELECT groups_dim.name AS group_name, YEARWEEK(messages_fact.timestamp) AS week, COUNT(messages_fact.message_id) AS total_messages
FROM groups_dim
JOIN chats_groups_bridge ON groups_dim.group_id = chats_groups_bridge.group_id
JOIN chats_dim ON chats_groups_bridge.chat_id = chats_dim.chat_id
JOIN messages_fact ON chats_dim.chat_id = messages_fact.chat_id
GROUP BY groups_dim.name, YEARWEEK(messages_fact.timestamp)
ORDER BY groups_dim.name, YEARWEEK(messages_fact.timestamp);
```

This query joins the groups_dim, chats_groups_bridge, chats_dim, and messages_fact tables, and groups the results by the group name and the week (using the YEARWEEK function). The COUNT function is used to count the number of messages in each group and week. The results are ordered by group name and week.

</blockquote>

What do you think? Pretty basic, but should do the trick!

## AI gives feedback to itself and acts on it

Oh no… as time goes by the query is taking longer and longer. Let’s ask AI to analyse the query.

<blockquote>

**The prompt for Step 6: Analyse the performance of the query**

…the previous prompt plus responses to them…

Step 6: Analyse the performance of the query

Please analyse the performance of the query and suggest any improvements assuming that a lot of messages are added each day.

</blockquote>

<blockquote>
**The response for Step 6: Analyse the performance of the query**

To improve the performance of the query, the following steps can be taken:

1. Partition the messages_fact table by date: Partitioning the messages_fact table by date can improve query performance by reducing the amount of data that needs to be scanned. This can be done using the PARTITION BY clause in MySQL.

2. Use a materialized view: A materialized view is a precomputed table that stores the results of a query. Using a materialized view can improve query performance by reducing the amount of computation required to generate the results. The materialized view can be refreshed periodically to ensure that the results are up-to-date.

3. Use a columnstore index: A columnstore index is a type of index that stores data by column rather than by row. This can improve query performance by reducing the amount of data that needs to be scanned. Columnstore indexes are available in MySQL 8.0.

4. Use a caching layer: A caching layer can improve query performance by storing the results of frequently executed queries in memory. This can reduce the amount of time required to execute the query.

5. Use a distributed database: If the amount of data is too large to be stored on a single machine, a distributed database can be used to improve query performance. A distributed database distributes the data across multiple machines, allowing queries to be executed in parallel.

Overall, partitioning the messages_fact table by date and using a materialized view are likely to have the biggest impact on query performance. Using a columnstore index, a caching layer, or a distributed database may also be beneficial depending on the specific requirements of the system.

</blockquote>

Right, kind of obvious isn’t it? Let’s add partitioning.

<blockquote>

**The prompt to add partitioning**

…the previous prompt plus responses to them…

Partition the messages_fact table by date: Partitioning the messages_fact table by date can improve query performance by reducing the amount of data that needs to be scanned. This can be done using the PARTITION BY clause in MySQL.

</blockquote>

<blockquote>

**The response with partitioning**

To partition the messages_fact table by date, you can use the PARTITION BY clause in MySQL. Here's an example of how you can partition the table by date:

```sql
CREATE TABLE messages_fact (
message_id VARCHAR(255) PRIMARY KEY,
timestamp TIMESTAMP,
sender_id VARCHAR(255),
chat_id VARCHAR(255),
content TEXT,
FOREIGN KEY (sender_id) REFERENCES users_dim(user_id),
FOREIGN KEY (chat_id) REFERENCES chats_dim(chat_id)
)
PARTITION BY RANGE COLUMNS(timestamp) (
PARTITION p1 VALUES LESS THAN ('2022-01-01'),
PARTITION p2 VALUES LESS THAN ('2022-02-01'),
PARTITION p3 VALUES LESS THAN ('2022-03-01'),
PARTITION p4 VALUES LESS THAN ('2022-04-01')
-- Add more partitions as needed
);
```

</blockquote>

That’s better. We usually don't want the whole history of messages per group, but the last three months or the past year. This would speed thing up. Partitioning by date is pretty standard way to speed up, but the above is a bit ugly and clunky.

Would the AI have included partitioning the first time, if we had included the information "a lot of messages are added each day" to the initial prompt? Maybe, maybe not. My gut feeling is that if you try to add too many things at once, the results are not so reliable.

## AI adapts to a new business rule

One last thing. I forgot to mention that users and chats can be added and removed in the middle of the week.

<blockquote>

**The prompt for users and chats changing groups in the middle of the week**

…the previous prompt plus responses to them…

Step 7: Users or chats can change group in a middle of the week

Users or chats can change group in a middle of the week. We need to take this into account when calculating the total amount of messages in a group per week.

Please, give me a step-by-step plan how to modify the existing SQL queries.

</blockquote>

<blockquote>

**The response with a new business rule**

Step 1: Modify the schema for the data warehouse

The first step is to modify the schema for the data warehouse to include a bridge table that tracks the group membership of users and chats. This will allow us to accurately calculate the total amount of messages in a group per week, even if users or chats change groups in the middle of the week.

…the schema changes…

Step 2: Modify the SQL query to retrieve the total amount of messages in a group per week

The next step is to modify the SQL query to retrieve the total amount of messages in a group per week to take into account users or chats changing groups in the middle of the week.

Here is the modified SQL query:

```sql
SELECT groups_dim.name AS group_name, YEARWEEK(messages_fact.timestamp) AS week, COUNT(messages_fact.message_id) AS total_messages
FROM groups_dim
JOIN chats_groups_bridge ON groups_dim.group_id = chats_groups_bridge.group_id
JOIN chats_dim ON chats_groups_bridge.chat_id = chats_dim.chat_id
JOIN users_groups_bridge ON groups_dim.group_id = users_groups_bridge.group_id
JOIN users_dim ON users_groups_bridge.user_id = users_dim.user_id
JOIN messages_fact ON chats_dim.chat_id = messages_fact.chat_id AND users_dim.user_id = messages_fact.sender_id
WHERE messages_fact.timestamp BETWEEN users_groups_bridge.start_date AND COALESCE(users_groups_bridge.end_date, NOW())
AND messages_fact.timestamp BETWEEN chats_groups_bridge.start_date AND COALESCE(chats_groups_bridge.end_date, NOW())
GROUP BY groups_dim.name, YEARWEEK(messages_fact.timestamp)
ORDER BY groups_dim.name, YEARWEEK(messages_fact.timestamp);
```

This query joins the groups_dim, chats_groups_bridge, chats_dim, users_groups_bridge, users_dim, and messages_fact tables, and filters the results based on the start_date and end_date columns in the bridge tables. The COALESCE function is used to handle cases where the end_date is NULL (i.e., the user or chat is still a member of the group). The results are grouped by the group name and the week (using the YEARWEEK function), and the COUNT function is used to count the number of messages in each group and week. The results are ordered by group name and week.

Step 3: Analyse the performance of the modified query

To improve the performance of the modified query, the same steps as in Step 6 can be taken. In addition, the following steps can be taken:

1. Use a more efficient date range query: The current query filters the results based on the start_date and end_date columns in the bridge tables using the BETWEEN operator. This can be inefficient for large datasets. A more efficient approach is to use a date range query that uses the >= and < operators.

…more tips on how to improve the query performance…

</blockquote>

Enough, isn’t it? At this point, you either think that this is pretty impressive or that this does not prove anything as it is not a real use-case… or both like I do. I'm not even trying to fix the issues here by prompting, because I believe there is a smarter way to get better results. But let's do a few more tests, before we dive into that.

## AI explains the code

How about if we face the above SQL query in the code? Of course, no comments. Let's ask ChatGPT to explain it with "in the context of an instant messaging application" as the only context.

<blockquote>

**The prompt for explaining the SQL query**

```sql
SELECT groups_dim.name AS group_name, YEARWEEK(messages_fact.timestamp) AS week, COUNT(messages_fact.message_id) AS total_messages
FROM groups_dim
JOIN chats_groups_bridge ON groups_dim.group_id = chats_groups_bridge.group_id
JOIN chats_dim ON chats_groups_bridge.chat_id = chats_dim.chat_id
JOIN users_groups_bridge ON groups_dim.group_id = users_groups_bridge.group_id
JOIN users_dim ON users_groups_bridge.user_id = users_dim.user_id
JOIN messages_fact ON chats_dim.chat_id = messages_fact.chat_id AND users_dim.user_id = messages_fact.sender_id
WHERE messages_fact.timestamp BETWEEN users_groups_bridge.start_date AND COALESCE(users_groups_bridge.end_date, NOW())
AND messages_fact.timestamp BETWEEN chats_groups_bridge.start_date AND COALESCE(chats_groups_bridge.end_date, NOW())
GROUP BY groups_dim.name, YEARWEEK(messages_fact.timestamp)
ORDER BY groups_dim.name, YEARWEEK(messages_fact.timestamp);
```

Please, explain the above query in the context of an instant messaging application.

</blockquote>

<blockquote>
**The response explaining the query**

Certainly! This query is used in the context of an instant messaging application to gather statistical information about the messaging activity within different groups. Let's break down the query step by step to understand what it's doing:

SELECT Statement: The query starts with the SELECT statement, which specifies the fields to be retrieved and processed in the result set.

groups_dim.name AS group_name: This renames the name column from the groups_dim table as group_name in the result set. This column represents the name of each group.

YEARWEEK(messages_fact.timestamp) AS week: This extracts the year and week number from the timestamp column of the messages_fact table. This is used to group the messages by weeks.

COUNT(messages_fact.message_id) AS total_messages: This counts the number of messages in each group and week. It calculates the total number of messages sent within each group during a specific week.

FROM Clause: This specifies the tables from which the data will be retrieved.

groups_dim, chats_groups_bridge, chats_dim, users_groups_bridge, users_dim, messages_fact: These are various dimension and fact tables related to groups, chats, users, and messages in the instant messaging application's database.
JOIN Clauses: The query performs multiple joins to connect the relevant tables together.

JOIN chats_groups_bridge: Joins the bridge table that links groups to chats.

JOIN chats_dim: Joins the dimension table for chats.

JOIN users_groups_bridge: Joins the bridge table that links groups to users.

JOIN users_dim: Joins the dimension table for users.

JOIN messages_fact: Joins the fact table for messages.

The joins are used to establish relationships between groups, chats, users, and messages.

WHERE Clause: This filters the data based on specific conditions.

messages_fact.timestamp BETWEEN users_groups_bridge.start_date AND COALESCE(users_groups_bridge.end_date, NOW()): Ensures that only messages sent within the valid date range of user-group associations are considered.

messages_fact.timestamp BETWEEN chats_groups_bridge.start_date AND COALESCE(chats_groups_bridge.end_date, NOW()): Ensures that only messages sent within the valid date range of chat-group associations are considered.

GROUP BY Clause: This groups the data for aggregation.

GROUP BY groups_dim.name, YEARWEEK(messages_fact.timestamp): Groups the data by group name and week, so that the subsequent aggregation functions operate on these groups.
ORDER BY Clause: Specifies the order in which the results are presented.

ORDER BY groups_dim.name, YEARWEEK(messages_fact.timestamp): Orders the results first by group name and then by the year and week.

In summary, this query retrieves messaging activity statistics for each group in the instant messaging application. It calculates the total number of messages sent within each group during different weeks, taking into account user-group and chat-group associations, as well as valid date ranges for these associations. The results are grouped and ordered by group name and week for analysis.

</blockquote>

This might be helpful, especially the last summary. If I read the summary first, it becomes much faster to understand the query. Or, much harder if it hallucinates something. Here is definitely some potential, but we need more hands-on experience on this.

## AI refactors

The SQL query is pretty ugly, isn't it? Let's ask ChatGPT to refactor it and use CTEs.

<blockquote>

**Prompt for refactoring the SQL query**

```sql
SELECT groups_dim.name AS group_name, YEARWEEK(messages_fact.timestamp) AS week, COUNT(messages_fact.message_id) AS total_messages
FROM groups_dim
JOIN chats_groups_bridge ON groups_dim.group_id = chats_groups_bridge.group_id
JOIN chats_dim ON chats_groups_bridge.chat_id = chats_dim.chat_id
JOIN users_groups_bridge ON groups_dim.group_id = users_groups_bridge.group_id
JOIN users_dim ON users_groups_bridge.user_id = users_dim.user_id
JOIN messages_fact ON chats_dim.chat_id = messages_fact.chat_id AND users_dim.user_id = messages_fact.sender_id
WHERE messages_fact.timestamp BETWEEN users_groups_bridge.start_date AND COALESCE(users_groups_bridge.end_date, NOW())
AND messages_fact.timestamp BETWEEN chats_groups_bridge.start_date AND COALESCE(chats_groups_bridge.end_date, NOW())
GROUP BY groups_dim.name, YEARWEEK(messages_fact.timestamp)
ORDER BY groups_dim.name, YEARWEEK(messages_fact.timestamp);
```

Please, explain the above query in the context of an instant messaging application. And then refactor the query to use SQL's CTEs.

</blockquote>

<blockquote>
**Response with the refactored SQL query**

The given SQL query is designed to retrieve information from an instant messaging application's database. It appears to be focused on aggregating messaging activity within different groups over specific weeks. Let's break down the query step by step:

1. **SELECT Clause:**

   - `groups_dim.name AS group_name`: Alias for the name of the group.
   - `YEARWEEK(messages_fact.timestamp) AS week`: This function calculates the year and week number for each message's timestamp. It's used to group messages by weeks.
   - `COUNT(messages_fact.message_id) AS total_messages`: Counts the number of messages in each group-week combination.

2. **FROM Clause:**

   - The `FROM` clause specifies the tables used in the query: `groups_dim`, `chats_groups_bridge`, `chats_dim`, `users_groups_bridge`, `users_dim`, and `messages_fact`.
   - Joins are used to connect these tables and establish relationships between them.

3. **JOIN Clauses:**

   - Joins are used to link various dimensions and facts of the data model.
   - For instance, `chats_groups_bridge` connects groups to chats, `users_groups_bridge` connects users to groups, and `messages_fact` holds the message-related information.

4. **WHERE Clause:**

   - The `WHERE` clause contains conditions that filter the data.
   - `messages_fact.timestamp BETWEEN users_groups_bridge.start_date AND COALESCE(users_groups_bridge.end_date, NOW())` ensures that the message timestamp falls within the valid period for both the user's membership in the group and the user's participation in the chat.
   - Similar conditions are applied for chats in the `chats_groups_bridge`.

5. **GROUP BY Clause:**

   - The `GROUP BY` clause groups the results by `group_name` and `week`, allowing for aggregation of messages within each group and week.

6. **ORDER BY Clause:**
   - The `ORDER BY` clause sorts the results by `group_name` and `week`.

Now, let's refactor the query using SQL's Common Table Expressions (CTEs) for better readability and maintainability:

```sql
WITH ValidUserGroups AS (
SELECT
groups_dim.group_id,
users_dim.user_id,
COALESCE(users_groups_bridge.start_date, '1900-01-01') AS user_group_start,
COALESCE(users_groups_bridge.end_date, NOW()) AS user_group_end
FROM
groups_dim
JOIN users_groups_bridge ON groups_dim.group_id = users_groups_bridge.group_id
JOIN users_dim ON users_groups_bridge.user_id = users_dim.user_id
),
ValidChatGroups AS (
SELECT
groups_dim.group_id,
chats_dim.chat_id,
COALESCE(chats_groups_bridge.start_date, '1900-01-01') AS chat_group_start,
COALESCE(chats_groups_bridge.end_date, NOW()) AS chat_group_end
FROM
groups_dim
JOIN chats_groups_bridge ON groups_dim.group_id = chats_groups_bridge.group_id
JOIN chats_dim ON chats_groups_bridge.chat_id = chats_dim.chat_id
)
SELECT
groups_dim.name AS group_name,
YEARWEEK(messages_fact.timestamp) AS week,
COUNT(messages_fact.message_id) AS total_messages
FROM
groups_dim
JOIN ValidUserGroups ON groups_dim.group_id = ValidUserGroups.group_id
JOIN ValidChatGroups ON groups_dim.group_id = ValidChatGroups.group_id
JOIN messages_fact ON ValidChatGroups.chat_id = messages_fact.chat_id AND ValidUserGroups.user_id = messages_fact.sender_id
WHERE
messages_fact.timestamp BETWEEN ValidUserGroups.user_group_start AND ValidUserGroups.user_group_end
AND messages_fact.timestamp BETWEEN ValidChatGroups.chat_group_start AND ValidChatGroups.chat_group_end
GROUP BY
groups_dim.name,
YEARWEEK(messages_fact.timestamp)
ORDER BY
groups_dim.name,
YEARWEEK(messages_fact.timestamp);
```

In this refactored version, I've used two CTEs, `ValidUserGroups` and `ValidChatGroups`, to simplify and organize the logic for handling valid user-group and chat-group relationships. This makes the main query cleaner and easier to understand.

</blockquote>

"...cleaner and easier to understand." Yeah, I guess it is, but the same thing as with code interpretation, definitely some potential, but we need more hands-on experience on this.

## Final thoughts

A minimal MySQL data warehouse for an instant messaging app with a single query... That's not very difficult, is it? No, it is not. But it is enough for me to believe that the next step is worth taking. Let's add some in-context learning by giving relevant examples of SQL in the similar context. That might be enough to automate some boring trivial tasks. We could go one step further and set up up a vector database of examples for semantic search and then feed the best matches to perform retrieval augmented generation (RAG). We could add style guide, and domain knowledge to adjust the examples to our specific case and write `dbt` instead of standard SQL. Maybe even some fine-tuning if in-context learning is not enough.

And suddenly we might have an AI-assisted data warehouse accellerator in our hands.

And testing! Automated unit testing to all generated code. Write test, write code, test code, fix code, rinse-and-repeat. We just need to be very careful how to set up this thing, but any improvement would be a big thing for me. I already write most of the test code with AI-assistance, but the further we can push the better.

Our goal should be to find the sweet spot where we get the maximum benefit from this amazing technology called generative AI. The 80/20 principle probably applies here. We must find the 20% of the task types that are suitable for AI, but that cover 80% of all cases or resources spent.

Ok ok, but Copilots and Duets are coming to big data engineering and analytics, why not just use them? By all means do, I will. But if you want to stay ahead of the game, you need to push the limits a bit, don't you? To learn the ropes that are constantly changing. And in the end, it is also fun!

Please, join us in the pursuit of pushing the boundaries of how business value can be captured smarter and faster!
