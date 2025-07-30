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
