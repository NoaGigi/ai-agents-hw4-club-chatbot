# Codex prompt: Homework 4 Club Chatbot

Create and maintain `Club_Chatbot.ipynb` as a Google Colab-compatible notebook for Homework 4. The official `AI_Agents_Homework_LangGraph_Topic_Chatbot.pdf` is the primary source of truth; keep this prompt and `AGENTS.md` consistent with it. If they conflict, the PDF wins.

## Deliverable and notebook organization

- Put all implementation code in `Club_Chatbot.ipynb`; do not create standalone Python files unless explicitly requested.
- Use clear Markdown headings and explain important choices in beginner-readable language. Include these sections: Installation & Secrets, State & Prompt Guard, Supabase Checkpointer & Graph, Standalone Graph Test, and Gradio UI.
- The first code cell installs exactly the required packages:

  ```python
  !pip -q install langgraph langchain langchain-openai langgraph-checkpoint-postgres "psycopg[binary]" gradio
  ```

- Load `OPENAI_API_KEY` and `SUPABASE_DB_URL` from Google Colab Secrets with `google.colab.userdata.get`. Never put secrets in code or outputs.
- Define the model name once in a `MODEL_NAME` constant and initialize the model with `init_chat_model(MODEL_NAME)`. The PDF allows `openai:gpt-6-luna` or another low-cost model. Do not repeat a literal model name elsewhere.

## Clubs, state, and graph

- Use these five clubs: `Astronomy`, `Cooking`, `Football`, `World Geography`, and `Python Programming`.
- Define a `TypedDict` state with `messages: Annotated[list[AnyMessage], add_messages]` and `topic: str`. Do not add a reducer to `topic`, so the latest selected club replaces the previous one.
- Build a fresh `SystemMessage` from the current `state["topic"]` inside `chatbot` for every call. Do not save the system message in `messages`.
- The system instruction identifies the assistant as a tutor at Curious Minds after-school center and locks it to the current `{topic}` club. It must answer only questions related to that club. For anything outside the current topic, including sneaky framing and follow-ups about earlier topics still present in history, it must politely refuse, explicitly name the current club, and suggest switching clubs using the dropdown.
- The only graph node is `chatbot`. It makes exactly one LLM call with `[system_message] + state["messages"]` and returns `{"messages": [response]}`. Build `START -> chatbot -> END`; do not add other nodes, tools, loops, or memory mechanisms.
- Compile the graph with the Supabase checkpointer described below. Before building the UI, include a standalone notebook cell that invokes the graph and demonstrates topic locking, switching clubs in the same thread, and refusing a question about the previous topic after the switch. Inspect the saved state with `graph.get_state(config).values`. Include the homework's example sequence: Cooking question about pancakes, off-topic Football question while on Cooking, switch to Football, then ask about pancakes and verify the refusal.
- Render the graph with `display(Image(graph.get_graph().draw_mermaid_png()))`.

## Persistent Supabase checkpointing

- Connect using the Session pooler URL from the `SUPABASE_DB_URL` Colab Secret:

  ```python
  conn = psycopg.connect(
      userdata.get("SUPABASE_DB_URL"),
      autocommit=True,
      row_factory=dict_row,
      prepare_threshold=0,
  )
  ```

- Create `PostgresSaver(conn)`, call `checkpointer.setup()`, and compile with `builder.compile(checkpointer=checkpointer)`.
- Do not use `InMemorySaver` or a Python dict/list as the conversation store. The saved conversation list must be read from database checkpoints. UI-only state such as the currently selected `thread_id` is fine.
- Each conversation has its own `str(uuid.uuid4())` `thread_id`. Reopening a conversation restores its messages and topic. A new conversation creates a new ID, clears the visible chat, resets the topic to Astronomy, and refreshes the saved list.

## Gradio interface

- Use `gr.Blocks` and provide a club dropdown (default `Astronomy`), a saved-conversations dropdown, a `+ New conversation` button, a `gr.Chatbot`, and a text input with submit behavior.
- Read conversations directly from Supabase checkpoints. Show human-readable labels such as `"{topic} — {first_user_message[:35]}..."` while using the associated `thread_id` as the dropdown value. Do not make an in-memory list the source of saved conversations.
- Sending a message invokes:

  ```python
  graph.invoke(
      {
          "messages": [HumanMessage(content=msg)],
          "topic": current_topic,
      },
      {"configurable": {"thread_id": current_thread_id}},
  )
  ```

- After sending, update the chat display from the checkpointed messages and refresh the saved-conversations dropdown.
- Switching clubs keeps the exact same `thread_id`; the next message uses the new topic and preserves prior messages.
- Selecting a saved conversation calls `graph.get_state({"configurable": {"thread_id": selected_thread_id}}).values` and restores both messages and the saved topic in the UI.

## Safety and scope

- Never hardcode, display, print, commit, or push API keys, passwords, or database URIs. Use Colab Secrets only. Do not expose secrets in notebook outputs, this prompt, or GitHub.
- Keep the graph simple and aligned with the PDF. Optional bonus features are out of scope unless explicitly requested.
- Do not commit or push automatically. Before each meaningful commit, summarize the changes and ask the user for approval.
- The PDF asks the student to run the notebook in Colab, test topic restrictions and switching, and verify persistence after restarting the runtime. Do not claim these checks passed unless they were actually performed.
