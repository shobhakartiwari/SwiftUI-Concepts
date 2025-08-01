# SwiftUI-Concepts 
- Great content from iOS Devel https://www.linkedin.com/in/vitoria-pinheiro
# 1. SwiftUI: `init()` vs `onAppear()` — What’s the Difference?

In SwiftUI, both `init()` and `onAppear()` are used to run code at the start of a view’s lifecycle—but they behave **very differently**.

Using the wrong one can lead to **unexpected bugs**, **unnecessary performance hits**, or even **duplicate API calls**. Let’s break it down.

---

## 📦 `init()` in SwiftUI

- Called **immediately** when the view struct is created—even if it’s **never rendered**.
- SwiftUI can **recreate views frequently** as part of its rendering engine.
- Best used for **lightweight logic**: initializing constants, bindings, or default state.

### ⚠️ Caution:
> Avoid heavy or async work in `init()`—it may run **multiple times** and **before the view appears onscreen**.

---

## 🎬 `onAppear()`

- Called **only when the view actually appears** on screen.
- Ideal for:
  - Analytics or tracking
  - Network calls / async work
  - Animations
  - Logging
- Runs **once per appearance**, unless the view is removed and inserted again.

---

## 👇 Example Comparison

```swift
import SwiftUI

struct DetailView: View {
    let id: Int

    init(id: Int) {
        self.id = id
        print("init called") // May be called even if the view never appears
    }

    var body: some View {
        Text("Detail for item \(id)")
            .onAppear {
                print("View appeared") // Called when user actually sees the view
            }
    }
}

```

# 2. SwiftUI: `@StateObject` vs `@ObservedObject`

In SwiftUI, both `@StateObject` and `@ObservedObject` are used to monitor observable objects and update views when the data changes. But their **purpose and lifecycle** differ—and choosing the wrong one can cause subtle bugs.

---

## 🧱 `@StateObject`

Use this when **the view creates and owns the object**.

- SwiftUI takes ownership and ensures the object **persists across view re-renders**.
- Prevents the object from being reinitialized when the view reloads.
- Introduced in **SwiftUI 2.0** to fix bugs common with `@ObservedObject` when used incorrectly.

### ✅ Use Case:
```swift
struct ProfileView: View {
    @StateObject private var viewModel = ProfileViewModel()

    var body: some View {
        Text(viewModel.name)
    }
}
```

## 📨 @ObservedObject
- Use this when the object is passed in or owned elsewhere.
- The view does not own the object—it just observes changes.
- It can be recreated every time the parent view updates, which may reset state if misused.
```swift
struct ProfileDetailView: View {
    @ObservedObject var viewModel: ProfileViewModel

    var body: some View {
        Text(viewModel.details)
    }
}
```
- ✅ ProfileDetailView just observes the viewModel, so we use @ObservedObject.
- When to Use 
  - View **creates and owns** the object  | `@StateObject`    
  - View **receives and observes** object | `@ObservedObject` 

# 3. SwiftUI ObservableObject Change Announcement

In SwiftUI, when you want your view to automatically update in response to data changes, you use an `ObservableObject`—but how does it actually notify the UI?

There are two main ways to trigger updates:

## 1. `@Published` (Recommended for most cases)

Mark a property with `@Published`, and SwiftUI will automatically observe changes and refresh any view that depends on it.

Use this when:

- You want simple, property-level observation
- You don’t need custom logic before notifying the view

## 2. `objectWillChange.send()` (Manual control)

Call this method when you want full control over when the view updates, such as:

- Updating multiple values at once
- Wrapping non-`@Published` properties
- Handling changes from external sources (e.g. notifications, delegates)

SwiftUI doesn’t just observe data—it reacts to change. Knowing how to trigger that change is key to building reactive, declarative UIs.
```swift
import SwiftUI
import Combine

class ProfileViewModel: ObservableObject {
    @Published var name: String = "Vivi" // Auto-updates UI
    
    var age: Int = 28 {
        didSet {
            objectWillChange.send() // Manually trigger UI update
        }
    }
    
    func updateFromExternalSource(newName: String, newAge: Int) {
        objectWillChange.send() // Notify once for both changes
        name = newName
        age = newAge
    }
}
```

# 4. Programmatic Navigation in SwiftUI

While SwiftUI makes simple navigation clean and intuitive with `NavigationLink`, things get more complex when you need to navigate programmatically—that is, based on state, deep links, or external triggers.

This is essential when handling:

- Deep links from Spotlight, Siri, or widgets
- Navigation based on logic (e.g., login success, error redirects)
- Multi-screen flows that aren’t user-initiated taps

## Key Concept

Programmatic navigation in SwiftUI requires declaring all navigation paths up front, then using bindings or selection tags to trigger transitions.

## Two Common Patterns

1. **Boolean binding with `NavigationLink(isActive:)`**
2. **Tag + selection with `NavigationLink(tag:selection:)`** — useful for handling multiple navigation states

## Bonus: Handling Deep Links or External Events

Just update the `@State` or `@Binding` variable when the external trigger occurs.

## Tip

Be aware that navigation links must exist in the view hierarchy even if hidden, or the navigation won’t work properly.

SwiftUI’s programmatic navigation is all about state-driven flows—declarative and reactive, just like the rest of the framework.
```swift
struct ContentView: View {
    @State private var navigate = false

    var body: some View {
        NavigationView {
            VStack {
                Button("Go to Details") {
                    navigate = true
                }

                NavigationLink(destination: DetailView(), isActive: $navigate) {
                    EmptyView()
                }
            }
            .navigationTitle("Home")
        }
    }
}
```
# 5. Purpose of the ButtonStyle Protocol in SwiftUI

SwiftUI provides multiple built-in button styles like `.bordered`, `.plain`, `.link`, and `.automatic`—but sometimes, you need full control over how buttons look and behave.

That’s where the `ButtonStyle` protocol comes in.

## Why Use ButtonStyle

- Create reusable, consistent button designs across your app
- Fully customize visual appearance (e.g., color, shape, padding)
- Apply them globally or to individual buttons
- Adapt styling based on button state (e.g., pressed, hovered)

## How It Works

You define a struct that conforms to `ButtonStyle` and implement `makeBody(configuration:)`, which gives you access to:

- The button label
- The pressed state (`configuration.isPressed`)

## ButtonStyle vs. PrimitiveButtonStyle

### ButtonStyle
- Focuses on styling only, using the system’s built-in button behavior (e.g., tap handling, accessibility, animations).

### PrimitiveButtonStyle
- Gives lower-level control, allowing you to redefine the entire button interaction, not just the appearance. Use it when you want to fully customize how a button responds to events.

## TL;DR
Use `ButtonStyle` to define consistent design across your app. Reach for `PrimitiveButtonStyle` only when you need to customize how the button works, not just how it looks.
```swift
import SwiftUI

struct RoundedBlueButton: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
            .background(configuration.isPressed ? Color.blue.opacity(0.6) : Color.blue)
            .foregroundColor(.white)
            .clipShape(Capsule())
    }
}

struct ContentView: View {
    var body: some View {
        Button("Tap Me") {
            // action here
        }
        .buttonStyle(RoundedBlueButton())
    }
}
```

# 6. When to Use GeometryReader in SwiftUI

The simplest answer: `GeometryReader` lets you read the size and position of a view, enabling you to build layouts that adapt dynamically to the available space.

## Common Use Cases

- Creating proportional or responsive layouts
- Animating views based on their position on screen
- Applying modifiers conditionally based on view size or geometry
- Implementing parallax effects or scroll-based UI transformations

## How It Works

`GeometryReader` injects a `GeometryProxy` into your view, giving you access to:

- `size`: the width and height available to the view
- `safeAreaInsets`: safe area constraints (like notches)
- `frame(in:)`: the view’s position relative to a given coordinate space

## Caution

`GeometryReader` is powerful, but often over-used. It can break layout expectations if not carefully constrained and may lead to performance issues or unwanted complexity.

## Best Practice

Use `GeometryReader` when you truly need dynamic layout behavior based on geometry—not just to “get the width.”

Use `GeometryReader` wisely—it’s a powerful tool, not a layout shortcut.
```swift
import SwiftUI

struct ResponsiveBox: View {
    var body: some View {
        GeometryReader { geo in
            Rectangle()
                .fill(Color.blue)
                .frame(width: geo.size.width / 2, height: geo.size.width / 2)
        }
    }
}
```
# 7. Why SwiftUI Uses Structs for Views Instead of Classes

The short answer: structs are simpler and more efficient than classes.

## Deeper Insight

### Value Types = Predictable Behavior

SwiftUI views are defined as value types (structs) instead of reference types (classes). This means:

- Views are copied, not referenced, making them immutable by default
- You avoid common pitfalls like unexpected shared state or reference cycles
- Structs are lightweight and fast to create and destroy

### Why This Matters in SwiftUI

SwiftUI is a declarative UI framework, which means the system:

- Continuously re-evaluates and rebuilds views based on state
- Needs to do this frequently and efficiently

Using structs allows SwiftUI to:

- Create new view instances on demand without worrying about memory bloat
- Eliminate the need for manual lifecycle management
- Keep performance high, even during complex UI updates

## Pro Tip

Although views are recreated, SwiftUI maintains state using tools like `@State`, `@Binding`, and `@Environment`, which are stored outside the view struct itself.

## Bottom Line

SwiftUI uses structs for simplicity, safety, and performance—and it’s one of the key reasons the framework scales so well across platforms.

# 8.Understanding SwiftUI’s Environment for New Developers

In SwiftUI, the **environment** is like a shared container of values that any view in the view hierarchy can access. Think of it as a singleton-style dependency injector, but scoped to the view tree.

Instead of manually passing data through view initializers, you can inject values globally or locally into the environment using modifiers. These values can then be accessed using property wrappers like `@Environment` and `@EnvironmentObject`.

## Why It Matters
- **Decoupled and testable views**: Keeps your views independent from specific data sources.
- **Shared data access**: Easily share app settings, themes, locale, size classes, color schemes, and more.
- **Avoids prop drilling**: Eliminates the need to pass values through multiple layers of views.

## Key Concepts
- **`@Environment`**: Used for system-provided values, such as `colorScheme`, `sizeCategory`, or `locale`.
- **`@EnvironmentObject`**: Used for custom `ObservableObject` instances that are injected once and accessible anywhere in the view hierarchy below.

## Environment vs. Injecting via @ObservedObject
- **`@ObservedObject`**: Requires explicit passing through a view’s initializer, leading to tight coupling.
- **`@EnvironmentObject`**: Injected implicitly via the view hierarchy, enabling loose coupling. You don’t need to modify intermediate views to pass data down.

## Summary
SwiftUI’s environment helps you build modular, scalable, and reactive UIs. It promotes a clean architecture by separating the view structure from the data flow.
```swift
import SwiftUI

class Settings: ObservableObject {
    @Published var isDarkMode = false
}

struct RootView: View {
    @StateObject var settings = Settings()
    
    var body: some View {
        ContentView()
            .environmentObject(settings)
    }
}

struct ContentView: View {
    @EnvironmentObject var settings: Settings
    
    var body: some View {
        Toggle("Dark Mode", isOn: $settings.isDarkMode)
            .padding()
    }
}
```
# 9. SwiftUI ViewModel and ObservableObject Basics
- ObservedObject is used when we need to track a value from ViewModel, and the ViewModel is a class. To track that ViewModel in SwiftUI, I can create a variable in the view, and monitor changes so the view re-renders. For this, the ViewModel must conform to ObservableObject, and its properties must be marked with @Published.
- To make a ViewModel reactive in SwiftUI, it must conform to `ObservableObject`. Here's how it works:

- **Conforming to `ObservableObject`**: Signals to SwiftUI that the ViewModel may have properties that can change, without specifying which ones.
- **Using `@Published`**: Marks properties that should trigger a view re-render when their values change. SwiftUI automatically refreshes views that depend on these properties.
- **Using `@ObservedObject` in Views**: Tells the view to observe the ViewModel and update itself when changes occur.

# 10. What Does the @Published Property Wrapper Do in Swift?

`@Published` is used in classes conforming to the `ObservableObject` protocol to automatically notify SwiftUI views of changes, triggering reactive UI updates.

## How It Works

- Automatically triggers `objectWillChange` when the property’s value changes.
- Integrates with views observing via `@ObservedObject`, `@StateObject`, or `@EnvironmentObject`.
- Enables precise UI updates based on object state.

## Real-World Example: A Todo List Manager

## Summary

Use `@Published` to ensure SwiftUI is notified of data model changes. It’s the simplest way to bind observable data to the UI, keeping everything synchronized declaratively and reactively.
