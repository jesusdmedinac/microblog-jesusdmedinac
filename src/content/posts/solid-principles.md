---
pubDate: 2026-01-19
author: JesusDMedinaC
title: SOLID Principles
description: "The SOLID principles are a set of five design principles that are used to create maintainable and scalable software."
image:
  url: "/src/images/blog/hadouken.png"
  alt: "#"
tags: ["SOLID", "Principles", "Design", "Patterns"]
---

# SOLID Principles

Yes, this is another post about SOLID principles, but this time I want to show you how to apply them in your daily work and how to identify when you need to apply them.

## Context and Intent

Every day, every new project, I find myself in the same situation: I need to develop a new feature, or maintain an existing one on a legacy codebase. So far so good until I create a pull request. Then I find myself trying to defend my code changes justifying that I'm following best practices, SOLID principles, etc.

Why I need to explain code best practices to engineers who are supposed to know them? They know them, in theory, right? Well, let's see.

## The Problem

Before explaining the SOLID principles, let's see a real example of a codebase that doesn't follow them.

I will use a real example from a legacy codebase that I worked on with a few adaptations due to NDA restrictions.

Let's start by defining the app architecture: A simple mobile app with multiple activities and fragments, each with its own logic and state management. Each activity could relies on its own view model to manage the state and business logic. Also, each fragment could relies on its own view model. We start seeing some problems here, right? well, we can discuss that later.

If I ask you, what is the Activity/Fragment responsible for? you may say that it responsibility is to manage the UI and business logic. Wrong. It is not responsible for managing the business logic. It is responsible for managing the UI. So why are you using the Activity/Fragment to manage the business logic? 

```kotlin
// com.jesusdmedinac.app.activities.MyActivity
class MyActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_my)
        
        viewModel.someState.observe(this, Observer { state ->
            // Update UI based on state
        })
        
        // Business logic
        viewModel.doSomething()

        // More business logic
        viewModel.someManager.doSomething()

        // More business logic
        privateBusinessLogic()
    }

    private fun privateBusinessLogic() {
        // Business logic
    }
}

// com.jesusdmedinac.app.viewmodels.MyViewModel
class MyViewModel : ViewModel() {
    @Inject
    lateinit var someManager: SomeManager

    private val _someState = MutableLiveData<State>()
    val someState: LiveData<State> = _someState
    
    fun doSomething() {
        // Business logic
    }
}
```

First of all, why are you using `someManager` on the Activity/Fragment? It is not responsible for managing the business logic. It is responsible for managing the UI. But you may say that since `someManager` is public, it can be used by other components, right? Wrong. It is public due to dependency injection, but not to be used by other components. And even the ViewModel should not be using it. Depending on the manager responsibility, we could determine in which component it should be used.

Now, why are you using `privateBusinessLogic` on the Activity/Fragment? We could perfectly move it to the ViewModel, right? But what if that `privateBusinessLogic` depends on Activity/Fragment properties? We could make the `privateBusinessLogic` able to receive those properties as parameters defining a data class or interface to abstract the data types.

I found a lot of examples as this one, where the developers always try to add the code solution in the existing codebase, instead of refactoring in order to follow SOLID principles.

Let's see a refactored version of this code:

```kotlin
// com.jesusdmedinac.app.activities.MyActivity
class MyActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_my)
        
        viewModel.someState.observe(this, Observer { state ->
            // Update UI based on state
        })
        
        // Business logic
        viewModel.doSomething()

        // More business logic
        viewModel.doSomethingElse()

        // More business logic
        viewModel.doNonPrivateBusinessLogic()
    }
}

// com.jesusdmedinac.app.viewmodels.MyViewModel
interface SomeManager {
    fun doSomething()
}

class MyViewModel @Inject constructor(
    private val someManager: SomeManager
) : ViewModel() {
    private val _someState = MutableLiveData<State>()
    val someState: LiveData<State> = _someState
    
    fun doSomething() {
        // Business logic
    }

    fun doSomethingElse() {
        // Business logic
        someManager.doSomething()
    }

    fun doNonPrivateBusinessLogic() {
        // Business logic
    }
}
```

## The Android problem - Context everywhere

Another very common problem for Android devs is using Context everywhere. But how do you develop an Android app without using Context? you don't. So? that's why we need to use SOLID principles.

What is a Context on Android? It is a global object that provides access to application-specific resources and classes, as well as application-specific system classes.

For example, let's say you have to access the resources of your app. You can do it as follows:

```kotlin
// com.jesusdmedinac.app.activities.MyActivity
val context = this
val resources = context.resources
```

But, what happend if you need to access the resources from a non-Activity/Fragment class? Easy, right? You just need to pass the Context to the class constructor. Wrong!

```kotlin
// com.jesusdmedinac.app.viewmodels.MyViewModel
class MyViewModel @Inject constructor(
    private val context: Context
) : ViewModel() {
    fun doSomething() {
        // Business logic
        val resources = context.resources
        val string = resources.getString(R.string.some_string)
        // do something with string
    }
}
```

If you need to access the resources from a non-Activity/Fragment class, you should define an abstraction for the resources and inject it to the class constructor.

```kotlin
// com.jesusdmedinac.app.viewmodels.MyViewModel
interface ResourcesProvider {
    fun getString(resId: Int): String
}

class MyViewModel @Inject constructor(
    private val resourcesProvider: ResourcesProvider
) : ViewModel() {
    fun doSomething() {
        // Business logic
        val string = resourcesProvider.getString(R.string.some_string)
        // do something with string
    }
}
```

Or even better, access the resources from the Activity/Fragment and pass it to the ViewModel.

```kotlin
// com.jesusdmedinac.app.activities.MyActivity
val context = this
val resources = context.resources

class MyActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_my)
        
        val resources = this.resources
        val string = resources.getString(R.string.some_string)
        
        viewModel.doSomething(string)
    }
}

// com.jesusdmedinac.app.viewmodels.MyViewModel
class MyViewModel @Inject constructor() : ViewModel() {
    fun doSomething(string: String) {
        // Business logic
    }
}
```

As this example, there are many other examples of how to apply SOLID principles to Android development. The key idea is to always think about the responsibilities of each component and how they interact with each other.

Now let's deep dive into each principle.

## Single Responsibility Principle

SRP states that a component should have only one reason to change. In other words, a component should have only one responsibility.

### What does it mean?

What single responsibility means? in simple words, it means that components with two or more responsibilities are more likely to change due to changes in any of those responsibilities.

Let's see an example:

What comes to your mind when you think about an Android Activity? Most developers would say "a screen" or "a UI component". But what if I tell you that an Activity is not just a UI component, it's also a lifecycle manager, a configuration change handler, a state saver, and a lot more.

This is a violation of the Single Responsibility Principle, as the Activity has multiple reasons to change.

But where I will put all that logic? Easy, we can use multiple approaches:

1. **ViewModel**: For business logic and state management
2. **Repository**: For data access and business logic
3. **Use Cases**: For specific business operations
4. **Utils**: For utility functions
5. **Extensions**: For extending existing classes

And many more approaches, but the key idea is to separate concerns and have each component with a single responsibility.

So, when you are designing a new Activity, think about what responsibilities it has and try to separate them into different components.

## Open/Closed Principle

Now, let's talk about the Open/Closed Principle.

OCP states that a component should be **open for extension but closed for modification**. In other words, you should be able to add new functionality without modifying existing code.

### Why does this matter?

Modifying existing code can introduce bugs and break functionality that already works. Instead, we extend behavior through abstraction and composition.

### Example: Adding cache to a repository

Let's say we have a repository that fetches data from a remote API:

```kotlin
interface Repository {
    suspend fun fetchData(id: String): Data
}

class RemoteRepository(private val apiService: ApiService) : Repository {
    override suspend fun fetchData(id: String): Data {
        return apiService.getData(id)
    }
}
```

Now we want to add caching. We have two options:

#### ❌ Bad approach - Violating OCP

```kotlin
class RemoteRepository(private val apiService: ApiService) : Repository {
    private val cache = mutableMapOf<String, Data>()  // Modified existing class!
    
    override suspend fun fetchData(id: String): Data {
        return cache[id] ?: apiService.getData(id).also { 
            cache[id] = it 
        }
    }
}
```

This violates OCP because we're modifying the existing `RemoteRepository` class.

#### ✅ Good approach - Using composition (Decorator pattern)

```kotlin
class CachedRepository(
    private val repository: Repository  // Depends on abstraction, not concrete class
) : Repository {
    private val cache = mutableMapOf<String, Data>()
    
    override suspend fun fetchData(id: String): Data {
        return cache[id] ?: repository.fetchData(id).also { 
            cache[id] = it 
        }
    }
}
```

Now we can add caching without touching `RemoteRepository`:

```kotlin
// Usage
val remoteRepo = RemoteRepository(apiService)
val cachedRepo = CachedRepository(remoteRepo)  // Extended with cache!
```

### Taking it further: Multiple layers

We can keep extending without modifying:

```kotlin
class LoggingRepository(
    private val repository: Repository
) : Repository {
    override suspend fun fetchData(id: String): Data {
        Log.d("Repository", "Fetching data for id: $id")
        return repository.fetchData(id).also {
            Log.d("Repository", "Data fetched successfully")
        }
    }
}

// Combine multiple decorators
val repository = LoggingRepository(
    CachedRepository(
        RemoteRepository(apiService)
    )
)
```

### Real-world Android example: Data sources

In modern Android architecture, we often use multiple data sources:

```kotlin
class UserRepository(
    private val remoteDataSource: RemoteUserDataSource,
    private val localDataSource: LocalUserDataSource
) : Repository {
    override suspend fun fetchData(id: String): User {
        // Try local first
        localDataSource.getUser(id)?.let { return it }
        
        // Fetch from remote and cache locally
        return remoteDataSource.getUser(id).also {
            localDataSource.saveUser(it)
        }
    }
}
```

This is also OCP-compliant because both data sources implement abstractions, making them easily replaceable or extendable.

### Key takeaway

**Open for extension**: Add new behavior by creating new classes
**Closed for modification**: Don't change existing, working code

Use interfaces and composition to make your code flexible and maintainable.

## Liskov Substitution Principle

LSP states that objects of a superclass should be replaceable with objects of its subclasses without breaking the application. In other words, if you have a class `A` and a class `B` that extends `A`, you should be able to use `B` anywhere you use `A`.

### Why does this matter?

This principle is often misunderstood. Many developers think it's just about inheritance, but it's actually about **behavioral compatibility**. If your subclass changes the expected behavior of the parent class, you're violating LSP.

Let me show you a classic example that I've seen in many Android codebases.

### Example: DataSource with different behaviors

Let's say we have a `DataSource` interface that both `LocalDataSource` and `RemoteDataSource` implement:

```kotlin
interface DataSource {
    suspend fun getUser(id: String): User?
}
```

The contract is simple: return a `User` if found, or `null` if not. Now let's see what happens when implementations don't honor this contract:

#### ❌ Bad approach - Violating LSP

```kotlin
class LocalDataSource(
    private val database: UserDatabase
) : DataSource {
    override suspend fun getUser(id: String): User? {
        return database.userDao().findById(id)  // Returns null if not found ✓
    }
}

class RemoteDataSource(
    private val apiService: ApiService
) : DataSource {
    override suspend fun getUser(id: String): User? {
        return apiService.getUser(id)  // Throws HttpException if not found! ✗
    }
}
```

Why is this a problem? Let's see:

```kotlin
class UserRepository(
    private val dataSource: DataSource  // Could be Local or Remote
) {
    suspend fun getUser(id: String): User? {
        return dataSource.getUser(id)  // Expects null if not found
    }
}

// Works fine with LocalDataSource
val localRepo = UserRepository(LocalDataSource(database))
localRepo.getUser("123")  // Returns null if not found ✓

// Crashes with RemoteDataSource!
val remoteRepo = UserRepository(RemoteDataSource(apiService))
remoteRepo.getUser("123")  // Throws HttpException! ✗
```

The `RemoteDataSource` cannot be substituted for `LocalDataSource` without breaking the expected behavior. The caller expects `null`, not an exception.

#### ✅ Good approach - Honoring the contract

```kotlin
class LocalDataSource(
    private val database: UserDatabase
) : DataSource {
    override suspend fun getUser(id: String): User? {
        return database.userDao().findById(id)
    }
}

class RemoteDataSource(
    private val apiService: ApiService
) : DataSource {
    override suspend fun getUser(id: String): User? {
        return try {
            apiService.getUser(id)
        } catch (e: HttpException) {
            if (e.code() == 404) null  // Honor the contract: return null if not found
            else throw e  // Re-throw for actual errors (500, etc.)
        }
    }
}
```

Now both implementations honor the same contract. Any code that depends on `DataSource` will work correctly regardless of which implementation is used.

#### Alternative: Make the contract explicit with Result

If you want to be even more explicit about possible failures:

```kotlin
interface DataSource {
    suspend fun getUser(id: String): Result<User?>
}

class RemoteDataSource(
    private val apiService: ApiService
) : DataSource {
    override suspend fun getUser(id: String): Result<User?> {
        return try {
            Result.success(apiService.getUser(id))
        } catch (e: HttpException) {
            if (e.code() == 404) Result.success(null)
            else Result.failure(e)
        }
    }
}
```

Now the contract explicitly states that the operation can fail, and callers must handle it.

### Real-world Android example: Click listeners

Another common violation I've seen is with click listeners:

#### ❌ Bad approach

```kotlin
open class BaseClickListener : View.OnClickListener {
    override fun onClick(view: View) {
        // Base implementation
        trackClick(view)
    }

    protected fun trackClick(view: View) {
        Analytics.trackClick(view.id)
    }
}

class DisabledClickListener : BaseClickListener() {
    override fun onClick(view: View) {
        // Does nothing! Violates LSP
    }
}
```

If someone expects `BaseClickListener` behavior and receives `DisabledClickListener`, the tracking won't happen. This breaks the expected contract.

#### ✅ Good approach

```kotlin
interface ClickHandler {
    fun handleClick(view: View)
}

class TrackingClickHandler(
    private val analytics: Analytics
) : ClickHandler {
    override fun handleClick(view: View) {
        analytics.trackClick(view.id)
    }
}

class NoOpClickHandler : ClickHandler {
    override fun handleClick(view: View) {
        // Explicitly does nothing - this IS the expected behavior
    }
}

class CompositeClickHandler(
    private val handlers: List<ClickHandler>
) : ClickHandler {
    override fun handleClick(view: View) {
        handlers.forEach { it.handleClick(view) }
    }
}
```

Now each implementation clearly defines its behavior, and you can compose them as needed.

### Key takeaway

**Subclasses must honor the contract of their parent class**. If a subclass can't fully substitute its parent without breaking functionality, you probably need a different design.

When in doubt, prefer composition over inheritance.

## Interface Segregation Principle

ISP states that clients should not be forced to depend on interfaces they don't use. In other words, it's better to have many small, specific interfaces than one large, general-purpose interface.

### Why does this matter?

Large interfaces create unnecessary coupling. If a class implements an interface with 10 methods but only needs 2, it's forced to provide implementations (even empty ones) for the other 8. This leads to code pollution and maintenance headaches.

### Example: The "God" interface

I've seen this pattern too many times in Android projects:

#### ❌ Bad approach - Fat interface

```kotlin
interface UserRepository {
    fun getUser(id: String): User
    fun getAllUsers(): List<User>
    fun saveUser(user: User)
    fun deleteUser(id: String)
    fun updateUser(user: User)
    fun searchUsers(query: String): List<User>
    fun getUsersByRole(role: String): List<User>
    fun exportUsersToCSV(): File
    fun importUsersFromCSV(file: File)
    fun syncUsersWithServer()
    fun getCachedUsers(): List<User>
    fun clearCache()
}
```

Now imagine you have a screen that only needs to display a single user. You're forced to depend on this entire interface:

```kotlin
class UserProfileViewModel(
    private val userRepository: UserRepository  // Depends on 12 methods, uses only 1
) : ViewModel() {
    fun loadUser(id: String) {
        val user = userRepository.getUser(id)
        // ...
    }
}
```

What if you want to test this ViewModel? You need to mock all 12 methods, even though you only use one.

#### ✅ Good approach - Segregated interfaces

```kotlin
interface UserReader {
    fun getUser(id: String): User
    fun getAllUsers(): List<User>
    fun searchUsers(query: String): List<User>
    fun getUsersByRole(role: String): List<User>
}

interface UserWriter {
    fun saveUser(user: User)
    fun deleteUser(id: String)
    fun updateUser(user: User)
}

interface UserSync {
    fun syncUsersWithServer()
}

interface UserCache {
    fun getCachedUsers(): List<User>
    fun clearCache()
}

interface UserExporter {
    fun exportUsersToCSV(): File
    fun importUsersFromCSV(file: File)
}
```

Now your ViewModel only depends on what it needs:

```kotlin
class UserProfileViewModel(
    private val userReader: UserReader  // Only depends on what it uses
) : ViewModel() {
    fun loadUser(id: String) {
        val user = userReader.getUser(id)
        // ...
    }
}
```

And your repository can implement multiple interfaces:

```kotlin
class UserRepositoryImpl(
    private val apiService: ApiService,
    private val database: UserDatabase
) : UserReader, UserWriter, UserCache {
    // Implement only the methods this class is responsible for
}
```

### Real-world Android example: RecyclerView adapters

Another common violation is with RecyclerView click listeners:

#### ❌ Bad approach

```kotlin
interface ItemClickListener {
    fun onItemClick(item: Item)
    fun onItemLongClick(item: Item)
    fun onItemSwipe(item: Item)
    fun onItemDrag(item: Item)
    fun onItemFavorite(item: Item)
    fun onItemShare(item: Item)
    fun onItemDelete(item: Item)
}

class SimpleListAdapter(
    private val listener: ItemClickListener  // Needs to implement 7 methods!
) : RecyclerView.Adapter<ViewHolder>() {
    // ...
}
```

#### ✅ Good approach

```kotlin
fun interface OnItemClick {
    fun invoke(item: Item)
}

fun interface OnItemLongClick {
    fun invoke(item: Item)
}

class SimpleListAdapter(
    private val onItemClick: OnItemClick,
    private val onItemLongClick: OnItemLongClick? = null  // Optional!
) : RecyclerView.Adapter<ViewHolder>() {
    // ...
}

// Usage - clean and simple
val adapter = SimpleListAdapter(
    onItemClick = { item -> navigateToDetail(item) }
)
```

### Key takeaway

**Keep interfaces small and focused**. A client should never be forced to implement methods it doesn't need.

When you see an interface with more than 5 methods, ask yourself: can this be split into smaller, more cohesive interfaces?

## Dependency Inversion Principle

DIP states two things:

1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
2. Abstractions should not depend on details. Details should depend on abstractions.

### Why does this matter?

This is probably the most important principle for building testable and maintainable Android applications. Without DIP, your code becomes tightly coupled, making it impossible to test in isolation or swap implementations.

### Example: Direct dependencies

Let's say you have a ViewModel that needs to fetch users:

#### ❌ Bad approach - High-level depends on low-level

```kotlin
class UserViewModel : ViewModel() {
    // Direct dependency on concrete implementation
    private val apiService = RetrofitClient.createService(ApiService::class.java)
    private val database = Room.databaseBuilder(
        context,
        AppDatabase::class.java,
        "app-db"
    ).build()

    fun loadUsers() {
        viewModelScope.launch {
            try {
                val users = apiService.getUsers()
                database.userDao().insertAll(users)
            } catch (e: Exception) {
                // Handle error
            }
        }
    }
}
```

What's wrong here?

1. **Impossible to test**: You can't mock `apiService` or `database`
2. **Tightly coupled**: Changing from Retrofit to Ktor requires modifying this class
3. **No flexibility**: Can't easily switch between real and fake data sources

#### ✅ Good approach - Depend on abstractions

```kotlin
// Abstraction (interface)
interface UserRepository {
    suspend fun getUsers(): List<User>
    suspend fun saveUsers(users: List<User>)
}

// Low-level implementation
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) : UserRepository {
    override suspend fun getUsers(): List<User> {
        return apiService.getUsers()
    }

    override suspend fun saveUsers(users: List<User>) {
        userDao.insertAll(users)
    }
}

// High-level module depends on abstraction
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository  // Depends on interface, not implementation
) : ViewModel() {
    fun loadUsers() {
        viewModelScope.launch {
            try {
                val users = userRepository.getUsers()
                userRepository.saveUsers(users)
            } catch (e: Exception) {
                // Handle error
            }
        }
    }
}
```

Now you can easily test it:

```kotlin
class UserViewModelTest {
    @Test
    fun `loadUsers should fetch and save users`() = runTest {
        // Create a fake implementation for testing
        val fakeRepository = object : UserRepository {
            override suspend fun getUsers() = listOf(User("1", "John"))
            override suspend fun saveUsers(users: List<User>) { /* no-op */ }
        }

        val viewModel = UserViewModel(fakeRepository)
        viewModel.loadUsers()

        // Assert...
    }
}
```

### Real-world Android example: The dependency flow

In a well-architected Android app, the dependency flow should look like this:

```
UI Layer (Activity/Fragment/Composable)
    ↓ depends on
ViewModel (abstraction)
    ↓ depends on
Use Cases / Interactors (abstraction)
    ↓ depends on
Repository (abstraction)
    ↓ depends on
Data Sources (abstraction)
    ↓ implemented by
Concrete implementations (Retrofit, Room, SharedPreferences, etc.)
```

Each layer depends only on abstractions from the layer below. The concrete implementations are provided at runtime through dependency injection (Hilt, Koin, etc.).

### Taking it further: The Dependency Injection setup

Here's how you would wire everything with Hilt:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    fun provideApiService(): ApiService {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .build()
            .create(ApiService::class.java)
    }
}
```

Now your ViewModel doesn't know or care about Retrofit, Room, or any other implementation detail. It just knows it has a `UserRepository` that can fetch and save users.

### Key takeaway

**Depend on abstractions, not concretions**. Your high-level business logic should never directly instantiate low-level details like network clients or databases.

Use dependency injection to provide concrete implementations at runtime. This makes your code testable, flexible, and maintainable.

## Conclusion

SOLID principles are not just theoretical concepts for interviews. They are practical tools that help you write better code every day.

Let me summarize what we've learned:

1. **Single Responsibility**: One class, one reason to change
2. **Open/Closed**: Extend behavior without modifying existing code
3. **Liskov Substitution**: Subclasses must be substitutable for their base classes
4. **Interface Segregation**: Small, focused interfaces over large, general ones
5. **Dependency Inversion**: Depend on abstractions, not concrete implementations

The next time you're in a pull request review defending your code changes, remember: these principles are not about being pedantic. They're about building software that's easier to understand, test, and maintain.

And if your team doesn't follow these principles? Well, maybe it's time to have a conversation. Or share this post with them.