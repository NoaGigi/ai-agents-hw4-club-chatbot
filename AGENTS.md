# Homework 4: Club Chatbot

This project is Homework 4 for the AI Agents course. The official `AI_Agents_Homework_LangGraph_Topic_Chatbot.pdf` in this folder is the primary source of truth for project requirements. Follow it and the final `prompt.md` when one is added. If project guidance or a prompt conflicts with the PDF, follow the PDF and update this file or prompt to match it.

## Deliverable and implementation

- Put all implementation code in a Google Colab-compatible Jupyter notebook (`.ipynb`). Do not create standalone Python files unless explicitly requested.
- Build the chatbot with LangGraph and the concepts taught in class. Keep the graph small: exactly one `chatbot` node and `START -> chatbot -> END`. Do not add nodes or complexity beyond the PDF requirements. Optional bonus features are not required unless the user explicitly requests them.
- Organize the notebook into clear sections with Markdown headings. Explain important implementation decisions, especially where the state, topic instruction, persistent checkpointing, and interface behavior are implemented.
- The graph must run independently in a notebook cell before the Gradio interface is built.
- Use the PDF's installation requirements and compatible libraries: LangGraph, LangChain/OpenAI, `langgraph-checkpoint-postgres`, `psycopg[binary]`, and Gradio.

## Required behavior

- Use a shared state containing `messages` with the `add_messages` reducer and a `topic` value without a reducer. The current topic is part of checkpointed state.
- Implement exactly one `chatbot` node and exactly one LLM call per user message. Build a fresh `SystemMessage` from the current `state["topic"]` inside that node, send it with the message history, and return only the assistant reply. Do not store the system message in `messages`.
- Keep the bot on the selected club's topic. A child first selects a club; after that, answer only about that club, kindly refuse unrelated questions, and suggest switching clubs. After a topic switch, follow only the new topic, including when the child asks about a previous topic from earlier in the same conversation.
- Use the five clubs in the story (Astronomy, Cooking, Football, World Geography, and Python Programming), unless the project prompt specifies an equivalent list based on the PDF.
- Persist conversations in Supabase using LangGraph's `PostgresSaver`, connected with the Supabase Session pooler URL. Do not use `InMemorySaver` or a Python variable as the conversation store. Initialize the checkpoint tables with `checkpoint.setup()`.
- Use a stable, unique `thread_id` for each conversation. Load the saved conversation list from the database; reopening one must restore both messages and topic. A new conversation must create a new `thread_id` and clear the chat. Changing the club continues the same `thread_id` and updates the topic for the next message.
- Build the interface in Gradio with `gr.Blocks`, including a topic dropdown, chat window and message box, a New conversation button, and a readable list of saved conversations. Selecting a saved conversation restores its messages and topic. The conversation list should show useful identifying text, such as the topic and opening question, rather than only a UUID.
- Draw the graph in the notebook with `draw_mermaid_png()`.

## Secrets and sensitive data

- Never hardcode, display, print, or commit API keys, passwords, or database connection strings.
- In Colab, load `OPENAI_API_KEY` and `SUPABASE_DB_URL` from Google Colab Secrets using `google.colab.userdata.get`. Keep notebook access enabled for both secrets.
- Never paste a secret into notebook code, cell output, `prompt.md`, `AGENTS.md`, or GitHub. Never push secrets or other sensitive information to GitHub.
- Use the Supabase Session pooler URL as described in the PDF; keep the database password safe and out of the repository.

## Workflow and source of truth

- Follow the Homework 4 PDF and the project `prompt.md` as the source of truth. Keep this file consistent with them. Do not import requirements from earlier homework assignments when they conflict with Homework 4.
- Explain decisions clearly and keep code beginner-readable, with only the state, node, checkpointing, and interface behavior needed by the assignment.
- Do not commit or push automatically after small changes. Before each meaningful commit, summarize the proposed changes and ask the user for approval. Never commit or push secrets or sensitive information.
- The PDF asks the student to run, inspect, and fix the notebook in Colab, test topic restrictions and topic switching, and verify persistence after restarting the runtime. Do not claim those checks passed unless they were actually performed.
