---
date: '2026-06-22T17:32:02+02:00'
draft: false
title: 'Schema Linking for Text-to-SQL using LLM-generated Metadata Descriptions'
featured: false
---
While building an LLM-powered [Text-to-SQL framework](/posts/text-to-sql-intro/), many developers encounter a fundamental issue when scaling into production databases: it is not scalable to pass the whole database schema to the LLM prompt and expect the LLM to handle the sheer volume of data. Not only do models have limited context windows, but even those with a large enough one will eventually struggle to make sense of the database schema and distinguish which columns or tables are actually relevant to the user's question.

## Schema Linking
Schema linking aims to solve this issue by essentially finding the relevant tables and columns which then get passed on to the prompt instead of the whole schema. Imagine the user asks:

> “Which customers placed orders in March?”

The model must figure out:
- **customers** → table `customers`, columns `id`/`name`/`email`...
- **orders** → table `orders`
- **placed** → likely join condition via a foreign key (`orders.cust_id = customers.id`)
- **March** → filter on a date column, maybe `orders.order_date`

Without schema linking, the model would need to figure out these links from the whole schema. So the use case for schema linking is not just to tackle the limited context windows of language models, but to also greatly improve the efficiency by giving the LLM all the relevant information and not much more, giving it no space for possible hallucinations.

## The Argument Against Schema Linking
Some researchers argue that the issue of the scalability of dumping the whole schema into the generation prompt is just a matter of technological advancement, that both context windows will get larger and LLMs will get cheaper and, therefore, developing complex schema linking pipelines is not necessary. These researchers rather focus on developing fine-tuned agents for the task. However, as of now, prices of cloud services and AI tokens are not looking to get much cheaper any time soon and such use of fine-tuned models has not been attempted beyond benchmark-scale databases (BIRD).

## Real-world Databases
Real databases are messy. Column names may be cryptic (cust_id, ord_dt), user terminology may be different from schema terminology, different terminology can be used in different tables etc. Most of today's state-of-the-art Text-to-SQL frameworks leverage some kind of external domain-specific knowledge like documentation or evidence and therefore lack flexibility and are difficult to implement in practice. This may be the result of the fact that the current Text-to-SQL frameworks are mostly evaluated using two relevant benchmarks: BIRD and Spider 2.0.

For example, this is the evidence provided by the BIRD train dataset for the user's question _"Please calculate the average total price of shipped orders from German customers."_:
```
average total price = DIVIDE(MULTIPLY(quantityOrdered, priceEach)), COUNT(orderNumber)); German is a nationality of country = 'Germany'; shipped orders refers to status = 'Shipped';
```

The tasks in Spider 2.0 include external documentation markdown files. While some databases surely have well-maintained and detailed documentation, it is safe to assume that the majority of databases do not have documentation ready for LLM to perfectly understand.

Without documentation or evidence, it is currently impossible for the LLM to consistently find relevant columns and tables to the user's question in a complex schema by itself. However, when the LLM is given a prompt with a focused schema like this:

> “Show employee salaries by department.”

Focused schema:
```
staff(emp_id, full_name, salary_usd, dept_fk)
departments(dept_id, dept_name)
```

This makes the task simpler by multiple orders of magnitude. The LLM can then easily resolve that:
- **employee** → `staff`
- **salaries** → `salary_usd`
- **department** → `departments.dept_name`
- join path → `staff.dept_fk = departments.dept_id`


## Schema Linking Using LLM Metadata-driven Descriptions
To correctly assess the relevance of a column or a table to the user's question, it is first important to understand what each column and table actually contains. This is what good documentation does, after all. However, we assume that we have no documentation available for the database, so why not just create one? While we are at it, let's also make the documentation semantically rich, so we can then dump it into a vector database and retrieve relevant columns and tables by semantic similarity to the user's question.

Of course, this will not be done by hand and will introduce a preprocessing step into the Text-to-SQL pipeline. So first, it is needed to actually get the data from the database. For each column, retrieve data like column name, data type, non-null value count, unique values etc. and for each table, the table name, example rows, column count etc. This data could then be fed into an LLM, prompting it to create a SQL column description based on the data retrieved. Yet a more robust technique is to insert the raw data into a structured narrative. This is an example of a column narrative, created from a BEAVER train database:

```
Column 'ACADEMIC_TERMS_KEY' has VARCHAR(127) data type and allows NULLs. The column is located in the table ACADEMIC_TERMS which has 144 rows. In the physical database table layout, the column 'ACADEMIC_TERMS_KEY' sits and immediately precedes 'TERM_CODE' and 'TERM_DESCRIPTION'. 0 total values are NULL (0.0%). The column contains 144 unique distinct values, yielding a uniqueness ratio of 100.0%. Character length metadata: Min length is 6, max length is 6 (Average length: 6.0). The top most frequent values distributed in this column are: '1995FA' appearing 1 times (0.69%), '1995JA' appearing 1 times (0.69%), '1995SP' appearing 1 times (0.69%), '1995SU' appearing 1 times (0.69%), '1996FA' appearing 1 times (0.69%).
```

This then gets passed to the LLM to create a semantically rich description based on the narrative:

```
Serves as the unique, system-generated surrogate identifier for each distinct academic term record within the ACADEMIC_TERMS dimension table. This column is fully populated across all 144 rows with zero NULLs and exhibits 100% uniqueness, making it the definitive primary key for this entity. The values are strictly formatted as a fixed 6-character composite code that concatenates a 4-digit calendar year with a 2-character season or session abbreviation (e.g., 'FA' for Fall, 'SP' for Spring, 'SU' for Summer, 'JA' for January term). This column is the primary target for JOIN operations when linking fact tables (such as enrollments or course offerings) to their specific term context, and it should be used in SQL generation whenever a query requires precise term-level granularity or needs to traverse relationships anchored to a specific academic period.
```
_<small>(generated with Deepseek-V4-Pro)</small>_

While the descriptions may not always be perfect, they are semantically rich, exactly what is needed for the next step: upserting them into a vector database. When the user asks a question, the N most similar descriptions to the user's question are found and are retrieved along with their metadata, like table name or column name, as relevant to the user's question.


## Conclusion
This article gave a brief introduction to schema linking and to one of many approaches to achieve it. Most state-of-the-art Text-to-SQL solutions work with database documentation or evidence. The usage of LLM generated descriptions based on the retrieved data from the database is essential when working without the provided domain-specific knowledge. Furthermore, if done correctly, the descriptions will be semantically rich, and thus they become ideal candidates for schema linking via semantic similarity to the user's question.