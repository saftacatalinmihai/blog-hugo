---
title: "Computer"
date: 2025-07-08T18:01:06+03:00
draft: false
---

## 🖖 Re-Creating Star Trek's LCARS Computer With Modern LLMs

*A hands-on experiment in autonomous tool-building agents*

### 1 · Why LCARS?

For decades science-fiction fans have watched [Star Trek](https://en.wikipedia.org/wiki/Star_Trek) officers converse with a ship-wide computer [LCARS](https://en.wikipedia.org/wiki/LCARS) ("Library Computer Access/Retrieval System"). LCARS could:

* answer arbitrary questions by searching internal databases;
* execute real-world actions by routing commands to subsystems;
* politely say **why** it couldn't comply when limits were hit.

In 2025, [large language models (LLMs)](https://en.wikipedia.org/wiki/Large_language_model) plus [tool-calling APIs](https://platform.openai.com/docs/guides/function-calling) make that dream feel within reach. I decided to find out *how close* we can get—armed with [Replit](https://replit.com/) dev env, and a [Postgres](https://www.postgresql.org/) instance.

You can find the open source code here: https://github.com/saftacatalinmihai/Computer

---

### 2 · Beyond Science Fiction: Replacing Everyday Apps

> *"Computer, log wet diaper change for Stefan at 14:30."* — Me, talking to my watch instead of opening a baby tracker app

While building this system, I discovered something unexpected: **it's already replacing many of specialized apps in my daily life**. The same architecture that lets LCARS control starship functions works surprisingly well for mundane tasks that would normally require dedicated applications.

I find that it already can replace many apps like note-taking, journaling, baby tracker, food inventory... simple apps that are just CRUD apps for writing and reading from a DB. The natural language interface combined with a flexible database schema creates a universal interface that's often *more* convenient than purpose-built applications.

Consider how many apps on your phone are essentially just fancy interfaces for:
1. **C**reating records
2. **R**eading information back
3. **U**pdating existing data
4. **D**eleting things you no longer need

This is known as [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) - the four basic operations of persistent storage.

Each with its own account, learning curve, subscription fee, and data silo. What if one conversational interface could replace them all?

```
Captain's Log, Supplemental: The irony isn't lost on me that in trying to recreate 
sci-fi technology from the future, I've ended up replacing dozens of present-day 
apps with something that feels both more advanced and more intuitive.
```

---

### 3 · Architecture at a Glance

| Layer | Purpose | Notes |
|-------|---------|-------|
| **LLM Clients** | Natural-language "brain" | Multiple clients ([OpenRouter](https://openrouter.ai/), [Ollama](https://ollama.com/)) with flexible model selection |
| **Assistant Core** | Conversation management | Versioned architecture (v1-v4) with dynamic loading |
| **Tool Hub** | Concrete implementations | One file per tool; importable functions |
| **Database** | Knowledge store | Dual support for [SQLite](https://www.sqlite.org/) and [PostgreSQL](https://www.postgresql.org/) |
| **Interfaces** | User interaction | CLI REPL, Web ([Flask](https://flask.palletsprojects.com/)), and [Slack](https://slack.com/) integrations |
| **Self-Modification** | System evolution | Code reading and writing capabilities |

This architecture is specifically designed to enable flexible CRUD operations through natural language. The database layer supports dynamic schema creation, while the LLM "brain" translates user intentions into appropriate database operations.

{{< mermaid >}}
graph TD;
    User[User] --> |Natural Language| Interfaces
    subgraph Interfaces
        CLI[Command-Line REPL]
        Web[Web Interface]
        Slack[Slack Bot]
    end
    Interfaces --> |Requests| Assistant
    subgraph Assistant Core
        V1[Assistant v1]
        V2[Assistant v2]
        V3[Assistant v3]
        V4[Assistant v4]
        Loader[Assistant Loader]
    end
    Assistant --> |Queries| LLM[LLM Clients]
    LLM --> |Responses| Assistant
    Assistant --> |Tool Calls| Tools
    subgraph Tool Hub
        SQL[SQL Operations]
        Python[Python Execution]
        ASCII[ASCII Art Generation]
        Self[Self-Modification]
    end
    Tools --> |Results| Assistant
    SQL --> DB
    subgraph Database
        SQLite[SQLite]
        Postgres[PostgreSQL]
    end
{{< /mermaid >}}

---

### 4 · How Tools Work

1. **Tool Implementation**
   Each tool is a Python function in the `tools/` directory. The SQL tool is the heart of the CRUD functionality:

   ```python
   # From tools/sql.py
   def run_sql(conn):
       def inner(sql_statements: str):
           column_names = []
           rows = []
           parsed_sql_list = parse_sql(str(sql_statements))
           for sql_statement in parsed_sql_list:
               if not sql_statement.strip():
                   continue
               cursor = conn.cursor()
               cursor.execute(sql_statement)
               if cursor.description:
                   column_names = [description[0] for description in cursor.description]
                   rows = cursor.fetchall()
               else:
                   column_names = []
                   rows = []
               conn.commit()
           # Format and return results...
       return inner
   ```

2. **System Prompt for SQL Tool**
   The magic happens in the system prompt that guides the LLM on how to use the SQL tool:

   ```
   Tool: postgres_sql_run
   Description: Runs SQL against a Postgres database.
   Instructions:
   - You interact with a user who is not familiar with SQL.
   - Generate SQL code based on user requests. Keep it simple and use only Postgres-compatible features.
   - No multi-tenancy, user access control, complex data validation, or error handling beyond basic SQL.
   - For relative time (e.g., "yesterday"), use Postgres's relative time functions.
   - DO NOT insert sample or placeholder data. Only use data mentioned by the user.
   - After any DDL (CREATE, ALTER, DROP) or DML (INSERT, UPDATE, DELETE) statement, append a `SELECT * FROM table_name;` query for the modified table to show its new state.
   - Assume tables might contain data, but you don't know what it is. Your role is to generate SQL.
   - For querying, aim for broad matches (e.g., case-insensitive search if appropriate for the user's request).
   - Provide only the final, executable SQL code. No hypothetical examples.
   - If the schema seems insufficient for the user's request, include `CREATE TABLE` statements first, then the necessary DML.
   - The existing Postgres SQL Schema is: [CURRENT SCHEMA INJECTED HERE]
   ```

   This prompt is critical because it explicitly allows the LLM to **create new tables and modify schemas** when needed. The last instruction is particularly powerful: "If the schema seems insufficient for the user's request, include `CREATE TABLE` statements first..."

3. **Tool Schema Definition**
   The LLM needs to know how to call the tool, which is defined in a schema:

   ```json
   {
     "type": "function",
     "function": {
       "name": "postgres_sql_run",
       "description": "Runs one or more SQL statements against a Postgres database and returns the result of the last query.",
       "parameters": {
         "type": "object",
         "properties": {
           "sql_statements": {
             "type": "string",
             "description": "A single string containing one or more SQL statements separated by semicolons."
           }
         },
         "required": ["sql_statements"]
       }
     }
   }
   ```

4. **Call Flow with Schema Evolution**
   When a user makes a request that requires a new type of data storage:

   *User Request → LLM Analysis → Schema Evaluation → Table Creation/Modification → Data Operation → Response*

   For example, if a user says "Track that I watered my plants today" and no plant tracking table exists, the system will:
   1. Recognize the need for plant tracking
   2. Generate a `CREATE TABLE plants_watering(...)` statement
   3. Insert the watering record
   4. Return confirmation with the new table's state

This dynamic schema evolution is what makes the system so powerful for CRUD app replacement. Unlike traditional apps with fixed schemas, this approach lets the database structure evolve organically with user needs.

```
[Lieutenant Commander Data](https://en.wikipedia.org/wiki/Data_(Star_Trek)) would appreciate that the system can modify its own code,
much like his own neural net's ability to create new pathways based on experience.
```

---

### 5 · LLM Prompt Engineering

The system prompt is dynamically constructed with:

1. **Persona Definition**: Establishes the assistant's identity and capabilities
2. **Tool Descriptions**: Detailed information about available tools and their parameters
3. **Database Schema**: When using database tools, the schema is included for context
4. **Conversation History**: Previous interactions to maintain context
5. **Special Instructions**: Custom behavior guidelines based on the assistant version

The prompt construction is version-specific, with each assistant version (v1-v4) having its own prompt engineering approach. This allows the system to evolve its interaction style and capabilities over time.

For CRUD operations, including the current database schema in the prompt is essential. This gives the LLM awareness of existing tables and fields, allowing it to generate appropriate SQL for creating, reading, updating, or deleting records based on conversational requests.

---

### 6 · Self-Modification Capabilities

The system can modify itself through a sophisticated self-awareness mechanism:

1. **Code Reading**
   The assistant can read its own source code:

   ```python
   # Helper function to get current assistant code
   def get_current_assistant_code(file_path):
      """Reads and returns the content of the current assistant.py file."""
      try:
         with open(file_path, 'r', encoding='utf-8') as f:
               return f.read()
      except Exception as e:
         print(
               f"Critical Error: Could not read own source code at {file_path}. Error: {e}"
         )
         return "# Error: Could not read assistant code. Self-update features will be impaired."

   ```

2. **Version Management**
   When creating a new version, the system:

   * Reads its current implementation
   * Makes the requested modifications
   * Saves as a new version with incremented number (e.g., `assistant_v4.py`)
   * Updates the assistant loader to recognize the new version

3. **Tool Addition**
   New tools are added by:

   * Creating a new tool file in the `tools/` directory
   * Updating the assistant code to include the tool in its mapping
   * Defining the tool's schema for the LLM

   This is implemented through special placeholders in the code:
   * `#<TOOL_IMPORT>` - Where to add new tool imports
   * `#<TOOL_MAPPING>` - Where to add new tool mappings
   * `#<tool_schema>` - Where to add new tool schemas

This self-modification capability is what makes the system truly extensible for CRUD app replacement. When I need a new type of app functionality, I can simply ask the system to create a new tool that handles that specific use case, rather than downloading yet another specialized app.

```
"Working." - The Computer, [Star Trek: The Next Generation](https://en.wikipedia.org/wiki/Star_Trek:_The_Next_Generation)
```

---

### 7 · Multiple Interfaces

The system provides three distinct interfaces:

1. **Command-Line REPL (bot_repl.py)**
   ```python
   # Interactive command-line interface
   while True:
       user_input = input("> ")
       if user_input.lower() in ["exit", "quit"]:
           break
       response = assistant.process_message(user_input)
       print(response)
   ```

2. **Web Interface (main.py)**
   A Flask-based web application and API with:
   * User authentication
   * Conversation history
   * Streaming responses

3. **Slack Bot (bot_slack.py)**
   Integration with Slack for team communication:
   * Responds to direct messages
   * Handles slash commands
   * Supports threaded conversations

Having multiple interfaces means I can interact with my CRUD functionality from wherever is most convenient:
- Terminal when I'm already working in the command line
- Web interface from any device with a browser
- Slack when collaborating with family or team members

This multi-interface approach eliminates one of the biggest limitations of traditional apps: being locked into a single access method.

---

### 8 · Evolution Through Versions

The system has evolved through multiple versions:

| Version | Key Features Added |
|---------|-------------------|
| **v1** | Basic conversation, SQLite support |
| **v2** | Tool execution, Python code evaluation |
| **v3** | Self-modification, improved error handling |
| **v4** | PostgreSQL support, ASCII art generation, enhanced self-modification |

Each version builds upon the previous one, with the `assistant_loader.py` module dynamically loading the appropriate version based on configuration or user request.

This versioning approach allows:
* Stable reference points for development
* Ability to roll back if issues occur when new tools are added by the assistant and it breaks something.
* Clear tracking of capability evolution
* Testing new features without breaking existing functionality

The progression from SQLite to PostgreSQL in particular has been crucial for CRUD app replacement, as it provides a stable DB in a managed cloud needed for daily from multiple entry points.

---

### 9 · Real-World Applications: The CRUD App Replacement Showcase

Beyond the technical implementation, this system has proven immediately useful for everyday tasks that benefit from structured data storage with natural language access:

#### App Replacement Showcase

| Traditional App | LCARS Implementation | Key Advantages |
|-----------------|----------------------|----------------|
| **Note-taking App** | "Computer, note that I need to buy milk" | - Natural language input<br>- No app switching<br>- Automatic organization |
| **Baby Tracker** | "Log diaper change for Stefan" | - Shared family access<br>- Custom queries<br>- No subscription fees |
| **Food Inventory** | "Add milk to fridge, expires July 15" | - Voice input from anywhere<br>- Expiration alerts<br>- Shopping list integration |
| **Journaling App** | "Computer, journal entry: today I..." | - Multi-modal entries<br>- Searchable by content<br>- Integrated with other data |
| **Habit Tracker** | "Record 30 minutes of meditation" | - Natural reminders<br>- Pattern analysis<br>- No rigid templates |

#### 1. **Personal Activity Logging**
   * **Plant Care Tracking**: Recording when plants were watered, with the ability to later ask "When did I last water the plants?"
   * **Habit Tracking**: Logging exercise, medication, or other recurring activities with natural language queries

#### 2. **Baby Care Assistant**
   * Replacing specialized apps with simple natural language commands:
     ```
     > Recording wet diaper change for Stefan
     > Starting sleep time for Stefan
     > Stefan just finished feeding for 15 minutes
     ```
   * Later querying patterns: "How many wet diapers today?" or "When was Stefan's last feeding?"

#### 3. **Household Management**
   * **Inventory Tracking**: "Adding milk to fridge, expires on July 15th"
   * **Maintenance Records**: "Just changed the furnace filter" with later queries like "When was the furnace filter last changed?"

The system's key advantages in these scenarios:

* **Dynamic Schema Evolution**: The assistant can create new tables or modify existing ones based on conversation context
* **Database Transparency**: Users can directly access the PostgreSQL database with standard clients for advanced queries
* **Device Integration**: Using the Web API with [Apple Shortcuts](https://support.apple.com/guide/shortcuts/welcome/ios) enables voice interaction from any Apple device, including [Apple Watch](https://www.apple.com/watch/)
* **Conversational Interface**: Natural language input/output eliminates the need to learn specialized app interfaces

This practical utility demonstrates how the technical capabilities translate into real value for users, bridging the gap between science fiction and everyday convenience.

```
Captain's Log: It's remarkable how quickly I've abandoned specialized apps in favor of
simply talking to my computer. The freedom from app-switching alone has saved me countless
hours, not to mention the mental load of remembering which app contains which information.
```

---

### 10 · Why One System Beats Many Apps

The LCARS-inspired approach offers several advantages over using multiple specialized apps:

1. **Unified Data Store**: All information lives in one database, enabling cross-domain queries like "Show me all the days I exercised but didn't sleep well"

2. **Consistent Interface**: Learn one interaction pattern that works across all use cases, rather than navigating different UIs for each app

3. **No Context Switching**: Eliminate the cognitive load of jumping between apps—just talk to your computer naturally

4. **Evolving Functionality**: The system grows with your needs through self-modification, rather than waiting for app developers to add features

5. **Privacy Control**: Your data stays on your infrastructure, not spread across multiple third-party services

6. **Cost Efficiency**: Replace multiple subscription services with a single self-hosted solution

7. **Custom Workflows**: Create personalized workflows that span traditional app boundaries

The most surprising benefit has been how the unified approach reveals connections between previously siloed information. When your exercise data, sleep tracking, and work productivity all live in the same system, patterns emerge that would be invisible when using separate apps.

---

### 11 · Challenges and Next Steps

1. **Current Challenges**
   * **Security**: Python code execution presents significant security risks
   * **Error Handling**: Improving error recovery and user-friendly error messages
   * **Memory Management**: Optimizing conversation history for long interactions
   * **Tool Dependencies**: Managing dependencies between tools and assistant versions
   * **Schema Management**: Balancing flexibility with consistency in database structure

2. **Planned Improvements**
   * **Dynamic MCP server integration** Looking up [MCP servers](https://modelcontextprotocol.io/introduction) and installing them as tools to be used by the assistant.
   * **Multi-Modal Capabilities**: Adding [image and audio processing](https://en.wikipedia.org/wiki/Multimodal_learning)
   * **Improved Tool Management**: More robust tool versioning and dependency tracking
   * **Security Hardening**: [Sandboxed execution environments](https://en.wikipedia.org/wiki/Sandbox_(computer_security)) and permission models
   * **Advanced Visualization**: Beyond ASCII art to rich interactive visualizations
   * **Mobile-First Interfaces**: Optimized experiences for on-the-go CRUD operations
   * **Community Tool Repository**: Sharing specialized tools for different CRUD app replacements

3. **Future CRUD App Replacements**
   * **Health Tracking**: Comprehensive wellness monitoring beyond single-purpose health apps
   * **Project Management**: Replacing task management apps with natural language project coordination
   * **Learning Systems**: Personal knowledge management and spaced repetition learning
   * **IoT Integration**: Connecting [Internet of Things](https://en.wikipedia.org/wiki/Internet_of_things) devices to the CRUD ecosystem
   * **Financial Tracking**: Personal finance management through conversation

```
"There are always possibilities." - [Spock](https://en.wikipedia.org/wiki/Spock)
```

---

## ☑️ Takeaways

* **Self-modifying AI assistants** are now achievable with modern LLMs and tool-calling APIs
* **Multiple interfaces** (CLI, Web, Slack) provide flexible access points for different use cases
* **Versioning** both the assistant and its tools creates a stable evolution path
* **Database operations** (SQLite/PostgreSQL), Python execution, and visualization capabilities form a powerful foundation
* **Self-modification** requires careful design with placeholders and version management
* **CRUD app replacement** demonstrates immediate practical utility beyond technical novelty
* **Unified data approach** reveals insights impossible with siloed app ecosystems
* The system demonstrates a practical implementation of the **LCARS vision** from Star Trek

This project shows how close we've come to the science fiction dream of conversational computers that can extend their own capabilities—while highlighting the engineering challenges that remain in making such systems robust, secure, and truly autonomous. More surprisingly, it demonstrates how quickly such systems can replace dozens of specialized apps with a single, more flexible interface.

— *End log.*

** Part of this blog post was written with the help (not by) AI (LLMs).