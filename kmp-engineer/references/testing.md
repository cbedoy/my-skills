# KMP Unit Testing Conventions

## Stack

| Tool | Purpose |
|---|---|
| Mokkery | Mocking (`mock`, `MockMode.autoUnit`, `everySuspend`, `every`, `verify`, `verifySuspend`) |
| Turbine | Flow testing (`.test {}`, `awaitItem`, `awaitComplete`, `cancelAndIgnoreRemainingEvents`) |
| coroutines-test | `runTest`, `StandardTestDispatcher`, `TestCoroutineDispatchers` |

---

## Universal rules

- Test names in English backtick format: `` `should X when Y` ``
- No conditional logic in tests
- One concept per assertion block
- Helpers and fakes at the bottom in `// region helpers`
- `MockMode.autoUnit` for all mocks
- Dispatchers are always injected via `TestCoroutineDispatchers` — never `Dispatchers.setMain` directly
- Test both success and typed error paths — never only the happy path

---

## ViewModel tests

```kotlin
class FeatureViewModelTest {

    private val getItemUseCase = mock<GetItemUseCase>(MockMode.autoUnit)
    private val dispatchers = TestCoroutineDispatchers()
    private lateinit var viewModel: FeatureViewModel

    @BeforeTest
    fun setUp() {
        Dispatchers.setMain(dispatchers.main)
    }

    @AfterTest
    fun tearDown() {
        Dispatchers.resetMain()
    }

    // region initial state
    @Test
    fun `should emit Loading as initial state`() = runTest {
        viewModel = FeatureViewModel(getItemUseCase, dispatchers)
        assertEquals(FeatureUiState.Loading, viewModel.uiState.value)
    }
    // endregion

    // region success
    @Test
    fun `should emit Content state when use case returns item`() = runTest {
        setupDefaultSuccessResponses()
        viewModel = FeatureViewModel(getItemUseCase, dispatchers)

        viewModel.uiState.test {
            viewModel.loadItem(FAKE_ITEM_ID)
            awaitItem() // Loading
            val state = awaitItem()
            assertIs<FeatureUiState.Content>(state)
            assertEquals(fakeItem, state.item)
            cancelAndIgnoreRemainingEvents()
        }
    }
    // endregion

    // region error
    @Test
    fun `should emit Error state when use case returns domain error`() = runTest {
        everySuspend { getItemUseCase(any()) } returns flowOf(
            Result.failure(FeatureDomainError.NotFound)
        )
        viewModel = FeatureViewModel(getItemUseCase, dispatchers)

        viewModel.uiState.test {
            viewModel.loadItem(FAKE_ITEM_ID)
            awaitItem() // Loading
            val state = awaitItem()
            assertIs<FeatureUiState.Error>(state)
            cancelAndIgnoreRemainingEvents()
        }
    }
    // endregion

    // region helpers
    private fun setupDefaultSuccessResponses() {
        everySuspend { getItemUseCase(FAKE_ITEM_ID) } returns flowOf(Result.success(fakeItem))
    }
    // endregion
}
```

**Rules:**
- Always consume `Loading` before asserting final state.
- `cancelAndIgnoreRemainingEvents()` at end of every Turbine block.
- Use `TestCoroutineDispatchers` — enables deterministic test execution.
- Assert on typed `DomainError` subtypes, never on message strings.

---

## UseCase tests

```kotlin
class GetItemUseCaseTest {

    private val repository = mock<ItemRepository>(MockMode.autoUnit)

    @Test
    fun `should return item flow from repository`() = runTest {
        everySuspend { repository.getItem(FAKE_ITEM_ID) } returns flowOf(Result.success(fakeItem))

        val useCase = GetItemUseCase(repository)
        val result = useCase(FAKE_ITEM_ID).first()

        assertEquals(Result.success(fakeItem), result)
    }

    @Test
    fun `should propagate domain error from repository`() = runTest {
        val error = FeatureDomainError.NotFound
        everySuspend { repository.getItem(any()) } returns flowOf(Result.failure(error))

        val useCase = GetItemUseCase(repository)
        val result = useCase(FAKE_ITEM_ID).first()

        assertEquals(error, result.exceptionOrNull())
    }
}
```

**Rules:**
- No Dispatchers setup — UseCase tests don't need it.
- One test per use case method, plus one per relevant error variant.
- `assertEquals` directly on the flow result.

---

## Repository tests

```kotlin
class ItemRepositoryImplTest {

    private val remoteDataSource = mock<ItemRemoteDataSource>(MockMode.autoUnit)
    private val dispatchers = TestCoroutineDispatchers()
    private lateinit var repository: ItemRepository

    @BeforeTest
    fun setUp() {
        repository = ItemRepositoryImpl(remoteDataSource, dispatchers)
    }

    @Test
    fun `should map response to domain model on success`() = runTest {
        everySuspend { remoteDataSource.fetchItem(FAKE_ITEM_ID) } returns fakeItemResponse

        repository.getItem(FAKE_ITEM_ID).test {
            val result = awaitItem()
            assertTrue(result.isSuccess)
            with(result.getOrThrow()) {
                assertEquals(fakeItemResponse.id, id)
                assertEquals(fakeItemResponse.name, name)
            }
            awaitComplete()
        }
    }

    @Test
    fun `should emit NotFound domain error when source throws 404`() = runTest {
        everySuspend { remoteDataSource.fetchItem(any()) } throws HttpException(404)

        repository.getItem(FAKE_ITEM_ID).test {
            val result = awaitItem()
            assertIs<FeatureDomainError.NotFound>(result.exceptionOrNull())
            awaitComplete()
        }
    }

    @Test
    fun `should emit UnexpectedError for unknown exceptions`() = runTest {
        everySuspend { remoteDataSource.fetchItem(any()) } throws RuntimeException("Boom")

        repository.getItem(FAKE_ITEM_ID).test {
            val result = awaitItem()
            assertIs<FeatureDomainError.UnexpectedError>(result.exceptionOrNull())
            awaitComplete()
        }
    }

    @Test
    fun `should forward item id unmodified to data source`() = runTest {
        everySuspend { remoteDataSource.fetchItem(FAKE_ITEM_ID) } returns fakeItemResponse

        repository.getItem(FAKE_ITEM_ID).test {
            awaitItem()
            awaitComplete()
        }

        verifySuspend { remoteDataSource.fetchItem(FAKE_ITEM_ID) }
    }
}
```

**Rules:**
- `awaitComplete()` — repository flows emit and complete.
- Verify mapping field by field in the success case.
- Test each typed `DomainError` subtype, not just generic failure.
- `verifySuspend` to confirm args arrive unmodified.

---

## DataSource tests

```kotlin
class ItemRemoteDataSourceImplTest {

    private val httpClient = mock<HttpClient>(MockMode.autoUnit)

    @Test
    fun `should return item response for valid id`() = runTest {
        everySuspend { httpClient.get<ItemResponse>(any()) } returns fakeItemResponse

        val dataSource = ItemRemoteDataSourceImpl(httpClient)
        val result = dataSource.fetchItem(FAKE_ITEM_ID)

        assertEquals(fakeItemResponse, result)
    }

    @Test
    fun `should forward item id in request path`() = runTest {
        everySuspend { httpClient.get<ItemResponse>("$BASE_PATH/$FAKE_ITEM_ID") } returns fakeItemResponse

        val dataSource = ItemRemoteDataSourceImpl(httpClient)
        dataSource.fetchItem(FAKE_ITEM_ID)

        verifySuspend { httpClient.get<ItemResponse>("$BASE_PATH/$FAKE_ITEM_ID") }
    }
}
```

**Rules:**
- No Dispatchers setup — DataSource tests are direct suspend calls.
- No Turbine — not flows.
- One test per public method.
- Test `null` / optional response variants where the API allows them.
- DataSource throws raw — do not assert on `DomainError` here.

---

## Fake data conventions

```kotlin
// region helpers
private const val FAKE_ITEM_ID = "item-abc-123"
private const val BASE_PATH = "/api/v1/items"

private val fakeItem = Item(
    id = FAKE_ITEM_ID,
    name = "Fake Item",
    description = "A fake item for testing"
)

private val fakeItemResponse = ItemResponse(
    id = FAKE_ITEM_ID,
    name = "Fake Item",
    description = "A fake item for testing"
)
// endregion
```
