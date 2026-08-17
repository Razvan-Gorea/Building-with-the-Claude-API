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


## Useful Links

- [Claude Python SDK](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/python)
- [Create a Message](https://platform.claude.com/docs/en/api/python/messages/create)
- [API Overview](https://platform.claude.com/docs/en/api/overview)