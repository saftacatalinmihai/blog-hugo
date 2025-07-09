---
title: "Computer"
date: 2025-07-08T18:01:06+03:00
draft: false
---

## 🖖 Re-Creating Star Trek's LCARS Computer With Modern LLMs

*A hands-on experiment in autonomous tool-building agents*

Imagine walking into your home and saying, "Computer, show me when I last watered the plants and remind me to buy groceries." No app switching, no hunting through interfaces, no remembering which service stores what data. Just a conversation with your digital assistant that understands context, remembers everything, and can act on your behalf.

This isn't science fiction anymore—it's Tuesday afternoon in my living room.

What started as a quest to recreate Star Trek's iconic LCARS computer has become something far more practical: a universal replacement for the dozens of specialized apps cluttering our digital lives. Like a Swiss Army knife for the smartphone age, this system demonstrates how the marriage of large language models and tool-calling APIs can transform the way we interact with technology.

But the journey from starship computer to everyday assistant revealed something unexpected: the most powerful sci-fi technologies aren't the ones that dazzle with complexity, but those that disappear into the background, becoming as natural as conversation itself.

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

While building this system, I discovered something unexpected: **it's already replacing many specialized apps in my daily life**. Like a universal translator that breaks down language barriers, this architecture dissolves the barriers between apps, creating a unified interface for all your digital needs.

Think of it as the difference between having a Swiss Army knife versus carrying a separate tool for every task. The same conversational interface that lets LCARS control starship functions works surprisingly well for mundane tasks that would normally require dedicated applications.

I find that it already can replace many apps like note-taking, journaling, baby tracker, food inventory... simple apps that are just CRUD apps for writing and reading from a database. It's like having a personal digital butler who knows where you keep everything and can fetch it on command.

Consider how many apps on your phone are essentially just fancy interfaces for:
1. **C**reating records
2. **R**eading information back
3. **U**pdating existing data
4. **D**eleting things you no longer need

This is known as [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) - the four basic operations of persistent storage. Each app is like a specialized shopkeeper in a digital mall, each with its own storefront, its own rules, and its own way of organizing inventory.

Each with its own account, learning curve, subscription fee, and data silo. What if one conversational interface could replace them all? What if, instead of visiting dozens of specialized shops, you could speak to one knowledgeable concierge who had access to everything?

```
Captain's Log, Supplemental: The irony isn't lost on me that in trying to recreate 
sci-fi technology from the future, I've ended up replacing dozens of present-day 
apps with something that feels both more advanced and more intuitive.
```

---

### 3 · Architecture at a Glance

Like the Enterprise itself, this system is built on a modular architecture where each component serves a specific purpose, but they all work together seamlessly. Think of it as a digital starship where each deck has its own function, but they're all connected by the same communication network.

| Layer | Purpose | Starship Analogy |
|-------|---------|------------------|
| **LLM Clients** | Natural-language "brain" | The ship's computer core with multiple backup systems |
| **Assistant Core** | Conversation management | The bridge crew, coordinating all operations |
| **Tool Hub** | Concrete implementations | Engineering and science departments, each with specialized skills |
| **Database** | Knowledge store | The ship's library and data banks |
| **Interfaces** | User interaction | Communication channels: viewscreen, comm badges, and terminals |
| **Self-Modification** | System evolution | The ship's ability to adapt and upgrade its own systems |

This architecture is specifically designed to enable flexible CRUD operations through natural language. The database layer supports dynamic schema creation, while the LLM "brain" translates user intentions into appropriate database operations—much like how the universal translator converts alien languages into understandable speech.

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

Tools in this system work like skilled crew members on a starship—each one has a specific expertise, but they all speak the same language and follow the same protocols. When Captain Picard says "Computer, show me all Federation ships in the Neutral Zone," the computer doesn't just magically know what to do. It breaks down the request, identifies the right databases to query, and formats the response appropriately.

Our system works similarly, but for everyday tasks.

1. **Tool Implementation**
   Each tool is a Python function in the `tools/` directory. The SQL tool is the heart of the CRUD functionality—think of it as the ship's librarian who knows exactly where every piece of information is stored and how to retrieve it:

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
   The magic happens in the system prompt that guides the LLM—like Starfleet's Prime Directive, these instructions shape how the AI behaves in every interaction. Just as officers receive detailed briefings before away missions, the LLM gets comprehensive instructions on how to handle database operations:

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

   This prompt is critical because it explicitly allows the LLM to **create new tables and modify schemas** when needed. It's like giving the ship's computer permission to create new database categories when it encounters previously unknown phenomena. The last instruction is particularly powerful: "If the schema seems insufficient for the user's request, include `CREATE TABLE` statements first..."

   This is where the system transcends traditional apps. Instead of being constrained by a fixed schema designed by someone else, the database grows organically like a living organism, adapting to your actual needs.

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
   When a user makes a request that requires new data storage, the system orchestrates a sophisticated dance between understanding, planning, and execution:

   *User Request → LLM Analysis → Schema Evaluation → Table Creation/Modification → Data Operation → Response*

   Picture this scenario: you walk into your living room and say, "Computer, track that I watered my plants today." If no plant-tracking table exists, the system becomes like a helpful assistant who realizes you need a new filing system. It doesn't just fail—it adapts.

   The system will:
   1. Recognize the need for plant tracking (like a good assistant understanding your intent)
   2. Generate a `CREATE TABLE plants_watering(...)` statement (creating the filing system)
   3. Insert the watering record (filing the information)
   4. Return confirmation with the new table's state (showing you the organized result)

This dynamic schema evolution is what makes the system so powerful for CRUD app replacement. Unlike traditional apps with rigid schemas—like filing cabinets with fixed compartments—this approach lets the database structure evolve organically, like a personal assistant who creates new organizational systems as you need them.

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

In the Star Trek universe, the most fascinating aspect of advanced computers isn't their computational power—it's their ability to learn, adapt, and evolve. Like Data's neural networks creating new pathways based on experience, this system can modify its own code and capabilities.

The system can modify itself through a sophisticated self-awareness mechanism that would make any Federation engineer proud:

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

The true test of any technology isn't in the laboratory—it's in the messy, unpredictable reality of daily life. Like the Enterprise crew facing unexpected challenges that require creative solutions, this system has proven its worth in scenarios that would stump traditional apps.

Beyond the technical implementation, this system has proven immediately useful for everyday tasks that benefit from structured data storage with natural language access. It's like having a skilled personal assistant who never forgets, never gets tired, and always knows exactly where to find what you need.

#### App Replacement Showcase

Consider the digital transformation in your pocket. Where once you might have juggled multiple apps like a circus performer keeping plates spinning, now you have a single, conversational interface that understands context and remembers everything.

| Traditional App | LCARS Implementation | Key Advantages |
|-----------------|----------------------|----------------|
| **Note-taking App** | "Computer, note that I need to buy milk" | - Natural language input<br>- No app switching<br>- Automatic organization |
| **Baby Tracker** | "Log diaper change for Stefan" | - Shared family access<br>- Custom queries<br>- No subscription fees |
| **Food Inventory** | "Add milk to fridge, expires July 15" | - Voice input from anywhere<br>- Expiration alerts<br>- Shopping list integration |
| **Journaling App** | "Computer, journal entry: today I..." | - Multi-modal entries<br>- Searchable by content<br>- Integrated with other data |
| **Habit Tracker** | "Record 30 minutes of meditation" | - Natural reminders<br>- Pattern analysis<br>- No rigid templates |

#### 1. **Personal Activity Logging**
   It's like having a digital memory palace that never forgets. Every interaction becomes a breadcrumb in a trail of personal data that you can query naturally:
   * **Plant Care Tracking**: Recording when plants were watered, with the ability to later ask "When did I last water the plants?"
   * **Habit Tracking**: Logging exercise, medication, or other recurring activities with natural language queries

#### 2. **Baby Care Assistant**
   New parents know the exhaustion of trying to remember feeding schedules, diaper changes, and sleep patterns. This system becomes like a wise grandmother who remembers everything and offers gentle reminders:
   * Replacing specialized apps with simple natural language commands:
     ```
     > Recording wet diaper change for Stefan
     > Starting sleep time for Stefan
     > Stefan just finished feeding for 15 minutes
     ```
   * Later querying patterns: "How many wet diapers today?" or "When was Stefan's last feeding?"

#### 3. **Household Management**
   Like the ship's computer tracking the status of every system on the Enterprise, this system can monitor the countless details of home management:
   * **Inventory Tracking**: "Adding milk to fridge, expires on July 15th"
   * **Maintenance Records**: "Just changed the furnace filter" with later queries like "When was the furnace filter last changed?"

The system's key advantages in these scenarios mirror the benefits of having a ship's computer that knows every system, every crew member, and every mission detail:

* **Dynamic Schema Evolution**: Like the ship's computer adapting to new alien technologies, the assistant can create new tables or modify existing ones based on conversation context
* **Database Transparency**: Users can directly access the PostgreSQL database with standard clients for advanced queries—like having direct access to the ship's data banks when needed
* **Device Integration**: Using the Web API with [Apple Shortcuts](https://support.apple.com/guide/shortcuts/welcome/ios) enables voice interaction from any Apple device, including [Apple Watch](https://www.apple.com/watch/)—your personal communicator badge
* **Conversational Interface**: Natural language input/output eliminates the need to learn specialized app interfaces—speak to your computer the way you'd speak to a colleague

This practical utility demonstrates how the technical capabilities translate into real value for users, bridging the gap between science fiction and everyday convenience. It's the difference between having a tool that works for you versus having to work for your tools.

```
Captain's Log: It's remarkable how quickly I've abandoned specialized apps in favor of
simply talking to my computer. The freedom from app-switching alone has saved me countless
hours, not to mention the mental load of remembering which app contains which information.
```

---

### 10 · Why One System Beats Many Apps

In the original Star Trek series, the ship's computer wasn't a collection of separate programs—it was a unified intelligence that could connect information across all ship systems. This LCARS-inspired approach offers several advantages that mirror the benefits of having a single, omniscient digital assistant rather than a collection of specialized servants:

1. **Unified Data Store**: All information lives in one database, enabling cross-domain queries like "Show me all the days I exercised but didn't sleep well"—connections that would be impossible when your fitness tracker and sleep app can't talk to each other

2. **Consistent Interface**: Learn one interaction pattern that works across all use cases, rather than navigating different UIs for each app. It's like having a universal translator for your digital life

3. **No Context Switching**: Eliminate the cognitive load of jumping between apps—just talk to your computer naturally, like having a conversation with a knowledgeable friend who remembers everything

4. **Evolving Functionality**: The system grows with your needs through self-modification, rather than waiting for app developers to add features. It's like having a technology that adapts to you, rather than requiring you to adapt to it

5. **Privacy Control**: Your data stays on your infrastructure, not spread across multiple third-party services like digital breadcrumbs scattered across the internet

6. **Cost Efficiency**: Replace multiple subscription services with a single self-hosted solution—like consolidating dozens of monthly bills into one

7. **Custom Workflows**: Create personalized workflows that span traditional app boundaries, like a digital butler who understands your unique preferences and routines

The most surprising benefit has been how the unified approach reveals connections between previously siloed information. It's like switching from a flashlight that illuminates one small area to stadium lights that show the entire field. When your exercise data, sleep tracking, and work productivity all live in the same system, patterns emerge that would be invisible when using separate apps.

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

Building a real-world LCARS system has taught me that the most powerful sci-fi technologies aren't the ones that dazzle with impossible physics—they're the ones that solve fundamental human problems. The tricorder was revolutionary not because it used exotic particles, but because it put all the information a person might need into a single, intuitive interface.

* **Self-modifying AI assistants** are now achievable with modern LLMs and tool-calling APIs, bringing us closer to the adaptive intelligence we've dreamed of
* **Multiple interfaces** (CLI, Web, Slack) provide flexible access points for different use cases—like having multiple ways to contact the ship's computer
* **Versioning** both the assistant and its tools creates a stable evolution path, allowing the system to grow without breaking
* **Database operations** (SQLite/PostgreSQL), Python execution, and visualization capabilities form a powerful foundation for replacing entire categories of apps
* **Self-modification** requires careful design with placeholders and version management, but enables unprecedented adaptability
* **CRUD app replacement** demonstrates immediate practical utility beyond technical novelty—this isn't just a proof of concept, it's a daily driver
* **Unified data approach** reveals insights impossible with siloed app ecosystems, like having a conversation with your data instead of interrogating it
* The system demonstrates a practical implementation of the **LCARS vision** from Star Trek, proving that some sci-fi dreams are closer than we think

This project shows how close we've come to the science fiction dream of conversational computers that can extend their own capabilities—while highlighting the engineering challenges that remain in making such systems robust, secure, and truly autonomous. More surprisingly, it demonstrates how quickly such systems can replace dozens of specialized apps with a single, more flexible interface.

The future isn't just about having smarter computers—it's about having computers that are easier to talk to. And sometimes, the most futuristic technology is the one that feels the most natural.

— *End log.*

** Part of this blog post was written with the help (not by) AI (LLMs).