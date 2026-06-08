---
date: '2026-06-08T21:08:04+02:00'
draft: false
title: 'Text-to-SQL: The 2026 Introduction'
featured: false
---
In today's software-driven world, data has become the backbone of countless services used by billions of people worldwide, with SQL serving as the primary language for managing it. With the vast majority of the world's structured data stored in relational databases, SQL remains the most widely used language for querying and managing information. From analysts and data scientists to software engineers, millions of professionals rely on SQL every day to access, analyze, and transform data. 

Essentially, SQL is the key that unlocks the value of data, meaning that without SQL syntax knowledge, a person would not be able to access the data. In this article, I want to introduce a tool that simplifies and democratizes access to data analysis and management by bypassing the need for the SQL knowledge: Text-to-SQL.

# What is "Text-to-SQL"
Text-to-SQL is a process of transforming natural language problems in the database field into structured query languages ​​that can be executed in relational databases. This technology enables complex data analysis and management without the knowledge of SQL syntax. Recently, the adoption of Large Language Models (LLMs) into business workflows spawned a new use case scenario: integrating your own database and data analysis into a seamless conversation with an AI chatbot, essentially creating an automated subject matter expert (SME) AI assistant.

# History
Text-to-SQL technology originated in the 1970s with rule-based systems relying on hard-coded semantic grammars. The deep learning boom of the late 2010s shifted the focus toward machine translation, utilizing Seq2Seq architectures and LSTMs. By 2020, relation-aware graph networks have emerged to map complex schema hierarchies. Today, state-of-the-art systems utilize Large Language Models integrated into agentic pipelines, eliminating the historical need for manual schema mapping.

# The challenges
The challenge in generating correct SQL from a natural language prompt using LLMs basically involves the LLM understanding the database. Although that may sound like a straightforward task, there are many difficulties to tackle.

## Domain-specific knowledge
Production databases are not made with the intention of an LLM to understand them. They are often full of abbreviated column names, various data types and irregular values, so finding relevant data could pose a challenge even to a specialized data analyst without the necessary domain-specific knowledge. This introduces two approaches: building a Text-to-SQL solution for databases with available domain knowledge (good documentation or evidence), and building one that assumes no domain knowledge will be supplied.

Assuming the latter, the LLM must, for example, figure out that "building street address" needs to be mapped to columns STREET_NUMBER, STREET_NUMBER_SUFFIX, STREET_NAME, STREET_SUFFIX, and ADDRESS_PURPOSE without any documentation to guide it. Another example could be that a company has a database with their clients containing a column "is_prem" and values of either 0 or 1. The LLM must figure out this means if the person has a subscription or not.

## Non Subject Matter Expert user base
If the target user has to know SQL or have the domain-specific knowledge of the database in order to use a Text-to-SQL tool, it is, in my opinion, completely missing the point, simply put. The assumption should be that the average user of this tool is not a SME, so for questions like "Give me the names of the top 5 performing soccer players", the LLM must have the domain-specific knowledge in order to map the request correctly to the database schema (e.g. soccer = table named football; top performing = most goals scored).

## Database size
In the most raw version, the only input is the user question and the database schema. In some cases, the LLM has the domain-specific knowledge at its disposal as well. For smaller and benchmark-sized databases, it is possible to dump the whole schema with the user question and the possibly available documentation or evidence into a singular prompt. When using a fine-tuned model with a large enough context window, this result can yield some promising results as well. However, this approach is not really scalable beyond these smaller databases at the moment. 

For enterprise-sized databases containing hundreds of tables, each table with thousands of columns and each column with millions of rows, this approach simply arrives at a bottleneck. Despite some studies suggesting that this is only a matter of technological advancement, scalability still remains one of the biggest difficulties when building a Text-to-SQL solution.

## Hallucinations
Tasking a Large Language Model with SQL generation on its own can naturally result in hallucinations. That is not a bug, the LLM is always just predicting the next token. The SQL syntax sadly does not care about semantic similarity and, as a result, if the column name in the SQL is not an exact match with the column name in the database, the SQL will not work correctly. When tasked with a large input context, LLMs are notorious for that they inherently suffer from the "Lost-in-the-Middle" Phenomenon, essentially meaning, that for large prompts (depending on the model), most of the context in the middle is lost. What is lost is then often hallucinated.

# Benchmarks
As with all things LLM, the rapid development of Text-to-SQL tools is tracked through various benchmarks. Early benchmarks in the pre-LLM era, like WikiSQL, contained now trivial tasks paired with a single table extracted from Wikipedia. The queries only required basic filtering (like WHERE clauses) and simple aggregations (like COUNT or MAX) on that specific table.

After LLMs were adopted for Text-to-SQL, single-table benchmarks became redundant, as LLMs excelled at these tasks. The single table benchmarks were then replaced by benchmarks like Spider or BIRD, containing more complex tasks involving JOIN operations and relational mapping.

Today, the state-of-the-art tools and methods for Text-to-SQL are mostly evaluated by the Spider and BIRD benchmarks. The highest scoring method/model on the BIRD benchmark currently stands at a remarkable 81.95% execution accuracy in the Test score. That is compared to the 92.96% execution accuracy of a team consisting of data engineers and database students. 

Even though the Spider and BIRD benchmarks offer the ideal difficulty-to-complexity ratio for today's state-of-the-art models, they fail to properly test for the actual production scenario. Both benchmarks use either full documentation (Spider) or provide evidence (BIRD), giving the LLM domain-specific knowledge it might not otherwise have access to. Both of these benchmarks do not test for enterprise-sized databases either. In my opinion, this motivates the current submissions and development to focus on approaches that may not be at all scaleable beyond the benchmark-sized databases.

# The future
As we close in on perfect scores on some of the Spider benchmarks and get closer and closer to the human performance on BIRD, a new benchmark emerges. While acknowledging the issues with Spider and BIRD I mentioned earlier, BEAVER introduces a true enterprise benchmark for Text-to-SQL. Yet the question remains as to what extent is a true enterprise setting even manageable with the top scoring method on the BEAVER benchmark, ReFoRCE, using the Claude Sonnet 4.5, achieving mere 11.4 execution accuracy.

# Conclusion
Text-to-SQL is a complex process of transforming natural language questions into SQL. Although many solutions exist, ranging from subject matter expert assisted generation and single-table solutions to the state-of-the-art multi-stage agentic pipeline solutions, they all still fail to achieve a true enterprise-ready state.