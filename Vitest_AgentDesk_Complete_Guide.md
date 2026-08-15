# Vitest From Zero to Advanced — Practical Real-World Testing Guide

## Project: AgentDesk

A beginner-friendly TypeScript application used to learn Vitest through realistic testing scenarios:

- Ollama Cloud model API calls
- SQLite persistence
- Caching and TTL
- Chat history
- Chat compaction
- Researcher/Coder/Reviewer agents
- Agent-to-agent communication
- Agentic workflows
- Retries and timeouts
- Unit, integration, and end-to-end testing

The goal is not merely to memorize Vitest APIs. The goal is to understand **why, when, and how to test a real application**.

---

# Table of Contents

1. [What You Will Build](#1-what-you-will-build)
2. [Why This Project](#2-why-this-project)
3. [Learning Roadmap](#3-learning-roadmap)
4. [JavaScript and TypeScript Basics](#4-javascript-and-typescript-basics)
5. [Understanding Testing](#5-understanding-testing)
6. [Installing Vitest](#6-installing-vitest)
7. [Your First Test](#7-your-first-test)
8. [`describe`, `it`, and `test`](#8-describe-it-and-test)
9. [`expect` and Matchers](#9-expect-and-matchers)
10. [Arrange, Act, Assert](#10-arrange-act-assert)
11. [Unit Testing](#11-unit-testing)
12. [Edge Cases](#12-edge-cases)
13. [Testing Errors](#13-testing-errors)
14. [Async Testing](#14-async-testing)
15. [Mocks](#15-mocks)
16. [`vi.fn()`](#16-vifn)
17. [`vi.spyOn()`](#17-vispyon)
18. [`vi.mock()`](#18-vimock)
19. [Mock Return Values and Failures](#19-mock-return-values-and-failures)
20. [Test Lifecycle](#20-test-lifecycle)
21. [Test Isolation](#21-test-isolation)
22. [SQLite](#22-sqlite)
23. [Testing Repositories](#23-testing-repositories)
24. [Integration Testing](#24-integration-testing)
25. [Caching](#25-caching)
26. [Fake Timers](#26-fake-timers)
27. [Ollama Cloud](#27-ollama-cloud)
28. [Testing External AI Services](#28-testing-external-ai-services)
29. [Chat History](#29-chat-history)
30. [Chat Compaction](#30-chat-compaction)
31. [Agents](#31-agents)
32. [Multi-Agent Communication](#32-multi-agent-communication)
33. [Agentic Workflows](#33-agentic-workflows)
34. [Retries](#34-retries)
35. [Timeouts](#35-timeouts)
36. [Parameterized Tests](#36-parameterized-tests)
37. [Testing AI Applications](#37-testing-ai-applications)
38. [Unit vs Integration vs E2E](#38-unit-vs-integration-vs-e2e)
39. [Coverage](#39-coverage)
40. [Advanced Vitest](#40-advanced-vitest)
41. [Testing Strategy](#41-testing-strategy)
42. [Final Project Structure](#42-final-project-structure)
43. [What You Will Know](#43-what-you-will-know)
44. [Professional Testing Mindset](#44-professional-testing-mindset)

---

# 1. What You Will Build

We will build a deliberately small application called **AgentDesk**.

It is an AI task platform where a user gives a task and multiple agents collaborate.

Conceptually:

```text
                    +-------------+
                    |    User     |
                    +------+------+
                           |
                           v
                  +-----------------+
                  |  Agent Manager  |
                  +--------+--------+
                           |
              +------------+------------+
              |            |            |
              v            v            v
        +----------+  +----------+  +----------+
        |Researcher|  |  Coder   |  | Reviewer |
        +----+-----+  +----+-----+  +----+-----+
             |             |             |
             +-------------+-------------+
                           |
                           v
                    +-------------+
                    | AI Service  |
                    +------+------+
                           |
                           v
                    +-------------+
                    | Ollama Cloud|
                    +-------------+

                    +-------------+
                    |    Cache    |
                    +-------------+

                    +-------------+
                    |   SQLite    |
                    +-------------+
```

The application will contain:

```text
AgentDesk
│
├── Ollama Cloud integration
├── SQLite database
├── Cache
├── Conversation history
├── Chat compaction
├── Researcher agent
├── Coder agent
├── Reviewer agent
├── Agent message bus
├── Agent workflow
├── Retry handling
└── Timeout handling
```

---

# 2. Why This Project

A common beginner mistake is learning testing with examples such as:

```ts
function add(a: number, b: number) {
  return a + b;
}
```

Then writing:

```ts
expect(add(2, 3)).toBe(5);
```

This teaches the syntax, but not enough real-world testing.

AgentDesk creates realistic problems.

For example:

## External API

We need to test Ollama without making every test call the internet.

```text
Test
 |
 v
Agent
 |
 v
Fake AI Service
```

instead of:

```text
Test
 |
 v
Agent
 |
 v
Internet
 |
 v
Ollama Cloud
```

## Database

We need to test SQLite without corrupting production data.

## Cache

We need to test:

- cache hit
- cache miss
- expiration
- invalidation
- failed requests

## Agents

We need to test:

- which agent runs
- what information it receives
- what happens when an agent fails
- what happens when a reviewer rejects work
- whether another agent gets another attempt

## AI

LLM output is not always deterministic.

Therefore, we need to learn how to test **AI orchestration** without depending on exact model output.

---

# 3. Learning Roadmap

The course progresses like this:

```text
JavaScript basics
        |
        v
TypeScript basics
        |
        v
Vitest fundamentals
        |
        v
Unit testing
        |
        v
Mocks / spies / stubs
        |
        v
Async testing
        |
        v
SQLite
        |
        v
Integration testing
        |
        v
Caching
        |
        v
Fake timers
        |
        v
Ollama
        |
        v
Chat history
        |
        v
Chat compaction
        |
        v
Agents
        |
        v
Multi-agent systems
        |
        v
Agentic workflows
        |
        v
Retries / timeouts
        |
        v
Advanced Vitest
        |
        v
Professional testing strategy
```

---

# 4. JavaScript and TypeScript Basics

Because the guide assumes you are new to JavaScript and TypeScript, we will only learn the concepts necessary for the project.

## 4.1 Variables

```ts
const name = "Harsh";
let count = 0;

count = count + 1;
```

Prefer `const` when the variable does not need reassignment.

---

## 4.2 Functions

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

Here:

```text
a: number
b: number
: number
```

are TypeScript type annotations.

---

## 4.3 Objects

```ts
const user = {
  id: 1,
  name: "Harsh"
};
```

Access properties:

```ts
user.name;
user.id;
```

---

## 4.4 Arrays

```ts
const names = ["Harsh", "Rahul", "Amit"];
```

Access:

```ts
names[0];
```

Result:

```text
Harsh
```

---

## 4.5 Interfaces

Interfaces describe object shapes.

```ts
interface User {
  id: number;
  name: string;
}
```

Then:

```ts
const user: User = {
  id: 1,
  name: "Harsh"
};
```

---

## 4.6 Union Types

An agent message can have different roles:

```ts
type MessageRole =
  | "system"
  | "user"
  | "assistant"
  | "tool";
```

This means `MessageRole` can only contain one of those values.

---

## 4.7 Async/Await

An API request takes time.

```ts
async function getUser() {
  const result = await fetchUser();

  return result;
}
```

`await` waits for a Promise to complete.

---

## 4.8 Promises

A Promise represents a value that will become available later.

```ts
const result: Promise<string>;
```

Eventually:

```text
pending
   |
   +----> fulfilled
   |
   +----> rejected
```

This is extremely important for testing APIs, databases, agents, and Ollama.

---

## 4.9 Classes

We may use classes for services:

```ts
class CacheService {
  get(key: string) {
    // ...
  }

  set(key: string, value: string) {
    // ...
  }
}
```

---

## 4.10 Modules

One file can export:

```ts
export function countTokens(text: string) {
  // ...
}
```

Another file can import:

```ts
import { countTokens } from "./token-counter";
```

Vitest's module mocking will later depend heavily on understanding imports and exports.

---

# 5. Understanding Testing

The simplest definition:

> A test executes application code and checks whether the observed behavior matches what we expect.

Conceptually:

```text
Input
  |
  v
Application
  |
  v
Output
  |
  v
Assertion
  |
  +---- PASS
  |
  +---- FAIL
```

A test normally contains:

```text
Arrange
   |
   v
Act
   |
   v
Assert
```

---

# 6. Installing Vitest

Create the project:

```bash
mkdir agentdesk
cd agentdesk
npm init -y
```

Install TypeScript:

```bash
npm install typescript
```

Install Vitest:

```bash
npm install -D vitest @types/node
```

Create a TypeScript configuration:

```bash
npx tsc --init
```

Add scripts to `package.json`:

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

Run:

```bash
npm test
```

For a one-time test run:

```bash
npm run test:run
```

During development, `vitest` normally runs in watch mode.

---

# 7. Your First Test

Create:

```text
src/utils/token-counter.ts
```

```ts
export function countTokens(text: string): number {
  const cleaned = text.trim();

  if (!cleaned) {
    return 0;
  }

  return cleaned.split(/\s+/).length;
}
```

Create:

```text
tests/unit/token-counter.test.ts
```

```ts
import { describe, expect, it } from "vitest";
import { countTokens } from "../../src/utils/token-counter";

describe("countTokens", () => {
  it("counts words", () => {
    const result = countTokens("hello world");

    expect(result).toBe(2);
  });
});
```

Run:

```bash
npm test
```

You now have a real unit test.

---

# 8. `describe`, `it`, and `test`

## `describe`

Groups related tests.

```ts
describe("Calculator", () => {
  // tests
});
```

Think:

```text
Calculator
├── addition
├── subtraction
├── multiplication
└── division
```

## `it`

Defines one test.

```ts
it("adds numbers", () => {
  expect(2 + 3).toBe(5);
});
```

## `test`

Vitest also supports:

```ts
test("adds numbers", () => {
  expect(2 + 3).toBe(5);
});
```

`it` and `test` are effectively aliases.

A readable convention is:

```ts
describe("Calculator", () => {
  it("adds numbers", () => {});
  it("subtracts numbers", () => {});
});
```

---

# 9. `expect` and Matchers

`expect()` is how we make assertions.

## Exact primitive values

```ts
expect(5).toBe(5);
```

## Objects

```ts
expect(user).toEqual({
  id: 1,
  name: "Harsh"
});
```

## Truthiness

```ts
expect(value).toBeTruthy();
expect(value).toBeFalsy();
```

## Null

```ts
expect(value).toBeNull();
```

## Arrays

```ts
expect(users).toHaveLength(3);
expect(users).toContain("Harsh");
```

## Strings

```ts
expect(message).toContain("hello");
```

## Errors

```ts
expect(() => divide(10, 0)).toThrow();
```

Or:

```ts
expect(() => divide(10, 0))
  .toThrow("Cannot divide by zero");
```

## Async success

```ts
await expect(getUser())
  .resolves
  .toEqual(user);
```

## Async failure

```ts
await expect(getUser())
  .rejects
  .toThrow();
```

---

# 10. Arrange, Act, Assert

This is one of the most useful habits in testing.

Example:

```ts
it("creates a user", () => {
  // Arrange
  const name = "Harsh";

  // Act
  const user = createUser(name);

  // Assert
  expect(user.name).toBe("Harsh");
});
```

Think:

```text
ARRANGE
Prepare everything

ACT
Run the behavior

ASSERT
Check the result
```

Keep this mental model throughout the entire project.

---

# 11. Unit Testing

A unit test normally tests one small piece of behavior in isolation.

Example:

```ts
function isValidAge(age: number): boolean {
  return age >= 18;
}
```

Tests:

```ts
describe("isValidAge", () => {
  it("accepts adults", () => {
    expect(isValidAge(18)).toBe(true);
  });

  it("rejects minors", () => {
    expect(isValidAge(17)).toBe(false);
  });
});
```

Unit tests should normally be:

- fast
- deterministic
- isolated
- easy to understand

---

# 12. Edge Cases

Never test only the happy path.

For:

```ts
countTokens("hello world")
```

we should also ask:

```text
What about empty text?
What about spaces?
What about multiple spaces?
What about one word?
```

Tests:

```ts
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

A good tester asks:

> What is the boundary?

---

# 13. Testing Errors

Suppose:

```ts
function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error("Cannot divide by zero");
  }

  return a / b;
}
```

Test:

```ts
it("throws when dividing by zero", () => {
  expect(() => divide(10, 0))
    .toThrow("Cannot divide by zero");
});
```

For async:

```ts
it("handles API failure", async () => {
  await expect(getUser())
    .rejects
    .toThrow("User not found");
});
```

This becomes very important when testing:

- Ollama failure
- database failure
- cache failure
- agent failure
- network failure

---

# 14. Async Testing

Most real applications are asynchronous.

Example:

```ts
async function askAI(prompt: string): Promise<string> {
  return "AI response";
}
```

Test:

```ts
it("returns an AI response", async () => {
  const result = await askAI("Hello");

  expect(result).toBe("AI response");
});
```

Important:

```ts
it("...", async () => {
```

and:

```ts
await ...
```

Without proper `await`, a test can finish before the asynchronous operation finishes.

---

# 15. Mocks

Mocking is one of the most important Vitest concepts for this project.

Suppose:

```text
Agent
  |
  v
Ollama Cloud
```

We do not want every test to call the real cloud API.

Instead:

```text
Agent
  |
  v
Fake AI Service
```

The fake service can return exactly what the test needs.

Vitest provides:

- `vi.fn()`
- `vi.spyOn()`
- `vi.mock()`

---

# 16. `vi.fn()`

Create a fake function:

```ts
import { vi } from "vitest";

const fakeAI = vi.fn();
```

Call it:

```ts
fakeAI("hello");
```

Check whether it was called:

```ts
expect(fakeAI).toHaveBeenCalled();
```

Check call count:

```ts
expect(fakeAI).toHaveBeenCalledTimes(1);
```

Check arguments:

```ts
expect(fakeAI)
  .toHaveBeenCalledWith("hello");
```

This is incredibly useful when testing agents.

---

# 17. `vi.spyOn()`

Suppose:

```ts
const logger = {
  log(message: string) {
    console.log(message);
  }
};
```

Test:

```ts
const spy = vi.spyOn(logger, "log");

logger.log("hello");

expect(spy)
  .toHaveBeenCalledWith("hello");
```

Think:

```text
vi.fn()
    |
    +--> create a fake function

vi.spyOn()
    |
    +--> watch an existing function
```

---

# 18. `vi.mock()`

Sometimes you want to mock an entire module.

Example:

```ts
vi.mock("ollama");
```

This is useful when application code imports a dependency directly.

Conceptually:

```text
Real module
    |
    v
   MOCK
    |
    v
Fake module
```

Module mocking is more advanced than `vi.fn()`, so learn `vi.fn()` and `vi.spyOn()` first.

---

# 19. Mock Return Values and Failures

## Synchronous result

```ts
const mock = vi.fn();

mock.mockReturnValue("hello");
```

## Async result

```ts
const mock = vi.fn();

mock.mockResolvedValue("hello");
```

Then:

```ts
const result = await mock();

expect(result).toBe("hello");
```

## Async failure

```ts
mock.mockRejectedValue(
  new Error("Ollama unavailable")
);
```

## Different results on different calls

```ts
mock
  .mockRejectedValueOnce(new Error("network"))
  .mockRejectedValueOnce(new Error("network"))
  .mockResolvedValueOnce("success");
```

This is perfect for retry tests.

---

# 20. Test Lifecycle

Vitest provides lifecycle hooks.

```ts
beforeEach(() => {
  // before every test
});
```

```ts
afterEach(() => {
  // after every test
});
```

Also:

```ts
beforeAll(() => {});
afterAll(() => {});
```

Typical pattern:

```ts
describe("Cache", () => {
  let cache: MemoryCache;

  beforeEach(() => {
    cache = new MemoryCache();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it("stores values", () => {
    // ...
  });
});
```

---

# 21. Test Isolation

Imagine:

```text
Test A
creates user 1

Test B
expects zero users
```

If Test B sees user 1, the tests are coupled.

That's bad.

A test suite should ideally allow:

```text
Test A alone → PASS
Test B alone → PASS
A + B → PASS
B + A → PASS
```

The exact order should not matter.

For databases:

```text
beforeEach
    |
    v
fresh test DB
```

For mocks:

```text
afterEach
    |
    v
restore mocks
```

For cache:

```text
beforeEach
    |
    v
fresh cache
```

---

# 22. SQLite

AgentDesk will use SQLite for:

```text
users
conversations
messages
agents
agent_messages
workflow_events
```

A simplified schema:

```text
users
-----
id
name

conversations
-------------
id
user_id
created_at

messages
--------
id
conversation_id
role
content
created_at

agent_messages
--------------
id
workflow_id
from_agent
to_agent
content
created_at
```

The important architectural idea is to separate database access.

Instead of:

```text
Agent
  |
  +--> SQL
  |
  +--> SQL
  |
  +--> SQL
```

use:

```text
Agent
  |
  v
Repository
  |
  v
SQLite
```

---

# 23. Testing Repositories

Define:

```ts
interface MessageRepository {
  save(message: Message): Promise<void>;

  getConversation(
    conversationId: string
  ): Promise<Message[]>;
}
```

Then:

```text
SQLiteMessageRepository
```

implements it.

Test:

```ts
it("saves a message", async () => {
  const message = {
    conversationId: "chat-1",
    role: "user",
    content: "Hello"
  };

  await repository.save(message);

  const messages =
    await repository.getConversation("chat-1");

  expect(messages).toHaveLength(1);
  expect(messages[0].content).toBe("Hello");
});
```

This test uses a real SQLite database, so it is an integration test.

---

# 24. Integration Testing

A unit test:

```text
Agent
 |
 v
Fake AI
```

An integration test:

```text
Repository
 |
 v
SQLite
```

Another integration test:

```text
Agent
 |
 v
Workflow
 |
 v
Repository
 |
 v
SQLite
```

Integration tests verify that multiple real pieces work together.

They are usually slower than unit tests.

---

# 25. Test Database

Never use production data for tests.

Use:

```text
production.db

test.db
```

Better:

```text
each test
   |
   v
fresh isolated database
```

For example:

```ts
beforeEach(() => {
  database = createTestDatabase();
});

afterEach(() => {
  database.close();
});
```

The exact SQLite library can be chosen for the implementation, but the testing principle remains the same.

---

# 26. Caching

Our AI service can use a cache.

Architecture:

```text
Request
   |
   v
Cache?
  / \
hit  miss
 |     |
 v     v
return Ollama
       |
       v
     cache
       |
       v
     return
```

A simple interface:

```ts
interface Cache {
  get(key: string): string | null;

  set(
    key: string,
    value: string,
    ttlSeconds: number
  ): void;
}
```

---

# 27. Cache Tests

## Cache hit

```ts
it("returns cached value", () => {
  cache.set("answer", "42", 60);

  expect(cache.get("answer"))
    .toBe("42");
});
```

## Cache miss

```ts
it("returns null for missing values", () => {
  expect(cache.get("missing"))
    .toBeNull();
});
```

## Expiration

We need to simulate time.

That leads to fake timers.

---

# 28. Fake Timers

Suppose cache TTL is:

```text
60 seconds
```

We do not want a test that literally waits 60 seconds.

Use:

```ts
vi.useFakeTimers();
```

Then:

```ts
vi.advanceTimersByTime(60_000);
```

Conceptually:

```text
Real time

0 sec -------------------- 60 sec


Test time

0 sec
 |
 +-- advanceTimersByTime(60000)
 |
 v
60 sec
```

This can test:

- cache TTL
- retry delays
- scheduled tasks
- timeout behavior
- token refresh
- polling

Always clean up:

```ts
afterEach(() => {
  vi.useRealTimers();
});
```

---

# 29. Ollama Cloud

AgentDesk will isolate Ollama behind an AI service.

Architecture:

```text
Researcher
    |
Coder
    |
Reviewer
    |
    v
AIService
    |
    v
Ollama
```

A simple interface:

```ts
interface AIService {
  chat(messages: Message[]): Promise<string>;
}
```

Then:

```ts
class OllamaService implements AIService {
  async chat(
    messages: Message[]
  ): Promise<string> {
    // real Ollama request
  }
}
```

Agents only know:

```ts
ai.chat(messages);
```

They do not need to know how Ollama works.

---

# 30. Why Dependency Injection Matters

Without dependency injection:

```ts
class ResearcherAgent {
  async run(task: string) {
    const ollama = new Ollama(...);

    return ollama.chat(...);
  }
}
```

Testing becomes harder.

Instead:

```ts
class ResearcherAgent {
  constructor(
    private ai: AIService
  ) {}

  async run(task: string) {
    return this.ai.chat(...);
  }
}
```

Now production can use:

```text
OllamaService
```

while tests can use:

```text
FakeAIService
```

This is one of the most important architectural concepts in the project.

---

# 31. Testing External AI Services

Do not make hundreds of unit tests call Ollama Cloud.

Use:

```text
Unit Tests
   |
   v
Fake AI
```

Then a small number of integration/live tests can verify:

```text
Application
   |
   v
OllamaService
   |
   v
Ollama Cloud
```

For example:

```text
tests/live/ollama.test.ts
```

These tests require a valid API key.

They should be separate from ordinary:

```bash
npm test
```

A possible command:

```bash
npm run test:ollama
```

The exact model should be configurable through environment variables.

---

# 32. What to Test About Ollama

Do not normally assert:

```ts
expect(response)
  .toBe("the exact LLM response");
```

Instead test:

```text
Was the correct request constructed?
Was the correct model selected?
Were messages formatted correctly?
Was the response parsed correctly?
Was an API failure handled?
Was a timeout handled?
Was retry triggered?
```

For live tests:

```text
Does authentication work?
Does the endpoint respond?
Can we parse a valid response?
```

---

# 33. Chat History

A conversation might look like:

```text
system
user
assistant
user
assistant
user
assistant
```

Represent it:

```ts
interface Message {
  role: MessageRole;
  content: string;
}
```

Then:

```ts
function addMessage(
  messages: Message[],
  message: Message
): Message[] {
  return [...messages, message];
}
```

Test:

```ts
it("adds a message", () => {
  const messages: Message[] = [];

  const result = addMessage(messages, {
    role: "user",
    content: "Hello"
  });

  expect(result).toHaveLength(1);
  expect(result[0].content).toBe("Hello");
});
```

---

# 34. Chat Compaction

Long conversations create a context problem.

Example:

```text
100 messages
     |
     v
too much context
```

Compaction:

```text
100 messages
     |
     v
summarize old messages
     |
     v
summary + recent messages
```

Example:

```text
Before:

system
user
assistant
user
assistant
...
100 messages

After:

system
summary
recent user message
recent assistant message
...
```

---

# 35. Testing Compaction

Tests:

```text
short conversation
    -> no compaction

long conversation
    -> compaction happens

system message
    -> preserved

recent messages
    -> preserved

old messages
    -> summarized

AI summarizer
    -> called when required

AI summarizer
    -> not called when unnecessary
```

Example:

```ts
it("does not compact a short conversation", async () => {
  const result =
    await compactConversation(shortMessages, ai);

  expect(ai.chat)
    .not.toHaveBeenCalled();

  expect(result)
    .toEqual(shortMessages);
});
```

Long conversation:

```ts
it("compacts a long conversation", async () => {
  ai.chat.mockResolvedValue(
    "Summary of conversation"
  );

  const result =
    await compactConversation(longMessages, ai);

  expect(ai.chat)
    .toHaveBeenCalledTimes(1);

  expect(result).toContainEqual({
    role: "system",
    content: "Summary of conversation"
  });
});
```

---

# 36. Agents

Start with one agent.

## Researcher

Responsibilities:

```text
receive task
     |
     v
ask AI
     |
     v
return research
```

Implementation:

```ts
class ResearcherAgent {
  constructor(
    private ai: AIService
  ) {}

  async run(task: string): Promise<string> {
    return this.ai.chat([
      {
        role: "system",
        content: "You are a research agent."
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

# 37. Testing Researcher

```ts
it("asks AI to research the task", async () => {
  const ai = {
    chat: vi.fn()
      .mockResolvedValue("Research result")
  };

  const agent =
    new ResearcherAgent(ai);

  const result =
    await agent.run("Research Vitest mocking");

  expect(result)
    .toBe("Research result");

  expect(ai.chat)
    .toHaveBeenCalledTimes(1);
});
```

We are testing:

```text
result
+
interaction
```

---

# 38. Coder Agent

Workflow:

```text
Researcher
    |
    v
research result
    |
    v
Coder
```

The coder receives the research.

Test:

```text
research result
       |
       v
coder
       |
       v
implementation
```

We can verify that the research was included in the coder's input.

Example conceptually:

```ts
expect(coderAI.chat)
  .toHaveBeenCalledWith(
    expect.arrayContaining([
      expect.objectContaining({
        content: research
      })
    ])
  );
```

---

# 39. Reviewer Agent

Workflow:

```text
Researcher
     |
     v
Coder
     |
     v
Reviewer
```

Reviewer evaluates the work.

Its output could be:

```text
PASS
```

or:

```text
FAIL
```

The workflow reacts accordingly.

---

# 40. Multi-Agent Communication

Add a message bus:

```ts
interface AgentMessage {
  from: string;
  to: string;
  type: string;
  payload: unknown;
}
```

Example:

```ts
await bus.publish({
  from: "researcher",
  to: "coder",
  type: "research.complete",
  payload: research
});
```

Conceptually:

```text
Researcher
    |
    v
Message Bus
    |
    v
Coder
```

Then:

```text
Coder
    |
    v
Message Bus
    |
    v
Reviewer
```

---

# 41. Testing Agent Communication

Test:

```text
Did researcher publish?
Did it publish to coder?
Was correct message type used?
Was research included?
Did coder receive it?
```

Example:

```ts
expect(bus.publish)
  .toHaveBeenCalledWith({
    from: "researcher",
    to: "coder",
    type: "research.complete",
    payload: research
  });
```

---

# 42. Agentic Workflow

Now combine the agents:

```text
User Task
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
    +---- PASS ----> END
    |
    +---- FAIL ----> Coder
                         |
                         v
                      Reviewer
```

This creates an interesting testing problem.

---

# 43. Testing Workflow Approval

Mock reviewer:

```ts
reviewer.review
  .mockResolvedValue("PASS");
```

Run:

```ts
await workflow.run("Build feature");
```

Assertions:

```ts
expect(coder.run)
  .toHaveBeenCalledTimes(1);

expect(reviewer.review)
  .toHaveBeenCalledTimes(1);
```

The workflow should stop.

---

# 44. Testing Workflow Rejection

Reviewer:

```ts
reviewer.review
  .mockResolvedValueOnce("FAIL")
  .mockResolvedValueOnce("PASS");
```

Run:

```ts
await workflow.run("Build feature");
```

Expected:

```text
Coder attempt 1
     |
Reviewer -> FAIL
     |
Coder attempt 2
     |
Reviewer -> PASS
     |
END
```

Test:

```ts
expect(coder.run)
  .toHaveBeenCalledTimes(2);

expect(reviewer.review)
  .toHaveBeenCalledTimes(2);
```

---

# 45. Testing Failures

AI failure:

```ts
ai.chat.mockRejectedValue(
  new Error("Ollama unavailable")
);
```

Then:

```ts
await expect(
  agent.run("hello")
).rejects.toThrow(
  "Ollama unavailable"
);
```

This is essential.

Real applications fail.

Good tests prove the application handles those failures.

---

# 46. Retries

Suppose Ollama temporarily fails.

Desired behavior:

```text
Attempt 1 -> FAIL
Attempt 2 -> FAIL
Attempt 3 -> SUCCESS
```

Mock:

```ts
ai.chat
  .mockRejectedValueOnce(
    new Error("network")
  )
  .mockRejectedValueOnce(
    new Error("network")
  )
  .mockResolvedValueOnce(
    "success"
  );
```

Then:

```ts
const result = await retry(
  () => ai.chat(messages),
  3
);

expect(result)
  .toBe("success");

expect(ai.chat)
  .toHaveBeenCalledTimes(3);
```

---

# 47. Testing Retry Exhaustion

What if all attempts fail?

```ts
ai.chat
  .mockRejectedValue(
    new Error("network")
  );
```

Expected:

```text
attempt 1 -> fail
attempt 2 -> fail
attempt 3 -> fail
              |
              v
            throw
```

Test:

```ts
await expect(
  retry(() => ai.chat(messages), 3)
).rejects.toThrow("network");

expect(ai.chat)
  .toHaveBeenCalledTimes(3);
```

---

# 48. Timeouts

AI can take too long.

Application requirement:

```text
AI request
   |
   +-- 5 seconds
   |
   v
timeout
```

Test with fake timers rather than waiting five seconds.

Concept:

```ts
vi.useFakeTimers();

const promise = callWithTimeout(
  () => ai.chat(messages),
  5000
);

vi.advanceTimersByTime(5000);

await expect(promise)
  .rejects
  .toThrow("timeout");

vi.useRealTimers();
```

The exact implementation will be developed carefully in the project because timers + Promises can be confusing for beginners.

---

# 49. Parameterized Tests

Suppose:

```text
1 -> valid
2 -> valid
3 -> valid
0 -> invalid
-1 -> invalid
```

Instead of five repetitive tests:

```ts
it.each([
  [1, true],
  [2, true],
  [3, true],
  [0, false],
  [-1, false]
])(
  "validates %s",
  (value, expected) => {
    expect(isValid(value))
      .toBe(expected);
  }
);
```

This is useful for:

- validation
- permissions
- cache TTL
- status codes
- agent states
- retry counts
- input boundaries

---

# 50. Testing AI Applications

Traditional software:

```text
input
  |
  v
function
  |
  v
exact output
```

AI application:

```text
input
  |
  v
agent
  |
  v
LLM
  |
  v
decision
  |
  v
tool
  |
  v
another agent
  |
  v
output
```

LLM output can vary.

Therefore, do not build your entire test suite around exact generated text.

Instead test deterministic application behavior.

For example:

```text
Correct agent called?
Correct tool called?
Correct context passed?
Conversation saved?
Cache used?
Retry triggered?
Workflow stopped?
Reviewer consulted?
Compaction triggered?
Failure handled?
```

---

# 51. The Key AI Testing Principle

## Mock the intelligence, test the orchestration.

For example:

```ts
ai.chat.mockResolvedValue("PASS");
```

Then test:

```text
AI says PASS
      |
      v
workflow ends
```

Another test:

```ts
ai.chat.mockResolvedValue("FAIL");
```

Then:

```text
AI says FAIL
      |
      v
coder runs again
```

You do not need a real LLM to prove that your workflow handles `PASS` and `FAIL` correctly.

---

# 52. Unit vs Integration vs E2E

## Unit

```text
Function
  |
  v
Fake dependencies
```

Characteristics:

- very fast
- isolated
- deterministic

Examples:

```text
token counting
validation
compaction logic
retry logic
workflow decisions
```

---

## Integration

```text
Service
  |
  v
Real dependency
```

Examples:

```text
Repository -> SQLite
Agent -> Workflow
Cache -> AI Service
Message Bus -> Agents
```

---

## End-to-End

```text
User
 |
 v
Application
 |
 +--> Workflow
 |
 +--> Agents
 |
 +--> Cache
 |
 +--> Database
 |
 +--> AI
```

E2E tests are slower and more complex.

---

# 53. Testing Pyramid

Think:

```text
                 /\
                /  \
               / E2E\
              /------\
             /        \
            /Integration\
           /------------\
          /              \
         /  Unit Tests   \
        /------------------\
```

Most tests should normally be unit tests.

Some integration tests.

A smaller number of E2E tests.

Why?

Unit tests are:

```text
fast
cheap
isolated
```

E2E tests are:

```text
slow
expensive
more fragile
```

---

# 54. Coverage

Coverage tells you how much code was executed by tests.

Useful categories include:

```text
Statements
Branches
Functions
Lines
```

But remember:

> 100% coverage does not automatically mean 100% quality.

Example:

```ts
function divide(a: number, b: number) {
  return a / b;
}
```

You could get high coverage with poor assertions.

Coverage is a measurement tool, not proof of correctness.

---

# 55. What Good Coverage Looks Like

For a cache:

```text
cache hit       ✓
cache miss      ✓
expired         ✓
overwrite       ✓
invalid TTL     ✓
```

For workflow:

```text
PASS            ✓
FAIL             ✓
retry            ✓
max retries      ✓
AI error         ✓
timeout          ✓
```

That is more valuable than blindly chasing a percentage.

---

# 56. Advanced Vitest

After mastering the core concepts, move into:

## Module mocking

```ts
vi.mock(...)
```

## Spies

```ts
vi.spyOn(...)
```

## Fake timers

```ts
vi.useFakeTimers()
```

## Parameterized tests

```ts
it.each(...)
```

## Test isolation

```text
mock cleanup
database cleanup
cache cleanup
timer cleanup
```

## Type testing

Vitest supports type-level assertions such as:

```ts
expectTypeOf(value)
  .toBeString();
```

and dedicated type-test files.

Do not learn this at the beginning.

---

# 57. Final AgentDesk Project Structure

By the end, the project can look like:

```text
agentdesk/
│
├── src/
│   │
│   ├── agents/
│   │   ├── researcher.ts
│   │   ├── coder.ts
│   │   ├── reviewer.ts
│   │   └── agent-manager.ts
│   │
│   ├── ai/
│   │   └── ollama.ts
│   │
│   ├── cache/
│   │   └── cache.ts
│   │
│   ├── chat/
│   │   ├── conversation.ts
│   │   └── compaction.ts
│   │
│   ├── db/
│   │   ├── database.ts
│   │   └── message-repository.ts
│   │
│   ├── workflow/
│   │   └── workflow.ts
│   │
│   ├── messaging/
│   │   └── message-bus.ts
│   │
│   └── utils/
│       ├── token-counter.ts
│       ├── retry.ts
│       └── timeout.ts
│
├── tests/
│   │
│   ├── unit/
│   │   ├── token-counter.test.ts
│   │   ├── cache.test.ts
│   │   ├── compaction.test.ts
│   │   ├── retry.test.ts
│   │   ├── timeout.test.ts
│   │   ├── researcher.test.ts
│   │   ├── coder.test.ts
│   │   └── reviewer.test.ts
│   │
│   ├── integration/
│   │   ├── database.test.ts
│   │   ├── repository.test.ts
│   │   ├── cache-ai.test.ts
│   │   ├── agent-workflow.test.ts
│   │   └── message-bus.test.ts
│   │
│   └── e2e/
│       └── complete-workflow.test.ts
│
├── package.json
├── tsconfig.json
├── vitest.config.ts
└── README.md
```

---

# 58. Test Architecture

The tests mirror the application architecture:

```text
                       E2E
                        |
                        v
                  Complete Workflow
                        |
                        v
                Integration Tests
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
    Agents           SQLite           Cache
       |                |                |
       +----------------+----------------+
                        |
                        v
                   Unit Tests
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
  Business Logic    Error Logic     Utilities
```

---

# 59. What We Should NOT Do

Do not create tests just to increase test count.

Bad:

```text
function has 10 lines
       |
       v
10 tests because there are 10 lines
```

Better:

```text
normal behavior
edge behavior
failure behavior
important business rules
```

For a cache:

```text
✓ cache hit
✓ cache miss
✓ expiration
✓ overwrite
✓ invalid TTL
```

For a workflow:

```text
✓ approval
✓ rejection
✓ retry
✓ maximum retries
✓ AI failure
✓ timeout
```

---

# 60. Testing Agentic Systems

Traditional application:

```text
Input
 |
 v
Function
 |
 v
Output
```

Agentic system:

```text
Input
 |
 v
Agent
 |
 v
LLM
 |
 v
Decision
 |
 v
Tool
 |
 v
Another Agent
 |
 v
LLM
 |
 v
Decision
 |
 v
Output
```

There are many possible failure points.

Therefore, test boundaries.

Examples:

```text
Agent received correct input
Agent called correct service
Agent published correct event
Agent passed correct context
Agent stopped correctly
Agent retried correctly
Agent persisted state
Agent recovered from failure
```

---

# 61. Professional Testing Mindset

A beginner often asks:

> "What should I test?"

A stronger tester asks:

> "What can go wrong?"

For each feature, ask:

```text
What is the happy path?

What are the boundaries?

What happens with empty input?

What happens with invalid input?

What happens if the dependency fails?

What happens if the dependency is slow?

What happens if it returns unexpected data?

What happens if the operation is repeated?

What happens if two operations happen together?

What happens after a retry?

What happens after a timeout?

What happens if cached data is stale?

What happens if the database is unavailable?
```

This mindset is more important than memorizing Vitest APIs.

---

# 62. The Complete Learning Sequence

Follow this order.

## Phase 1 — Foundations

```text
1. JavaScript variables
2. Functions
3. Objects
4. Arrays
5. Modules
6. Async/Await
7. Promises
8. Errors
9. TypeScript types
10. Interfaces
```

## Phase 2 — Vitest

```text
11. Install Vitest
12. First test
13. describe
14. it
15. expect
16. matchers
17. Arrange/Act/Assert
```

## Phase 3 — Unit Testing

```text
18. Pure functions
19. Edge cases
20. Errors
21. Business rules
22. Parameterized tests
```

## Phase 4 — Mocking

```text
23. vi.fn
24. mockReturnValue
25. mockResolvedValue
26. mockRejectedValue
27. vi.spyOn
28. vi.mock
29. Mock cleanup
```

## Phase 5 — Async

```text
30. Promise testing
31. API testing
32. Failure testing
33. Retry testing
34. Timeout testing
```

## Phase 6 — SQLite

```text
35. Database
36. Tables
37. CRUD
38. Repository
39. Test database
40. Database cleanup
41. Integration tests
```

## Phase 7 — Cache

```text
42. Cache design
43. Cache hit
44. Cache miss
45. TTL
46. Expiration
47. Fake timers
```

## Phase 8 — AI

```text
48. AI interface
49. Ollama service
50. Cloud authentication
51. Mock AI
52. API failures
53. Live tests
```

## Phase 9 — Chat

```text
54. Message model
55. Conversation
56. Persistence
57. Context limits
58. Compaction
59. Summary testing
```

## Phase 10 — Agents

```text
60. Researcher
61. Coder
62. Reviewer
63. Dependency injection
64. Agent tests
```

## Phase 11 — Multi-Agent

```text
65. Message bus
66. Agent communication
67. Sequential workflows
68. Parallel workflows
69. Failure handling
```

## Phase 12 — Advanced

```text
70. Fake timers
71. Module mocking
72. Type testing
73. Coverage
74. Test isolation
75. Fixtures
76. Factories
77. Integration architecture
78. E2E testing
```

---

# 63. Final Skills You Should Have

After completing the project, you should understand:

## Vitest Fundamentals

```text
describe
it
test
expect
```

## Assertions

```text
toBe
toEqual
toContain
toHaveLength
toThrow
resolves
rejects
```

## Lifecycle

```text
beforeEach
afterEach
beforeAll
afterAll
```

## Mocking

```text
vi.fn
vi.spyOn
vi.mock
mockReturnValue
mockResolvedValue
mockRejectedValue
mockImplementation
```

## Time

```text
fake timers
advanceTimersByTime
setSystemTime
```

## Advanced

```text
parameterized tests
module mocking
type testing
test isolation
coverage
integration testing
E2E testing
```

## Real Application Testing

```text
Ollama
SQLite
cache
repositories
conversation history
chat compaction
agents
multi-agent communication
workflows
retry
timeouts
external services
```

---

# 64. The Most Important Lessons

If you remember only a few things, remember these:

## 1. Test behavior, not implementation

Prefer:

```ts
expect(result).toEqual(expected);
```

over testing private implementation details.

---

## 2. Test boundaries

Think about:

```text
empty
invalid
maximum
minimum
missing
expired
failed
slow
repeated
```

---

## 3. Mock external dependencies

Do not make every unit test call:

```text
Ollama
Internet
real APIs
```

---

## 4. Keep tests isolated

Every test should ideally start from a clean state.

---

## 5. Use real dependencies selectively

Use real SQLite for integration tests.

Use fake Ollama for most unit tests.

Use real Ollama only for a small number of live tests.

---

## 6. AI tests should focus on orchestration

Do not depend heavily on exact LLM wording.

Test:

```text
what was called
what was passed
what happened next
```

---

## 7. Coverage is not quality

A test suite with 80% meaningful coverage can be better than one with 100% meaningless assertions.

---

# 65. The End Goal

At the end, you should be able to look at an application like:

```text
User
 |
 v
API
 |
 v
Agent Manager
 |
 +---- Researcher
 |
 +---- Coder
 |
 +---- Reviewer
 |
 v
AI Service
 |
 +---- Cache
 |
 +---- Ollama
 |
 v
SQLite
```

and immediately think:

```text
What can I unit test?

What should I mock?

What should be an integration test?

What should use a real database?

Where can state leak?

What can fail?

What needs fake timers?

What needs a live API test?

What should be tested end-to-end?

How do I make this deterministic?
```

That is the actual goal of learning Vitest.

Not memorizing:

```ts
vi.fn()
```

but knowing:

> **When should I use `vi.fn()` and why?**

Not memorizing:

```ts
vi.mock()
```

but knowing:

> **Which dependency should I isolate and why?**

Not memorizing:

```ts
vi.useFakeTimers()
```

but knowing:

> **Which time-dependent behavior should I control?**

And not merely knowing how to write:

```ts
expect(...)
```

but knowing:

> **What behavior actually deserves confidence?**

---

# 66. Recommended Final Test Distribution

A practical starting target:

```text
              Test Suite

           ┌──────────────┐
           │     E2E      │  5-10%
           ├──────────────┤
           │ Integration  │ 20-30%
           ├──────────────┤
           │     Unit     │ 60-75%
           └──────────────┘
```

The exact percentages are not rules.

The important principle is:

```text
Many fast tests
+
Some realistic integration tests
+
A few complete-system tests
```

---

# 67. Final Architecture

The complete application can ultimately look like:

```text
                         USER
                           |
                           v
                    +-------------+
                    | API / App    |
                    +------+------+
                           |
                           v
                  +------------------+
                  | Workflow Manager |
                  +--------+---------+
                           |
              +------------+------------+
              |            |            |
              v            v            v
         Researcher      Coder       Reviewer
              |            |            |
              +------------+------------+
                           |
                           v
                     Message Bus
                           |
                           v
                     AI Service
                           |
                    +------+------+
                    |             |
                    v             v
                 Cache         Ollama
                    |
                    v
                 SQLite
                    |
                    v
            Conversation History
                    |
                    v
               Compaction
```

And testing:

```text
                         E2E
                          |
                          v
                 Complete Workflow
                          |
                          v
                  Integration Tests
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
       Agents          SQLite          Cache
          |               |               |
          +---------------+---------------+
                          |
                          v
                     Unit Tests
                          |
        +-----------------+----------------+
        |                 |                |
        v                 v                v
    Business          Error Logic      Utilities
      Rules
```

---

# 68. Final Perspective

Vitest itself is not the difficult part.

The syntax is relatively small.

The difficult part is developing the ability to reason about software:

```text
What should happen?

What can go wrong?

What dependency should be isolated?

What state can leak?

What needs to be deterministic?

What should be tested at unit level?

What requires integration?

What should be left for E2E?

How do I test an external AI system?

How do I test asynchronous workflows?

How do I test failures?

How do I know the tests themselves are trustworthy?
```

AgentDesk is designed specifically to make you answer those questions through practice.

By the end, you should not just be able to say:

> "I know Vitest."

You should be able to take a real TypeScript application and design a sensible test strategy for it from scratch.
