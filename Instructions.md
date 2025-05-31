### Goal

Enable a locally hosted Ollama model to act as an autonomous **agent** that decides when to call your Python functions (“tools”), runs them, and weaves the results into its final answer.

---

### Step-by-step agent playbook

1. **Select a tool-aware model**
   Pull any Ollama build that advertises “tool” support, for example

   ```bash
   ollama pull qwen2.5:tool
   ```

2. **Declare each tool in code**

   ```python
   def get_weather(city: str) -> str: ...
   tools = [{
       "name": "get_weather",
       "description": "Return weather like 'Toronto: 12 °C, rain'.",
       "parameters": {
           "type": "object",
           "properties": {"city": {"type": "string"}},
           "required": ["city"]
       },
       "function": get_weather          # actual Python handle
   }]
   ```

3. **Craft a concise system prompt (best-practice template)**

   ```
   You are an assistant with access to external tools.

   Available tools:
   • get_weather(city: str) – looks up current weather.

   Rules:
   1. If a tool is needed, respond ONLY with a valid JSON object:
      { "name": "<tool_name>", "arguments": { ... } }
   2. After you receive the tool result, reply to the user in natural language.
   3. If no tool is needed, answer directly.
   ```

4. **Initialize chat history**

   ```python
   history = [{"role": "system", "content": SYSTEM_PROMPT}]
   ```

5. **Main loop**

   ```python
   while True:
       resp = ollama.chat(model="qwen2.5:tool",
                          messages=history,
                          tools=tools,
                          stream=False)
       msg = resp["message"]

       if "tool_calls" in msg:          # model requested a tool
           call = msg["tool_calls"][0]
           fn   = call["name"]
           args = call["arguments"]
           result = next(t["function"] for t in tools
                         if t["name"] == fn)(**args)

           history.append({
               "role": "assistant",
               "tool_call_id": call["id"],
               "name": fn,
               "content": result
           })
           continue                      # loop so model can use result
       else:
           print("Assistant:", msg["content"])
           break
   ```

6. **Follow output hygiene**

   * Tool functions must return short, deterministic strings or JSON.
   * Keep model `temperature` 0 – 0.2 to reduce hallucinated calls.
   * Retry the step if the model outputs invalid JSON.

7. **(Optional) Hand control to a framework**
   Point LangChain, CrewAI, or LiteLLM at `http://localhost:11434/v1` and choose their “OpenAI-tools” agent type; they will run the same loop for you.

With this recipe the model can autonomously decide when a tool is useful, call it, get your code’s output, and deliver a coherent answer to the user.
