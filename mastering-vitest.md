# Mastering Vitest — Learn Testing by Building a Real AI Code-Review Agent

> **Project used throughout this guide: `OllamaReview`** — a CLI tool that pulls a git diff, sends it to an **Ollama Cloud** model for structured review (bugs, security issues, style violations), enforces rate limits, caches results, retries failed/streamed requests, queues concurrent jobs, and renders a Markdown report.
>
> This is **not** a toy chatbot. It has the shape of a real production service: an HTTP client with retries and streaming, a token-bucket rate limiter, an LRU cache, a concurrency-limited job queue built on `EventEmitter`, a pure parsing layer, and an orchestrator that wires everything together with dependency injection. Every one of those pieces maps to a category of thing you will need to test in real TypeScript codebases — which is exactly why we build it this way.

## Table of Contents

1. [Why Vitest](#1-why-vitest)
2. [Project Architecture](#2-project-architecture)
3. [Project Setup](#3-project-setup)
4. [Part I — Fundamentals: `describe`, `it`, `expect`, lifecycle hooks](#4-part-i--fundamentals)
5. [Part II — Testing Pure Functions: the Diff Parser](#5-part-ii--testing-pure-functions-the-diff-parser)
6. [Part III — Mocking Basics: `vi.fn`, `vi.spyOn`](#6-part-iii--mocking-basics)
7. [Part IV — The Ollama Cloud Client: mocking `fetch`, `vi.mock`, `vi.hoisted`](#7-part-iv--the-ollama-cloud-client)
8. [Part V — Streaming & Async Generators](#8-part-v--streaming--async-generators)
9. [Part VI — Retries, Backoff & Fake Timers](#9-part-vi--retries-backoff--fake-timers)
10. [Part VII — Stateful Classes: Rate Limiter & LRU Cache](#10-part-vii--stateful-classes)
11. [Part VIII — The Job Queue: testing `EventEmitter` & concurrency](#11-part-viii--the-job-queue)
12. [Part IX — The Orchestrator: dependency injection & integration tests](#12-part-ix--the-orchestrator)
13. [Part X — Snapshot Testing the Report Generator](#13-part-x--snapshot-testing)
14. [Part XI — Type-Level Testing with `expectTypeOf`](#14-part-xi--type-level-testing)
15. [Part XII — Custom Matchers](#15-part-xii--custom-matchers)
16. [Part XIII — Coverage & CI](#16-part-xiii--coverage--ci)
17. [Cheat Sheet & Anti-Patterns](#17-cheat-sheet--anti-patterns)

---

## 1. Why Vitest

Vitest is a test runner built on Vite. Three things matter for TS/AI-app testing specifically:

- **Native ESM + TypeScript**, no Babel/ts-jest config wrestling — your test file imports `.ts` directly and it just works.
- **Jest-compatible API** (`describe`, `it`, `expect`, `vi` instead of `jest`) — so everything below transfers to Jest almost 1:1, but Vitest is faster because it reuses Vite's module graph and runs in workers.
- **First-class fake timers, module mocking, and in-source type testing** — all things you *need* when testing retry logic, rate limiters, and streaming AI clients, which is most of what this guide is about.

Install:

```bash
npm install -D vitest @vitest/coverage-v8 @vitest/ui
```

---

## 2. Project Architecture

```
ollama-review-agent/
├── src/
│   ├── types.ts              # Domain types & discriminated unions
│   ├── errors.ts             # Custom error hierarchy
│   ├── diff-parser.ts        # Pure functions: git diff → structured hunks
│   ├── ollama-client.ts      # HTTP client for Ollama Cloud (fetch, streaming, retries)
│   ├── rate-limiter.ts       # Token-bucket rate limiter
│   ├── lru-cache.ts          # Generic LRU cache
│   ├── review-queue.ts       # EventEmitter-based concurrency-limited job queue
│   ├── report-generator.ts   # Markdown report rendering
│   └── review-service.ts     # Orchestrator (dependency injection root)
├── tests/                    # Mirrors src/, one *.test.ts per module
├── vitest.config.ts
├── vitest.setup.ts
├── tsconfig.json
└── package.json
```

**Data flow:** `git diff` text → `diff-parser` → hunks → `review-service` asks `rate-limiter` for permission → checks `lru-cache` → on miss calls `ollama-client` (which streams NDJSON from Ollama Cloud, retrying on transient failures) → results pushed through `review-queue` for bounded concurrency → `report-generator` renders the final Markdown.

Every arrow in that sentence is a seam we will test in isolation, and then together.

---

## 3. Project Setup

**`package.json`** (relevant scripts):

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage"
  },
  "devDependencies": {
    "vitest": "^2.1.0",
    "@vitest/coverage-v8": "^2.1.0",
    "@vitest/ui": "^2.1.0",
    "typescript": "^5.6.0"
  }
}
```

**`vitest.config.ts`**:

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: false,            // we import describe/it/expect explicitly — see Part I
    environment: "node",       // this is a CLI/service, not a browser app
    setupFiles: ["./vitest.setup.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "html", "lcov"],
      exclude: ["**/*.d.ts", "src/index.ts"],
      thresholds: {
        lines: 85,
        functions: 85,
        branches: 80,
        statements: 85,
      },
    },
    testTimeout: 10_000,
  },
});
```

`globals: false` is a deliberate choice: explicit imports (`import { describe, it, expect } from "vitest"`) make test files self-documenting and let your editor autocomplete correctly without a global type-shim. If you'd rather write Jest-style code with no imports, flip it to `true` and add `"types": ["vitest/globals"]` to `tsconfig.json`. We use explicit imports throughout this guide.

**`vitest.setup.ts`** — runs once before the whole suite, good for global fixtures:

```ts
import { afterEach, vi } from "vitest";

// Make sure no test leaks a mock into the next one.
afterEach(() => {
  vi.restoreAllMocks();
});
```

---

## 4. Part I — Fundamentals

### 4.1 The domain types

```ts
// src/types.ts
export interface DiffHunk {
  filePath: string;
  startLine: number;
  addedLines: string[];
  removedLines: string[];
}

export type Severity = "info" | "warning" | "critical";

export interface ReviewFinding {
  filePath: string;
  line: number;
  severity: Severity;
  message: string;
}

// Discriminated union: every async pipeline needs a result type like this.
export type OllamaResult<T> =
  | { ok: true; value: T }
  | { ok: false; error: OllamaError };

export interface OllamaChatChunk {
  message: { content: string };
  done: boolean;
}
```

```ts
// src/errors.ts
export class OllamaError extends Error {
  constructor(
    message: string,
    public readonly code: "TIMEOUT" | "RATE_LIMIT" | "SERVER_ERROR" | "PARSE_ERROR",
    public readonly retryable: boolean,
  ) {
    super(message);
    this.name = "OllamaError";
  }
}

export class InvalidDiffError extends Error {
  constructor(reason: string) {
    super(`Invalid diff: ${reason}`);
    this.name = "InvalidDiffError";
  }
}
```

### 4.2 Anatomy of a test file

```ts
// tests/errors.test.ts
import { describe, it, expect } from "vitest";
import { OllamaError, InvalidDiffError } from "../src/errors";

describe("OllamaError", () => {
  it("carries a machine-readable error code", () => {
    const err = new OllamaError("timed out", "TIMEOUT", true);

    expect(err.code).toBe("TIMEOUT");
    expect(err.retryable).toBe(true);
    expect(err).toBeInstanceOf(Error);
    expect(err.message).toBe("timed out");
  });

  it("sets a distinguishing name for log filtering", () => {
    const err = new OllamaError("boom", "SERVER_ERROR", false);
    expect(err.name).toBe("OllamaError");
  });
});

describe("InvalidDiffError", () => {
  it("prefixes the reason for readability", () => {
    const err = new InvalidDiffError("missing @@ header");
    expect(err.message).toMatch(/^Invalid diff:/);
    expect(err.message).toContain("missing @@ header");
  });
});
```

Key building blocks:

| Construct | Purpose |
|---|---|
| `describe(name, fn)` | Groups related tests; nestable |
| `it` / `test` | A single test case (identical, `it` reads better in BDD-style prose) |
| `expect(value)` | Wraps a value so you can assert on it |
| `it.skip` / `it.only` / `it.todo` | Skip a test, isolate one test, or mark a stub for later |
| `it.concurrent` | Run this test in parallel with other concurrent tests in the same file |

### 4.3 Lifecycle hooks

```ts
import { describe, it, expect, beforeAll, beforeEach, afterEach, afterAll } from "vitest";

describe("lifecycle order", () => {
  beforeAll(() => console.log("1. once, before any test in this describe"));
  beforeEach(() => console.log("2. before every test"));
  afterEach(() => console.log("3. after every test"));
  afterAll(() => console.log("4. once, after all tests"));

  it("test A", () => {});
  it("test B", () => {});
});
```

Rule of thumb: **`beforeEach` for anything mutable** (fresh instances, reset mocks). **`beforeAll` only for expensive, truly immutable setup** — because state accidentally shared across tests via `beforeAll` is the single most common cause of flaky suites.

### 4.4 Common matchers you'll actually use

```ts
expect(2 + 2).toBe(4);                       // Object.is equality (primitives)
expect({ a: 1 }).toEqual({ a: 1 });          // deep equality (objects/arrays)
expect({ a: 1, b: 2 }).toMatchObject({ a: 1 }); // partial deep equality
expect([1, 2, 3]).toContain(2);
expect("Ollama Cloud").toMatch(/Cloud/);
expect(null).toBeNull();
expect(undefined).toBeUndefined();
expect(findings.length).toBeGreaterThan(0);
expect(fn).toThrow(InvalidDiffError);
await expect(promise).resolves.toBe("ok");
await expect(promise).rejects.toThrow(OllamaError);
expect(value).not.toBe(other);               // .not negates any matcher
```

---

## 5. Part II — Testing Pure Functions: the Diff Parser

Pure functions are the cheapest, highest-value tests you will ever write — no mocks, no async, just input/output. Always build a pure core around your side-effecting shell (this one parses text; it never touches the filesystem or network).

```ts
// src/diff-parser.ts
import { InvalidDiffError } from "./errors";
import type { DiffHunk } from "./types";

const FILE_HEADER = /^diff --git a\/(.+) b\/(.+)$/;
const HUNK_HEADER = /^@@ -\d+(?:,\d+)? \+(\d+)(?:,\d+)? @@/;

export function parseDiff(raw: string): DiffHunk[] {
  if (!raw.trim()) {
    throw new InvalidDiffError("empty diff");
  }

  const lines = raw.split("\n");
  const hunks: DiffHunk[] = [];
  let currentFile = "";
  let current: DiffHunk | null = null;

  for (const line of lines) {
    const fileMatch = line.match(FILE_HEADER);
    if (fileMatch) {
      currentFile = fileMatch[2];
      continue;
    }

    const hunkMatch = line.match(HUNK_HEADER);
    if (hunkMatch) {
      if (!currentFile) throw new InvalidDiffError("hunk without file header");
      current = {
        filePath: currentFile,
        startLine: Number(hunkMatch[1]),
        addedLines: [],
        removedLines: [],
      };
      hunks.push(current);
      continue;
    }

    if (!current) continue;
    if (line.startsWith("+") && !line.startsWith("+++")) {
      current.addedLines.push(line.slice(1));
    } else if (line.startsWith("-") && !line.startsWith("---")) {
      current.removedLines.push(line.slice(1));
    }
  }

  return hunks;
}

export function summarizeHunks(hunks: DiffHunk[]) {
  return {
    fileCount: new Set(hunks.map((h) => h.filePath)).size,
    linesAdded: hunks.reduce((sum, h) => sum + h.addedLines.length, 0),
    linesRemoved: hunks.reduce((sum, h) => sum + h.removedLines.length, 0),
  };
}
```

### 5.1 Table-driven tests with `test.each`

This is the single highest-leverage Vitest feature for pure functions: instead of copy-pasting near-identical `it` blocks, describe the *shape* once and feed it a table of cases.

```ts
// tests/diff-parser.test.ts
import { describe, it, expect } from "vitest";
import { parseDiff, summarizeHunks } from "../src/diff-parser";
import { InvalidDiffError } from "../src/errors";

const SAMPLE_DIFF = `diff --git a/src/math.ts b/src/math.ts
@@ -1,3 +1,4 @@
 export function add(a: number, b: number) {
-  return a + b
+  return a + b;
+}
`;

describe("parseDiff", () => {
  it("extracts the file path from the diff header", () => {
    const [hunk] = parseDiff(SAMPLE_DIFF);
    expect(hunk.filePath).toBe("src/math.ts");
  });

  it("captures added and removed lines separately", () => {
    const [hunk] = parseDiff(SAMPLE_DIFF);
    expect(hunk.removedLines).toEqual(["  return a + b"]);
    expect(hunk.addedLines).toEqual(["  return a + b;", "}"]);
  });

  it("throws InvalidDiffError on empty input", () => {
    expect(() => parseDiff("")).toThrow(InvalidDiffError);
    expect(() => parseDiff("   \n  ")).toThrow(/empty diff/);
  });

  it("throws when a hunk appears before any file header", () => {
    expect(() => parseDiff("@@ -1,1 +1,1 @@\n+x")).toThrow(/hunk without file header/);
  });

  // ---- table-driven edge cases ----
  it.each([
    { input: "diff --git a/x.ts b/x.ts\n@@ -1,0 +1,0 @@\n", expected: 0 },
    { input: SAMPLE_DIFF, expected: 1 },
    { input: SAMPLE_DIFF + SAMPLE_DIFF.replace("math.ts", "util.ts"), expected: 2 },
  ])("produces $expected hunk(s) for a given diff", ({ input, expected }) => {
    expect(parseDiff(input)).toHaveLength(expected);
  });

  // Same idea using a plain array + describe.each, useful when each case needs multiple assertions
  describe.each([
    { addedText: "+console.log('debug')", severity: "leftover debug statement" },
    { addedText: "+// TODO: fix", severity: "TODO marker" },
  ])("style smells: $severity", ({ addedText }) => {
    it("is captured as an added line", () => {
      const diff = `diff --git a/a.ts b/a.ts\n@@ -1,0 +1,1 @@\n${addedText}\n`;
      const [hunk] = parseDiff(diff);
      expect(hunk.addedLines[0]).toBe(addedText.slice(1));
    });
  });
});

describe("summarizeHunks", () => {
  it("counts unique files, not hunks", () => {
    const hunks = [
      ...parseDiff(SAMPLE_DIFF),
      ...parseDiff(SAMPLE_DIFF), // same file again, e.g. two hunks in one file
    ];
    expect(summarizeHunks(hunks).fileCount).toBe(1);
  });

  it("sums added/removed lines across all hunks", () => {
    const hunks = parseDiff(SAMPLE_DIFF);
    expect(summarizeHunks(hunks)).toEqual({
      fileCount: 1,
      linesAdded: 2,
      linesRemoved: 1,
    });
  });

  it("returns zeros for an empty hunk list", () => {
    expect(summarizeHunks([])).toEqual({ fileCount: 0, linesAdded: 0, linesRemoved: 0 });
  });
});
```

**Why this section matters for "core JS/TS concepts":** regex extraction, array `.reduce`/`.map`, `Set` for uniqueness, string slicing, discriminated control flow, and custom error classes — all covered with zero mocks. If your pure-logic layer isn't tested this thoroughly, no amount of mocking upstream will save you.


---

## 6. Part III — Mocking Basics

Before mocking a whole module (Part IV), master the two primitives everything else is built on.

### 6.1 `vi.fn()` — a fake function you can inspect

```ts
import { describe, it, expect, vi } from "vitest";

describe("vi.fn basics", () => {
  it("records calls and arguments", () => {
    const onFinding = vi.fn();

    onFinding({ severity: "critical", message: "SQL injection risk" });
    onFinding({ severity: "info", message: "unused import" });

    expect(onFinding).toHaveBeenCalledTimes(2);
    expect(onFinding).toHaveBeenNthCalledWith(1, expect.objectContaining({ severity: "critical" }));
    expect(onFinding).toHaveBeenLastCalledWith(expect.objectContaining({ severity: "info" }));
  });

  it("can return canned values, including different values per call", () => {
    const nextId = vi.fn()
      .mockReturnValueOnce("id-1")
      .mockReturnValueOnce("id-2")
      .mockReturnValue("id-fallback");

    expect(nextId()).toBe("id-1");
    expect(nextId()).toBe("id-2");
    expect(nextId()).toBe("id-fallback");
    expect(nextId()).toBe("id-fallback");
  });

  it("can simulate async success and failure", async () => {
    const fetchModel = vi.fn()
      .mockResolvedValueOnce({ model: "qwen3-coder:480b" })
      .mockRejectedValueOnce(new Error("network down"));

    await expect(fetchModel()).resolves.toEqual({ model: "qwen3-coder:480b" });
    await expect(fetchModel()).rejects.toThrow("network down");
  });

  it("can delegate to a real implementation", () => {
    const double = vi.fn((n: number) => n * 2);
    expect(double(21)).toBe(42);
    expect(double).toHaveBeenCalledWith(21);
  });
});
```

### 6.2 `vi.spyOn()` — wrap an existing method, keep or override behavior

```ts
describe("vi.spyOn", () => {
  it("observes calls to a real method without changing behavior", () => {
    const rateLimiter = { tryAcquire: () => true };
    const spy = vi.spyOn(rateLimiter, "tryAcquire");

    rateLimiter.tryAcquire();

    expect(spy).toHaveBeenCalledOnce();
    spy.mockRestore(); // give the real method back
  });

  it("can override behavior just like vi.fn", () => {
    const clock = { now: () => Date.now() };
    vi.spyOn(clock, "now").mockReturnValue(1_700_000_000_000);

    expect(clock.now()).toBe(1_700_000_000_000);
  });
});
```

**`vi.fn` vs `vi.spyOn`:** use `vi.fn()` for a callback you invented yourself (e.g. a listener passed into your code). Use `vi.spyOn(obj, "method")` when the function already exists on a real object/module and you want to intercept it in place.

### 6.3 Clearing, resetting, restoring — know the difference

```ts
vi.clearAllMocks();   // wipes call history (mock.calls, mock.results) — keeps implementations
vi.resetAllMocks();   // clearAllMocks() + removes any mockReturnValue/mockImplementation
vi.restoreAllMocks();  // resetAllMocks() + puts spied-on real implementations back
```

Put `vi.restoreAllMocks()` in a global `afterEach` (as we did in `vitest.setup.ts`) so no test can leak mock state into the next one — this single line prevents a huge class of "works alone, fails in the full suite" bugs.


---

## 7. Part IV — The Ollama Cloud Client

This is the heart of the AI-app-specific testing: an HTTP client that talks to Ollama Cloud's `/api/chat` endpoint, streams newline-delimited JSON, and retries on transient errors.

```ts
// src/ollama-client.ts
import { OllamaError } from "./errors";
import type { OllamaChatChunk, OllamaResult } from "./types";

export interface OllamaClientOptions {
  apiKey: string;
  baseUrl?: string;
  model?: string;
  maxRetries?: number;
  fetchImpl?: typeof fetch;
}

export class OllamaClient {
  private readonly baseUrl: string;
  private readonly model: string;
  private readonly maxRetries: number;
  private readonly fetchImpl: typeof fetch;

  constructor(private readonly opts: OllamaClientOptions) {
    this.baseUrl = opts.baseUrl ?? "https://ollama.com";
    this.model = opts.model ?? "qwen3-coder:480b-cloud";
    this.maxRetries = opts.maxRetries ?? 3;
    this.fetchImpl = opts.fetchImpl ?? fetch;
  }

  async chat(prompt: string): Promise<OllamaResult<string>> {
    let attempt = 0;
    let lastError: OllamaError | null = null;

    while (attempt <= this.maxRetries) {
      try {
        const response = await this.fetchImpl(`${this.baseUrl}/api/chat`, {
          method: "POST",
          headers: {
            Authorization: `Bearer ${this.opts.apiKey}`,
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            model: this.model,
            messages: [{ role: "user", content: prompt }],
            stream: true,
          }),
        });

        if (response.status === 429) {
          throw new OllamaError("rate limited by Ollama Cloud", "RATE_LIMIT", true);
        }
        if (response.status >= 500) {
          throw new OllamaError(`server error ${response.status}`, "SERVER_ERROR", true);
        }
        if (!response.ok) {
          throw new OllamaError(`request failed: ${response.status}`, "SERVER_ERROR", false);
        }
        if (!response.body) {
          throw new OllamaError("empty response body", "PARSE_ERROR", false);
        }

        return { ok: true, value: await this.consumeStream(response.body) };
      } catch (err) {
        lastError = this.normalizeError(err);
        if (!lastError.retryable || attempt === this.maxRetries) break;

        const backoffMs = 2 ** attempt * 250; // 250, 500, 1000...
        await this.sleep(backoffMs);
        attempt++;
      }
    }

    return { ok: false, error: lastError! };
  }

  async *streamChat(prompt: string): AsyncGenerator<string> {
    const response = await this.fetchImpl(`${this.baseUrl}/api/chat`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${this.opts.apiKey}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ model: this.model, messages: [{ role: "user", content: prompt }], stream: true }),
    });

    if (!response.ok || !response.body) {
      throw new OllamaError(`stream failed: ${response.status}`, "SERVER_ERROR", response.status >= 500);
    }

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = "";

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      buffer += decoder.decode(value, { stream: true });

      let newlineIndex: number;
      while ((newlineIndex = buffer.indexOf("\n")) >= 0) {
        const line = buffer.slice(0, newlineIndex).trim();
        buffer = buffer.slice(newlineIndex + 1);
        if (!line) continue;

        const chunk = this.parseChunk(line);
        if (chunk.message.content) yield chunk.message.content;
        if (chunk.done) return;
      }
    }
  }

  private async consumeStream(body: ReadableStream<Uint8Array>): Promise<string> {
    let full = "";
    for await (const piece of this.streamFromBody(body)) full += piece;
    return full;
  }

  private async *streamFromBody(body: ReadableStream<Uint8Array>): AsyncGenerator<string> {
    const reader = body.getReader();
    const decoder = new TextDecoder();
    let buffer = "";
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      buffer += decoder.decode(value, { stream: true });
      let idx: number;
      while ((idx = buffer.indexOf("\n")) >= 0) {
        const line = buffer.slice(0, idx).trim();
        buffer = buffer.slice(idx + 1);
        if (!line) continue;
        const chunk = this.parseChunk(line);
        if (chunk.message.content) yield chunk.message.content;
      }
    }
  }

  private parseChunk(line: string): OllamaChatChunk {
    try {
      return JSON.parse(line) as OllamaChatChunk;
    } catch {
      throw new OllamaError(`malformed chunk: ${line.slice(0, 50)}`, "PARSE_ERROR", false);
    }
  }

  private normalizeError(err: unknown): OllamaError {
    if (err instanceof OllamaError) return err;
    if (err instanceof Error && err.name === "AbortError") {
      return new OllamaError("request timed out", "TIMEOUT", true);
    }
    return new OllamaError(err instanceof Error ? err.message : "unknown error", "SERVER_ERROR", true);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}
```

### 7.1 Mocking global `fetch` directly

The simplest strategy: inject `fetch` (we already accept `fetchImpl` in the constructor for exactly this reason — **design for testability**). No `vi.mock` needed at all.

```ts
// tests/ollama-client.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { OllamaClient } from "../src/ollama-client";
import { OllamaError } from "../src/errors";

function jsonLineStream(chunks: object[]): ReadableStream<Uint8Array> {
  const encoder = new TextEncoder();
  return new ReadableStream({
    start(controller) {
      for (const chunk of chunks) {
        controller.enqueue(encoder.encode(JSON.stringify(chunk) + "\n"));
      }
      controller.close();
    },
  });
}

function makeResponse(status: number, body: ReadableStream<Uint8Array> | null) {
  return { ok: status >= 200 && status < 300, status, body } as Response;
}

describe("OllamaClient.chat", () => {
  it("concatenates streamed content chunks into one string", async () => {
    const fetchImpl = vi.fn().mockResolvedValue(
      makeResponse(
        200,
        jsonLineStream([
          { message: { content: "The " }, done: false },
          { message: { content: "diff " }, done: false },
          { message: { content: "looks good." }, done: true },
        ]),
      ),
    );

    const client = new OllamaClient({ apiKey: "test-key", fetchImpl });
    const result = await client.chat("Review this diff");

    expect(result).toEqual({ ok: true, value: "The diff looks good." });
    expect(fetchImpl).toHaveBeenCalledWith(
      "https://ollama.com/api/chat",
      expect.objectContaining({ method: "POST" }),
    );
  });

  it("sends the API key as a bearer token", async () => {
    const fetchImpl = vi.fn().mockResolvedValue(makeResponse(200, jsonLineStream([{ message: { content: "ok" }, done: true }])));
    const client = new OllamaClient({ apiKey: "secret-123", fetchImpl });

    await client.chat("hi");

    const [, init] = fetchImpl.mock.calls[0];
    const headers = init.headers as Record<string, string>;
    expect(headers.Authorization).toBe("Bearer secret-123");
  });

  it("returns a typed ok:false result instead of throwing on a 400", async () => {
    const fetchImpl = vi.fn().mockResolvedValue(makeResponse(400, null));
    const client = new OllamaClient({ apiKey: "k", fetchImpl, maxRetries: 0 });

    const result = await client.chat("bad prompt");

    expect(result.ok).toBe(false);
    if (!result.ok) {
      expect(result.error).toBeInstanceOf(OllamaError);
      expect(result.error.code).toBe("SERVER_ERROR");
      expect(result.error.retryable).toBe(false);
    }
  });

  it("surfaces a PARSE_ERROR when a chunk is not valid JSON", async () => {
    const encoder = new TextEncoder();
    const badStream = new ReadableStream<Uint8Array>({
      start(controller) {
        controller.enqueue(encoder.encode("not-json\n"));
        controller.close();
      },
    });
    const fetchImpl = vi.fn().mockResolvedValue(makeResponse(200, badStream));
    const client = new OllamaClient({ apiKey: "k", fetchImpl, maxRetries: 0 });

    const result = await client.chat("hi");

    expect(result.ok).toBe(false);
    if (!result.ok) expect(result.error.code).toBe("PARSE_ERROR");
  });
});
```

### 7.2 `vi.mock` + `vi.hoisted` — mocking a whole module

Injecting `fetch` works great for a leaf client. But once other modules `import { OllamaClient } from "./ollama-client"` and you're testing *them*, you don't want to rebuild a fake `ReadableStream` every time — you mock the whole module instead.

```ts
// tests/review-service.test.ts (excerpt — full file in Part IX)
import { describe, it, expect, vi } from "vitest";

// vi.hoisted lets us define the mock factory's internals before vi.mock's hoisting
// moves the vi.mock call itself to the top of the file (this avoids "used before
// initialization" errors when the mock needs a variable you also want to assert on).
const { mockChat } = vi.hoisted(() => ({
  mockChat: vi.fn(),
}));

vi.mock("../src/ollama-client", () => ({
  OllamaClient: vi.fn().mockImplementation(() => ({
    chat: mockChat,
  })),
}));

import { ReviewService } from "../src/review-service";

describe("ReviewService with a fully mocked OllamaClient", () => {
  it("passes the model's raw response through to the parser", async () => {
    mockChat.mockResolvedValue({
      ok: true,
      value: JSON.stringify([{ line: 3, severity: "warning", message: "unused variable" }]),
    });

    // ... construct ReviewService and assert (see Part IX for the full setup)
  });
});
```

**Important gotcha:** `vi.mock` calls are hoisted to the top of the file by Vitest's transform step — *before* your imports run. That means any variable you reference inside the factory must itself be created in a way that survives hoisting, which is precisely what `vi.hoisted()` is for. Forgetting this is the #1 source of "cannot access 'x' before initialization" errors when mocking modules.


---

## 8. Part V — Streaming & Async Generators

`streamChat` is an `AsyncGenerator<string>` — a core modern-JS construct for AI apps (token-by-token UI updates). Testing it means consuming it exactly the way real callers would: with `for await...of`.

```ts
// tests/ollama-client.stream.test.ts
import { describe, it, expect, vi } from "vitest";
import { OllamaClient } from "../src/ollama-client";
import { OllamaError } from "../src/errors";

function sseLikeStream(pieces: string[]): ReadableStream<Uint8Array> {
  const encoder = new TextEncoder();
  let i = 0;
  return new ReadableStream({
    pull(controller) {
      if (i < pieces.length) {
        controller.enqueue(encoder.encode(pieces[i]));
        i++;
      } else {
        controller.close();
      }
    },
  });
}

describe("OllamaClient.streamChat", () => {
  it("yields each content token as it arrives", async () => {
    const chunks = [
      JSON.stringify({ message: { content: "def" }, done: false }) + "\n",
      JSON.stringify({ message: { content: "ects" }, done: false }) + "\n",
      JSON.stringify({ message: { content: " found: 2" }, done: true }) + "\n",
    ];
    const fetchImpl = vi.fn().mockResolvedValue({
      ok: true,
      status: 200,
      body: sseLikeStream(chunks),
    } as Response);

    const client = new OllamaClient({ apiKey: "k", fetchImpl });

    const received: string[] = [];
    for await (const token of client.streamChat("review this")) {
      received.push(token);
    }

    expect(received).toEqual(["def", "ects", " found: 2"]);
  });

  it("handles a JSON object split across two network chunks", async () => {
    // Simulates TCP fragmenting a single line across reader.read() calls.
    const raw = JSON.stringify({ message: { content: "hello world" }, done: true }) + "\n";
    const mid = Math.floor(raw.length / 2);
    const fetchImpl = vi.fn().mockResolvedValue({
      ok: true,
      status: 200,
      body: sseLikeStream([raw.slice(0, mid), raw.slice(mid)]),
    } as Response);

    const client = new OllamaClient({ apiKey: "k", fetchImpl });
    const tokens: string[] = [];
    for await (const t of client.streamChat("x")) tokens.push(t);

    expect(tokens).toEqual(["hello world"]);
  });

  it("stops iterating as soon as done:true is received, ignoring trailing junk", async () => {
    const chunks = [
      JSON.stringify({ message: { content: "a" }, done: true }) + "\n",
      "this line should never be parsed\n",
    ];
    const fetchImpl = vi.fn().mockResolvedValue({
      ok: true,
      status: 200,
      body: sseLikeStream(chunks),
    } as Response);

    const client = new OllamaClient({ apiKey: "k", fetchImpl });
    const tokens: string[] = [];
    for await (const t of client.streamChat("x")) tokens.push(t);

    expect(tokens).toEqual(["a"]);
  });

  it("throws OllamaError immediately if the initial response is not ok", async () => {
    const fetchImpl = vi.fn().mockResolvedValue({ ok: false, status: 503, body: null } as Response);
    const client = new OllamaClient({ apiKey: "k", fetchImpl });

    await expect(async () => {
      for await (const _ of client.streamChat("x")) {
        // never reached
      }
    }).rejects.toThrow(OllamaError);
  });
});
```

**Pattern to remember:** an `AsyncGenerator` is tested by *draining* it — either with `for await...of` into an array, or by manually calling `.next()` when you need to assert on intermediate state between yields:

```ts
it("can be driven step by step with .next()", async () => {
  const client = new OllamaClient({ apiKey: "k", fetchImpl: /* ... */ vi.fn() });
  const gen = client.streamChat("x");

  const first = await gen.next();
  expect(first.done).toBe(false);
  // ... assert, then continue
});
```


---

## 9. Part VI — Retries, Backoff & Fake Timers

`chat()` retries with exponential backoff via real `setTimeout`. Awaiting real backoff (250ms, 500ms, 1000ms...) in a test suite that runs thousands of times is how CI pipelines become 20 minutes long. **Fake timers** solve this: time advances instantly, under your control.

```ts
// tests/ollama-client.retry.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { OllamaClient } from "../src/ollama-client";

function makeResponse(status: number) {
  return { ok: status < 300, status, body: null } as Response;
}

describe("OllamaClient retry/backoff", () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it("retries a 429 up to maxRetries, then gives up", async () => {
    const fetchImpl = vi.fn().mockResolvedValue(makeResponse(429));
    const client = new OllamaClient({ apiKey: "k", fetchImpl, maxRetries: 2 });

    const resultPromise = client.chat("hi");

    // The client is now `await`-ing setTimeout for the first backoff.
    // Fast-forward through all pending timers instead of waiting in real time.
    await vi.runAllTimersAsync();

    const result = await resultPromise;

    expect(fetchImpl).toHaveBeenCalledTimes(3); // 1 initial + 2 retries
    expect(result.ok).toBe(false);
  });

  it("uses exponential backoff: 250ms, then 500ms", async () => {
    const fetchImpl = vi.fn().mockResolvedValue(makeResponse(500));
    const client = new OllamaClient({ apiKey: "k", fetchImpl, maxRetries: 2 });

    const resultPromise = client.chat("hi");

    // First attempt happens synchronously (before any timer fires).
    await vi.advanceTimersByTimeAsync(0);
    expect(fetchImpl).toHaveBeenCalledTimes(1);

    // Advance past the first backoff window (250ms) — triggers retry #1.
    await vi.advanceTimersByTimeAsync(250);
    expect(fetchImpl).toHaveBeenCalledTimes(2);

    // Advance past the second backoff window (500ms) — triggers retry #2.
    await vi.advanceTimersByTimeAsync(500);
    expect(fetchImpl).toHaveBeenCalledTimes(3);

    await resultPromise;
  });

  it("does NOT retry a non-retryable error (e.g. 400 Bad Request)", async () => {
    const fetchImpl = vi.fn().mockResolvedValue(makeResponse(400));
    const client = new OllamaClient({ apiKey: "k", fetchImpl, maxRetries: 3 });

    const result = await client.chat("hi"); // no timers to fake-advance — it never sleeps

    expect(fetchImpl).toHaveBeenCalledTimes(1);
    expect(result.ok).toBe(false);
  });

  it("succeeds on the second attempt after one transient failure", async () => {
    const fetchImpl = vi.fn()
      .mockResolvedValueOnce(makeResponse(503))
      .mockResolvedValueOnce({
        ok: true,
        status: 200,
        body: new ReadableStream({
          start(c) {
            c.enqueue(new TextEncoder().encode(JSON.stringify({ message: { content: "ok" }, done: true }) + "\n"));
            c.close();
          },
        }),
      } as Response);

    const client = new OllamaClient({ apiKey: "k", fetchImpl, maxRetries: 3 });
    const resultPromise = client.chat("hi");

    await vi.runAllTimersAsync();
    const result = await resultPromise;

    expect(result).toEqual({ ok: true, value: "ok" });
    expect(fetchImpl).toHaveBeenCalledTimes(2);
  });
});
```

### 9.1 Fake timer API you'll reach for constantly

| API | What it does |
|---|---|
| `vi.useFakeTimers()` | Replace `setTimeout`/`setInterval`/`Date.now` with controllable fakes |
| `vi.advanceTimersByTime(ms)` | Synchronously fast-forward, firing due timers (no pending promises resolved) |
| `vi.advanceTimersByTimeAsync(ms)` | Same, but also flushes microtasks/promises in between — **use this when timer callbacks contain `await`**, which is almost always in real async code |
| `vi.runAllTimers()` / `vi.runAllTimersAsync()` | Fire every pending timer, including ones scheduled by other timers, until none remain |
| `vi.setSystemTime(date)` | Freeze `Date.now()`/`new Date()` to a fixed instant — essential for testing cache TTLs and rate-limiter windows |
| `vi.useRealTimers()` | Restore real timers — **always pair this with `useFakeTimers` in `afterEach`** |

**The #1 fake-timer bug:** calling `vi.advanceTimersByTime()` (sync) when the code under test awaits a promise inside the timer callback. The timer fires, but the `await` never gets a chance to resolve before your assertion runs. Always prefer the `*Async` variants for code that mixes `setTimeout` and `async/await` — which is exactly what our retry loop does.


---

## 10. Part VII — Stateful Classes

### 10.1 Token-bucket rate limiter

```ts
// src/rate-limiter.ts
export class TokenBucketRateLimiter {
  private tokens: number;
  private lastRefillMs: number;

  constructor(
    private readonly capacity: number,
    private readonly refillRatePerSecond: number,
    private readonly now: () => number = Date.now,
  ) {
    this.tokens = capacity;
    this.lastRefillMs = this.now();
  }

  private refill(): void {
    const elapsedSeconds = (this.now() - this.lastRefillMs) / 1000;
    const refillAmount = elapsedSeconds * this.refillRatePerSecond;
    if (refillAmount > 0) {
      this.tokens = Math.min(this.capacity, this.tokens + refillAmount);
      this.lastRefillMs = this.now();
    }
  }

  tryAcquire(cost = 1): boolean {
    this.refill();
    if (this.tokens >= cost) {
      this.tokens -= cost;
      return true;
    }
    return false;
  }

  async acquire(cost = 1): Promise<void> {
    while (!this.tryAcquire(cost)) {
      await new Promise((resolve) => setTimeout(resolve, 50));
    }
  }
}
```

```ts
// tests/rate-limiter.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { TokenBucketRateLimiter } from "../src/rate-limiter";

describe("TokenBucketRateLimiter", () => {
  // Inject a controllable clock instead of relying on Date.now() directly —
  // this is dependency injection at its simplest, and it makes the whole
  // suite deterministic without needing vi.useFakeTimers at all.
  let currentTime = 0;
  const clock = () => currentTime;

  beforeEach(() => {
    currentTime = 0;
  });

  it("starts full and allows `capacity` requests immediately", () => {
    const limiter = new TokenBucketRateLimiter(5, 1, clock);
    for (let i = 0; i < 5; i++) {
      expect(limiter.tryAcquire()).toBe(true);
    }
    expect(limiter.tryAcquire()).toBe(false);
  });

  it("refills proportionally to elapsed time", () => {
    const limiter = new TokenBucketRateLimiter(5, 2, clock); // 2 tokens/sec
    for (let i = 0; i < 5; i++) limiter.tryAcquire();
    expect(limiter.tryAcquire()).toBe(false);

    currentTime += 1000; // 1 second passes -> +2 tokens
    expect(limiter.tryAcquire()).toBe(true);
    expect(limiter.tryAcquire()).toBe(true);
    expect(limiter.tryAcquire()).toBe(false);
  });

  it("never refills above capacity", () => {
    const limiter = new TokenBucketRateLimiter(3, 100, clock);
    limiter.tryAcquire(); // 2 left
    currentTime += 10_000; // huge time jump
    expect(limiter.tryAcquire(3)).toBe(true); // capped at 3, so exactly enough
    expect(limiter.tryAcquire()).toBe(false);
  });

  it("supports variable request cost", () => {
    const limiter = new TokenBucketRateLimiter(10, 0, clock);
    expect(limiter.tryAcquire(7)).toBe(true);
    expect(limiter.tryAcquire(4)).toBe(false); // only 3 left
    expect(limiter.tryAcquire(3)).toBe(true);
  });

  describe("acquire() — the async waiting variant", () => {
    beforeEach(() => vi.useFakeTimers());
    afterEach(() => vi.useRealTimers());

    it("resolves once tokens become available", async () => {
      const limiter = new TokenBucketRateLimiter(1, 1, clock);
      limiter.tryAcquire(); // drain the only token

      const acquirePromise = limiter.acquire();
      let resolved = false;
      acquirePromise.then(() => (resolved = true));

      await vi.advanceTimersByTimeAsync(50);
      expect(resolved).toBe(false); // clock (our injected one) hasn't moved, still no tokens

      currentTime += 2000; // now the bucket has refilled
      await vi.advanceTimersByTimeAsync(50);
      expect(resolved).toBe(true);
    });
  });
});
```

### 10.2 Generic LRU cache

```ts
// src/lru-cache.ts
export class LRUCache<K, V> {
  private readonly map = new Map<K, { value: V; expiresAt: number }>();

  constructor(
    private readonly maxSize: number,
    private readonly ttlMs: number,
    private readonly now: () => number = Date.now,
  ) {}

  get(key: K): V | undefined {
    const entry = this.map.get(key);
    if (!entry) return undefined;

    if (this.now() >= entry.expiresAt) {
      this.map.delete(key);
      return undefined;
    }

    // Re-insert to mark as most-recently-used (Map preserves insertion order).
    this.map.delete(key);
    this.map.set(key, entry);
    return entry.value;
  }

  set(key: K, value: V): void {
    this.map.delete(key);
    if (this.map.size >= this.maxSize) {
      const oldestKey = this.map.keys().next().value as K;
      this.map.delete(oldestKey);
    }
    this.map.set(key, { value, expiresAt: this.now() + this.ttlMs });
  }

  get size(): number {
    return this.map.size;
  }
}
```

```ts
// tests/lru-cache.test.ts
import { describe, it, expect } from "vitest";
import { LRUCache } from "../src/lru-cache";

describe("LRUCache", () => {
  it("returns undefined for a missing key", () => {
    const cache = new LRUCache<string, number>(2, 1000);
    expect(cache.get("missing")).toBeUndefined();
  });

  it("stores and retrieves values", () => {
    const cache = new LRUCache<string, string>(2, 1000);
    cache.set("diff-hash-1", "review result A");
    expect(cache.get("diff-hash-1")).toBe("review result A");
  });

  it("evicts the least-recently-used entry when full", () => {
    const cache = new LRUCache<string, number>(2, 1000);
    cache.set("a", 1);
    cache.set("b", 2);
    cache.set("c", 3); // capacity 2 -> "a" evicted

    expect(cache.get("a")).toBeUndefined();
    expect(cache.get("b")).toBe(2);
    expect(cache.get("c")).toBe(3);
  });

  it("counts a get() as recent use, protecting it from eviction", () => {
    const cache = new LRUCache<string, number>(2, 1000);
    cache.set("a", 1);
    cache.set("b", 2);
    cache.get("a");       // "a" is now more recent than "b"
    cache.set("c", 3);    // should evict "b", not "a"

    expect(cache.get("a")).toBe(1);
    expect(cache.get("b")).toBeUndefined();
  });

  it("expires entries after the TTL using an injected clock", () => {
    let time = 0;
    const cache = new LRUCache<string, number>(10, 1000, () => time);

    cache.set("k", 42);
    time += 999;
    expect(cache.get("k")).toBe(42);

    time += 2; // total elapsed 1001ms, past the 1000ms TTL
    expect(cache.get("k")).toBeUndefined();
  });

  it("tracks size correctly, including after eviction", () => {
    const cache = new LRUCache<string, number>(2, 1000);
    cache.set("a", 1);
    cache.set("b", 2);
    cache.set("c", 3);
    expect(cache.size).toBe(2);
  });
});
```

Notice the pattern repeated across the rate limiter and the cache: **inject the clock as a constructor parameter with a `Date.now` default.** This is a small design decision that turns "flaky, slow, real-time-dependent tests" into "instant, deterministic tests," and it works with or without `vi.useFakeTimers`. Prefer it for pure time-based logic; reach for `vi.useFakeTimers` when the code under test uses `setTimeout` internally (like `acquire()` and the retry loop) since you can't inject your way out of a raw `setTimeout` call.


---

## 11. Part VIII — The Job Queue

A concurrency-limited queue built on `EventEmitter`, used to review many files without hammering Ollama Cloud with unlimited parallel requests.

```ts
// src/review-queue.ts
import { EventEmitter } from "node:events";

export interface ReviewJob<T> {
  id: string;
  run: () => Promise<T>;
}

export interface QueueEvents<T> {
  "job:start": (id: string) => void;
  "job:success": (id: string, result: T) => void;
  "job:error": (id: string, error: Error) => void;
  "queue:idle": () => void;
}

export class ConcurrentReviewQueue<T> extends EventEmitter {
  private readonly pending: ReviewJob<T>[] = [];
  private active = 0;

  constructor(private readonly concurrency: number) {
    super();
  }

  add(job: ReviewJob<T>): void {
    this.pending.push(job);
    this.drain();
  }

  private drain(): void {
    while (this.active < this.concurrency && this.pending.length > 0) {
      const job = this.pending.shift()!;
      this.active++;
      this.emit("job:start", job.id);

      job
        .run()
        .then((result) => this.emit("job:success", job.id, result))
        .catch((error: Error) => this.emit("job:error", job.id, error))
        .finally(() => {
          this.active--;
          if (this.active === 0 && this.pending.length === 0) {
            this.emit("queue:idle");
          }
          this.drain();
        });
    }
  }

  get activeCount(): number {
    return this.active;
  }

  get pendingCount(): number {
    return this.pending.length;
  }
}
```

```ts
// tests/review-queue.test.ts
import { describe, it, expect, vi } from "vitest";
import { ConcurrentReviewQueue } from "../src/review-queue";

/** Waits for an EventEmitter to fire `event`, resolving with its args. */
function once<T extends unknown[]>(emitter: EventEmitter, event: string): Promise<T> {
  return new Promise((resolve) => emitter.once(event, (...args: T) => resolve(args)));
}

describe("ConcurrentReviewQueue", () => {
  it("never runs more than `concurrency` jobs at once", async () => {
    const queue = new ConcurrentReviewQueue<string>(2);
    let maxConcurrent = 0;

    const makeJob = (id: string, delayMs: number) => ({
      id,
      run: () =>
        new Promise<string>((resolve) => {
          setTimeout(() => resolve(`done:${id}`), delayMs);
        }),
    });

    queue.on("job:start", () => {
      maxConcurrent = Math.max(maxConcurrent, queue.activeCount);
    });

    const idle = once(queue, "queue:idle");
    for (let i = 0; i < 5; i++) queue.add(makeJob(`job-${i}`, 10));
    await idle;

    expect(maxConcurrent).toBeLessThanOrEqual(2);
  });

  it("emits job:success with the resolved value", async () => {
    const queue = new ConcurrentReviewQueue<number>(1);
    const successHandler = vi.fn();
    queue.on("job:success", successHandler);

    queue.add({ id: "square-6", run: () => Promise.resolve(36) });
    await once(queue, "queue:idle");

    expect(successHandler).toHaveBeenCalledWith("square-6", 36);
  });

  it("emits job:error instead of throwing when a job rejects", async () => {
    const queue = new ConcurrentReviewQueue<never>(1);
    const errorHandler = vi.fn();
    queue.on("job:error", errorHandler);

    const boom = new Error("Ollama Cloud 503");
    queue.add({ id: "bad-job", run: () => Promise.reject(boom) });
    await once(queue, "queue:idle");

    expect(errorHandler).toHaveBeenCalledWith("bad-job", boom);
  });

  it("continues processing later jobs after an earlier one fails", async () => {
    const queue = new ConcurrentReviewQueue<string>(1);
    const results: string[] = [];
    queue.on("job:success", (id) => results.push(id));

    queue.add({ id: "a", run: () => Promise.reject(new Error("fail")) });
    queue.add({ id: "b", run: () => Promise.resolve("ok") });

    await once(queue, "queue:idle");

    expect(results).toEqual(["b"]);
  });

  it("emits queue:idle exactly once for a batch of jobs added synchronously", async () => {
    const queue = new ConcurrentReviewQueue<number>(3);
    const idleHandler = vi.fn();
    queue.on("queue:idle", idleHandler);

    for (let i = 0; i < 6; i++) {
      queue.add({ id: `j${i}`, run: () => Promise.resolve(i) });
    }
    await once(queue, "queue:idle");
    // flush any microtasks so a hypothetical double-emit would have already happened
    await new Promise((r) => setTimeout(r, 0));

    expect(idleHandler).toHaveBeenCalledTimes(1);
  });

  it("exposes pendingCount and activeCount for observability", () => {
    const queue = new ConcurrentReviewQueue<string>(1);
    queue.add({ id: "slow", run: () => new Promise(() => {}) }); // never resolves
    queue.add({ id: "queued", run: () => Promise.resolve("x") });

    expect(queue.activeCount).toBe(1);
    expect(queue.pendingCount).toBe(1);
  });
});
```

This section demonstrates a pattern many tutorials skip: **testing event-driven async code without arbitrary `setTimeout(resolve, N)` sleeps in the test itself.** The `once()` helper turns "wait for the emitter to say it's done" into a proper awaitable promise, which is both faster and non-flaky compared to guessing a sleep duration.


---

## 12. Part IX — The Orchestrator

`ReviewService` is the dependency-injection root: it takes *interfaces*, not concrete classes, so tests can substitute fully controlled fakes for every collaborator. This is the pattern that makes large real-world apps testable at all.

```ts
// src/report-generator.ts (needed by the orchestrator; full version in Part X)
import type { ReviewFinding } from "./types";

export function generateMarkdownReport(findings: ReviewFinding[]): string {
  if (findings.length === 0) return "# Review Report\n\nNo issues found. ✅\n";

  const bySeverityOrder = { critical: 0, warning: 1, info: 2 } as const;
  const sorted = [...findings].sort((a, b) => bySeverityOrder[a.severity] - bySeverityOrder[b.severity]);

  const lines = ["# Review Report", ""];
  for (const f of sorted) {
    const badge = f.severity === "critical" ? "🔴" : f.severity === "warning" ? "🟡" : "🔵";
    lines.push(`- ${badge} **${f.filePath}:${f.line}** — ${f.message}`);
  }
  return lines.join("\n") + "\n";
}
```

```ts
// src/review-service.ts
import { parseDiff } from "./diff-parser";
import { generateMarkdownReport } from "./report-generator";
import type { ReviewFinding } from "./types";

export interface ModelClient {
  chat(prompt: string): Promise<{ ok: boolean; value?: string; error?: Error }>;
}

export interface RateLimiter {
  acquire(): Promise<void>;
}

export interface ResultCache {
  get(key: string): ReviewFinding[] | undefined;
  set(key: string, value: ReviewFinding[]): void;
}

export interface ReviewServiceDeps {
  modelClient: ModelClient;
  rateLimiter: RateLimiter;
  cache: ResultCache;
  hashDiff: (diff: string) => string;
}

export class ReviewService {
  constructor(private readonly deps: ReviewServiceDeps) {}

  async reviewDiff(rawDiff: string): Promise<string> {
    const hunks = parseDiff(rawDiff);
    const cacheKey = this.deps.hashDiff(rawDiff);

    const cached = this.deps.cache.get(cacheKey);
    if (cached) return generateMarkdownReport(cached);

    await this.deps.rateLimiter.acquire();

    const prompt = this.buildPrompt(hunks.length);
    const response = await this.deps.modelClient.chat(prompt);

    if (!response.ok || !response.value) {
      throw response.error ?? new Error("model call failed");
    }

    const findings = this.parseFindings(response.value);
    this.deps.cache.set(cacheKey, findings);
    return generateMarkdownReport(findings);
  }

  private buildPrompt(hunkCount: number): string {
    return `You are a senior code reviewer. Analyze these ${hunkCount} diff hunks and return JSON findings.`;
  }

  private parseFindings(raw: string): ReviewFinding[] {
    try {
      const parsed = JSON.parse(raw);
      if (!Array.isArray(parsed)) throw new Error("expected array");
      return parsed as ReviewFinding[];
    } catch {
      return [];
    }
  }
}
```

### 12.1 Testing with hand-written fakes (no `vi.mock` needed)

Because every dependency is an interface, we can write tiny fake implementations instead of mocking modules — often clearer than mocks for orchestration-level tests.

```ts
// tests/review-service.test.ts
import { describe, it, expect, vi } from "vitest";
import { ReviewService, type ModelClient, type RateLimiter, type ResultCache } from "../src/review-service";
import type { ReviewFinding } from "../src/types";

const SAMPLE_DIFF = `diff --git a/a.ts b/a.ts\n@@ -1,0 +1,1 @@\n+const x = 1\n`;

function makeDeps(overrides: Partial<{
  chatResult: { ok: boolean; value?: string; error?: Error };
  cached: ReviewFinding[] | undefined;
}> = {}) {
  const modelClient: ModelClient = {
    chat: vi.fn().mockResolvedValue(
      overrides.chatResult ?? { ok: true, value: JSON.stringify([]) },
    ),
  };
  const rateLimiter: RateLimiter = { acquire: vi.fn().mockResolvedValue(undefined) };
  const store = new Map<string, ReviewFinding[]>();
  if (overrides.cached) store.set("hash-abc", overrides.cached);
  const cache: ResultCache = {
    get: vi.fn((key: string) => store.get(key)),
    set: vi.fn((key: string, value: ReviewFinding[]) => void store.set(key, value)),
  };
  const hashDiff = vi.fn().mockReturnValue("hash-abc");

  return { modelClient, rateLimiter, cache, hashDiff };
}

describe("ReviewService.reviewDiff", () => {
  it("acquires a rate-limit permit before calling the model", async () => {
    const deps = makeDeps();
    const service = new ReviewService(deps);

    await service.reviewDiff(SAMPLE_DIFF);

    expect(deps.rateLimiter.acquire).toHaveBeenCalledOnce();
    expect(deps.modelClient.chat).toHaveBeenCalledOnce();
  });

  it("returns a cache hit without touching the rate limiter or the model", async () => {
    const findings: ReviewFinding[] = [{ filePath: "a.ts", line: 1, severity: "info", message: "cached" }];
    const deps = makeDeps({ cached: findings });
    const service = new ReviewService(deps);

    const report = await service.reviewDiff(SAMPLE_DIFF);

    expect(deps.rateLimiter.acquire).not.toHaveBeenCalled();
    expect(deps.modelClient.chat).not.toHaveBeenCalled();
    expect(report).toContain("cached");
  });

  it("stores findings in the cache after a successful model call", async () => {
    const findings: ReviewFinding[] = [{ filePath: "a.ts", line: 1, severity: "critical", message: "SQLi risk" }];
    const deps = makeDeps({ chatResult: { ok: true, value: JSON.stringify(findings) } });
    const service = new ReviewService(deps);

    await service.reviewDiff(SAMPLE_DIFF);

    expect(deps.cache.set).toHaveBeenCalledWith("hash-abc", findings);
  });

  it("propagates the model's error when the call fails", async () => {
    const modelError = new Error("Ollama Cloud unavailable");
    const deps = makeDeps({ chatResult: { ok: false, error: modelError } });
    const service = new ReviewService(deps);

    await expect(service.reviewDiff(SAMPLE_DIFF)).rejects.toThrow("Ollama Cloud unavailable");
  });

  it("degrades gracefully to an empty finding list on malformed model JSON", async () => {
    const deps = makeDeps({ chatResult: { ok: true, value: "not valid json {{{" } });
    const service = new ReviewService(deps);

    const report = await service.reviewDiff(SAMPLE_DIFF);

    expect(report).toContain("No issues found");
  });

  it("propagates InvalidDiffError from the parser without calling the model", async () => {
    const deps = makeDeps();
    const service = new ReviewService(deps);

    await expect(service.reviewDiff("")).rejects.toThrow("Invalid diff");
    expect(deps.modelClient.chat).not.toHaveBeenCalled();
  });
});
```

**Fakes vs. `vi.mock`, when to use which:**

- **Hand-written fakes conforming to an interface** (as above) — best for orchestration/integration tests. They read like real collaborators, catch interface-shape mistakes, and don't require hoisting gymnastics.
- **`vi.mock` on a concrete module** — best when you can't inject a dependency (legacy code, a module with side effects at import time, or a third-party SDK you don't own).

Prefer designing for injectable interfaces first; reach for `vi.mock` as the fallback, not the default.


---

## 13. Part X — Snapshot Testing

Snapshots are ideal for the report generator: a fair amount of formatting logic, whose output you mostly want to eyeball once and then be alerted to any unintended change.

```ts
// tests/report-generator.test.ts
import { describe, it, expect } from "vitest";
import { generateMarkdownReport } from "../src/report-generator";
import type { ReviewFinding } from "../src/types";

describe("generateMarkdownReport", () => {
  it("renders a clean report when there are no findings", () => {
    expect(generateMarkdownReport([])).toMatchInlineSnapshot(`
      "# Review Report

      No issues found. ✅
      "
    `);
  });

  it("sorts findings by severity: critical, then warning, then info", () => {
    const findings: ReviewFinding[] = [
      { filePath: "b.ts", line: 4, severity: "info", message: "consider renaming" },
      { filePath: "a.ts", line: 12, severity: "critical", message: "possible SQL injection" },
      { filePath: "c.ts", line: 7, severity: "warning", message: "unhandled promise rejection" },
    ];

    expect(generateMarkdownReport(findings)).toMatchSnapshot();
  });
});
```

Two flavors, both shown above:

- **`toMatchInlineSnapshot()`** — Vitest writes the snapshot *directly into the test file* the first time you run it (and rewrites it with `--update`/`-u`). Best for small, stable outputs you want reviewers to see in the diff itself.
- **`toMatchSnapshot()`** — stored in a sibling `__snapshots__/report-generator.test.ts.snap` file. Best for larger or many outputs, keeping test files readable.

Run `vitest -u` (or `vitest --update`) to intentionally accept new snapshot output after a deliberate change — **never do this reflexively just to make a red test pass**; read the diff first. A snapshot test's entire value comes from a human reviewing what changed.


---

## 14. Part XI — Type-Level Testing

Vitest can assert on **types themselves**, not just runtime values — crucial for a library-like layer such as `OllamaResult<T>`, where a bug might never throw at runtime but still silently widen a type and defeat the whole point of the discriminated union.

```ts
// tests/types.test-d.ts   <-- note the .test-d.ts suffix: type-only test file
import { describe, it, expectTypeOf } from "vitest";
import type { OllamaResult, ReviewFinding, Severity } from "../src/types";

describe("OllamaResult<T> discriminated union", () => {
  it("narrows to `value: T` when ok is true", () => {
    const result = { ok: true, value: "hello" } as OllamaResult<string>;
    if (result.ok) {
      expectTypeOf(result.value).toBeString();
      // @ts-expect-error - `error` doesn't exist on the ok:true branch
      result.error;
    }
  });

  it("narrows to `error: OllamaError` when ok is false", () => {
    const result = {} as OllamaResult<number>;
    if (!result.ok) {
      expectTypeOf(result.error).toHaveProperty("code");
      expectTypeOf(result.error.code).toEqualTypeOf<
        "TIMEOUT" | "RATE_LIMIT" | "SERVER_ERROR" | "PARSE_ERROR"
      >();
    }
  });

  it("Severity is a closed string union, not a bare string", () => {
    expectTypeOf<Severity>().not.toBeString(); // it's narrower than `string`
    expectTypeOf<Severity>().toEqualTypeOf<"info" | "warning" | "critical">();
  });

  it("ReviewFinding.line is a number, not a string", () => {
    expectTypeOf<ReviewFinding>().toHaveProperty("line").toEqualTypeOf<number>();
  });
});
```

Run type tests with `vitest typecheck` (or add `test.typecheck.enabled: true` in `vitest.config.ts`). These tests **never execute at runtime** — they're compiled and checked by `tsc`, and a failure means a type regression, not a logic bug. This is how you guard against someone loosening `Severity` to `string` six months from now and silently breaking every switch statement that exhaustively handles it.


---

## 15. Part XII — Custom Matchers

When you assert the same shape repeatedly (e.g. "is this a well-formed `ReviewFinding`"), extend `expect` instead of repeating boilerplate.

```ts
// tests/setup/custom-matchers.ts
import { expect } from "vitest";
import type { ReviewFinding } from "../../src/types";

interface CustomMatchers<R = unknown> {
  toBeValidFinding(): R;
  toHaveSeverity(severity: string): R;
}

declare module "vitest" {
  interface Assertion<T = any> extends CustomMatchers<T> {}
  interface AsymmetricMatchersContaining extends CustomMatchers {}
}

expect.extend({
  toBeValidFinding(received: ReviewFinding) {
    const validSeverities = ["info", "warning", "critical"];
    const pass =
      typeof received.filePath === "string" &&
      received.filePath.length > 0 &&
      typeof received.line === "number" &&
      received.line > 0 &&
      validSeverities.includes(received.severity) &&
      typeof received.message === "string";

    return {
      pass,
      message: () =>
        pass
          ? `expected ${JSON.stringify(received)} not to be a valid finding`
          : `expected ${JSON.stringify(received)} to be a valid ReviewFinding (filePath, line>0, valid severity, message)`,
    };
  },

  toHaveSeverity(received: ReviewFinding, expected: string) {
    const pass = received.severity === expected;
    return {
      pass,
      message: () => `expected severity "${received.severity}" to be "${expected}"`,
    };
  },
});
```

Register it in `vitest.config.ts`:

```ts
export default defineConfig({
  test: {
    setupFiles: ["./vitest.setup.ts", "./tests/setup/custom-matchers.ts"],
  },
});
```

Usage — reads like a domain-specific language for your own test suite:

```ts
import { describe, it, expect } from "vitest";
import "../setup/custom-matchers"; // ensures TS picks up the type augmentation in this file too

it("every finding returned by the parser is well-formed", () => {
  const findings = [{ filePath: "a.ts", line: 3, severity: "warning", message: "todo" }];
  for (const f of findings) {
    expect(f).toBeValidFinding();
    expect(f).toHaveSeverity("warning");
  }
});
```


---

## 16. Part XIII — Coverage & CI

### 16.1 Coverage

```bash
npx vitest run --coverage
```

With the thresholds set in `vitest.config.ts` (Section 3), a coverage run that dips below 85% lines/functions or 80% branches **fails the build**, not just prints a warning. Open `coverage/index.html` locally to see exactly which branches in `ollama-client.ts`'s retry loop or `review-service.ts`'s error paths aren't exercised yet — this is usually where the interesting edge-case tests are still missing (e.g., "what if `maxRetries` is 0", "what if the cache returns an empty array vs. `undefined`").

Coverage is a **detector of untested code**, not a target to chase for its own sake — 100% coverage with weak assertions (`expect(result).toBeDefined()` everywhere) is worse than 85% coverage with the sharp, specific assertions used throughout this guide.

### 16.2 GitHub Actions CI

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npx vitest run --coverage
      - run: npx vitest typecheck   # runs the *.test-d.ts type tests from Part XI
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-report
          path: coverage/
```

Never let the CI job be the *first* place your `OLLAMA_API_KEY` might leak: notice that every test in this guide injects a fake `fetchImpl` or a fake `ModelClient` — **no test in this entire suite makes a real network call.** That's not an accident; it's the point of dependency injection. A test suite for an AI app that occasionally calls a real cloud model is not a test suite, it's a flaky, slow, rate-limited liability.


---

## 17. Cheat Sheet & Anti-Patterns

### 17.1 Quick reference

| Need | API |
|---|---|
| Group tests | `describe("name", () => { ... })` |
| A test case | `it("does x", () => {})` / `test("does x", () => {})` |
| Run once before/after all tests in a block | `beforeAll` / `afterAll` |
| Run before/after each test | `beforeEach` / `afterEach` |
| Skip / isolate / stub a test | `it.skip` / `it.only` / `it.todo` |
| Parallel tests within a file | `it.concurrent` |
| Table-driven tests | `it.each([...])(...)` / `describe.each([...])(...)` |
| Fake function | `vi.fn()`, `.mockReturnValue()`, `.mockResolvedValue()`, `.mockImplementation()` |
| Spy on a real method | `vi.spyOn(obj, "method")` |
| Mock a whole module | `vi.mock("./path", () => ({ ... }))` |
| Reference a hoisted var inside `vi.mock` | `vi.hoisted(() => ({ ... }))` |
| Clear/reset/restore mocks | `vi.clearAllMocks()` / `vi.resetAllMocks()` / `vi.restoreAllMocks()` |
| Fake timers | `vi.useFakeTimers()` / `vi.useRealTimers()` |
| Advance time (async-safe) | `vi.advanceTimersByTimeAsync(ms)` / `vi.runAllTimersAsync()` |
| Freeze the system clock | `vi.setSystemTime(date)` |
| Assert a promise | `await expect(p).resolves.toBe(x)` / `await expect(p).rejects.toThrow(Err)` |
| Snapshot | `toMatchSnapshot()` / `toMatchInlineSnapshot()` |
| Type-only assertions | `expectTypeOf(x).toEqualTypeOf<T>()` in a `*.test-d.ts` file |
| Extend `expect` | `expect.extend({ myMatcher(received, expected) { ... } })` |
| Partial object match | `expect(x).toMatchObject({...})` / `expect.objectContaining({...})` |
| Coverage | `vitest run --coverage` |
| Watch mode UI | `vitest --ui` |

### 17.2 Anti-patterns to avoid (all demonstrated being avoided above)

1. **Sleeping with real timers in tests** (`await new Promise(r => setTimeout(r, 300))`) instead of `vi.useFakeTimers()` — makes your suite slow and eventually flaky on loaded CI runners.
2. **Not injecting `fetch`/the clock/collaborators** — if you can't swap a dependency, you're forced into brittle module-level `vi.mock` calls or, worse, real network calls in tests.
3. **Asserting only `toBeDefined()` / `toBeTruthy()`** — weak assertions pass even when the actual shape of the data is wrong. Prefer `toEqual`, `toMatchObject`, or a custom matcher.
4. **Forgetting `vi.restoreAllMocks()` in `afterEach`** — a `vi.spyOn` in test #1 can silently change behavior in test #47 run later in the same file.
5. **Using `vi.advanceTimersByTime` (sync) on code with `await` inside the timer callback** — the promise never gets a chance to settle before your assertion; use the `*Async` variant.
6. **One giant `it("works")` block per module** — you lose the ability to tell *which* behavior broke from the test name alone. Prefer many small, precisely named `it` blocks (see how every test above states exactly one behavior in its title).
7. **Testing implementation details** (e.g., asserting a private field was set) **instead of observable behavior** (the return value, an emitted event, a thrown error). Implementation details change; behavior contracts shouldn't, and testing them keeps your suite from becoming a refactoring tax.
8. **Blindly running `vitest -u` to fix a red snapshot test** without reading the diff — this turns snapshot testing from a safety net into a rubber stamp.

### 17.3 Where to go from here

- Wire the real `OllamaClient` against Ollama Cloud's actual `/api/chat` endpoint in a *separate*, explicitly-opt-in integration test (e.g. gated behind an `RUN_INTEGRATION=1` env var) so it never runs in normal CI — this validates your mocks are honest without slowing down every commit.
- Add `msw` (Mock Service Worker) if you outgrow hand-rolled `fetch` mocks and want to intercept at the network layer across many client methods at once.
- Try mutation testing (e.g. Stryker) against this codebase once coverage is green — it will reveal which of your "passing" tests aren't actually asserting anything meaningful, a great gut-check after working through this whole guide.
