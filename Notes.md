## Claude Model Family

- Claude Opus: Highest intelligence, most expensive, use for the most complicated tasks, supports reasoning. Higher latency.
- Claude Sonnet: Balances speed, quality and cost. Best for common tasks. Supports reasoning.
- Claude Haiku: Most cost-efficient model, latency optimized, no reasoning support, use for simpliest tasks.

## Accessing the API (Example Workflow)

1. Client sends request to your server
2. Server uses Anthhrophic SDK or plain http request to send the client request to the Anthrophic API
3. The request hits the API and the following steps happen in the following order:
    - Tokenisation: entire input is split into tokens (can be seen as words)
    - Embedding: each token generates an embedding (a list of numbers representing meaning)
    - Contextualization: each embedding gets adjusted to those around it (adds context and refinement for every token)
    - Generation: final embeddings are sent to an output layer, probability + some randomness used to generate output
4. API sends output of the model back to your server, then client
5. Client renders the response

## Multi-Turn Conversations

Claude doesn't have memory, you need to create and store a list of messages that gets past to Claude.

## Prompts

- Prompt Engineering:
    - Set of best practices and guidance to improving your prompts

- Prompt Evaluation:
    - Automated testing to measure how well your prompts work
    - Run it through a pipeline to score it,then iterate on the prompt

## Prompt Eval Workflow

1. Initial Prompt Draft
2. Create an Eval Dataset (Questions)(Manually or AI generated)
3. Feed through Claude
4. Feed through a grader (get score for every question in eval dataset then average)
5. The averaged score is your prompt score, use it to determine if your prompt needs to be improved

## Graders:

- Code:
    - Programmatically evaluate the result
    - Useful for:
        - Checking output length
        - Readability
        - Syntax evaluation
        - Containing certain words

- Model:
    - Ask a model to assign a score to the output or compare two versions
    - Useful for:
        - Response quality
        - Helpfulness
        - Safety
        - Completeness
        - Quality of instruction following

- Human:
    - Ask a human to assign a score to the output or compare two versions
    

## Prompt Engineering

1. Set a goal
2. Write an initial prompt
3. Eval the prompt
4. Apply a prompt engineering technique
5. Re-eval to verify better performance
6. Repeat steps 4 to 5 until desired performance is achieved

## Prompt Engineering techniques:

- Be clear and direct:
    - Clear:
        - Use simple language
        - State what you want explicitly
        - Lead your prompt with a simple statement of the model's task
    - Direct:
        - Use instructions, not questions
        - Use direct action verbs
- Be specific:
    - List out guidelines:
        - List qualities that the output should have
        - Provide steps the model should follow

    - Use qualities for output on every prompt
    - Use steps when you want claude to take a specific path of critical thinking, decision making or to consider a wider view
    - If possible try to combine both approaches (qualities & steps)

- Providing structure:
    - XML tags to separate distinct portions of the prompt (useful when including lots of content)

- Provide Examples:
    - Give Claude sample input/output pairs:
        - Useful for edge cases or complex output formats
        - "One-Shot": provide a single example
        - "Multi-Shot": provide multiple examples
        - Recommended combining with XML tags for structure!
        - Explain why your example outputs are considered ideal
        - Keep examples relevant to your specific task
        - Include examples that address your most common failure cases

## Tool Use

- Helps get current up to date data that Claude can use

- Example tool use workflow:
    1. Initial Request: You send Claude a question along with instructions on how to get extra data from external sources
    2. Tool Request: Claude analyzes the question and decides it needs additional information, then asks for specific details about what data it needs
    3. Data Retrieval: Your server runs code to fetch the requested information from external APIs or databases
    4. Final Response: You send the retrieved data back to Claude, which then generates a complete response using both the original question and the fresh data

## Tool Function

- Plain Python function that will be executed when Claude decides it needs some additional information to help the user
- Best Practices:
    - Use well-named, descriptive arguments
    - Validate the inputs, raising an error if they fail validation
    - Return meaningful errors - Claude will try to call to use you function a second time.

## JSON Schema For Tools

- The JSON Schema helps Claude understand what arguements your function requires
- Best Practices:
    - Explain what the tool does, when to use it, and what it returns
    - Aim for 3-4 sentences
    - Provide super detailed descriptions

`tools = [
    {
        "name": "get_weather",
        "description": "Get the current weather for a given location.",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City and state, e.g. San Francisco, CA",
                }
            },
            "required": ["location"],
        },
    }
]`

## Running the Tool Function

- When Claude responds with a tool use block, you extract the input parameters and call your function. Here's how to access the tool parameters:

`response.content[0].input`

- This gives you a dictionary of the arguments Claude wants to pass to your function. Since your function expects keyword arguments rather than a dictionary, you use Python's unpacking syntax:

`get_current_datetime(**response.content[0].input)`

## Tool Result Block

- After running the tool function, you need to send the results back to Claude using a tool result block This block goes inside a user message and tells Claude what happened when you executed the tool.

- The tool result block has several important properties:
    - `tool_use_id` - Must match the id of the ToolUse block that this ToolResult corresponds to
    - `content` - Output from running your tool, serialized as a string
    - `is_error` - True if an error occurred

## Sending Tool Results to Claude — Step by Step

1. **Claude requests a tool call**
   Claude responds with a tool use block containing the input parameters it wants to pass to your function (e.g. `response.content[0]` holds the ToolUse block, with `.input` giving the argument dictionary).

2. **Extract parameters and run the function**
   Pull the arguments from `response.content[0].input`, then unpack them into your function using Python's `**` syntax, e.g. `get_current_datetime(**response.content[1].input)`.

3. **Handle multiple tool calls if present**
   Claude can request several tool calls in one response (e.g. two separate calculations). Each has a unique `id`, and you must match results back to the correct call using that id — even if results come back out of order.

4. **Build a tool result block**
   Package your function's output into a `tool_result` block with three key fields:
   - `tool_use_id` — must match the originating ToolUse block's `id`
   - `content` — your function's output, serialized as a string
   - `is_error` — `True` if the tool call failed

5. **Append the tool result as a user message**
   Add a new message to your `messages` list with `role: "user"` and `content` set to the tool_result block, so Claude sees what happened when the tool ran.

6. **Send the follow-up request with full history**
   Call `client.messages.create` again, including the complete conversation history (original user message, assistant's tool use message, and your new tool result message) plus the `tools` schema — Claude still needs the schema to understand the tool references, even though no further call is expected.

7. **Receive Claude's final natural-language response**
   Claude incorporates the tool result into a coherent answer for the user, completing the tool use workflow.

## Retrieval Augumented Generation

- Option 1: Give all the extracted text to claude
    - Pro: Straightforward approach
    - Con: Costs more to process, takes longer to process

- Option 2: Break documents into chunks, only give the most relevant chunks to claude with the users prompt
    - Pro: Scales well up to large documents, runs faster and costs less, works well with multiple documents
    - Con: Requires preprocessing step to chunk documents. Needs a search mechanism to find relevant chunks

## Chunking Strategies

- Structure-based: Best results when you control document formatting (like internal company reports)
- Sentence-based: Good middle ground for most text documents
- Size-based: Most reliable fallback that works with any content type, including code

## The Full RAG Flow

1. Chunk source text
2. Generate embeddings (using an embedding model), then normalize
3. Store embeddings in a vector database e.g Pinecone, pgvector store
4. Embed the user's query with the same embedding model and normalize
5. Send the query (embedding form) to the vector database
6. Find similar embeddings to the user query using cosine similarity for example
7. Take the user query plus most relevant text chunk and send it to claude for a response

## Cosine Similarity

Key points about cosine similarity:
    
    - Results range from -1 to 1
    - Values close to 1 mean high similarity
    - Values close to -1 mean very different
    - 0 means perpendicular (no relationship)

## Cosine Distance

Key points about cosine distance:

    - Calculated as 1 - cosine similarity
    - Same direction, 0.0
    - If perpendicular, 1.0
    - If complete opposite, 2.0

## Useful Links

- [Claude Python SDK](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/python)
- [Create a Message](https://platform.claude.com/docs/en/api/python/messages/create)
- [API Overview](https://platform.claude.com/docs/en/api/overview)
- [Tool use with Claude](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)
- [VoyageAI](https://www.voyageai.com/)