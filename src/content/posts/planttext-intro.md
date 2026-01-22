---
pubDate: 2025-11-30
author: JesusDMedinaC
title: PlantText Intro
description: "PlantText is a focused online editor built around PlantUML, the textual language for describing diagrams."
image:
  url: "/src/images/blog/planttext.png"
  alt: "#"
tags: ["PlantText", "PlantUML", "Android"]
---

If you've ever sketched an architecture on a napkin only to dread converting it into UML, PlantText might be the shortcut you've been missing—let me show you why.

## What is PlantText?

PlantText is a focused online editor built around PlantUML, the textual language for describing diagrams. Instead of dragging shapes or fighting sluggish desktop tools, you write short snippets such as `@startuml` blocks, press Generate, and receive sequence, class, or component diagrams instantly. PlantText handles syntax highlighting, version links, and embeds, so you can stay in your flow while collaborating with teammates.

## Why designers and developers stay

The editor keeps live previews in sync with your text, so updating a method signature or component boundary takes seconds.
Diagrams are stored as links you can paste in pull requests, tickets, or documentation without exporting images manually.
Works everywhere because PlantText relies on PlantUML notation, you're not locked into a proprietary format—drop the same source into IDE plugins, CI pipelines, or documentation generators.
It also easy to generete from AI (ChatGPT, Claude, etc.) or provide it to AI to share context.

## My first class diagram

Let's create a class diagram for a simple Android app. For that, we will use the following app structure:

Our app uses a simple clean architecture with presentation, domain, and data modules. Our presentation module contains the UI and ViewModels, our domain module contains the use cases, repositories declarations, and entities, and our data module contains the data sources, repositories implementations, and data models.

Let see the actual code sample:

```kotlin
// Presentation layer

@Composable
fun MainScreen(
  viewModel: MainViewModel,
) {
    ...
}

class MainViewModel(private val getUserUseCase: GetUserUseCase) : ViewModel() {
    ...
}

data class UserUI(val id: String, val name: String)
// Domain Layer

interface GetUserUseCase {
    suspend fun getUser(userId: String): User
}

class GetUserUseCaseImpl(private val userRepository: UserRepository) : GetUserUseCase {
    ...
}

interface UserRepository {
    suspend fun getUser(userId: String): User
}

data class User(val id: String, val name: String)
// Data Layer
class UserRepositoryImpl(private val dataSource: UserDataSource) : UserRepository {
    ...
}

interface UserDataSource {
    suspend fun getUser(userId: String): DataUser
}

class UserDataSourceImpl(private val dataSource: UserDataSource) : UserDataSource {
    ...
}

data class DataUser(val id: String, val name: String)
```

### Defining the classes

Declaring classes on PlantText is as simple as writing the class name and its properties. For example, let's define the User class:

```planttext
@startuml

class User {
    +id: String
    +name: String
}
@enduml
```

Let's break down the syntax: `@startuml` is the start of the diagram. `class User` is the class name. `+id: String` and `+name: String` are the property name and type. We use + to indicate that the property is public. `@enduml` is the end of the diagram.

Now, what about interfaces? well, I think you now the answer, right?

```planttext
@startuml

interface GetUserUseCase {
    +getUser(userId: String): User
}
@enduml
```

The only difference is the keyword `interface`. So maybe you now what's next... of course! the relationships.

We can define implementations using `implements` keyword, and `extends` for inheritance.

```planttext
@startuml

interface GetUserUseCase {
    +getUser(userId: String): User
}

class GetUserUseCaseImpl implements GetUserUseCase

@enduml
```

It's so simple, right? 

So the full diagram would be:

```planttext
@startuml
' Presentation Layer
class MainViewModel extends ViewModel {
    - getUserUseCase: GetUserUseCase
    + MainViewModel(getUserUseCase: GetUserUseCase)
    + getUser(userId: String): UserUI
}

' Domain Layer
interface GetUserUseCase {
    +getUser(userId: String): User
}

class GetUserUseCaseImpl implements GetUserUseCase {
    GetUserUseCaseImpl(private val userRepository: UserRepository)
    +getUser(userId: String): User
}

interface UserRepository {
    +getUser(userId: String): User
}

' Data Layer
class UserRepositoryImpl implements UserRepository {
    UserRepositoryImpl(private val dataSource: UserDataSource)
    +getUser(userId: String): User
}

interface UserDataSource {
    +getUser(userId: String): User
}

class UserDataSourceImpl implements UserDataSource {
    UserDataSourceImpl(private val dataSource: UserDataSource)
    +getUser(userId: String): User
}

@enduml
```

With this simple declarations, we'll get a diagram like this:

![Class Diagram](/src/images/blog/planttext-class-diagram.png)

But, it seems like there is something missing... where are the relationships?

### Relationships

PlantText also supports relationships between classes. For example, let's define a relationship between MainViewModel and GetUserUseCase:

```planttext
@startuml

MainViewModel --> GetUserUseCase

@enduml
```

This simple notation express a dependency relationship between MainViewModel and GetUserUseCase. So, in order to define all missing relationships, we can use the following notation:

```planttext
@startuml
MainViewModel --> GetUserUseCaseImpl
GetUserUseCaseImpl --> UserRepositoryImpl
UserRepositoryImpl --> UserDataSourceImpl
UserDataSourceImpl --> UserDataSource
@enduml
```

Let's take a look at the updated diagram:

![Class Diagram](/src/images/blog/planttext-class-diagram-relationships.png)

What about the direction of the arrow? well, PlantText supports different types of relationships:

- `-->` : Dependency
- `<--` : Inheritance
- `*--` : Realization
- `o--` : Association
- `o<--` : Composition
- `o*--` : Aggregation

Depending on the direction of the arrow, we can define the type of relationship. A wrong direction of the arrow can lead to a wrong relationship.

## Conclusion

With this basic introduction, you are now ready to start playing around with PlantText. You can find more information about PlantText in the [official documentation](https://planttext.com/).

---

If you want to read more content about Android, AI and Mobile development subscribe to my channel and follow me on all my networks as @jesusdmedinac.