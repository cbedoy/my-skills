# Code Quality — ktlint & Detekt in KMP

## ktlint

Applied via Gradle plugin. All produced code must pass ktlint formatting rules.

### Key rules enforced

| Rule | Example |
|---|---|
| No wildcard imports | `import com.example.feature.model.Item` ✅ vs `import com.example.feature.model.*` ❌ |
| Trailing commas on multiline | Last param/arg in multiline must have trailing comma |
| Max line length (120 chars) | Break long chains, function signatures, lambdas |
| No blank lines at start/end of block | Clean opening/closing braces |
| Consistent spacing around operators | `val x = a + b` not `val x=a+b` |
| Opening braces on same line | `fun foo() {` not `fun foo()\n{` |

### Multiline formatting

```kotlin
// ✅ trailing comma, one param per line
fun createItem(
    id: String,
    name: String,
    description: String,  // trailing comma
): Item

// ✅ multiline lambda with trailing comma
val items = listOf(
    Item("1", "Alpha"),
    Item("2", "Beta"),  // trailing comma
)
```

---

## Detekt

Applied via `DetektConventionPlugin` under `build-logic`. Config in `detekt.yml`.

### Rules always active in KMP projects

**Complexity:**
- `ComplexMethod` — max cyclomatic complexity per function (default 10). Refactor with early returns or extract functions.
- `LongMethod` — functions over ~60 lines should be split.
- `LongParameterList` — max 6 params. Use data class or builder if exceeded.
- `NestedBlockDepth` — max 4 levels. Matches the Compose nesting rule.
- `TooManyFunctions` — class/file with too many functions is a god class. Split it.

**Style:**
- `MagicNumber` — no inline numeric literals. Use named constants or `companion object` vals.
- `UnusedPrivateMember` — no dead code.
- `ForbiddenComment` — `FIXME` and `STOPSHIP` fail CI. `TODO` is a warning.
- `MaxLineLength` — 120 chars (aligned with ktlint).

**Potential bugs:**
- `UnnecessaryAbstractClass` — abstract class with no abstract members → use interface.
- `EqualsWithHashCodeExist` — if you override `equals`, override `hashCode`.
- `UnreachableCode` — dead branches fail the build.

**KMP-specific (coroutines):**
- `GlobalCoroutineUsage` — `GlobalScope` is forbidden. Use `viewModelScope`, `lifecycleScope`, or injected scope.
- `RedundantSuspendModifier` — suspend functions that don't call any suspending operation.
- `SuspendFunWithFlowReturnType` — `suspend fun` returning `Flow` is almost always wrong. Return `Flow` directly (cold stream).

```kotlin
// ❌ Detekt violation
suspend fun getItems(): Flow<List<Item>>

// ✅ correct
fun getItems(): Flow<List<Item>>
```

---

## Naming (Detekt + conventions)

```kotlin
// ✅ constants in SCREAMING_SNAKE_CASE inside companion object or top-level
companion object {
    private const val DEFAULT_TIMEOUT_MS = 5_000L
}

// ✅ sealed interface entries: PascalCase
sealed interface ItemUiState {
    data object Loading : ItemUiState
    data class Content(val item: Item) : ItemUiState
}

// ❌ Detekt flags this
val MY_VARIABLE = "hello"   // not a constant, use camelCase
```

---

## Baseline & suppression

- Use `detekt:baseline.xml` only for legacy violations, not new code.
- Inline suppression (`@Suppress("...")`) allowed only with a comment explaining why.
- Never suppress `GlobalCoroutineUsage` or `MagicNumber` without justification.

```kotlin
@Suppress("MagicNumber") // 1000 = millis per second, extracted for readability
private const val MILLIS_PER_SECOND = 1_000
```

---

## CI enforcement

Both tools run in CI. A build with violations must not merge.

Gradle tasks:
```bash
./gradlew ktlintCheck        # check formatting
./gradlew ktlintFormat       # auto-fix formatting
./gradlew detekt             # run static analysis
```

For KMP multimodule, run per-module or from root with `--continue` to see all violations at once:
```bash
./gradlew detekt --continue
```
