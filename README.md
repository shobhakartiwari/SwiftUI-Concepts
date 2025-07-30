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
