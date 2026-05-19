# Addressables Notes

## Dynamic Asset Management 
- help you improve the performance of your game by allowing you to determine when, and from where, assets are loaded in and out of memory during runtime.

## Different approaches to asset management

### Direct references

- When you drag an asset into a scene or onto a component through the Inspector window of the Editor, you're making a direct reference to that asset. 
- When you build your application, these assets are saved in a separate file associated with your scene. 
- When your application is running on a user's device and the scene is loaded, the Unity Player loads the entire asset file into memory before the scene, ensuring that the scene has access to everything it directly references.
- If you use only direct references in your game, then your approach to asset management is not dynamic. Loading and unloading the scene is the only control offered.<br>
→  Risk of building a game with a large build size that performs slowly, especially on devices with less memory such as mobile.

### Resources folder

- The Resources folder and the Resources API provided a simple way to manage assets in memory. In the project folder structure of older Unity projects, you might see one or more Resources folders containing assets.
- During the Player build process, the Editor finds the assets in any folders named Resources and bundles them into a serialized file, with metadata and indexing information, that's packaged with your application. 
- The Resources API allows you to write scripts to load and unload the assets in folders named Resources.
- The Resources system has several disadvantages over newer systems. It doesn't allow for a fine-grained approach to memory management, it slows down startup time, and it can bloat the size of your built application. Moreover, it doesn't support delivery of content from a CDN.

### AssetBundle system
- The AssetBundle system organizes assets into containers called AssetBundles. Like the Resources folder, the AssetBundle system creates sets of assets into separate files. Unlike Resources folders, AssetBundles can be stored locally with the Player or remotely in the cloud.
- The AssetBundles system, through its API, minimizes the impact on network and system resources. It does this by allowing you to download the bundles on an as-needed basis, so that you can add DLC and post-release content updates. For example, you can deliver new content for your players to view, earn, or purchase, without requiring them to download a new version of your game. Once bundles are downloaded, AssetBundles API provides a way to load and unload assets from bundles.
-  AssetBundles has limitations that have required developers to implement their own solutions:
    + The AssetBundles system is an API that can only be used in scripts. There is a limited user interface for defining AssetBundles in the Unity Editor, but building AssetBundles requires scripting.
    + The AssetBundles API by itself does not keep track of asset dependencies between AssetBundles. For example, if you want to load a prefab from AssetBundle A, you will need to locate any of its dependencies such as meshes, materials, and textures that may be located in other AssetBundles, and ensure those dependent AssetBundles are loaded at runtime before you attempt to load the prefab.
    + Memory allocation and deallocation is direct and manual, so it's possible to unload an asset from an AssetBundle while other code still depends on that asset, potentially resulting in missing content issues or memory leakage. This can become problematic in code that creates race conditions, and requires fortifying your code against these problems.
    + AssetBundles runtime API is not aware whether you put your built bundles locally or remotely. This requires you to keep track of the location of that AssetBundle, whether it's on a web server or on disk.

### Addressables

- The Addressables system is built on top of the AssetBundles API to give you a user interface in the Unity Editor, and to automate processes that before could only be managed manually in scripts. 
- With the Addressables system, you can organize your assets in the Unity Editor, and let the system handle dependencies, asset locations, and memory allocation and deallocation.

## Why should you use Addressables?

### Flexibility
- gives you the flexibility to specify where an asset is hosted. 
- You can install assets so that they exist alongside your application on disk or download them on demand from a remote web server. 
- You can then change where a specific asset exists, such as from local to remote, or from a monolithic bundle to more granular bundles, all without needing to rewrite code.

### Dependency management
- automatically loads all dependencies of any assets you load so that all meshes, shaders, animations, and other dependent assets load before the asset requested loads. 
- For example, if you request to load a character, the system will automatically load the character's meshes, materials, and animations, and the materials' textures and shaders as well.

### Memory management
- automates much of the drudge work of memory management. 
- As you load assets into and out of your game in scripts, the system keeps track of memory allocation. 
- You can also use the system's robust profiler to further improve the memory efficiency of your game.

### Efficient content packing
- Because the Addressables system maps complex dependency chains, it helps you pack assets efficiently, even if you move or rename assets. 
- You can choose how granular you want your assets to be at a location, favoring packing them separately or grouped together, and can easily prepare assets for both local and remote deployment to support on-demand or downloadable content (DLC), which can reduce application sizes.

### Cloud build and content delivery
- The Addressables system has been integrated into Unity Gaming Services (UGS), specifically Cloud Content Delivery and Cloud Build, giving you end-to-end services for live game updates and building your applications in the cloud.

### Scriptable Build Pipeline
- The Addressables system uses the Scriptable Build Pipeline (SBP), which is more robust than the legacy AssetBundle build pipeline. You can use the pre-defined build flows or, if you want to use a more advanced method, create your own using the divided-up APIs.

### Localization
The system is also integrated with Unity’s Localization package so that you can use various languages in your projects.

## What is an addressable?

- When you enable an asset as addressable, Unity generates an address, which is a string identifier that the Addressables system associates with the asset’s location, regardless of whether the asset resides locally with your built game or on a content delivery network.

- In your scripts, you can use the asset's address instead of keeping track of its location, so that you don't have to write code that specifies where — only what. This way, if an asset location changes, you won't have to rewrite your code.

## Prerequisites

### Asynchronous operations

#### Coroutines

##### Concept
- A coroutine is a method that can pause execution and resume later.
- In Unity, coroutines usually run across multiple frames.
- Coroutines do not create a new thread. They still run on the main Unity thread.
- They are useful for spreading logic over time without putting everything inside `Update()`.

##### Use cases
- Delays
- Timed actions
- Animations without `Update()`
- Cooldowns
- Sequenced logic
- Waiting for conditions before continuing

##### Syntax

```csharp
StartCoroutine(MyCoroutine());

IEnumerator MyCoroutine() {
    yield return new WaitForSeconds(2f);
}
```

To stop a coroutine later, store the returned `Coroutine` reference:

```csharp
Coroutine runningCoroutine;

runningCoroutine = StartCoroutine(MyCoroutine());
StopCoroutine(runningCoroutine);
```

##### Yield return types
- `yield return null`
    + Pauses until the next frame.
- `yield return new WaitForSeconds(waitTime)`
    + Pauses for the specified scaled time in seconds.
    + Affected by `Time.timeScale`.
    + Real wait time is approximately `waitTime / Time.timeScale`.
    + The exact resume time can vary:
        + if the coroutine starts during a long frame, the wait starts at the end of that frame
        + the coroutine resumes on the first frame after the wait time has passed, not immediately at the exact timestamp
- `yield return new WaitForSecondsRealtime(waitTime)`
    + Pauses for the specified real, unscaled time.
    + Not affected by `Time.timeScale`.
- `yield return new WaitForEndOfFrame()`
    + Pauses until the end of the frame, after cameras and GUI are rendered and before the frame is displayed.
- `yield return new WaitUntil(() => condition)`
    + Pauses until the supplied delegate returns `true`.
    + The delegate is checked each frame after `Update()` and before `LateUpdate()`.
- `yield return new WaitWhile(() => condition)`
    + Pauses while the supplied delegate returns `true`.
    + Resumes when the delegate returns `false`.

##### Stopping a coroutine
- `StopCoroutine(MyCoroutine());`
- `StopCoroutine("MyCoroutine");`
- `StopCoroutine(myCoroutine);`
- Coroutines also stop if:
    + the `GameObject` the script is attached to becomes inactive
    + the `MonoBehaviour` is destroyed with `Destroy()`
- Disabling the `MonoBehaviour` with `enabled = false` does not stop its coroutines.
- `StopCoroutine(null)` throws a `NullReferenceException`.
- `StopAllCoroutines()` stops all coroutines running on that `MonoBehaviour`.
- Be consistent with how you start and stop a coroutine:
    + if you start it by method name, stop it by method name
    + if you start it with an `IEnumerator`, stop that same `IEnumerator`
    + if you store the returned `Coroutine`, stop that `Coroutine` reference

##### Nested coroutines
- A coroutine can wait for another coroutine to finish.

```csharp
IEnumerator ParentCoroutine() {
    yield return StartCoroutine(ChildCoroutine());
    Debug.Log("Child coroutine finished");
}
```

- Use nested coroutines when a sequence has clear smaller steps.
- Avoid deeply nested coroutine chains because they can become hard to follow and debug.

##### Tradeoffs
- Advantages:
    + simple syntax for time-based behavior
    + avoids bloated `Update()` methods
    + good for readable sequences
- Disadvantages:
    + still runs on the main thread
    + harder to cancel cleanly if references are not stored
    + exceptions can be less obvious than normal synchronous code
    + complex coroutine flows can become difficult to trace

##### When not to use
- Heavy CPU computation
- Real multithreading
- High-frequency physics logic
- Logic that needs precise per-frame control
- Complex async workflows that need return values, error handling, or cancellation tokens

##### Alternative solutions comparison
- `Update()`
    + Best for continuous per-frame logic.
    + Common for input, movement, timers, and state checks.
- `Invoke()` / `InvokeRepeating()`
    + Simple for delayed or repeated method calls.
    + Less flexible than coroutines.
- `async` / `await` with `Task`
    + Good for general C# asynchronous work, especially I/O.
    + Not automatically tied to Unity object lifetime.
    + Be careful when touching Unity APIs from background threads.
- UniTask
    + Unity-friendly alternative to `Task`.
    + Supports async/await style with lower allocation overhead.
    + Good for larger async workflows, cancellation tokens, and readable sequencing.
- Unity Jobs System
    + Best for CPU-heavy work that can run in parallel.
    + More complex, but designed for performance.


#### Delegates

##### Concept
- A delegate is a type-safe reference to a method.
- It lets you store a method in a variable, pass a method as an argument, or call a method later.
- A delegate defines the method signature it can point to:
    + return type
    + parameter types
- Any method with the same signature can be assigned to that delegate.

##### Why delegates are useful
- They allow flexible code without hard-coding which method should run.
- They are commonly used for:
    + callbacks
    + event systems
    + UI button actions
    + sorting/filtering logic
    + async completion handlers
    + custom behavior passed into reusable methods

##### Basic syntax

```csharp
public delegate void MyDelegate();

public class DelegateExample {
    MyDelegate myDelegate;

    void SayHello() {
        Debug.Log("Hello");
    }

    void Start() {
        myDelegate = SayHello;
        myDelegate();
    }
}
```

- `public delegate void MyDelegate();` declares a delegate type.
- `MyDelegate myDelegate;` creates a delegate variable.
- `myDelegate = SayHello;` stores a method reference.
- `myDelegate();` invokes the stored method.

##### Delegates with parameters

```csharp
public delegate void DamageHandler(int damageAmount);

void TakeDamage(int amount) {
    Debug.Log("Took damage: " + amount);
}

void Start() {
    DamageHandler onDamage = TakeDamage;
    onDamage(25);
}
```

- The assigned method must match the delegate signature.
- `TakeDamage(int amount)` can be assigned because it returns `void` and takes one `int`.

##### Delegates with return values

```csharp
public delegate int CalculateScore(int baseScore, int multiplier);

int MultiplyScore(int baseScore, int multiplier) {
    return baseScore * multiplier;
}

void Start() {
    CalculateScore calculateScore = MultiplyScore;
    int finalScore = calculateScore(10, 3);

    Debug.Log(finalScore);
}
```

- Delegates can return values.
- The method return type must match the delegate return type.

##### Multicast delegates
- A delegate can reference more than one method.
- Use `+=` to add a method.
- Use `-=` to remove a method.

```csharp
public delegate void PlayerAction();

PlayerAction onPlayerDied;

void PlayDeathAnimation() {
    Debug.Log("Play death animation");
}

void ShowGameOverScreen() {
    Debug.Log("Show game over screen");
}

void Start() {
    onPlayerDied += PlayDeathAnimation;
    onPlayerDied += ShowGameOverScreen;

    onPlayerDied();
}
```

- Methods are called in the order they were added.
- Multicast delegates are best when the delegate returns `void`.
- If a multicast delegate has a return value, only the last method's return value is kept.

##### Null checking
- A delegate is `null` when no method has been assigned to it.
- Calling a `null` delegate causes a `NullReferenceException`.

```csharp
if (onPlayerDied != null) {
    onPlayerDied();
}
```

Shorter syntax:

```csharp
onPlayerDied?.Invoke();
```

With parameters:

```csharp
onDamageTaken?.Invoke(25);
```

##### Anonymous methods and lambda expressions
- Delegates can store named methods, anonymous methods, or lambda expressions.

```csharp
public delegate void MessageHandler(string message);

void Start() {
    MessageHandler handler = delegate(string message) {
        Debug.Log(message);
    };

    handler("Hello from an anonymous method");
}
```

Lambda version:

```csharp
MessageHandler handler = message => {
    Debug.Log(message);
};

handler("Hello from a lambda");
```

- Lambdas are useful for short behavior.
- Named methods are usually better when the logic is reused or needs to be unsubscribed later.

##### Built-in delegate types
- C# already provides common delegate types, so you often do not need to define your own.

`Action`
- Represents a method that returns `void`.

```csharp
Action onComplete;

void Start() {
    onComplete = () => Debug.Log("Complete");
    onComplete?.Invoke();
}
```

`Action<T>`
- Represents a method that returns `void` and takes parameters.

```csharp
Action<int> onHealthChanged;

void Start() {
    onHealthChanged = health => Debug.Log("Health: " + health);
    onHealthChanged?.Invoke(75);
}
```

`Func<T>`
- Represents a method that returns a value.
- The last generic type is the return type.

```csharp
Func<int, int, int> add;

void Start() {
    add = (a, b) => a + b;

    int result = add(2, 3);
    Debug.Log(result);
}
```

- `Func<int, int, int>` means:
    + first `int`: first parameter
    + second `int`: second parameter
    + third `int`: return type

`Predicate<T>`
- Represents a method that takes one value and returns `bool`.

```csharp
Predicate<int> isAlive = health => health > 0;

bool result = isAlive(10);
```

##### Unity example: callback after loading

```csharp
using System;
using System.Collections;
using UnityEngine;

public class Loader : MonoBehaviour {
    public void LoadData(Action onComplete) {
        StartCoroutine(LoadRoutine(onComplete));
    }

    IEnumerator LoadRoutine(Action onComplete) {
        yield return new WaitForSeconds(2f);

        Debug.Log("Data loaded");
        onComplete?.Invoke();
    }
}
```

Usage:

```csharp
public class GameManager : MonoBehaviour {
    [SerializeField] Loader loader;

    void Start() {
        loader.LoadData(OnLoadComplete);
    }

    void OnLoadComplete() {
        Debug.Log("Start game");
    }
}
```

##### Unity example: custom behavior

```csharp
using System;
using UnityEngine;

public class Enemy : MonoBehaviour {
    public void Attack(Action<int> applyDamage) {
        int damage = 10;
        applyDamage?.Invoke(damage);
    }
}
```

Usage:

```csharp
enemy.Attack(damage => {
    playerHealth -= damage;
    Debug.Log("Player health: " + playerHealth);
});
```

##### Delegates vs direct method calls
- Direct method call:
    + simple
    + clear
    + best when the caller knows exactly what method should run
- Delegate:
    + more flexible
    + lets another object decide what method should run
    + useful when the caller should not know the exact implementation

##### Delegates vs events
- Delegates are method references.
- Events are built on top of delegates.
- An event protects the delegate so outside classes can subscribe and unsubscribe, but cannot directly invoke it.
- Use a plain delegate for local callbacks or private behavior.
- Use an event when other objects need to subscribe to something that happened.

##### Common mistakes
- Calling a delegate without checking for `null`.
- Using a lambda when you need to unsubscribe later.
- Forgetting to unsubscribe from long-lived delegates or events.
- Using delegates when a simple method call would be clearer.
- Creating complex delegate chains that are hard to trace.

##### When to use
- Use delegates when you want to pass behavior as data.
- Use delegates when one class should call back into another class without tightly depending on it.
- Use delegates when you need flexible, reusable logic.

##### When not to use
- Do not use delegates just to make code look advanced.
- Do not use delegates when a normal method call is simpler.
- Do not use delegates for large systems where an event, interface, or state machine would make the flow clearer.

#### Events

##### Concept
- An event is a notification that something happened.
- Events are built on top of delegates.
- They let one class announce something without needing to know which other classes are listening.
- The class that owns the event is called the publisher.
- The classes that listen to the event are called subscribers.

##### Why events are useful
- Events reduce tight coupling between classes.
- The publisher does not need direct references to every listener.
- Multiple subscribers can react to the same event.
- Events are commonly used for:
    + player death
    + health changes
    + score changes
    + button clicks
    + item pickups
    + async operation completion
    + game state changes

##### Basic syntax

```csharp
public delegate void PlayerDiedHandler();

public event PlayerDiedHandler OnPlayerDied;
```

- `PlayerDiedHandler` defines the method signature.
- `OnPlayerDied` is the event that other classes can subscribe to.
- Outside classes can use `+=` and `-=`.
- Outside classes cannot directly invoke the event.

##### Raising an event
- The class that declares the event is responsible for invoking it.
- Use `?.Invoke()` to avoid a `NullReferenceException` when there are no subscribers.

```csharp
public class Player {
    public delegate void PlayerDiedHandler();
    public event PlayerDiedHandler OnPlayerDied;

    int health = 100;

    public void TakeDamage(int damage) {
        health -= damage;

        if (health <= 0) {
            OnPlayerDied?.Invoke();
        }
    }
}
```

##### Subscribing to an event

```csharp
public class GameManager {
    Player player;

    public void StartGame() {
        player = new Player();
        player.OnPlayerDied += HandlePlayerDied;
    }

    void HandlePlayerDied() {
        Debug.Log("Game over");
    }
}
```

- `+=` subscribes a method to the event.
- The subscribed method must match the event delegate signature.

##### Unsubscribing from an event

```csharp
player.OnPlayerDied -= HandlePlayerDied;
```

- `-=` removes a method from the event.
- Unsubscribe when the listener no longer needs the event.
- This helps avoid bugs where destroyed or inactive objects still receive notifications.

##### Unity lifecycle pattern
- In Unity, a common pattern is:
    + subscribe in `OnEnable()`
    + unsubscribe in `OnDisable()`

```csharp
using UnityEngine;

public class HealthDisplay : MonoBehaviour {
    [SerializeField] PlayerHealth playerHealth;

    void OnEnable() {
        playerHealth.OnHealthChanged += UpdateHealthText;
    }

    void OnDisable() {
        playerHealth.OnHealthChanged -= UpdateHealthText;
    }

    void UpdateHealthText(int currentHealth) {
        Debug.Log("Health: " + currentHealth);
    }
}
```

- This works well for scene objects that can be enabled and disabled.
- If the object subscribes once and lives for the whole scene, subscribing in `Start()` can also be fine.
- If the publisher might be destroyed first, check references before unsubscribing.

##### Event with parameters

```csharp
using UnityEngine;

public class PlayerHealth : MonoBehaviour {
    public delegate void HealthChangedHandler(int currentHealth);
    public event HealthChangedHandler OnHealthChanged;

    [SerializeField] int health = 100;

    public void TakeDamage(int damage) {
        health -= damage;
        OnHealthChanged?.Invoke(health);
    }
}
```

Subscriber:

```csharp
using UnityEngine;

public class HealthLogger : MonoBehaviour {
    [SerializeField] PlayerHealth playerHealth;

    void OnEnable() {
        playerHealth.OnHealthChanged += LogHealth;
    }

    void OnDisable() {
        playerHealth.OnHealthChanged -= LogHealth;
    }

    void LogHealth(int currentHealth) {
        Debug.Log("Current health: " + currentHealth);
    }
}
```

##### Using Action for events
- Instead of declaring a custom delegate, you can use `Action`.
- Add `using System;` to use `Action`.

```csharp
using System;
using UnityEngine;

public class PlayerHealth : MonoBehaviour {
    public event Action<int> OnHealthChanged;

    [SerializeField] int health = 100;

    public void TakeDamage(int damage) {
        health -= damage;
        OnHealthChanged?.Invoke(health);
    }
}
```

- `Action<int>` means the event sends one `int` and returns `void`.
- This is shorter than creating a custom delegate type.

##### Using EventHandler
- C# also provides the `EventHandler` pattern.
- This is common in standard C# code.
- The first parameter is the object that raised the event.
- The second parameter contains event data.

```csharp
using System;

public class HealthChangedEventArgs : EventArgs {
    public int CurrentHealth { get; }

    public HealthChangedEventArgs(int currentHealth) {
        CurrentHealth = currentHealth;
    }
}

public class PlayerHealth {
    public event EventHandler<HealthChangedEventArgs> OnHealthChanged;

    int health = 100;

    public void TakeDamage(int damage) {
        health -= damage;
        OnHealthChanged?.Invoke(this, new HealthChangedEventArgs(health));
    }
}
```

Subscriber:

```csharp
void HandleHealthChanged(object sender, HealthChangedEventArgs args) {
    Debug.Log("Current health: " + args.CurrentHealth);
}
```

##### Event access rules
- The class that declares the event can:
    + subscribe
    + unsubscribe
    + invoke the event
- Other classes can only:
    + subscribe with `+=`
    + unsubscribe with `-=`
- This prevents outside classes from accidentally clearing or invoking the event.

Example:

```csharp
playerHealth.OnHealthChanged += UpdateUI;   // allowed
playerHealth.OnHealthChanged -= UpdateUI;   // allowed
playerHealth.OnHealthChanged?.Invoke(100);  // not allowed outside PlayerHealth
```

##### Events vs delegates
- Delegate:
    + can be assigned directly with `=`
    + can be invoked by outside classes if accessible
    + useful for callbacks and private/local behavior
- Event:
    + restricts outside access
    + can only be raised by the declaring class
    + better for broadcasting that something happened

##### Events vs UnityEvent
- C# event:
    + written in code
    + faster and more type-safe
    + not visible in the Inspector
    + good for core gameplay systems
- `UnityEvent`:
    + visible in the Inspector
    + lets designers connect responses without code
    + easier to configure in scenes or prefabs
    + can be harder to trace in code

##### Common Unity example: player death

```csharp
using System;
using UnityEngine;

public class PlayerHealth : MonoBehaviour {
    public event Action OnPlayerDied;

    [SerializeField] int health = 100;

    public void TakeDamage(int damage) {
        if (health <= 0) {
            return;
        }

        health -= damage;

        if (health <= 0) {
            health = 0;
            OnPlayerDied?.Invoke();
        }
    }
}
```

Subscriber:

```csharp
using UnityEngine;

public class GameOverController : MonoBehaviour {
    [SerializeField] PlayerHealth playerHealth;

    void OnEnable() {
        playerHealth.OnPlayerDied += ShowGameOver;
    }

    void OnDisable() {
        playerHealth.OnPlayerDied -= ShowGameOver;
    }

    void ShowGameOver() {
        Debug.Log("Game over");
    }
}
```

##### Common mistakes
- Forgetting to unsubscribe from events.
- Subscribing multiple times by accident.
- Using anonymous lambdas when you need to unsubscribe later.
- Making events public as plain delegates instead of using the `event` keyword.
- Putting too much logic inside event subscribers.
- Assuming subscribers are called in a specific order.

##### When to use
- Use events when one object needs to announce that something happened.
- Use events when multiple objects may need to react.
- Use events when the publisher should not directly control the subscribers.

##### When not to use
- Do not use events when a simple method call is clearer.
- Do not use events for logic that must happen in a strict order.
- Do not use events when the publisher needs a return value from the listener.
- Do not overuse events for every small interaction, because the flow can become hard to trace.

##### Mental model
- Delegate: "Here is a method you can call."
- Event: "Tell me when this thing happens."
- Publisher: "Something happened."
- Subscriber: "I want to react when that happens."

##### Difference between Multicast delegates vs. Events
- A multicast delegate and an event can both call multiple methods.
- The main difference is access control.
- A multicast delegate is just a delegate variable that can hold multiple method references.
- An event is a protected wrapper around a delegate.

##### Multicast delegate
- Can store multiple methods.
- Can be invoked directly if the delegate is accessible.
- Can be overwritten with `=`.
- Can be cleared by assigning `null`.
- Gives outside code more control, which can be risky.

```csharp
public Action OnPlayerDied;

void Start() {
    OnPlayerDied += PlayDeathAnimation;
    OnPlayerDied += ShowGameOverScreen;

    OnPlayerDied?.Invoke();
}
```

Problem:

```csharp
player.OnPlayerDied = null;
player.OnPlayerDied = SomeOtherMethod;
player.OnPlayerDied?.Invoke();
```

- If `OnPlayerDied` is public, outside classes can accidentally remove all subscribers, replace the whole delegate, or invoke it directly.

##### Event
- Also supports multiple subscribers.
- Outside classes can only use `+=` and `-=`.
- Outside classes cannot invoke the event.
- Outside classes cannot overwrite the whole event.
- The declaring class stays in control.

```csharp
public event Action OnPlayerDied;

void Die() {
    OnPlayerDied?.Invoke();
}
```

Outside class:

```csharp
player.OnPlayerDied += ShowGameOverScreen;  // allowed
player.OnPlayerDied -= ShowGameOverScreen;  // allowed
player.OnPlayerDied = null;                 // not allowed
player.OnPlayerDied?.Invoke();              // not allowed
```

##### Quick comparison
- Multicast delegate:
    + "Here is a list of methods that can be called."
    + Flexible, but less protected.
    + Good for private callbacks or internal method chains.
- Event:
    + "Something happened, and subscribers can react."
    + Safer for public notifications.
    + Good when other classes need to listen without controlling the publisher.

##### Rule of thumb
- Use a multicast delegate when the delegate is private or controlled inside one class.
- Use an event when other classes need to subscribe.
- If the member is public and represents something that happened, prefer `event`.

### Structs

- The Addressables API uses a struct, named **AsyncOperationHandle**, to help you keep track of asynchronous operations in your code.
- A struct is a value type in C# that can be used to group related data together in a single object. They are similar to classes in that they can contain fields and methods, but unlike classes, they are passed by value instead of by reference. This means that when you create a struct and pass it to a method or assign it to a variable, a copy of the struct is created rather than a reference to the original object.
- The **AsyncOperationHandle** struct has a Boolean value to indicate whether a method succeeded or failed, and the return value of the request operation.

### Serialization
- Serialization is the process of transforming and storing data between sessions of an application, and deserialization is the process of taking that stored data so that it can be reconstructed when an application runs again.
- Unity serializes fields during build time so that it can deserialize the data stored in them while the application is running. The [SerializeField] attribute works to mark private fields as serialized, and serialized fields become available in the Inspector for editing, just like public variables.
- As you use the Addressables system and API, you'll use serialization to optimize the ways your project consumes assets.

## AssetReference

An AssetReference is a type that references an addressable asset. It is intended to be used as a serializable field in Unity classes like MonoBehaviour or ScriptableObject. When you add an AssetReference to one of these classes, you can assign an address to it in the Inspector with an object picker. The selection is limited to addressable assets.

On the surface, you'll add AssetReferences to your scripts just like you would add direct references, through public fields and private serializable fields. AssetReferences do not store a direct reference to the asset. The AssetReference stores the global unique identifier (GUID) of the asset, which is used by the Addressables system to store the object for retrieval at runtime.

The main benefit of using AssetReference over a string address is that it will restrict the selection of addresses from the Inspector, which avoids problems like typos in string addresses.

## Reference counting

One benefit of the Addressables system is that it helps you manage the loading and unloading of the assets in and out of memory. This is done internally through a reference counting system, which manages the way the resources are shared, but also decides when the resources are actually loaded and unloaded from memory.

## Build content using Addressables

- When you make a build while using the Addressables system, you generate two sets of files:
    - A player build, which is similar to the application file you would build without using the Addressables system, except that it also contains your local Addressables content.
    - Addressables content, which includes AssetBundles, runtime settings, a content catalog, and other files that will either be stored locally, with the player build, or uploaded to your content delivery network (CDN) or other hosting services.

- When you build your full application, the default workflow is to build your Addressables content first and then make a player build, but you have some control over this workflow through build scripts.

- You also have some control over the way your project compiles for the Unity Editor's Play mode. Using a Play Mode script, you can either compile your project normally or test your addressable assets, and you can save time by only building Addressables content for Play mode when you need to.

### Play Mode script

- When you're developing your game, you enter Play mode many times to test your changes. The Addressables system builds and uses AssetBundles, which can take time to build, especially in a large project with a lot of addressable assets. This could slow down Play mode iteration and burden your development process.

- A Play Mode script is an Editor-only feature of the Addressables system that allows you to control how the system will deliver assets during Play mode. You can use a Play Mode script to bypass Addressable content builds, allowing you to iterate your work at the quick pace that Play mode allows.

- You'll mainly be using these Play Mode scripts:
    - Use Asset Database (formerly known as Fast Mode): this option allows you to use Play mode without building Addressables content. It mocks up built Addressables content by locating and loading assets through the Editor asset database.
    - Use Existing Build: loads assets from AssetBundles created by an earlier build. Before you use this option, you must run a full build to create a set of Addressables content.

- By default, the Play Mode script is set to Use Asset Database for the convenience of development iteration. 

### Default build script

- When you build an application that uses the Addressables system, the general workflow is to build Addressables content as a separate step first before building the Unity Player application so that the application can use the AssetBundles. The Addressables system provides its own standard build scripts for building content, but you can also write your own build script if you need to customize your content build pipeline.

- When you first introduce the Addressables system into your project, it is configured to use the Default Build script. In addition to building the content, the location for where the content files are built depends on several factors such as whether the addressable assets are configured to be local or remote.

- Note: In most cases when you modify a setting (for example, Profile, Groups, the Addressables system settings) and you use the Use Existing Build option, you need to build your Addressables content to apply the changes. Otherwise, you will still see the outdated content in your build.

- You can find the Default Build script in Packages > Addressables > Editor > Build > DataBuilders > BuildScriptPackedMode.cs.

### Play Mode script build cache

- The build cache is the local set of Addressables content that's used when you select Use Existing Build.
- Select: Build > Clear Build Cache > Content Builders > Default Build Script.
- You'll see an error message in the Console window again because you destroyed the built content necessary to use an existing build.

### Scriptable Build Pipeline cache

- The Addressables system uses the Scriptable Build Pipeline (SBP) to build AssetBundles. The SBP is more robust than the legacy AssetBundle build pipeline.

- When you build your content, the SBP creates .info files that speed up subsequent builds. The SBP reads data from these .info files instead of regenerating data that hasn't changed. These files are cached in the Library/BuildCache folder of your project.

- To clear the SBP's build cache, follow these instructions:
    1. On Windows from the main menu, select Edit > Preferences > Scriptable Build Pipeline > Build Cache, and on Mac selecting from the main toolbar Unity > Settings which opens Preferences > Scriptable Build Pipeline > Build Cache.
    2. Make note of the current cache size.
    3. Select Purge Cache.
    4. In the Purge Build Cache dialog pops up, select Yes.

- Note: You can also clear the cache from the Addressables Groups window. First, make sure that you are using the existing build (Play Mode Script dropdown). Then, from the toolbar of the Addressables Groups window, select Build > Clear Build Cache > All from the dropdown.