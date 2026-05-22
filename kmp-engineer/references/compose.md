# Compose Multiplatform — UI Guidelines

## State hoisting

State must live **outside** the Composable that displays it.

**Wrong — internal state, not hoistable:**
```kotlin
@Composable
fun CounterCard() {
    var count by remember { mutableStateOf(0) } // ❌ trapped inside
    Text("$count")
    Button(onClick = { count++ }) { Text("Add") }
}
```

**Correct — true state hoisting:**
```kotlin
@Composable
fun CounterCard(
    count: Int,
    onIncrement: () -> Unit,
    modifier: Modifier = Modifier
) {
    // ✅ state comes in, events go out
    Text("$count")
    Button(onClick = onIncrement) { Text("Add") }
}
```

Rules:
- Leaf Composables receive **typed state values**, not `StateFlow` or `ViewModel`.
- State flows down, events flow up — always.
- `remember` is allowed for **UI-only** ephemeral state (animation targets, focus, scroll position).
- Never `collectAsState()` deep inside a leaf Composable — collect at screen level and pass down.

---

## No callback hell — UiAction pattern

If a Composable needs more than **3 lambdas**, replace them with a sealed `UiAction`:

**Wrong — callback hell:**
```kotlin
@Composable
fun ItemCard(
    item: Item,
    onFavorite: () -> Unit,        // ❌ growing list of lambdas
    onShare: () -> Unit,
    onDelete: () -> Unit,
    onEdit: () -> Unit,
    onNavigate: (String) -> Unit
)
```

**Correct — sealed UiAction:**
```kotlin
sealed interface ItemUiAction {
    data object Favorite : ItemUiAction
    data object Share : ItemUiAction
    data object Delete : ItemUiAction
    data class Edit(val itemId: String) : ItemUiAction
    data class Navigate(val destination: String) : ItemUiAction
}

@Composable
fun ItemCard(
    item: Item,
    onAction: (ItemUiAction) -> Unit,  // ✅ single stable lambda
    modifier: Modifier = Modifier
)
```

The ViewModel or screen-level Composable handles the `when(action)` dispatch.

---

## No deep nesting

Max ~2 levels of meaningful nesting before extracting a sub-Composable.

**Wrong — pyramid of doom:**
```kotlin
@Composable
fun ItemDetailScreen(...) {
    Column {
        Row {
            Box {
                Column {
                    Row { // ❌ nobody knows what this renders
                        Text(item.title)
                        Icon(...)
                    }
                }
            }
        }
    }
}
```

**Correct — extract aggressively:**
```kotlin
@Composable
fun ItemDetailScreen(
    state: ItemDetailUiState,
    onAction: (ItemDetailUiAction) -> Unit
) {
    when (state) {
        is ItemDetailUiState.Loading -> LoadingContent()
        is ItemDetailUiState.Content -> ItemDetailContent(state.item, onAction)
        is ItemDetailUiState.Error -> ErrorContent(state.message, onAction)
    }
}

@Composable
private fun ItemDetailContent(
    item: Item,
    onAction: (ItemDetailUiAction) -> Unit,
    modifier: Modifier = Modifier
) {
    Column(modifier) {
        ItemHeader(item, onAction)
        ItemBody(item)
    }
}
```

---

## Screen structure

Screen-level Composables are **thin coordinators**:

```kotlin
@Composable
fun FeatureScreen(
    viewModel: FeatureViewModel = koinInject() // or kotlin-inject equivalent
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    FeatureContent(
        state = uiState,
        onAction = viewModel::onAction
    )
}

// Separately testable, no ViewModel dependency:
@Composable
fun FeatureContent(
    state: FeatureUiState,
    onAction: (FeatureUiAction) -> Unit,
    modifier: Modifier = Modifier
) {
    ...
}
```

Rules:
- `FeatureScreen` = DI + state collection. Nothing else.
- `FeatureContent` = pure Composable, previeable, testable in isolation.
- Never pass `ViewModel` reference to child Composables.

---

## Minimizing recompositions

- Use `remember { }` for expensive calculations.
- Use `derivedStateOf { }` when a value depends on another observable state.
- Prefer `@Stable` and `@Immutable` on state classes passed to Composables.
- Avoid creating lambdas inline in frequently recomposing Composables — hoist them or use `remember`.
- Avoid reading `StateFlow` inside deeply nested Composables — read at screen level, pass values down.

```kotlin
// ❌ new lambda every recomposition
ItemList(items = items, onClick = { id -> navigate(id) })

// ✅ stable reference
val onItemClick = remember<(String) -> Unit> { { id -> navigate(id) } }
ItemList(items = items, onClick = onItemClick)
```

---

## Composable size and responsibility

| Type | Responsibility | Should have preview? |
|---|---|---|
| Screen | DI + state collection | No (has ViewModel) |
| Content | Full-page layout, delegates to components | Yes |
| Component | Single reusable UI piece | Yes |
| Slot | Layout placeholder (header, footer) | Yes |

Every non-Screen Composable must have a `@Preview`.

```kotlin
@Preview
@Composable
private fun ItemCardPreview() {
    AppTheme {
        ItemCard(
            item = previewItem,
            onAction = {}
        )
    }
}
```

---

## Material 3

- Use `MaterialTheme.colorScheme.*` — never hardcoded colors.
- Use `MaterialTheme.typography.*` — never hardcoded text sizes.
- Use `MaterialTheme.shapes.*` for corners.
- Extend via `CompositionLocalProvider` for custom design tokens when needed.
