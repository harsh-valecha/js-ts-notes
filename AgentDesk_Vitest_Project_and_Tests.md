# AgentDesk — Complete Beginner-Friendly TypeScript Project + Vitest Tests

This document contains the **actual project code and tests** for the AgentDesk application described in the attached Vitest guide.

The goal is simple:

- copy the files exactly
- run the commands
- understand one file at a time
- run the tests
- then change the code and watch the tests protect you

The project intentionally uses simple TypeScript instead of a large framework.

---

# 1. What We Are Building

AgentDesk is a small backend-style AI application:

```text
User Task
   |
   v
Agent Manager / Workflow
   |
   +------> Researcher
   |
   +------> Coder
   |
   +------> Reviewer
   |
   v
AI Service
   |
   +------> Cache
   |
   +------> Ollama Cloud
   |
   v
SQLite
   |
   v
Conversation History
```

The application demonstrates:

- TypeScript
- Vitest
- unit tests
- mocks
- spies
- async tests
- SQLite
- repository pattern
- cache + TTL
- fake timers
- Ollama Cloud
- chat history
- chat compaction
- agents
- agent-to-agent communication
- workflow testing
- retries
- timeouts
- integration tests
- end-to-end-style tests

---

# 2. Final Project Structure

Create this structure:

```text
agentdesk/
│
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── .gitignore
├── .env.example
│
├── src/
│   ├── types.ts
│   │
│   ├── ai/
│   │   ├── ai-service.ts
│   │   └── ollama-service.ts
│   │
│   ├── agents/
│   │   ├── researcher.ts
│   │   ├── coder.ts
│   │   ├── reviewer.ts
│   │   └── agent-manager.ts
│   │
│   ├── cache/
│   │   └── memory-cache.ts
│   │
│   ├── chat/
│   │   ├── conversation.ts
│   │   └── compaction.ts
│   │
│   ├── db/
│   │   ├── database.ts
│   │   └── message-repository.ts
│   │
│   ├── messaging/
│   │   └── message-bus.ts
│   │
│   ├── workflow/
│   │   └── workflow.ts
│   │
│   └── utils/
│       ├── token-counter.ts
│       ├── retry.ts
│       └── timeout.ts
│
└── tests/
    ├── unit/
    │   ├── token-counter.test.ts
    │   ├── cache.test.ts
    │   ├── retry.test.ts
    │   ├── timeout.test.ts
    │   ├── conversation.test.ts
    │   ├── compaction.test.ts
    │   ├── researcher.test.ts
    │   ├── coder.test.ts
    │   └── reviewer.test.ts
    │
    ├── integration/
    │   ├── repository.test.ts
    │   ├── message-bus.test.ts
    │   └── workflow.test.ts
    │
    └── e2e/
        └── complete-workflow.test.ts
```

---

# 3. Install Everything

Create the project:

```bash
mkdir agentdesk
cd agentdesk
npm init -y
```

Install runtime packages:

```bash
npm install better-sqlite3 dotenv
```

Install development packages:

```bash
npm install -D typescript vitest @types/node @types/better-sqlite3
```

Then create TypeScript configuration:

```bash
npx tsc --init
```

---

# 4. package.json

Replace your `package.json` with:

```json
{
  "name": "agentdesk",
  "version": "1.0.0",
  "description": "Beginner-friendly TypeScript application for learning Vitest",
  "type": "module",
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:unit": "vitest run tests/unit",
    "test:integration": "vitest run tests/integration",
    "test:e2e": "vitest run tests/e2e",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  },
  "dependencies": {
    "better-sqlite3": "^11.0.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "@types/better-sqlite3": "^7.6.0",
    "@types/node": "^22.0.0",
    "typescript": "^5.0.0",
    "vitest": "^3.0.0"
  }
}
```

If npm installs newer compatible versions, that is fine.

---

# 5. tsconfig.json

Use:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "types": ["node", "vitest/globals"],
    "outDir": "dist"
  },
  "include": ["src", "tests", "vitest.config.ts"]
}
```

---

# 6. vitest.config.ts

Create:

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: false,
    environment: "node",
    clearMocks: true,
    restoreMocks: true
  }
});
```

We keep `globals: false` because importing `describe`, `it`, and `expect` makes it easier for a beginner to see where these functions come from.

---

# 7. .gitignore

Create:

```gitignore
node_modules/
dist/
.env
*.db
coverage/
```

---

# 8. .env.example

Create:

```env
OLLAMA_API_KEY=your_ollama_cloud_api_key
OLLAMA_MODEL=your_model_name
OLLAMA_URL=https://ollama.com/api/chat
```

Copy it to `.env` only if you want to run a real Ollama Cloud request.

Never commit `.env`.

---

# 9. Common Types

## src/types.ts

```ts
export type MessageRole =
  | "system"
  | "user"
  | "assistant"
  | "tool";

export interface Message {
  id?: number;
  conversationId?: string;
  role: MessageRole;
  content: string;
  createdAt?: string;
}

export interface AIService {
  chat(messages: Message[]): Promise<string>;
}

export interface Agent {
  name: string;
  run(task: string): Promise<string>;
}

export interface AgentMessage {
  id?: number;
  from: string;
  to: string;
  type: string;
  payload: unknown;
  createdAt?: string;
}
```

---

# 10. Token Counter

This is intentionally simple.

It is **not a real LLM tokenizer**.

It gives us an easy deterministic utility for learning tests.

## src/utils/token-counter.ts

```ts
export function countTokens(text: string): number {
  const cleaned = text.trim();

  if (!cleaned) {
    return 0;
  }

  return cleaned.split(/\s+/).length;
}
```

---

# 11. Token Counter Tests

## tests/unit/token-counter.test.ts

```ts
import { describe, expect, it } from "vitest";
import { countTokens } from "../../src/utils/token-counter.js";

describe("countTokens", () => {
  it("counts words", () => {
    expect(countTokens("hello world")).toBe(2);
  });

  it("handles multiple spaces", () => {
    expect(countTokens("hello     world")).toBe(2);
  });

  it("handles one word", () => {
    expect(countTokens("hello")).toBe(1);
  });

  it("returns zero for empty text", () => {
    expect(countTokens("")).toBe(0);
  });

  it("returns zero for whitespace", () => {
    expect(countTokens("     ")).toBe(0);
  });
});
```

---

# 12. Memory Cache

The cache stores values in memory.

Each value has an expiry time.

## src/cache/memory-cache.ts

```ts
interface CacheEntry {
  value: string;
  expiresAt: number;
}

export class MemoryCache {
  private entries = new Map<string, CacheEntry>();

  get(key: string): string | null {
    const entry = this.entries.get(key);

    if (!entry) {
      return null;
    }

    if (Date.now() >= entry.expiresAt) {
      this.entries.delete(key);
      return null;
    }

    return entry.value;
  }

  set(
    key: string,
    value: string,
    ttlSeconds: number
  ): void {
    if (ttlSeconds <= 0) {
      throw new Error("TTL must be greater than zero");
    }

    const expiresAt =
      Date.now() + ttlSeconds * 1000;

    this.entries.set(key, {
      value,
      expiresAt
    });
  }

  delete(key: string): void {
    this.entries.delete(key);
  }

  clear(): void {
    this.entries.clear();
  }

  size(): number {
    return this.entries.size;
  }
}
```

---

# 13. Cache Tests

## tests/unit/cache.test.ts

```ts
import {
  afterEach,
  describe,
  expect,
  it,
  vi
} from "vitest";

import { MemoryCache } from "../../src/cache/memory-cache.js";

describe("MemoryCache", () => {
  let cache: MemoryCache;

  beforeEach(() => {
    cache = new MemoryCache();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it("returns null for a missing key", () => {
    expect(cache.get("missing")).toBeNull();
  });

  it("stores and returns a value", () => {
    cache.set("answer", "42", 60);

    expect(cache.get("answer")).toBe("42");
  });

  it("overwrites an existing value", () => {
    cache.set("answer", "42", 60);
    cache.set("answer", "43", 60);

    expect(cache.get("answer")).toBe("43");
  });

  it("expires a value after TTL", () => {
    vi.useFakeTimers();

    cache.set("answer", "42", 60);

    expect(cache.get("answer")).toBe("42");

    vi.advanceTimersByTime(60_000);

    expect(cache.get("answer")).toBeNull();
  });

  it("throws when TTL is zero", () => {
    expect(() => {
      cache.set("answer", "42", 0);
    }).toThrow("TTL must be greater than zero");
  });

  it("deletes a value", () => {
    cache.set("answer", "42", 60);

    cache.delete("answer");

    expect(cache.get("answer")).toBeNull();
  });

  it("clears all values", () => {
    cache.set("one", "1", 60);
    cache.set("two", "2", 60);

    cache.clear();

    expect(cache.size()).toBe(0);
  });
});
```

Notice the important testing idea:

```ts
vi.useFakeTimers();
```

We do not actually wait 60 seconds.

---

# 14. Retry Utility

## src/utils/retry.ts

```ts
export async function retry<T>(
  operation: () => Promise<T>,
  maxAttempts: number
): Promise<T> {
  if (maxAttempts < 1) {
    throw new Error(
      "maxAttempts must be at least 1"
    );
  }

  let lastError: unknown;

  for (
    let attempt = 1;
    attempt <= maxAttempts;
    attempt++
  ) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;
    }
  }

  throw lastError;
}
```

This is deliberately simple.

There is no delay yet.

That makes it easier to learn the retry concept first.

---

# 15. Retry Tests

## tests/unit/retry.test.ts

```ts
import {
  describe,
  expect,
  it,
  vi
} from "vitest";

import { retry } from "../../src/utils/retry.js";

describe("retry", () => {
  it("returns immediately when operation succeeds", async () => {
    const operation = vi
      .fn()
      .mockResolvedValue("success");

    const result = await retry(
      operation,
      3
    );

    expect(result).toBe("success");
    expect(operation).toHaveBeenCalledTimes(1);
  });

  it("retries after failures", async () => {
    const operation = vi
      .fn()
      .mockRejectedValueOnce(
        new Error("network")
      )
      .mockRejectedValueOnce(
        new Error("network")
      )
      .mockResolvedValueOnce("success");

    const result = await retry(
      operation,
      3
    );

    expect(result).toBe("success");
    expect(operation).toHaveBeenCalledTimes(3);
  });

  it("throws after all attempts fail", async () => {
    const operation = vi
      .fn()
      .mockRejectedValue(
        new Error("network")
      );

    await expect(
      retry(operation, 3)
    ).rejects.toThrow("network");

    expect(operation).toHaveBeenCalledTimes(3);
  });

  it("rejects an invalid attempt count", async () => {
    const operation = vi
      .fn()
      .mockResolvedValue("success");

    await expect(
      retry(operation, 0)
    ).rejects.toThrow(
      "maxAttempts must be at least 1"
    );

    expect(operation).not.toHaveBeenCalled();
  });
});
```

This demonstrates:

```ts
vi.fn()
mockResolvedValue()
mockRejectedValue()
mockRejectedValueOnce()
toHaveBeenCalledTimes()
rejects.toThrow()
```

---

# 16. Timeout Utility

We need a way to stop waiting for an operation after a certain amount of time.

## src/utils/timeout.ts

```ts
export function withTimeout<T>(
  operation: Promise<T>,
  milliseconds: number
): Promise<T> {
  if (milliseconds <= 0) {
    return Promise.reject(
      new Error("Timeout must be greater than zero")
    );
  }

  const timeout = new Promise<never>(
    (_, reject) => {
      const timer = setTimeout(() => {
        reject(new Error("Operation timed out"));
      }, milliseconds);

      // This prevents the timer from keeping Node alive
      // after the operation finishes first.
      timer.unref?.();
    }
  );

  return Promise.race([
    operation,
    timeout
  ]);
}
```

---

# 17. Timeout Tests

## tests/unit/timeout.test.ts

```ts
import {
  afterEach,
  describe,
  expect,
  it,
  vi
} from "vitest";

import { withTimeout } from "../../src/utils/timeout.js";

describe("withTimeout", () => {
  afterEach(() => {
    vi.useRealTimers();
  });

  it("returns the operation result", async () => {
    const result = await withTimeout(
      Promise.resolve("success"),
      5_000
    );

    expect(result).toBe("success");
  });

  it("rejects when operation takes too long", async () => {
    vi.useFakeTimers();

    const slowOperation = new Promise<string>(
      () => {
        // Never resolves.
      }
    );

    const promise = withTimeout(
      slowOperation,
      5_000
    );

    const assertion = expect(
      promise
    ).rejects.toThrow(
      "Operation timed out"
    );

    vi.advanceTimersByTime(5_000);

    await assertion;
  });

  it("rejects invalid timeout values", async () => {
    await expect(
      withTimeout(
        Promise.resolve("hello"),
        0
      )
    ).rejects.toThrow(
      "Timeout must be greater than zero"
    );
  });

  it("passes through an operation error", async () => {
    await expect(
      withTimeout(
        Promise.reject(
          new Error("AI failed")
        ),
        5_000
      )
    ).rejects.toThrow("AI failed");
  });
});
```

---

# 18. Conversation

A conversation is simply an array of messages.

## src/chat/conversation.ts

```ts
import type { Message } from "../types.js";

export function addMessage(
  messages: Message[],
  message: Message
): Message[] {
  return [
    ...messages,
    message
  ];
}

export function getRecentMessages(
  messages: Message[],
  count: number
): Message[] {
  if (count <= 0) {
    return [];
  }

  return messages.slice(-count);
}
```

The original array is not modified.

That makes this function easy to reason about.

---

# 19. Conversation Tests

## tests/unit/conversation.test.ts

```ts
import {
  describe,
  expect,
  it
} from "vitest";

import {
  addMessage,
  getRecentMessages
} from "../../src/chat/conversation.js";

describe("conversation", () => {
  it("adds a message", () => {
    const messages = [];

    const result = addMessage(
      messages,
      {
        role: "user",
        content: "Hello"
      }
    );

    expect(result).toHaveLength(1);
    expect(result[0].content)
      .toBe("Hello");

    // Original array is unchanged.
    expect(messages).toHaveLength(0);
  });

  it("returns the requested recent messages", () => {
    const messages = [
      {
        role: "user" as const,
        content: "one"
      },
      {
        role: "assistant" as const,
        content: "two"
      },
      {
        role: "user" as const,
        content: "three"
      }
    ];

    const result =
      getRecentMessages(messages, 2);

    expect(result).toHaveLength(2);
    expect(result[0].content)
      .toBe("two");
    expect(result[1].content)
      .toBe("three");
  });

  it("returns an empty array for zero", () => {
    const result =
      getRecentMessages(
        [
          {
            role: "user",
            content: "hello"
          }
        ],
        0
      );

    expect(result).toEqual([]);
  });
});
```

---

# 20. Chat Compaction

We will compact a conversation when it becomes too large.

The algorithm is intentionally simple:

```text
messages <= limit
       |
       v
return messages unchanged

messages > limit
       |
       v
keep system message
       +
summarize old messages
       +
keep recent messages
```

## src/chat/compaction.ts

```ts
import type {
  AIService,
  Message
} from "../types.js";

export async function compactConversation(
  messages: Message[],
  ai: AIService,
  maxMessages: number,
  recentCount: number = 4
): Promise<Message[]> {
  if (messages.length <= maxMessages) {
    return messages;
  }

  const systemMessage = messages.find(
    (message) => message.role === "system"
  );

  const nonSystemMessages =
    messages.filter(
      (message) =>
        message.role !== "system"
    );

  const recentMessages =
    nonSystemMessages.slice(-recentCount);

  const oldMessages =
    nonSystemMessages.slice(
      0,
      Math.max(
        0,
        nonSystemMessages.length -
          recentCount
      )
    );

  const textToSummarize =
    oldMessages
      .map(
        (message) =>
          `${message.role}: ${message.content}`
      )
      .join("\n");

  const summary =
    await ai.chat([
      {
        role: "system",
        content:
          "Summarize the conversation briefly."
      },
      {
        role: "user",
        content: textToSummarize
      }
    ]);

  const result: Message[] = [];

  if (systemMessage) {
    result.push(systemMessage);
  }

  result.push({
    role: "system",
    content: `Conversation summary: ${summary}`
  });

  result.push(...recentMessages);

  return result;
}
```

---

# 21. Compaction Tests

## tests/unit/compaction.test.ts

```ts
import {
  describe,
  expect,
  it,
  vi
} from "vitest";

import { compactConversation } from "../../src/chat/compaction.js";

describe("compactConversation", () => {
  it("does not compact a short conversation", async () => {
    const ai = {
      chat: vi.fn()
    };

    const messages = [
      {
        role: "user" as const,
        content: "Hello"
      },
      {
        role: "assistant" as const,
        content: "Hi"
      }
    ];

    const result =
      await compactConversation(
        messages,
        ai,
        5
      );

    expect(result).toEqual(messages);
    expect(ai.chat).not.toHaveBeenCalled();
  });

  it("compacts a long conversation", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockResolvedValue(
          "The user discussed Vitest."
        )
    };

    const messages = [
      {
        role: "system" as const,
        content: "You are helpful."
      },
      {
        role: "user" as const,
        content: "I want to learn testing."
      },
      {
        role: "assistant" as const,
        content: "Start with unit tests."
      },
      {
        role: "user" as const,
        content: "Then learn mocks."
      },
      {
        role: "assistant" as const,
        content: "Use vi.fn."
      },
      {
        role: "user" as const,
        content: "What about integration?"
      }
    ];

    const result =
      await compactConversation(
        messages,
        ai,
        4,
        2
      );

    expect(ai.chat)
      .toHaveBeenCalledTimes(1);

    expect(result).toContainEqual({
      role: "system",
      content:
        "You are helpful."
    });

    expect(result).toContainEqual({
      role: "system",
      content:
        "Conversation summary: The user discussed Vitest."
    });

    expect(result).toContainEqual({
      role: "user",
      content:
        "What about integration?"
    });
  });

  it("propagates AI errors", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockRejectedValue(
          new Error("AI unavailable")
        )
    };

    const messages = Array.from(
      { length: 6 },
      (_, index) => ({
        role: "user" as const,
        content: `Message ${index}`
      })
    );

    await expect(
      compactConversation(
        messages,
        ai,
        4
      )
    ).rejects.toThrow(
      "AI unavailable"
    );
  });
});
```

---

# 22. AI Service Interface

The application should not directly depend on Ollama everywhere.

## src/ai/ai-service.ts

```ts
import type { AIService } from "../types.js";

export type { AIService };
```

The important part is the interface in `src/types.ts`:

```ts
export interface AIService {
  chat(messages: Message[]): Promise<string>;
}
```

This lets us replace the real AI service with a fake one in unit tests.

---

# 23. Ollama Cloud Service

We will use the Ollama HTTP API directly with `fetch`.

That keeps the example small and makes the HTTP boundary visible.

## src/ai/ollama-service.ts

```ts
import type {
  AIService,
  Message
} from "../types.js";

interface OllamaResponse {
  message?: {
    role?: string;
    content?: string;
  };
}

export interface OllamaServiceOptions {
  apiKey: string;
  model: string;
  url?: string;
}

export class OllamaService
  implements AIService
{
  private readonly apiKey: string;
  private readonly model: string;
  private readonly url: string;

  constructor(
    options: OllamaServiceOptions
  ) {
    this.apiKey = options.apiKey;
    this.model = options.model;
    this.url =
      options.url ??
      "https://ollama.com/api/chat";
  }

  async chat(
    messages: Message[]
  ): Promise<string> {
    const response = await fetch(
      this.url,
      {
        method: "POST",
        headers: {
          "Content-Type":
            "application/json",
          Authorization:
            `Bearer ${this.apiKey}`
        },
        body: JSON.stringify({
          model: this.model,
          messages: messages.map(
            (message) => ({
              role: message.role,
              content: message.content
            })
          ),
          stream: false
        })
      }
    );

    if (!response.ok) {
      throw new Error(
        `Ollama request failed: ${response.status}`
      );
    }

    const data =
      (await response.json()) as OllamaResponse;

    const content =
      data.message?.content;

    if (!content) {
      throw new Error(
        "Ollama response did not contain message content"
      );
    }

    return content;
  }
}
```

---

# 24. Testing Ollama Without Calling Ollama

Most tests should **not** call the cloud.

Instead, create a fake:

```ts
const ai = {
  chat: vi
    .fn()
    .mockResolvedValue("fake response")
};
```

Then inject it into an agent.

This is much faster and deterministic.

---

# 25. Researcher Agent

## src/agents/researcher.ts

```ts
import type {
  AIService,
  Agent
} from "../types.js";

export class ResearcherAgent
  implements Agent
{
  readonly name = "researcher";

  constructor(
    private readonly ai: AIService
  ) {}

  async run(
    task: string
  ): Promise<string> {
    return this.ai.chat([
      {
        role: "system",
        content:
          "You are a research agent. Give useful research for another developer."
      },
      {
        role: "user",
        content: task
      }
    ]);
  }
}
```

---

# 26. Researcher Tests

## tests/unit/researcher.test.ts

```ts
import {
  describe,
  expect,
  it,
  vi
} from "vitest";

import { ResearcherAgent } from "../../src/agents/researcher.js";

describe("ResearcherAgent", () => {
  it("returns the AI research result", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockResolvedValue(
          "Vitest uses expect for assertions."
        )
    };

    const agent =
      new ResearcherAgent(ai);

    const result =
      await agent.run(
        "Research Vitest assertions"
      );

    expect(result).toBe(
      "Vitest uses expect for assertions."
    );

    expect(ai.chat)
      .toHaveBeenCalledTimes(1);
  });

  it("passes the task to the AI", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockResolvedValue("research")
    };

    const agent =
      new ResearcherAgent(ai);

    await agent.run("Research SQLite");

    expect(ai.chat).toHaveBeenCalledWith(
      [
        {
          role: "system",
          content:
            "You are a research agent. Give useful research for another developer."
        },
        {
          role: "user",
          content: "Research SQLite"
        }
      ]
    );
  });

  it("propagates AI failures", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockRejectedValue(
          new Error("Ollama unavailable")
        )
    };

    const agent =
      new ResearcherAgent(ai);

    await expect(
      agent.run("Research Vitest")
    ).rejects.toThrow(
      "Ollama unavailable"
    );
  });
});
```

---

# 27. Coder Agent

The coder receives research and creates an implementation.

## src/agents/coder.ts

```ts
import type {
  AIService,
  Agent
} from "../types.js";

export class CoderAgent
  implements Agent
{
  readonly name = "coder";

  constructor(
    private readonly ai: AIService
  ) {}

  async run(
    task: string
  ): Promise<string> {
    return this.ai.chat([
      {
        role: "system",
        content:
          "You are a coding agent. Produce a simple implementation."
      },
      {
        role: "user",
        content: task
      }
    ]);
  }

  async runWithResearch(
    task: string,
    research: string
  ): Promise<string> {
    return this.ai.chat([
      {
        role: "system",
        content:
          "You are a coding agent. Use the research to create a simple implementation."
      },
      {
        role: "user",
        content:
          `Task:\n${task}\n\nResearch:\n${research}`
      }
    ]);
  }
}
```

---

# 28. Coder Tests

## tests/unit/coder.test.ts

```ts
import {
  describe,
  expect,
  it,
  vi
} from "vitest";

import { CoderAgent } from "../../src/agents/coder.js";

describe("CoderAgent", () => {
  it("returns the AI coding result", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockResolvedValue(
          "function add(a, b) { return a + b; }"
        )
    };

    const agent =
      new CoderAgent(ai);

    const result =
      await agent.run("Create an add function");

    expect(result).toContain(
      "function add"
    );
  });

  it("passes research to the AI", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockResolvedValue("code")
    };

    const agent =
      new CoderAgent(ai);

    await agent.runWithResearch(
      "Create a cache",
      "Use a Map with expiry times"
    );

    expect(ai.chat).toHaveBeenCalledWith(
      [
        {
          role: "system",
          content:
            "You are a coding agent. Use the research to create a simple implementation."
        },
        {
          role: "user",
          content:
            "Task:\nCreate a cache\n\nResearch:\nUse a Map with expiry times"
        }
      ]
    );
  });
});
```

---

# 29. Reviewer Agent

The reviewer gives a simple `PASS` or `FAIL`.

## src/agents/reviewer.ts

```ts
import type {
  AIService,
  Agent
} from "../types.js";

export class ReviewerAgent
  implements Agent
{
  readonly name = "reviewer";

  constructor(
    private readonly ai: AIService
  ) {}

  async run(
    task: string
  ): Promise<string> {
    const result =
      await this.ai.chat([
        {
          role: "system",
          content:
            "You are a code reviewer. Reply with PASS or FAIL followed by a short reason."
        },
        {
          role: "user",
          content: task
        }
      ]);

    return result;
  }
}
```

---

# 30. Reviewer Tests

## tests/unit/reviewer.test.ts

```ts
import {
  describe,
  expect,
  it,
  vi
} from "vitest";

import { ReviewerAgent } from "../../src/agents/reviewer.js";

describe("ReviewerAgent", () => {
  it("returns a passing review", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockResolvedValue(
          "PASS - implementation is simple"
        )
    };

    const agent =
      new ReviewerAgent(ai);

    const result =
      await agent.run(
        "Review this implementation"
      );

    expect(result).toContain("PASS");
  });

  it("returns a failing review", async () => {
    const ai = {
      chat: vi
        .fn()
        .mockResolvedValue(
          "FAIL - missing validation"
        )
    };

    const agent =
      new ReviewerAgent(ai);

    const result =
      await agent.run(
        "Review this implementation"
      );

    expect(result).toContain("FAIL");
  });
});
```

---

# 31. Agent Manager

This class creates the three agents from one AI service.

## src/agents/agent-manager.ts

```ts
import type { AIService } from "../types.js";

import { ResearcherAgent } from "./researcher.js";
import { CoderAgent } from "./coder.js";
import { ReviewerAgent } from "./reviewer.js";

export class AgentManager {
  readonly researcher: ResearcherAgent;
  readonly coder: CoderAgent;
  readonly reviewer: ReviewerAgent;

  constructor(
    ai: AIService
  ) {
    this.researcher =
      new ResearcherAgent(ai);

    this.coder =
      new CoderAgent(ai);

    this.reviewer =
      new ReviewerAgent(ai);
  }
}
```

---

# 32. Message Bus

The message bus stores messages in memory.

## src/messaging/message-bus.ts

```ts
import type {
  AgentMessage
} from "../types.js";

export class MessageBus {
  private messages: AgentMessage[] = [];

  publish(
    message: AgentMessage
  ): void {
    this.messages.push({
      ...message,
      createdAt:
        message.createdAt ??
        new Date().toISOString()
    });
  }

  getAll(): AgentMessage[] {
    return [...this.messages];
  }

  getMessagesFor(
    agentName: string
  ): AgentMessage[] {
    return this.messages.filter(
      (message) =>
        message.to === agentName
    );
  }

  clear(): void {
    this.messages = [];
  }
}
```

---

# 33. Message Bus Tests

## tests/integration/message-bus.test.ts

```ts
import {
  describe,
  expect,
  it
} from "vitest";

import { MessageBus } from "../../src/messaging/message-bus.js";

describe("MessageBus", () => {
  it("publishes and retrieves a message", () => {
    const bus =
      new MessageBus();

    bus.publish({
      from: "researcher",
      to: "coder",
      type: "research.complete",
      payload: {
        result: "Use a Map"
      }
    });

    const messages =
      bus.getAll();

    expect(messages).toHaveLength(1);
    expect(messages[0].from)
      .toBe("researcher");
    expect(messages[0].to)
      .toBe("coder");
  });

  it("returns messages for a specific agent", () => {
    const bus =
      new MessageBus();

    bus.publish({
      from: "researcher",
      to: "coder",
      type: "research.complete",
      payload: "research"
    });

    bus.publish({
      from: "coder",
      to: "reviewer",
      type: "code.complete",
      payload: "code"
    });

    expect(
      bus.getMessagesFor("coder")
    ).toHaveLength(1);

    expect(
      bus.getMessagesFor("reviewer")
    ).toHaveLength(1);
  });

  it("can be cleared", () => {
    const bus =
      new MessageBus();

    bus.publish({
      from: "researcher",
      to: "coder",
      type: "research.complete",
      payload: "research"
    });

    bus.clear();

    expect(bus.getAll())
      .toEqual([]);
  });
});
```

---

# 34. SQLite Database

We use an in-memory SQLite database during tests.

That means:

```text
Database disappears
when test finishes
```

No test database file is left behind.

## src/db/database.ts

```ts
import Database from "better-sqlite3";

export function createDatabase(
  filename: string = ":memory:"
): Database.Database {
  const db =
    new Database(filename);

  db.pragma("foreign_keys = ON");

  db.exec(`
    CREATE TABLE IF NOT EXISTS messages (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      conversation_id TEXT NOT NULL,
      role TEXT NOT NULL,
      content TEXT NOT NULL,
      created_at TEXT NOT NULL
    )
  `);

  return db;
}
```

---

# 35. Message Repository

The repository hides SQL from the rest of the application.

## src/db/message-repository.ts

```ts
import type Database from "better-sqlite3";

import type { Message } from "../types.js";

export class MessageRepository {
  constructor(
    private readonly db: Database.Database
  ) {}

  save(
    conversationId: string,
    message: Message
  ): Message {
    const createdAt =
      message.createdAt ??
      new Date().toISOString();

    const statement = this.db.prepare(`
      INSERT INTO messages
      (
        conversation_id,
        role,
        content,
        created_at
      )
      VALUES (?, ?, ?, ?)
    `);

    const result =
      statement.run(
        conversationId,
        message.role,
        message.content,
        createdAt
      );

    return {
      ...message,
      id: Number(result.lastInsertRowid),
      conversationId,
      createdAt
    };
  }

  getConversation(
    conversationId: string
  ): Message[] {
    const statement =
      this.db.prepare(`
        SELECT
          id,
          conversation_id AS conversationId,
          role,
          content,
          created_at AS createdAt
        FROM messages
        WHERE conversation_id = ?
        ORDER BY id ASC
      `);

    return statement.all(
      conversationId
    ) as Message[];
  }

  count(
    conversationId: string
  ): number {
    const statement =
      this.db.prepare(`
        SELECT COUNT(*) AS count
        FROM messages
        WHERE conversation_id = ?
      `);

    const row =
      statement.get(
        conversationId
      ) as { count: number };

    return row.count;
  }
}
```

---

# 36. Repository Integration Tests

## tests/integration/repository.test.ts

```ts
import {
  afterEach,
  beforeEach,
  describe,
  expect,
  it
} from "vitest";

import { createDatabase } from "../../src/db/database.js";
import { MessageRepository } from "../../src/db/message-repository.js";

describe("MessageRepository", () => {
  let db: ReturnType<typeof createDatabase>;
  let repository: MessageRepository;

  beforeEach(() => {
    db = createDatabase();
    repository =
      new MessageRepository(db);
  });

  afterEach(() => {
    db.close();
  });

  it("saves a message", () => {
    const saved =
      repository.save(
        "chat-1",
        {
          role: "user",
          content: "Hello"
        }
      );

    expect(saved.id)
      .toBeTypeOf("number");

    expect(saved.content)
      .toBe("Hello");
  });

  it("gets conversation messages", () => {
    repository.save(
      "chat-1",
      {
        role: "user",
        content: "Hello"
      }
    );

    repository.save(
      "chat-1",
      {
        role: "assistant",
        content: "Hi"
      }
    );

    const messages =
      repository.getConversation(
        "chat-1"
      );

    expect(messages)
      .toHaveLength(2);

    expect(messages[0].content)
      .toBe("Hello");

    expect(messages[1].content)
      .toBe("Hi");
  });

  it("does not mix conversations", () => {
    repository.save(
      "chat-1",
      {
        role: "user",
        content: "Chat one"
      }
    );

    repository.save(
      "chat-2",
      {
        role: "user",
        content: "Chat two"
      }
    );

    const messages =
      repository.getConversation(
        "chat-1"
      );

    expect(messages)
      .toHaveLength(1);

    expect(messages[0].content)
      .toBe("Chat one");
  });

  it("returns message count", () => {
    repository.save(
      "chat-1",
      {
        role: "user",
        content: "One"
      }
    );

    repository.save(
      "chat-1",
      {
        role: "user",
        content: "Two"
      }
    );

    expect(
      repository.count("chat-1")
    ).toBe(2);
  });
});
```

This is a genuine integration test because the repository talks to real SQLite.

---

# 37. Workflow

The workflow is:

```text
Task
 |
 v
Researcher
 |
 v
Coder
 |
 v
Reviewer
 |
 +---- PASS ----> finished
 |
 +---- FAIL ----> coder again
```

We will allow a maximum number of coding attempts.

## src/workflow/workflow.ts

```ts
import type { Agent } from "../types.js";

import { MessageBus } from "../messaging/message-bus.js";

export interface WorkflowOptions {
  researcher: Agent;
  coder: {
    runWithResearch(
      task: string,
      research: string
    ): Promise<string>;
  };
  reviewer: Agent;
  bus: MessageBus;
  maxAttempts?: number;
}

export interface WorkflowResult {
  research: string;
  code: string;
  review: string;
  attempts: number;
  approved: boolean;
}

export class AgentWorkflow {
  private readonly maxAttempts: number;

  constructor(
    private readonly options: WorkflowOptions
  ) {
    this.maxAttempts =
      options.maxAttempts ?? 3;
  }

  async run(
    task: string
  ): Promise<WorkflowResult> {
    const research =
      await this.options.researcher.run(
        task
      );

    this.options.bus.publish({
      from: "researcher",
      to: "coder",
      type: "research.complete",
      payload: research
    });

    let code = "";

    for (
      let attempt = 1;
      attempt <= this.maxAttempts;
      attempt++
    ) {
      code =
        await this.options.coder.runWithResearch(
          task,
          research
        );

      this.options.bus.publish({
        from: "coder",
        to: "reviewer",
        type: "code.complete",
        payload: code
      });

      const review =
        await this.options.reviewer.run(
          code
        );

      this.options.bus.publish({
        from: "reviewer",
        to: "workflow",
        type: "review.complete",
        payload: review
      });

      const approved =
        review
          .trim()
          .toUpperCase()
          .startsWith("PASS");

      if (approved) {
        return {
          research,
          code,
          review,
          attempts: attempt,
          approved: true
        };
      }

      if (
        attempt === this.maxAttempts
      ) {
        return {
          research,
          code,
          review,
          attempts: attempt,
          approved: false
        };
      }
    }

    throw new Error(
      "Workflow reached an unexpected state"
    );
  }
}
```

---

# 38. Workflow Integration Tests

## tests/integration/workflow.test.ts

```ts
import {
  describe,
  expect,
  it,
  vi
} from "vitest";

import { AgentWorkflow } from "../../src/workflow/workflow.js";
import { MessageBus } from "../../src/messaging/message-bus.js";

describe("AgentWorkflow", () => {
  it("stops after reviewer approves", async () => {
    const researcher = {
      name: "researcher",
      run: vi
        .fn()
        .mockResolvedValue(
          "Research result"
        )
    };

    const coder = {
      runWithResearch: vi
        .fn()
        .mockResolvedValue(
          "Code result"
        )
    };

    const reviewer = {
      name: "reviewer",
      run: vi
        .fn()
        .mockResolvedValue(
          "PASS - looks good"
        )
    };

    const bus =
      new MessageBus();

    const workflow =
      new AgentWorkflow({
        researcher,
        coder,
        reviewer,
        bus,
        maxAttempts: 3
      });

    const result =
      await workflow.run(
        "Build a cache"
      );

    expect(result.approved)
      .toBe(true);

    expect(result.attempts)
      .toBe(1);

    expect(coder.runWithResearch)
      .toHaveBeenCalledTimes(1);

    expect(reviewer.run)
      .toHaveBeenCalledTimes(1);
  });

  it("runs coder again after rejection", async () => {
    const researcher = {
      name: "researcher",
      run: vi
        .fn()
        .mockResolvedValue(
          "Research result"
        )
    };

    const coder = {
      runWithResearch: vi
        .fn()
        .mockResolvedValueOnce(
          "Bad code"
        )
        .mockResolvedValueOnce(
          "Good code"
        )
    };

    const reviewer = {
      name: "reviewer",
      run: vi
        .fn()
        .mockResolvedValueOnce(
          "FAIL - fix the bug"
        )
        .mockResolvedValueOnce(
          "PASS - fixed"
        )
    };

    const bus =
      new MessageBus();

    const workflow =
      new AgentWorkflow({
        researcher,
        coder,
        reviewer,
        bus,
        maxAttempts: 3
      });

    const result =
      await workflow.run(
        "Build a cache"
      );

    expect(result.approved)
      .toBe(true);

    expect(result.attempts)
      .toBe(2);

    expect(coder.runWithResearch)
      .toHaveBeenCalledTimes(2);

    expect(reviewer.run)
      .toHaveBeenCalledTimes(2);
  });

  it("stops after maximum attempts", async () => {
    const researcher = {
      name: "researcher",
      run: vi
        .fn()
        .mockResolvedValue(
          "Research"
        )
    };

    const coder = {
      runWithResearch: vi
        .fn()
        .mockResolvedValue(
          "Still bad code"
        )
    };

    const reviewer = {
      name: "reviewer",
      run: vi
        .fn()
        .mockResolvedValue(
          "FAIL - still broken"
        )
    };

    const bus =
      new MessageBus();

    const workflow =
      new AgentWorkflow({
        researcher,
        coder,
        reviewer,
        bus,
        maxAttempts: 3
      });

    const result =
      await workflow.run(
        "Build something"
      );

    expect(result.approved)
      .toBe(false);

    expect(result.attempts)
      .toBe(3);

    expect(coder.runWithResearch)
      .toHaveBeenCalledTimes(3);

    expect(reviewer.run)
      .toHaveBeenCalledTimes(3);
  });

  it("records agent communication", async () => {
    const researcher = {
      name: "researcher",
      run: vi
        .fn()
        .mockResolvedValue(
          "Research"
        )
    };

    const coder = {
      runWithResearch: vi
        .fn()
        .mockResolvedValue(
          "Code"
        )
    };

    const reviewer = {
      name: "reviewer",
      run: vi
        .fn()
        .mockResolvedValue(
          "PASS"
        )
    };

    const bus =
      new MessageBus();

    const workflow =
      new AgentWorkflow({
        researcher,
        coder,
        reviewer,
        bus
      });

    await workflow.run(
      "Build feature"
    );

    const messages =
      bus.getAll();

    expect(messages)
      .toHaveLength(3);

    expect(messages[0].type)
      .toBe(
        "research.complete"
      );

    expect(messages[1].type)
      .toBe(
        "code.complete"
      );

    expect(messages[2].type)
      .toBe(
        "review.complete"
      );
  });
});
```

---

# 39. Complete End-to-End-Style Test

This test combines:

- fake AI
- real agents
- real message bus
- real workflow
- real SQLite
- real repository
- conversation persistence

The AI itself stays fake so the test remains deterministic.

## tests/e2e/complete-workflow.test.ts

```ts
import {
  afterEach,
  beforeEach,
  describe,
  expect,
  it,
  vi
} from "vitest";

import type {
  AIService
} from "../../src/types.js";

import { AgentManager } from "../../src/agents/agent-manager.js";
import { MessageBus } from "../../src/messaging/message-bus.js";
import { AgentWorkflow } from "../../src/workflow/workflow.js";
import { createDatabase } from "../../src/db/database.js";
import { MessageRepository } from "../../src/db/message-repository.js";

describe("AgentDesk complete workflow", () => {
  let db: ReturnType<typeof createDatabase>;
  let repository: MessageRepository;

  beforeEach(() => {
    db = createDatabase();
    repository =
      new MessageRepository(db);
  });

  afterEach(() => {
    db.close();
  });

  it("runs a complete workflow", async () => {
    const ai: AIService = {
      chat: vi
        .fn()
        .mockResolvedValueOnce(
          "Research about caching"
        )
        .mockResolvedValueOnce(
          "Simple cache implementation"
        )
        .mockResolvedValueOnce(
          "PASS - implementation is good"
        )
    };

    const manager =
      new AgentManager(ai);

    const bus =
      new MessageBus();

    const workflow =
      new AgentWorkflow({
        researcher:
          manager.researcher,

        coder:
          manager.coder,

        reviewer:
          manager.reviewer,

        bus,

        maxAttempts: 3
      });

    const result =
      await workflow.run(
        "Create a cache"
      );

    expect(result.approved)
      .toBe(true);

    expect(result.attempts)
      .toBe(1);

    expect(ai.chat)
      .toHaveBeenCalledTimes(3);

    repository.save(
      "chat-1",
      {
        role: "user",
        content:
          "Create a cache"
      }
    );

    repository.save(
      "chat-1",
      {
        role: "assistant",
        content:
          result.code
      }
    );

    const messages =
      repository.getConversation(
        "chat-1"
      );

    expect(messages)
      .toHaveLength(2);

    expect(messages[0].content)
      .toBe(
        "Create a cache"
      );

    expect(messages[1].content)
      .toBe(result.code);

    expect(
      bus.getAll()
    ).toHaveLength(3);
  });
});
```

---

# 40. Testing the Real Ollama Service Without Ollama

The `OllamaService` has an HTTP boundary.

We can test that boundary by mocking `fetch`.

This is better than making a real network request in every test.

## tests/unit/ollama-service.test.ts

Add this file:

```ts
import {
  afterEach,
  describe,
  expect,
  it,
  vi
} from "vitest";

import { OllamaService } from "../../src/ai/ollama-service.js";

describe("OllamaService", () => {
  afterEach(() => {
    vi.restoreAllMocks();
  });

  it("sends the correct request", async () => {
    const fetchMock =
      vi.spyOn(
        globalThis,
        "fetch"
      );

    fetchMock.mockResolvedValue(
      new Response(
        JSON.stringify({
          message: {
            role: "assistant",
            content: "Hello from Ollama"
          }
        }),
        {
          status: 200,
          headers: {
            "Content-Type":
              "application/json"
          }
        }
      )
    );

    const service =
      new OllamaService({
        apiKey: "test-key",
        model: "test-model",
        url: "https://example.com/api/chat"
      });

    const result =
      await service.chat([
        {
          role: "user",
          content: "Hello"
        }
      ]);

    expect(result)
      .toBe("Hello from Ollama");

    expect(fetchMock)
      .toHaveBeenCalledTimes(1);

    expect(fetchMock)
      .toHaveBeenCalledWith(
        "https://example.com/api/chat",
        expect.objectContaining({
          method: "POST"
        })
      );
  });

  it("throws when the API returns an error", async () => {
    vi.spyOn(
      globalThis,
      "fetch"
    ).mockResolvedValue(
      new Response(
        "Server error",
        {
          status: 500
        }
      )
    );

    const service =
      new OllamaService({
        apiKey: "test-key",
        model: "test-model"
      });

    await expect(
      service.chat([
        {
          role: "user",
          content: "Hello"
        }
      ])
    ).rejects.toThrow(
      "Ollama request failed: 500"
    );
  });

  it("throws when response content is missing", async () => {
    vi.spyOn(
      globalThis,
      "fetch"
    ).mockResolvedValue(
      new Response(
        JSON.stringify({
          message: {}
        }),
        {
          status: 200
        }
      )
    );

    const service =
      new OllamaService({
        apiKey: "test-key",
        model: "test-model"
      });

    await expect(
      service.chat([
        {
          role: "user",
          content: "Hello"
        }
      ])
    ).rejects.toThrow(
      "Ollama response did not contain message content"
    );
  });
});
```

This is a very useful real-world pattern:

```text
Your code
   |
   v
fetch
   |
   v
MOCKED HTTP response
```

---

# 41. Optional Live Ollama Test

Do not run this as part of the normal test suite.

Create:

```text
tests/e2e/ollama-live.test.ts
```

```ts
import {
  describe,
  expect,
  it
} from "vitest";

import { OllamaService } from "../../src/ai/ollama-service.js";

const apiKey =
  process.env.OLLAMA_API_KEY;

const model =
  process.env.OLLAMA_MODEL;

describe.skipIf(
  !apiKey || !model
)(
  "Ollama Cloud live test",
  () => {
    it(
      "can call Ollama Cloud",
      async () => {
        const service =
          new OllamaService({
            apiKey: apiKey!,
            model: model!
          });

        const result =
          await service.chat([
            {
              role: "user",
              content:
                "Reply with exactly: hello"
            }
          ]);

        expect(result)
          .toBeTypeOf("string");

        expect(result.length)
          .toBeGreaterThan(0);
      },
      30_000
    );
  }
);
```

Run it only when your environment contains:

```text
OLLAMA_API_KEY
OLLAMA_MODEL
```

The test intentionally does **not** assert the exact LLM response because LLM output can vary.

---

# 42. Important Difference: Fake AI vs Real AI

Most tests:

```text
Test
 |
 v
Agent
 |
 v
Fake AI
```

A small number of live tests:

```text
Test
 |
 v
OllamaService
 |
 v
Ollama Cloud
```

This gives us:

```text
fast tests
+
deterministic tests
+
some real API confidence
```

---

# 43. Add Ollama Test Script

You can add this to `package.json`:

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:unit": "vitest run tests/unit",
    "test:integration": "vitest run tests/integration",
    "test:e2e": "vitest run tests/e2e",
    "test:ollama": "vitest run tests/e2e/ollama-live.test.ts",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

# 44. How `vi.fn()` Works

Example:

```ts
const ai = {
  chat: vi.fn()
};
```

At first:

```text
chat()
 |
 +--> does nothing
```

Then:

```ts
ai.chat.mockResolvedValue(
  "hello"
);
```

Now:

```text
chat()
 |
 +--> Promise("hello")
```

You can inspect calls:

```ts
expect(ai.chat)
  .toHaveBeenCalledTimes(1);
```

And arguments:

```ts
expect(ai.chat)
  .toHaveBeenCalledWith(
    expectedMessages
  );
```

---

# 45. `mockResolvedValue`

Use this for successful async functions.

```ts
const mock = vi
  .fn()
  .mockResolvedValue("success");
```

Equivalent mental model:

```text
mock()
   |
   v
Promise.resolve("success")
```

---

# 46. `mockRejectedValue`

Use this to simulate failure.

```ts
const mock = vi
  .fn()
  .mockRejectedValue(
    new Error("network error")
  );
```

Mental model:

```text
mock()
   |
   v
Promise.reject(error)
```

---

# 47. `mockResolvedValueOnce`

Use this when different calls need different results.

```ts
const reviewer = vi
  .fn()
  .mockResolvedValueOnce("FAIL")
  .mockResolvedValueOnce("PASS");
```

Result:

```text
Call 1 -> FAIL
Call 2 -> PASS
```

This is perfect for workflow tests.

---

# 48. `vi.spyOn`

Suppose you have:

```ts
const logger = {
  log(message: string) {
    console.log(message);
  }
};
```

Test:

```ts
const spy =
  vi.spyOn(
    logger,
    "log"
  );

logger.log("hello");

expect(spy)
  .toHaveBeenCalledWith(
    "hello"
  );
```

Use a spy when you want to observe an existing function.

---

# 49. `vi.mock`

Use module mocking when the dependency is imported directly.

Example concept:

```ts
vi.mock("./some-module.js");
```

For this beginner project, dependency injection is used more often because it is easier to understand:

```ts
constructor(
  private readonly ai: AIService
) {}
```

Then the test can provide:

```ts
{
  chat: vi.fn()
}
```

This is one of the reasons dependency injection is useful.

---

# 50. Unit Test vs Integration Test in This Project

## Unit

These are mostly isolated:

```text
token-counter
cache
retry
timeout
conversation
compaction
researcher
coder
reviewer
```

Dependencies are usually fake or nonexistent.

---

## Integration

These combine real components:

```text
MessageBus
Repository + SQLite
Workflow + real agents
```

---

## E2E-style

This combines:

```text
AgentManager
+
real agents
+
workflow
+
message bus
+
SQLite
+
fake AI
```

The AI remains fake because deterministic E2E-style tests are much easier to debug.

---

# 51. Run All Tests

From the project root:

```bash
npm install
```

Then:

```bash
npm run test:run
```

You should see tests for:

```text
token counter
cache
retry
timeout
conversation
compaction
researcher
coder
reviewer
Ollama service
repository
message bus
workflow
complete workflow
```

---

# 52. Run Tests in Watch Mode

Use:

```bash
npm test
```

Then edit a source file.

Vitest will automatically rerun relevant tests.

This is extremely useful while learning.

---

# 53. Run Only Unit Tests

```bash
npm run test:unit
```

---

# 54. Run Only Integration Tests

```bash
npm run test:integration
```

---

# 55. Run E2E Tests

```bash
npm run test:e2e
```

---

# 56. Useful Vitest Commands

Run a specific file:

```bash
npx vitest run tests/unit/cache.test.ts
```

Run tests matching a name:

```bash
npx vitest run -t "expires a value"
```

Run watch mode:

```bash
npx vitest
```

---

# 57. Coverage

The basic command is:

```bash
npm run test:coverage
```

If your Vitest installation reports that the coverage provider is missing, install:

```bash
npm install -D @vitest/coverage-v8
```

Then run:

```bash
npm run test:coverage
```

Coverage helps answer:

```text
Which code did my tests execute?
```

It does **not** answer:

```text
Are my tests good?
```

---

# 58. A Very Important Exercise

After everything passes, deliberately break the application.

For example, change:

```ts
return cleaned.split(/\s+/).length;
```

to:

```ts
return cleaned.split(" ").length;
```

Run:

```bash
npm run test:unit
```

A test should fail for multiple spaces.

This is exactly what you want.

A test suite is useful when it catches bugs.

---

# 59. Exercise: Break the Cache

Change:

```ts
if (Date.now() >= entry.expiresAt)
```

to:

```ts
if (Date.now() > entry.expiresAt)
```

Think about the difference at the exact expiration boundary.

Run:

```bash
npm run test:unit
```

The point is to start thinking about boundary conditions.

---

# 60. Exercise: Break Retry

Change:

```ts
attempt <= maxAttempts
```

to:

```ts
attempt < maxAttempts
```

Then run:

```bash
npm run test:unit
```

You should see the maximum-attempt behavior change.

This teaches why off-by-one errors matter.

---

# 61. Exercise: Break the Workflow

Change:

```ts
const approved =
  review
    .trim()
    .toUpperCase()
    .startsWith("PASS");
```

to:

```ts
const approved =
  review === "PASS";
```

Now a response like:

```text
PASS - implementation is good
```

will no longer pass.

Run:

```bash
npm run test:run
```

A workflow test should catch it.

---

# 62. Exercise: Make the Reviewer Fail

In:

```text
tests/integration/workflow.test.ts
```

change:

```ts
.mockResolvedValue(
  "PASS - looks good"
)
```

to:

```ts
.mockResolvedValue(
  "FAIL - broken"
)
```

Now the expected workflow result should change.

This is how mocking lets us test branches that are difficult to reproduce with a real AI.

---

# 63. Exercise: Test an Ollama Failure

Change:

```ts
mockResolvedValue(
  "Research result"
)
```

to:

```ts
mockRejectedValue(
  new Error("Ollama unavailable")
)
```

Then verify the application propagates the error.

This is much better than waiting for the real cloud service to fail.

---

# 64. What Each Test Type Is Teaching You

| Test | Main lesson |
|---|---|
| token-counter | basic assertions |
| cache | fake timers |
| retry | mocks + failures |
| timeout | async + timers |
| conversation | pure functions |
| compaction | async mocks |
| researcher | dependency injection |
| coder | checking arguments |
| reviewer | behavior based on AI result |
| repository | SQLite integration |
| message bus | state and communication |
| workflow | orchestration |
| complete workflow | multiple components |
| Ollama service | HTTP boundary mocking |
| live Ollama | real external service |

---

# 65. Mental Model for the Whole Application

Think about the application as layers.

```text
                    USER
                      |
                      v
                 WORKFLOW
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
     RESEARCHER     CODER      REVIEWER
          |           |           |
          +-----------+-----------+
                      |
                      v
                  AI SERVICE
                      |
             +--------+--------+
             |                 |
             v                 v
           CACHE            OLLAMA
             |
             v
           SQLITE
```

And tests:

```text
                    E2E
                     |
                     v
                  Workflow
                     |
              Integration
                     |
        +------------+------------+
        |            |            |
        v            v            v
      SQLite       Bus         Agents
        |
        v
       Unit
```

---

# 66. Why the AI Service Is an Interface

This:

```ts
interface AIService {
  chat(messages: Message[]): Promise<string>;
}
```

means:

> Anything that has a `chat()` method returning a Promise of a string can be used as the AI service.

Real implementation:

```text
OllamaService
```

Test implementation:

```text
{
  chat: vi.fn()
}
```

That is dependency injection.

It is one of the most useful concepts you will learn from this project.

---

# 67. Why We Use SQLite in Integration Tests

Suppose the repository has:

```ts
save()
getConversation()
count()
```

A unit test could mock SQLite.

But then we would not know whether our SQL actually works.

Integration test:

```text
MessageRepository
       |
       v
real SQLite
       |
       v
real SQL
```

This catches problems that a mocked database cannot catch.

---

# 68. Why We Do Not Use Real Ollama Everywhere

Imagine 100 tests.

If each test called the cloud:

```text
100 tests
   |
   v
100 network requests
```

Problems:

- slow
- network failures
- API cost/rate limits
- model output changes
- hard to reproduce
- tests become flaky

Instead:

```text
95+ tests
   |
   v
fake AI

small number
   |
   v
real Ollama
```

This is much more practical.

---

# 69. What Does "Test the Orchestration" Mean?

Suppose the reviewer says:

```text
PASS
```

We do not care whether a real LLM generated the perfect wording.

We care that:

```text
reviewer -> PASS
             |
             v
workflow stops
```

Suppose reviewer says:

```text
FAIL
```

We care that:

```text
reviewer -> FAIL
             |
             v
coder runs again
```

The deterministic part is our software.

That is what our tests should strongly protect.

---

# 70. Recommended Learning Order

Do not try to understand all files at once.

Use this order:

## Day 1

Read and run:

```text
token-counter
token-counter.test
```

Learn:

```text
describe
it
expect
toBe
```

---

## Day 2

Read:

```text
cache
cache.test
```

Learn:

```text
beforeEach
afterEach
vi.useFakeTimers
```

---

## Day 3

Read:

```text
retry
retry.test
```

Learn:

```text
vi.fn
mockResolvedValue
mockRejectedValue
mockResolvedValueOnce
```

---

## Day 4

Read:

```text
researcher
researcher.test
```

Learn:

```text
interfaces
dependency injection
mock functions
```

---

## Day 5

Read:

```text
database
repository
repository.test
```

Learn:

```text
SQLite
SQL
integration testing
```

---

## Day 6

Read:

```text
message bus
workflow
workflow.test
```

Learn:

```text
orchestration
agent communication
branch testing
```

---

## Day 7

Read:

```text
Ollama service
Ollama tests
```

Learn:

```text
fetch
HTTP
spies
mocking external services
```

---

# 71. Beginner Rule

When you see code you do not understand, do not jump immediately to advanced documentation.

Ask:

```text
What is this variable?

What type is it?

What does this function receive?

What does it return?

Is it synchronous or asynchronous?

What dependency is being passed in?

What does the test arrange?

What does the test execute?

What does the test assert?
```

If you can answer those questions, most of this project becomes much easier.

---

# 72. Final Test Strategy

For AgentDesk:

```text
                    TEST STRATEGY

                         E2E
                          |
                    few complete tests
                          |
                          v
                    INTEGRATION
                          |
              +-----------+-----------+
              |           |           |
           SQLite      Workflow      Bus
              |           |           |
              +-----------+-----------+
                          |
                          v
                        UNIT
                          |
       +------------------+------------------+
       |         |         |        |        |
      Cache     Retry    Agents  Chat     Utilities
```

Keep most tests:

```text
fast
isolated
deterministic
```

Use real dependencies where they provide confidence that mocks cannot.

---

# 73. Final Checklist

Before saying "I know Vitest", make sure you can explain these:

- [ ] What is a unit test?
- [ ] What is an integration test?
- [ ] What is an E2E test?
- [ ] What is `describe()`?
- [ ] What is `it()`?
- [ ] What is `expect()`?
- [ ] What does `toBe()` do?
- [ ] What does `toEqual()` do?
- [ ] What does `toThrow()` do?
- [ ] What does `.resolves` do?
- [ ] What does `.rejects` do?
- [ ] Why use `beforeEach()`?
- [ ] Why use `afterEach()`?
- [ ] What is `vi.fn()`?
- [ ] What is `vi.spyOn()`?
- [ ] What is `vi.mock()`?
- [ ] What is `mockResolvedValue()`?
- [ ] What is `mockRejectedValue()`?
- [ ] Why use fake timers?
- [ ] Why mock Ollama?
- [ ] Why use real SQLite in integration tests?
- [ ] What is dependency injection?
- [ ] How do you test a retry?
- [ ] How do you test a timeout?
- [ ] How do you test cache expiration?
- [ ] How do you test workflow approval?
- [ ] How do you test workflow rejection?
- [ ] Why should AI tests avoid exact generated text?
- [ ] What is the difference between testing an AI and testing AI orchestration?

---

# 74. One Last Mental Model

Whenever you write a test, think:

```text
        WHAT AM I TESTING?
                |
                v
        WHAT CAN GO WRONG?
                |
                v
       WHAT DO I CONTROL?
                |
                v
        WHAT SHOULD I MOCK?
                |
                v
      WHAT SHOULD BE REAL?
                |
                v
      WHAT SHOULD I ASSERT?
```

For example:

```text
Cache expiration
       |
       v
Time is difficult to control
       |
       v
Use fake timers
       |
       v
Advance time
       |
       v
Assert cache returns null
```

For Ollama:

```text
Ollama is external
       |
       v
Network is slow/unreliable
       |
       v
Mock fetch for unit tests
       |
       v
Assert request + response handling
       |
       v
Use a small number of live tests
```

For workflow:

```text
Reviewer can PASS or FAIL
       |
       v
Mock reviewer
       |
       +---- PASS ----> workflow ends
       |
       +---- FAIL ----> coder retries
```

That is the core of professional automated testing.

---

# 75. Project Complete

You now have a small but realistic TypeScript application containing:

```text
TypeScript
   +
Vitest
   +
SQLite
   +
Cache
   +
TTL
   +
Chat history
   +
Chat compaction
   +
Ollama Cloud
   +
Researcher
   +
Coder
   +
Reviewer
   +
Message Bus
   +
Agent Workflow
   +
Retries
   +
Timeouts
   +
Unit Tests
   +
Integration Tests
   +
E2E-style Tests
```

The important part is not how large the project is.

The important part is that every feature gives you something practical to test.

Start with:

```bash
npm install
npm run test:run
```

Then study one source file and its matching test file together.

For example:

```text
src/cache/memory-cache.ts
        +
tests/unit/cache.test.ts
```

Read the implementation first.

Then read the test.

Then deliberately break the implementation and run the test again.

That loop:

```text
READ
  ↓
RUN
  ↓
BREAK
  ↓
FAIL
  ↓
FIX
  ↓
PASS
```

is one of the fastest ways to actually learn testing.
