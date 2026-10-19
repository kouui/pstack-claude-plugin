# TypeScript 模式

每条规则在 `SKILL.md` 中都有对应的代码示例。其背后的原则与语言无关。参见 **type-system-discipline** 和 **boundary-discipline** 原则技能。

## 品牌类型

为原始类型打上品牌标记，使它们不会被混用。在边界处校验一次。下游代码信任该类型。

```ts
type AgentId = string & { readonly __brand: "AgentId" };

function isAgentId(input: string): input is AgentId {
  return isUUID(input);
}

function parseAgentId(input: string): AgentId {
  if (!isAgentId(input)) throw new Error(`Invalid agent id: ${input}`);
  return input;
}

function focusAgent(id: AgentId): void {
  /* input is trusted */
}
```

与 `readonly __brand: 'X'` 的结构保持一致。不要自创新的约定。

## 判别联合

用字面量判别式建模各种变体。每个变体共享同一个字段名，且每个变体的值都唯一，因此无法表示不可能的组合。

```ts
// Don't. Boolean + optionals lets contradictory states exist.
type DiffState = { loading: boolean; diff?: GitDiff; error?: string };

// Do. Only valid states exist.
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: GitDiff }
  | { kind: "error"; error: string };
```

选定一个判别字段名（`kind`、`type` 或 `tag`），并始终保持一致。

## 构造式建模

用全部合法的部件来构造类型，而不是用运行时检查去收窄一个宽松的类型。

非空，通过可变长元组实现：

```ts
type NonEmpty<T> = [T, ...T[]];

// Don't: T[] plus a length check every caller must repeat
function pickWinner(entries: string[]): string {
  if (entries.length === 0) throw new Error("no entries");
  return entries[Math.floor(Math.random() * entries.length)];
}

// Do: an empty value of the type can't exist
function pickWinner(entries: NonEmpty<string>): string {
  return entries[Math.floor(Math.random() * entries.length)];
}
```

如果传入的是普通的 `T[]`，用一个守卫函数收窄一次。该事实随后会保存在类型中：

```ts
const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
```

偶数长度，以成对的形式表示：

```ts
type Pairs<T> = [T, T][];
```

时间范围，以起点加时长表示：

```ts
// Don't: a comment holds the invariant
type TimeRange = { start: Date; end: Date }; // start <= end

// Do: a negative range can't be written; derive end when needed
type TimeRange = { start: Date; durationMs: number };
```

保持 `durationMs` 为普通数字。只有当原始数字可能被传入需要时长的位置时，才按“品牌类型”一节对其打标记，不要习惯性地这样做。选择能让错误状态无法被构造出来的表示，然后在其上提供你需要的读取方式（`pairs.flat()`、`rangeEnd()` 辅助函数）。

## 最简全函数类型

不要处处加强类型。当对 `T[]` 的每个操作都是全函数时，保留 `T[]`：

```ts
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0); // [] is 0, fine
```

当宽松的类型迫使使用处说谎时，才加强类型。信号包括 `!`、`arr[0] as T`，以及“不应该发生”的 throw：

```ts
// Don't: partiality smuggled past the compiler
function newestSession(sessions: Session[]): Session {
  return sessions.at(0)!;
}

// Do: strengthen the input; the assertion disappears
function newestSession(sessions: NonEmpty<Session>): Session {
  return sessions[0];
}
```

将结果弱化为 `Session | undefined` 是另一种全函数签名。

## 用 `unknown` 代替 `any`

外部数据始终是 `unknown`。使用前先收窄。

```ts
// Don't
function handle(input: any) {
  return input.foo.bar;
}

// Do
function handle(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    // narrowed; compiler verifies access
  }
}
```

外部来源包括 RPC 载荷、`JSON.parse`、`postMessage`、IPC、文件内容、环境变量、数据库结果。

## 先用 schema，再手写守卫

在为外部数据编写逐属性的类型守卫之前，先查找仓库中的运行时 schema 库和已有的 schema。让一个 schema 负责校验，并从它推导出 TypeScript 类型。不要同时维护一个 schema、一个重复的接口和一个守卫，让它们可能互相偏离。

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});

type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input);
}
```

当失败是预期的分支时，使用 `safeParse`。当仓库使用其他 schema 库时，使用其等效的推导辅助工具。不要为一个守卫新增 schema 依赖。本规则优先使用代码库已经信任的 schema 体系。

## 不使用 `as` 类型断言

每一个 `as` 都是潜在的运行时崩溃。只有在类型系统已经验证了该断言之后才进行转换。

```ts
import { z } from "zod";

// Don't
const user = data as User;

// Don't
function isUser(data: unknown): data is User {
  return typeof data === "object" && data !== null && "id" in data;
}

// Do
const userSchema = z.object({ id: z.string(), name: z.string() });
type User = z.infer<typeof userSchema>;

function parseUser(data: unknown): User {
  return userSchema.parse(data);
}
```

当类型先于校验器存在时，用它所证明的类型来标注校验器。这样，证明的内容少于类型要求时，编译器就会拒绝该校验器。从下面的对象中删除 `name`，赋值就会编译失败。

```ts
type User = { id: string; name: string };

const userSchema: z.ZodType<User> = z.object({ id: z.string(), name: z.string() });
```

在重构现有代码以去掉 `as` 时，先弄清楚 TypeScript 为什么无法推断：

- 缺少判别字段：添加一个，改用判别联合。
- 源类型过宽（例如 `Record<string, unknown>`）：将其收窄。
- 无类型的边界：使用拥有该形状的 schema 进行解析。只在不存在 schema 的地方添加 schema。
- 确实无法用类型表达：使用品牌类型或 `satisfies`。

## 收窄层级

从最优到最后手段：

1. **判别联合的 switch / if。** 编译器自动收窄。
2. **`in` 运算符。** `"key" in obj` 会收窄到包含该键的变体。
3. **`typeof` / `instanceof`。** 用于原始类型和类实例。
4. **用户自定义类型守卫。** 当以上方式都不够用时使用。
5. **`as` 类型断言。** 只有在校验之后才能使用。

```ts
function area(s: Shape): number {
  if ("radius" in s) return Math.PI * s.radius ** 2; // narrowed to circle
  return s.width * s.height; // narrowed to rect
}
```

## 类型守卫

守卫必须真正验证其断言。一个说谎的守卫比 `as` 更糟糕。

```ts
function isCircle(s: Shape): s is Shape & { kind: "circle" } {
  return s.kind === "circle";
}
```

能用判别式收窄时，优先使用判别式收窄。

## 穷尽性检查

在 default 分支中，将判别字段赋值给一个 `never` 类型的局部变量。

```ts
// Value-returning switch
function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}

// Void switch
function handle(s: Shape): void {
  switch (s.kind) {
    case "circle":
      drawCircle(s);
      break;
    case "rect":
      drawRect(s);
      break;
    default: {
      const _exhaustive: never = s;
      void _exhaustive;
    }
  }
}
```

有返回值的 switch 使用返回式写法，语句形式的 switch 使用 void 式写法。

## 用 `satisfies` 代替 `as`

`satisfies` 在校验的同时不会拓宽字面量类型。

```ts
// Don't. Widens, loses literal types.
const config = { theme: "dark", cols: 3 } as Config;

// Do. Validates AND preserves literal types.
const config = { theme: "dark", cols: 3 } satisfies Config;
// config.theme is "dark" (literal), not string
```

## 边界校验

在数据进入的地方校验一次。内部信任类型。参见 **boundary-discipline** 原则技能。

- **线格式**（proto、JSON-RPC）：使用 `ignoreUnknownFields` 解析，使前向兼容的变更不会破坏旧客户端。
- **持久化的 JSON：** 带版本号的数据块，解析时用 try/catch 包裹。
- **不要在调用链深处重复校验。**

## 从 schema 推导的类型

当 `.proto`、OpenAPI 规范、GraphQL schema 或数据库迁移已经定义了某个形状时，从生成的类型推导，而不是重复定义。

```ts
// Don't. Duplicate shape, drifts when the schema changes.
type CheckSummary = {
  totalCount: number;
  checks: { name: string; status: string }[];
};
function renderChecks(s: CheckSummary) {
  /* ... */
}

// Do. Derive from the generated schema type.
import type { ChecksMessage } from "<generated module>";
function renderChecks(s: Pick<ChecksMessage, "totalCount" | "checks">) {
  /* ... */
}
```

在编写新接口之前，先使用 `Pick`、`Omit`、`Parameters`、`ReturnType`、`Awaited`、`typeof`。

## 对象参数

```ts
// Don't. Swap two args, still compiles.
openFile(uri, {
  startLineNumber: 10,
  startColumn: 1,
  endLineNumber: 10,
  endColumn: 1,
});

// Do. Order-independent, self-documenting.
openFile({
  uri,
  selection: {
    startLineNumber: 10,
    startColumn: 1,
    endLineNumber: 10,
    endColumn: 1,
  },
});
```

在热路径上可以跳过：每帧渲染、分词器、解析器，以及任何分配成本重要的紧凑循环中的代码。
