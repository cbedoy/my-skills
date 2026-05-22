# KMP Architecture Reference

## Module structure

```
root/
├── build-logic/                    # Convention plugins (KmpLibraryPlugin, DetektPlugin, etc.)
├── feature-a-sdk/
│   ├── domain/
│   ├── data/
│   ├── datasource/
│   └── presentation/
├── feature-b-sdk/
│   └── ...
└── shared/                         # Cross-module utilities, CoroutineDispatchers, base errors
```

---

## Layer definitions

### Domain
```
domain/
├── model/          # Pure immutable data classes — zero framework/platform deps
├── usecase/        # Single-responsibility, one public operator fun invoke()
├── repository/     # Interfaces only
└── error/          # Typed domain error hierarchy (sealed interface)
```

**Rules:**
- Zero imports from `android.*`, `platform.*`, Ktor, Room, or any framework.
- All models are immutable (`val` only).
- Repository interfaces return `Flow<Result<T>>` for streams, `Result<T>` for one-shots.
- Use cases return `Flow<Result<T>>` or `Result<T>` — never raw types.

**Typed domain errors:**
```kotlin
sealed interface FeatureDomainError {
    data object NotFound : FeatureDomainError
    data object Unauthorized : FeatureDomainError
    data class NetworkError(val cause: Throwable) : FeatureDomainError
    data class UnexpectedError(val cause: Throwable) : FeatureDomainError
}
```

Never use `Exception` directly in domain. Map all throwables to typed errors at the data boundary.

---

### Data
```
data/
├── repository/     # Repository implementations
└── mapper/         # Extension functions: ResponseModel → DomainModel
```

**Rules:**
- Mappers are pure extension functions — no side effects, no DI.
- Never return response models upward — always map before crossing the layer boundary.
- Use `runCatching` to wrap data source calls. Map `Throwable` to `FeatureDomainError`.

**Repository pattern:**
```kotlin
class FeatureRepositoryImpl(
    private val remoteDataSource: FeatureRemoteDataSource,
    private val dispatchers: CoroutineDispatchers
) : FeatureRepository {

    override fun getItem(id: String): Flow<Result<Item>> = flow {
        val result = runCatching { remoteDataSource.fetchItem(id) }
            .map { it.toDomain() }
            .mapFailure { it.toFeatureDomainError() }
        emit(result)
    }.flowOn(dispatchers.io)
}

private fun Throwable.toFeatureDomainError(): FeatureDomainError = when (this) {
    is HttpException -> if (code() == 404) FeatureDomainError.NotFound
                        else FeatureDomainError.NetworkError(this)
    is UnauthorizedException -> FeatureDomainError.Unauthorized
    else -> FeatureDomainError.UnexpectedError(this)
}
```

---

### DataSource
```
datasource/
├── remote/         # Ktor-based — returns response models
└── local/          # SQLDelight or in-memory — returns local models
```

**Rules:**
- Methods map 1:1 to API endpoints or DB queries.
- No business logic — only data fetching/storage.
- Throws raw exceptions — the repository is responsible for mapping them.
- Use `runCatching` only if the DataSource itself needs to handle retries or fallbacks.

**Example:**
```kotlin
class FeatureRemoteDataSourceImpl(
    private val httpClient: HttpClient
) : FeatureRemoteDataSource {

    override suspend fun fetchItem(id: String): ItemResponse =
        httpClient.get("$BASE_PATH/$id").body()
}
```

---

### Presentation

```
presentation/
├── ui/
│   ├── screen/         # Screen-level Composables — thin, delegate to components
│   ├── component/      # Reusable leaf Composables
│   └── theme/          # Colors, Typography, Theme — Material 3 tokens
├── viewmodel/          # One ViewModel per screen/feature
└── state/              # UiState (sealed), UiAction (sealed), UiEvent (SharedFlow)
```

**ViewModel — injected dispatchers, typed state:**
```kotlin
class FeatureViewModel(
    private val getItemUseCase: GetItemUseCase,
    private val dispatchers: CoroutineDispatchers
) : ViewModel() {

    private val _uiState = MutableStateFlow<FeatureUiState>(FeatureUiState.Loading)
    val uiState: StateFlow<FeatureUiState> = _uiState.asStateFlow()

    fun loadItem(id: String) {
        viewModelScope.launch(dispatchers.main) {
            getItemUseCase(id)
                .flowOn(dispatchers.io)
                .collect { result ->
                    _uiState.value = result.fold(
                        onSuccess = { FeatureUiState.Content(it) },
                        onFailure = { FeatureUiState.Error(it.toUiMessage()) }
                    )
                }
        }
    }
}
```

**UiState:**
```kotlin
sealed interface FeatureUiState {
    data object Loading : FeatureUiState
    data class Content(val item: Item) : FeatureUiState
    data class Error(val message: String) : FeatureUiState
}
```

---

## CoroutineDispatchers

Defined in `shared` or per-module `commonMain`. Always injected.

```kotlin
// commonMain
interface CoroutineDispatchers {
    val io: CoroutineDispatcher
    val main: CoroutineDispatcher
    val default: CoroutineDispatcher
}

// Production implementation (commonMain via kotlin-inject)
class DefaultCoroutineDispatchers @Inject constructor() : CoroutineDispatchers {
    override val io: CoroutineDispatcher = Dispatchers.IO
    override val main: CoroutineDispatcher = Dispatchers.Main
    override val default: CoroutineDispatcher = Dispatchers.Default
}

// Test implementation
class TestCoroutineDispatchers(
    private val testDispatcher: TestCoroutineDispatcher = StandardTestDispatcher()
) : CoroutineDispatchers {
    override val io = testDispatcher
    override val main = testDispatcher
    override val default = testDispatcher
}
```

---

## Naming conventions

| Type | Convention | Example |
|---|---|---|
| Use case | Verb + noun + `UseCase` | `GetItemDetailUseCase` |
| Repository interface | Noun + `Repository` | `ItemRepository` |
| Repository impl | Noun + `RepositoryImpl` | `ItemRepositoryImpl` |
| DataSource interface | Noun + `DataSource` | `ItemRemoteDataSource` |
| DataSource impl | Noun + `DataSourceImpl` | `ItemRemoteDataSourceImpl` |
| UiState | Feature + `UiState` | `ItemDetailUiState` |
| UiAction | Feature + `UiAction` | `ItemDetailUiAction` |
| API model | Noun + `Response` / `Dto` | `ItemResponse` |
| Domain model | Plain noun | `Item` |
| Mapper | Extension on source type | `fun ItemResponse.toDomain(): Item` |
| Domain error | Feature + `DomainError` | `ItemDomainError` |

---

## expect/actual — when to use

**Use for:**
- Platform date/time formatting not covered by `kotlinx-datetime`
- Native secure storage
- File system access
- Platform lifecycle hooks

**Do NOT use for:**
- Anything covered by `kotlinx.*`
- Hiding architectural complexity
- Convenience wrappers writable in `commonMain`

---

## iOS distribution

- XCFramework via `kmmbridge` Gradle plugin
- Published to GitHub Packages or Maven
- `Package.swift` managed via `patchPackage.sh` for multi-binary support
- `composeResources` cross-copy via Xcode Build Phase to prevent `MissingResourceException`
