# LangChain EDA Assistant

![](architecture_diagram.jpg)

## Overview

Data analysis and visualization play a pivotal role in modern business operations, providing invaluable insights that drive strategic decision-making and organizational success. For business stakeholders, the ability to interpret and derive meaning from vast datasets is crucial in understanding market trends, customer behavior, and operational performance. 

**Large Language Models (LLMs)** present a paradigm shift in how non-technical stakeholders interact with data. These powerful NLP models enable users to converse with data in plain language, bypassing the need for SQL or programming expertise.

This project enables natural language interaction with an **ecommerce SQL database** through a **Streamlit chatbot**. It leverages:
- [LangChain](https://github.com/langchain-ai/langchain) to manage LLM agents
- [OpenAI GPT-4](https://openai.com/gpt-4) to interpret user intent
- [Plotly](https://plotly.com/python/) to generate visualizations

---

## Data Description
The e-commerce dataset consists of 7 interconnected datasets covering distribution centers, user events, inventory items, order details, products, and user profiles.

- **Distribution Centers dataset:** Contains information about distribution centers including unique identifiers, names, and geographical coordinates.
- **Events dataset:** Provides details about user events including unique identifiers, timestamps, user IDs, session IDs, and event types.
- **Inventory Items dataset:** Includes data on inventory items such as unique identifiers, timestamps for creation and sale, product details, and distribution center associations.
- **Order Items dataset:** Contains information on ordered items including unique identifiers, timestamps for creation, shipping, delivery, and return, along with associated product and user IDs.
- **Orders dataset:** Provides details about orders including unique identifiers, timestamps for creation, shipping, delivery, and return, along with order status and quantity of items.
- **Products dataset:** Includes information about products such as unique identifiers, costs, retail prices, categories, brands, and distribution center associations.
- **Users dataset:** Contains user profiles with details like unique identifiers, names, email addresses, demographics, location coordinates, traffic sources, and account creation timestamps.

## Tech Stack
- **Language:** Python 3.10
- **Libraries:** mysql-connector-python, mysqlclient, pydantic, langchain, matplotlib, plotly, streamlit, pymysql, python-dotenv
- **Model:** OpenAI GPT-4

## Approach
- **Database Creation:**
  - Install MySQL and MySQL Workbench on your system
  - Create a new MySQL database and load the CSVs into the database
- **Prompt Engineering Strategies:**
  - Acting as a Role
  - Language Translation
  - Travelling Guide
  - Combining LLMs and Prompts in multi-step workflows
- **LangChain Integration:**
  - Create SQL agent that transforms natural language queries into SQL queries
  - Create Python script agent that creates visualizations from SQL query outputs
- **Streamlit Chatbot Creation**
- **Integrating Memory and Guardrails**

## Project Structure
```
data/
  distribution_centers.csv
  ecommerce-data-description.docx
  ecommerce.db
  events.csv
  inventory_items.csv
  orders.csv
  order_items.csv
  products.csv
  users.csv

Query_To_Dashboard.ipynb
Readme.md
requirements.txt
src/
  agent.py
  chat_app.py
  helper.py
  stream_lit_demo.py
test/
```

## How it Works

### 1. chat_app.py – Main Streamlit App
This is the entry point of the application.

**Key Responsibilities:**
- Loads LLM agents (SQL & Python) and maintains session state
- Captures user queries from a chat interface
- Decides whether a query should be interpreted as a SQL query or a Python plot request
- Handles context injection for continuity in multi-turn conversations
- Displays results: plain text, charts, or error messages

**Important Logic:**
```python
if 'plot' in user_query.lower():
    use_python_agent()
else:
    use_sql_agent()
```

Visualization-related queries trigger the Python agent, otherwise the SQL agent is used.

**Streamlit Features:**
- `st.chat_input()` for natural user interaction
- Session memory for agent reuse (`st.session_state`)
- Reset functionality to clear agent history

### 2. agent.py – LLM Agent Setup Using LangChain
This file builds the LangChain-based agents.

**Agents Created:**
- SQL Agent
- Python Agent

#### SQL Agent
- Uses `create_sql_agent()` with:
  - `ChatOpenAI` as LLM
  - `SQLDatabaseToolkit` to access MySQL DB
  - Memory using `SQLChatMessageHistory` and `ConversationBufferMemory`
  - Custom instruction suffix (`CUSTOM_SUFFIX`) guiding the model to:
    - Avoid hallucinations
    - Use proper SQL syntax (e.g., `LOWER()` for case-insensitive queries)
    - Return only raw SQL results

```python
agent = create_sql_agent(
    llm=llm_agent,
    toolkit=toolkit,
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    suffix=CUSTOM_SUFFIX,
    memory=memory,
)
```

**Database Connection:**
Reads MySQL password from `.env` and uses SQLAlchemy URI:

```python
mysql+pymysql://root:{password}@localhost:3306/ecommerce
```

#### Python Agent
- Uses `create_openai_functions_agent()` from LangChain with:
  - A Python REPL Tool
  - Instructions that:
    - Force usage of Plotly (no matplotlib)
    - Force Python code execution (for reliability)
    - Return the response wrapped in a markdown Python block

```python
agent = create_openai_functions_agent(
    ChatOpenAI(...),
    tools=[PythonREPLTool()],
    prompt=custom_prompt
)
```

### 3. helper.py – Rendering Utility Functions
Contains Streamlit-friendly rendering functions.

- `display_text_with_images(text)`
  - Parses text and embedded image URLs
  - Renders markdown + images in order
  - Detects URLs like `https://xyz.com/image1.jpg`
- `display_python_code_plots(text)`
  - Extracts Python code from LLM markdown output

```python
pattern = r'```python\s(.*?)```'
```

Used in:

```python
code = display_python_code_plots(response['output'])
exec(code)
```

## Setup Instructions

1. **Install Dependencies**
```bash
pip install -r requirements.txt
```
Ensure the following are included:
- langchain, langchain_openai
- streamlit
- pymysql
- sqlalchemy
- plotly

2. **Setup .env File**
Create a `.env` file in the root directory:
```
OPENAI_API_KEY=your_openai_key
MYSQL_PASSWORD=your_mysql_password
```

3. **MySQL Database**
Ensure you have a MySQL DB running with:
- Database: `ecommerce`
- Table: `message_store` for memory
- Tables like `orders`, `users`, etc. for queries

4. **Run the App**
```bash
streamlit run chat_app.py
```

## How the Files Work Together

```
[chat_app.py]
     |
     |---> Uses [agent.py] to create:
     |       --> SQL Agent (SQL queries)
     |       --> Python Agent (plotting code)
     |
     |---> Uses [helper.py] to:
             --> Display responses
             --> Extract and render code blocks
```


![](demo.png)