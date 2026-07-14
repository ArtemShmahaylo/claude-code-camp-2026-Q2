# Explore Agent Architectures

## 1. An agent file with referenced files, for example CLAUDE.md and data/*.md

I attempted to use Claude Code as a Player Journey Agent with instructions stored in CLAUDE.md.

The agent was given:

- The MUD host and port
- Player credentials
- A goal to find a bakery and list its menu
- Markdown files for recording player and world state

The initial task was:

"Find the bakery and list what is on the menu."

During the first attempt, the Docker container running the MUD was not started. The agent detected that the game was unavailable, but instead of stopping, it searched the repository's world files. It found multiple bakery-related locations and returned information from the static game data.

The task was then repeated after starting the MUD with a stronger instruction:

"You must login to the game and find a bakery. List the menu."

## Technical Observations

The agent attempted to create its own MUD client rather than using a stable existing integration.

It created many temporary Python scripts using sockets and telnet. These scripts attempted to:

- Connect to localhost on port 4000
- Read the MUD login prompts
- Submit the player credentials
- Handle character creation or login state
- Send movement commands
- Navigate to a bakery
- Read the shop menu

The agent appeared to establish network connections to the MUD, but the final result did not contain a complete raw session transcript. Because of this, it was not possible to verify that the agent successfully logged in as the expected player, reached the bakery through the game, and obtained the menu from live game output.

During the same task, the agent also read the repository's world data files to determine the bakery location. This means the final answer was based on a mixture of live MUD interaction and static game data.

The agent reported that the bakery sold only water containers, despite the room description referring to bread and Danish pastries. This may indicate that the agent matched the wrong shop, room, or object data.

The agent created many similar temporary scripts before completing the task. This consumed significant time and made the execution difficult to audit or reproduce.

The markdown memory files did not provide enough structure to control the agent's interaction with the MUD. They stored information, but they did not provide a reliable connection, login, command, observation, or verification interface.

## Technical Conclusions

A prompt-only architecture using CLAUDE.md and markdown memory files is not sufficient for reliable MUD gameplay.

The agent needs a deterministic interface for interacting with the MUD. This interface should handle:

- TCP or Telnet connection details
- Login prompts and authentication
- Command submission
- Response collection
- Prompt detection
- Connection state
- Timeouts
- Session transcripts
- Error reporting

A small MUD SDK or command-line client should be created before building a more capable agentic loop.

The SDK should expose simple operations such as:

- connect
- login
- send_command
- read_response
- get_current_room
- move
- save_transcript
- disconnect

The agent should not be allowed to inspect the world source files during a gameplay test unless the experiment explicitly permits static world knowledge.

Every answer should include evidence showing:

- The commands sent to the MUD
- The responses returned by the MUD
- The room reached by the player
- The exact menu output
- Whether static files were accessed

An MCP server may later provide a cleaner tool interface around the MUD SDK. However, building an MCP server before creating a stable MUD client would add unnecessary complexity.

The next experiment should first create a reusable and testable MUD SDK. The Player Journey Agent can then use this SDK instead of generating temporary connection scripts during every run.

## 2. Agent Skills driven by main agent eg. ~/.skills

A very common way to drive specific functionality is via Agent Skills which is an open format for agents adopted by many coding harnesses and agent SDKs. 