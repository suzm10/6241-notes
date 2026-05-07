# LLVM Inlining Pass Analysis - Question 1

  

## (1) Criteria for In-linable Calls

  

LLVM uses multiple criteria to determine if a function **can** be inlined (i.e., is a valid candidate):

  

### Hard Constraints (Non-inlinable):


1. **Recursive Calls** (`IsRecursiveCall`, `AllowRecursiveCall`)

- Functions that call themselves are normally rejected unless `AllowRecursiveCall` is enabled

- Detected during analysis in `visitCallBase()`

- Location: `InlineCost.cpp:2465-2470`

  

2. **Indirect Branches** (`HasIndirectBr`)

- Functions containing `indirectbr` instructions cannot be inlined

- Reason: Cannot safely redirect indirect jumps across inlining boundaries

- Location: `InlineCost.cpp:2595-2604`

  

3. **Indirect Function Calls** (`IsIndirectCall`)

- Virtual/pointer-based function calls are penalized

- Can be analyzed with special logic (`BoostIndirectCalls`)

- Uses a reduced threshold (`IndirectCallThreshold = 100`)

  

4. **Uninlineable Intrinsics** (`HasUninlineableIntrinsic`)

- Certain intrinsic functions prohibit inlining

- Examples: functions with `returns_twice` semantics

  

5. **Stack Allocation Limits**

- Static allocations checked against `StackSizeThreshold` (default: unlimited)

- Recursive callers checked against `RecurStackSizeThreshold` (default: 1024 bytes)

- Location: `InlineCost.cpp:100-103`

  

6. **Dynamic Allocas**

- Functions with dynamic allocations may not be inlineable

- Special handling for recursive cases

- Location: `InlineCost.cpp:307, 309`

  

### Soft Constraints (Reduced Inlining):

  

- **Function Pointer Indirection**: Requires special analysis (`tryPromoteCall`)

- **Multiple Callers**: May increase overall cost

- **NoDuplicate Constraint**: Functions marked with `noduplicate` attribute

  

---

  

## (2) Cost Parameters and Threshold Determination

  

### Cost Calculation Parameters

  

LLVM assigns costs to different instruction types:

  

| Parameter | Value | Location |

|-----------|-------|----------|

| `InstrCost` | 5 | Line 97 |

| `CallPenalty` | 25 | Line 98 |

| `MemAccessCost` | 0 | Line 99 |

| `InlineAsmInstrCost` | 0 | Line 96 |

| `LoopPenalty` | 25 | InlineConstants::LoopPenalty |

  

### Threshold Determination

  

LLVM determines a **threshold** for each call site, and inlines if `Cost < Threshold`:

  

#### Default Thresholds (Line 81-93):

  

| Threshold | Value | Use Case |

|-----------|-------|----------|

| `DefaultThreshold` | 225 | Normal function calls |

| `InlineThreshold` | 225 | Generic inlining |

| `HintThreshold` | 325 | Functions with inline hints |

| `ColdCallSiteThreshold` | 45 | Cold (rarely executed) call sites |

| `ColdThreshold` | 45 | Cold functions |

| `HotCallSiteThreshold` | 3000 | Hot (frequently executed) call sites |

| `LocallyHotCallSiteThreshold` | 525 | Locally hot calls within a function |

| `OptSizeThreshold` | 50 | With `-Os` flag |

| `OptMinSizeThreshold` | 5 | With `-Oz` flag |

| `OptAggressiveThreshold` | 250 | With `-O3` flag |

  

### Threshold Adjustment Logic

  

The threshold is dynamically adjusted based on:

  

1. **Call Site Frequency** (via `BlockFrequencyInfo`):

- Hot call sites → higher threshold

- Cold call sites → lower threshold

- Location: `InlineCost.cpp:600-601` (`getHotCallSiteThreshold()`)

  

2. **Function Attributes**:

- `call-threshold-bonus` attribute adds to threshold

- Location: `InlineCost.cpp:626-628`

  

3. **Single-Basic-Block Functions**:

- Get bonus threshold increase (`SingleBBBonus`)

- Location: `InlineCost.cpp:794-795`

  

4. **Call Site Context**:

- Different thresholds for different optimization levels

- Different thresholds for import patterns in LTO

  

### Cost Benefit Analysis

  

LLVM also performs cost-benefit analysis:

- **Benefit**: Potential simplifications, eliminated calls

- **Cost**: Code size increase, register pressure

- Flags: `InlineEnableCostBenefitAnalysis`, `InlineSavingsMultiplier` (8), `InlineSavingsProfitableMultiplier` (4)

- Location: `InlineCost.cpp:88-89`

  

---

  

## (3) Total Budget (Capacity) for Inlining

  

### Budget Model

  

LLVM uses a **per-function budget** rather than a global program budget:

  

#### Function-Level Budget

- Each caller function has an implicit budget based on its size

- Inlining is stopped when the function becomes "too large"

- Not a pre-computed global budget, but a dynamic check

  

#### Budget Constraints (Location: `InlineCost.cpp:100-102`):

  

1. **Stack Size Threshold** (`StackSizeThreshold`)

- Default: unlimited (numeric_limits<size_t>::max())

- Limits total stack allocations in the function

  

2. **Recursive Stack Size** (`RecurStackSizeThreshold`)

- Default: 1024 bytes

- Special limit when caller is recursive

- Prevents excessive stack usage in recursive contexts

  

3. **SCC (Strongly Connected Component) Limitation**

- Prevents infinite inlining within SCCs

- Uses `InlineHistory` to track decisions

- Cost multiplier applied: `IntraSCCCostMultiplierValue = 2`

- Location: `Inliner.cpp:358-365`

  

#### How Budget is Enforced (Location: `Inliner.cpp:200-210`)

  

```

For each caller function:

- Collect all call sites

- Process them in priority order

- After each inlining, check if function is still worth inlining more

- Stop when function becomes too large relative to cost savings

```

  

The inliner monitors:

- **Function size**: Tracked via `F.getInstructionCount()`

- **Cost-benefit ratio**: Whether additional inlining provides worthwhile optimization

- **SCC membership**: Prevents repeated inlining through strongly connected components

  

### Budget Determination Strategy

  

Rather than a pre-allocated budget, LLVM uses:

  

1. **Eager Strategy**: Inline aggressively until cost threshold is exceeded

2. **Lazy Evaluation**: Re-evaluate budget after each inlining decision

3. **Heuristic Adaptation**: Adjust thresholds based on:

- Optimization level (-O0, -O2, -O3, etc.)

- Profile information (hot/cold paths)

- Link-time optimization phase (LTO)

- Function attributes and hints

  

### Key Insight

The "budget" is not a fixed number allocated upfront. Instead, LLVM continuously:

- Evaluates cost vs. threshold for each call site

- Stops inlining when the cost of inlining exceeds adjusted thresholds

- Adjusts thresholds dynamically based on call site properties

- Prevents runaway inlining through SCC tracking and inline history

  

---

  

## Summary Answer to Question 1

  

**1) Inlinability Criteria:**

- No recursion (unless enabled)

- No indirect branches

- No problematic intrinsics

- Stack allocation limits

- Can handle indirect calls with special analysis

  

**2) Cost Parameters & Thresholds:**

- Per-instruction cost (InstrCost=5), call penalties (25), etc.

- Base thresholds: 225 (default), 3000 (hot), 45 (cold)

- Dynamic adjustment based on: call site frequency, function size, optimization level, attributes

  

**3) Total Budget/Capacity:**

- No global budget; per-function budgets

- Limits: StackSizeThreshold, RecurStackSizeThreshold

- SCC multiplier prevents runaway inlining

- Budget enforced by continuous cost-benefit evaluation, not pre-allocation