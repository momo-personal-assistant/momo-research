## Background: why is session management necessary?

- context windows are limited
- high costs
- need to figure out how to know what content to throw out without losing context

## Key takeaway:

- overly aggressive compaction strategies lead to info loss
- you can’t predict which info is going to be vaulable 10 steps further, so you need to find a way to preserve it in a compacted way

## List of compaction strategies:

1. Keep the last N turns: only keeps most recent N turns of conversation and discards everything older → not recommended
   - depends on which value is needed for the specific app you’re trying to build
   - if the agent’s task is confined to a very particular task with expected outcomes, info that needs to be retained becomes extremely clear. in this case, semantic compaction would be more reasonable
2. Token-based Truncation: includes as many messages as possible within predefined token limit → doesn’t really help as well
3. Recursive Summarization: Older parts of conversation are replaced by AI generated summary → better cuz it doesn’t completely erase older histories
4. design around kv-cache hit rate (memory of past tokens model can reuse)
   - kv-cache exists because generating new tokens is expensive
   - agents have huge prefills (reading all previous context) and tiny decodes (producing the next token), so kv-cache helps skip recomputing all previous tokens
   - kv-cache hit means it found the prefix to reuse the cache. but even one token difference can breake the cache
   - common mistakes that cause cache breaks:
     - changing system prompts by adding timestamps
     - changing tool definitions: They usually sit at the front of the context. Changing them invalidates everything
     - non-deterministic serialization (ex. JSON objects with shuffled key order)
     - removing or inserting lines in the prefix. even whitespace or one token difference breaks it
   - How to keep it high:
     - Keep your prompt stable and deterministic
     - Make contexts append-only (never rewrite earlier messages)
     - Avoid dynamic tool loading (use masking instead)
     - Use session IDs or prefix caching if self-hosting
5. dont dynamically add/remove tools mid-iteration cuz it breaks the kv-cache
   - agents usually use hundreds of tools (browser*, shell*, search*, email*, etc)
   - if you dynamically remove tools, it destroys the kv-cache
     - ex.
     - Step 1: model uses `browser_open()`
     - Step 3: you removed that tool definition
       → The past context still says “browser_open()”, but the tool is no longer defined now. → leads to hallucination
   - mask the tools instead: keep all the tools in context, but just show the ones that the agent needs to see at that point
6. using the “file system” as context: content of web page can be dropped from context as long as the URL is preserved → this is designed to be restorable
7. manipulate attention by constantly rewrititing the todo list, and recite the objectives into the end of the context → so that it focuses on the most recent attention span
8. keep wrong stuff in: not erasing hallucinations or tool call failures, to keep evidence so that mdoel can adapt
9. don’t get few-shot too much stuff cuz it can backfire. if context is full of similar past action-observation pairs, the model will tend to follow that pattern even when it’s no longer optimal → increase diversity by controlled randomness

...more to come in this list.
