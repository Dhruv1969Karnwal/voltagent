---
title: Using LLM Providers
slug: /agents/providers
---

# Using LLM Providers with Agents

In VoltAgent, **Providers** are the connection layer between your `Agent` and the underlying Large Language Model (LLM) or AI service. They implement the `LLMProvider` interface and handle the complexities of specific APIs, such as authentication, request formatting, and response parsing.

This abstraction allows you to build agents that are **model agnostic**. You can easily switch the underlying LLM (e.g., from OpenAI to Groq, or from a cloud service to a local model) often just by changing the provider configuration, without needing to modify your core agent logic.

## Configuring an Agent with a Provider

When you create an `Agent` instance, you assign a configured provider instance to the `llm` property and specify the desired model identifier using the `model` property.

```typescript
import { Agent } from "@voltagent/core";

// 1. Import your chosen provider
// Example: using the VercelAIProvider
import { VercelAIProvider } from "@voltagent/vercel-ai";
// Import the model definition from the Vercel AI SDK
import { openai } from "@ai-sdk/openai";

// 2. Instantiate and configure the provider
// VercelAIProvider often requires no constructor arguments (relies on env vars)
const myProvider = new VercelAIProvider();

// 3. Create the Agent, passing the provider and model ID
const agent = new Agent({
  name: "My Configured Agent",
  description: "An agent ready to interact via the specified provider",
  llm: myProvider, // Assign the provider instance
  model: openai("gpt-4o-mini"), // Pass the model object from Vercel AI SDK
});

// 4. Now you can use the agent's methods (generateText, streamText, etc.)
async function run() {
  const response = await agent.generateText("How does the provider architecture help?");
  console.log(response.text);
}

run();
```

## The `model` Parameter

It's important to understand that the value you provide for the `model` parameter is interpreted _by the specific `llm` provider_ you have configured for the agent.

- Some providers (like `@voltagent/xsai`, `@voltagent/google-ai`, `@voltagent/groq-ai`) expect a **string** identifier (e.g., `'gpt-4o-mini'`, `'gemini-1.5-pro'`, `'llama3-70b-8192'`) that corresponds to a model available on the target API.
- Other providers (like `@voltagent/vercel-ai`) expect a **model object** imported from their corresponding SDK (e.g., `openai('gpt-4o')`, `anthropic('claude-3-5-sonnet-20240620')`).

Always refer to the documentation for the specific provider you are using to know what format is expected for the `model` parameter.

## Available Providers

VoltAgent offers built-in providers for various popular services and SDKs, including Vercel AI, Google AI (Gemini/Vertex), Groq, and any OpenAI-compatible API endpoint (via xsAI).

**For detailed information on each available provider, including specific installation, configuration, and usage instructions, please see the [Providers Overview](../providers/overview.md).**

### `@voltagent/ollama` (Ollama AI Provider)

Connects to Ollama and allowing you to use your locally hosted Ollama models with Voltagent.

## Prerequisites

1. [Install Ollama](https://ollama.ai/download) on your machine
2. Pull the models you want to use:
   ```bash
   ollama pull llama3
   # or any other model you want to use
   ```
3. Ensure Ollama is running:
   ```bash
   ollama serve
   ```

## Usage

```typescript
import { Agent } from "@voltagent/core";
import { OllamaProvider } from "@voltagent/ollama";

// Initialize the Ollama provider
const ollama = new OllamaProvider({
  // Optional: provide a custom URL if Ollama is not running on the default localhost:11434
  baseUrl: "http://localhost:11434",
});

// Create an agent using the Ollama provider
const agent = new Agent({
  name: "LocalLLMAgent",
  description: "An agent that uses a local Ollama model",
  llm: ollama,
  // Specify the model name - this should match a model you've pulled in Ollama
  model: "llama3",
});

// Use the agent
const response = await agent.generateText("Hello, what can you do for me?");
console.log(response.text);
```

## Provider-Specific Options

When calling agent methods, you can pass provider-specific options:

```typescript
const response = await agent.generateText("What is the capital of France?", {
  provider: {
    temperature: 0.7,
    maxTokens: 500,
    topP: 0.9,
    seed: 123,
  },
});
```

## Available Models

The available models depend on what you've pulled to your Ollama instance. Common models include:

- `llama3`
- `mistral`
- `gemma`
- `phi`
- `llama2`
- `llama2-uncensored`
- `orca-mini`
- `codellama`
- `stable-code`

Check the [Ollama model library](https://ollama.ai/library) for the full list of available models.

## Example with Streaming

```typescript
const stream = await agent.streamText("Tell me a story about a robot");

// Process the stream
for await (const chunk of stream.textStream) {
  process.stdout.write(chunk);
}
```

**Choose @voltagent/ollama when:** You want to use models available through Ollama, which provides a convenient local-first framework for running open-source language models. Ollama is well-suited for tasks that benefit from customizable, offline-friendly models, including text generation, lightweight coding tasks, and privacy-sensitive applications.
