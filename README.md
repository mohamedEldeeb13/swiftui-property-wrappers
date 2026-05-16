# SwiftUI Property Wrappers Explained

A practical guide to `@State`, `@Binding`, `@StateObject`, `@ObservedObject`, `@Environment`, `@EnvironmentObject`, and `@AppStorage`.

<details>
<summary>بالعربي</summary>

شرح عملي ومبسط لأهم Property Wrappers في SwiftUI مع أمثلة وتشبيهات تساعدك تختار النوع الصح في كل حالة.

</details>

> **Tip:** Every section below has a collapsible **بالعربي** block — click to expand the Arabic summary.

---

## Table of Contents

1. [What is a Property Wrapper?](#what-is-a-property-wrapper)
2. [Why SwiftUI Needs Property Wrappers](#why-swiftui-needs-property-wrappers)
3. [Important: Does `@State` Convert Value Types into Reference Types?](#important-does-state-convert-value-types-into-reference-types)
4. [The Main Question: Who Owns the Data?](#the-main-question-who-owns-the-data)
5. [`@State`](#state)
6. [`@Binding`](#binding)
7. [`@StateObject`](#stateobject)
8. [`ObservableObject` and `@Published`](#observableobject-and-published)
9. [`@ObservedObject`](#observedobject)
10. [`@Environment`](#environment)
11. [`@EnvironmentObject`](#environmentobject)
12. [`@AppStorage`](#appstorage)
13. [Apartment Analogy](#apartment-analogy)
14. [Quick Decision Guide](#quick-decision-guide)
15. [Summary Table](#summary-table)
16. [Common Mistakes](#common-mistakes)
17. [Recommended Usage in Real Projects](#recommended-usage-in-real-projects)
18. [Final Conclusion](#final-conclusion)

---

## What is a Property Wrapper?

A **Property Wrapper** is a Swift feature that adds extra behavior to a property.

Instead of making a property just store a value, a property wrapper can manage how the value is stored, read, updated, observed, or connected to another source.

Example:

```swift
@State private var count = 0
```

Here, `count` looks like a normal `Int`, but `@State` adds extra behavior:

- SwiftUI stores the value in a managed storage.
- SwiftUI observes changes to the value.
- When the value changes, SwiftUI recalculates the view body.
- You can create a binding using `$count`.

<details>
<summary>بالعربي</summary>

Property Wrapper يعني إن المتغير مش مجرد `var` عادي، لكن Swift بتضيف حواليه behavior معين.

مثال: `@State` بتخلي SwiftUI تراقب المتغير وتعمل update للـ UI لما قيمته تتغير.

</details>

---

## Why SwiftUI Needs Property Wrappers

SwiftUI views are usually `structs`.

```swift
struct CounterView: View {
    var body: some View {
        Text("Hello")
    }
}
```

A `struct` is a value type. SwiftUI can recreate the view many times while updating the UI.

SwiftUI does not work like UIKit.

In UIKit, you usually update the UI manually:

```swift
label.text = "New value"
```

In SwiftUI, the UI is a result of the current state:

```swift
Text(name)
```

The flow is:

```text
State changes → body recalculates → UI updates
```

So SwiftUI needs a way to know:

- Which values are important for the UI?
- Which values should be stored outside the temporary `View struct`?
- Which changes should trigger a UI update?

That is why SwiftUI uses property wrappers like:

```swift
@State
@Binding
@StateObject
@ObservedObject
@Environment
@EnvironmentObject
@AppStorage
```

<details>
<summary>بالعربي</summary>

SwiftUI مش بتخليك تغير الـ UI يدوي زي UIKit.

أنت بتغير الداتا، وSwiftUI تعيد بناء الـ View بناءً على الداتا الجديدة.

</details>

---

## Important: Does `@State` Convert Value Types into Reference Types?

No.

This is a very important point.

When you write:

```swift
@State private var count = 0
```

`count` is still an `Int`.

And `Int` is still a value type.

`@State` does **not** convert `Int`, `String`, `Bool`, or any value type into a reference type.

What actually happens is that SwiftUI manages the storage of this value outside the normal lifetime of the `View struct`.

So the correct understanding is:

```text
@State does not change the data type.
@State changes how the value is stored and observed by SwiftUI.
```

Example:

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

When `count` changes:

1. SwiftUI updates the state value.
2. SwiftUI knows that this view depends on that value.
3. SwiftUI recalculates the body.
4. The UI shows the new count.

<details>
<summary>بالعربي</summary>

`@State` مش بتحول الـ `Int` أو `String` إلى reference type.

هي فقط بتخلي SwiftUI تدير تخزين القيمة وتعمل refresh للـ View لما القيمة تتغير.

</details>

---

## The Main Question: Who Owns the Data?

The easiest way to choose the correct SwiftUI property wrapper is to ask:

```text
Who owns this data?
```

### Simple mental model

```text
This View owns a simple value
→ @State

Parent owns the value, child edits it
→ @Binding

This View creates and owns an object
→ @StateObject

Parent passes an object to this View
→ @ObservedObject

The value comes from SwiftUI environment
→ @Environment

A shared object comes from environment
→ @EnvironmentObject

A simple value is persisted in UserDefaults
→ @AppStorage
```

<details>
<summary>بالعربي</summary>

أهم سؤال:

مين مالك الداتا؟

الإجابة على السؤال ده غالبًا هتحدد الـ wrapper الصح.

</details>

---

## `@State`

Use `@State` when the value is local to the current view.

The view owns this value.

It is commonly used with simple value types like:

```swift
Bool
String
Int
Double
Date
Array
Struct
```

Example:

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        VStack(spacing: 16) {
            Text("Count: \(count)")

            Button("Increase") {
                count += 1
            }
        }
    }
}
```

Use `@State` for local UI state, such as:

```text
show password
selected tab
search text
is sheet presented
is alert shown
selected date
local form input
```

Example:

```swift
struct LoginView: View {
    @State private var phone = ""
    @State private var password = ""
    @State private var isPasswordVisible = false

    var body: some View {
        VStack {
            TextField("Phone", text: $phone)

            Button(isPasswordVisible ? "Hide Password" : "Show Password") {
                isPasswordVisible.toggle()
            }
        }
    }
}
```

### Why `private`?

Usually, `@State` should be private because the state is owned by the view itself.

```swift
@State private var isLoading = false
```

If another view needs to modify this value, you usually pass it as a `@Binding`.

<details>
<summary>بالعربي</summary>

استخدم `@State` لما القيمة تخص نفس الشاشة فقط.

مثال:

- إظهار أو إخفاء password
- فتح sheet
- search text
- selected tab

والقاعدة المهمة:

`@State` معناها إن الـ View دي هي المالكة للقيمة.

</details>

---

## `@Binding`

Use `@Binding` when the value is owned by a parent view, but the child view needs to read and modify it.

The child does not own the value.

It only has a connection to it.

Example:

```swift
struct ParentView: View {
    @State private var isOn = false

    var body: some View {
        VStack {
            Text(isOn ? "Enabled" : "Disabled")

            ToggleChildView(isOn: $isOn)
        }
    }
}
```

Child view:

```swift
struct ToggleChildView: View {
    @Binding var isOn: Bool

    var body: some View {
        Toggle("Status", isOn: $isOn)
    }
}
```

Here:

```swift
@State private var isOn = false
```

The value is owned by `ParentView`.

And:

```swift
@Binding var isOn: Bool
```

The child receives a binding to that value.

### What does `$` mean?

```swift
isOn
```

This is the value itself:

```swift
Bool
```

But:

```swift
$isOn
```

This is a binding:

```swift
Binding<Bool>
```

So:

```swift
Text(isOn ? "On" : "Off")
```

needs the value.

But:

```swift
Toggle("Status", isOn: $isOn)
```

needs a binding because the toggle must read and write the value.

### Manually creating a `Binding`

Sometimes the source value is not already a binding, or you need to transform it before exposing it to a child view. You can build a `Binding` by hand using `get` and `set`:

```swift
Binding(
    get: { someValue },
    set: { newValue in someValue = newValue }
)
```

Useful when:

```text
You want to derive a binding from a computed value
You want to bridge a non-binding source (like a manager class) into a SwiftUI control
You want to apply validation or transformation when the value changes
```

<details>
<summary>بالعربي</summary>

ممكن تعمل `Binding` يدوي بـ `get` و `set` لما القيمة مش جاهزة كـ Binding أو محتاج تعدل عليها قبل ما تمررها للـ child.

</details>

<details>
<summary>بالعربي</summary>

`@Binding` يعني إن القيمة مش مملوكة للـ child.

القيمة الأصلية موجودة في parent، والـ child واخد وصلة عليها يقدر يقرأ ويعدل.

`isOn` يعني القيمة نفسها.

`$isOn` يعني Binding على القيمة.

</details>

---

## `@StateObject`

Use `@StateObject` when the current view creates and owns an observable object.

This is commonly used for screen ViewModels.

Example:

```swift
final class LoginViewModel: ObservableObject {
    @Published var phone = ""
    @Published var isLoading = false
}
```

View:

```swift
struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()

    var body: some View {
        VStack {
            TextField("Phone", text: $viewModel.phone)

            if viewModel.isLoading {
                ProgressView()
            }
        }
    }
}
```

### What does `@StateObject` do?

`@StateObject` tells SwiftUI:

```text
This View owns this object.
Create it once for this View identity.
Keep the same instance even if body is recalculated.
Observe it for changes.
```

This means when the `body` refreshes, SwiftUI will not recreate the object.

```swift
@StateObject private var viewModel = LoginViewModel()
```

The `LoginViewModel` instance is created once for the same view identity.

### Does `@StateObject` make the object a reference type?

No.

The object is a reference type because it is a `class`.

```swift
final class LoginViewModel: ObservableObject
```

`@StateObject` does not convert it into a reference type.

It only tells SwiftUI to own, preserve, and observe the object.

### Tip: annotate ViewModels with `@MainActor`

When a ViewModel does async work and updates `@Published` properties, mark it with `@MainActor` so all updates run on the main thread:

```swift
@MainActor
final class LoginViewModel: ObservableObject {
    @Published var phone = ""
}
```

This avoids "Publishing changes from background threads is not allowed" warnings and keeps the UI updates safe.

<details>
<summary>بالعربي</summary>

لو الـ ViewModel فيه async، استخدم `@MainActor` عشان تضمن إن تحديث الـ `@Published` يحصل دايمًا على الـ main thread وما يجيش تحذير من SwiftUI.

</details>

<details>
<summary>بالعربي</summary>

استخدم `@StateObject` لما الـ View هي اللي بتعمل create للـ ViewModel.

مثال:

`LoginView` تعمل create لـ `LoginViewModel`.

هنا الأفضل:

```swift
@StateObject private var viewModel = LoginViewModel()
```

مش `@ObservedObject`.

</details>

---

## `ObservableObject` and `@Published`

For SwiftUI to observe an object, the object usually conforms to `ObservableObject`.

Example:

```swift
final class LoginViewModel: ObservableObject {
    @Published var phone = ""
}
```

### What does `ObservableObject` mean?

It means this object can notify SwiftUI when something important changes.

SwiftUI does not continuously check every property inside your ViewModel.

Instead, it waits for change notifications.

That is why a normal class is not enough.

The ViewModel must conform to `ObservableObject`, and the properties that should trigger UI updates should usually be marked with `@Published`.

### What does `@Published` mean?

It means this property will send change notifications when it changes.

Example:

```swift
viewModel.phone = "01012345678"
```

Flow:

```text
phone changed
→ @Published sends a change notification
→ ObservableObject notifies SwiftUI
→ SwiftUI recalculates the body
→ UI updates
```

### What if the class does not conform to `ObservableObject`?

Example:

```swift
final class LoginViewModel {
    var phone = ""
}
```

And in the view:

```swift
struct LoginView: View {
    private var viewModel = LoginViewModel()

    var body: some View {
        VStack {
            Text(viewModel.phone)

            Button("Change") {
                viewModel.phone = "010"
            }
        }
    }
}
```

The value of `phone` will actually change inside the class instance, but the UI will not update automatically.

Why?

Because SwiftUI needs someone to notify it that a value inside the ViewModel has changed, so it can recalculate the `body` and rebuild the affected views.

A normal `class` is only a reference type.

It does not automatically notify SwiftUI about changes.

This is the job of `ObservableObject`.

Correct:

```swift
final class LoginViewModel: ObservableObject {
    @Published var phone = ""
}
```

And in the View:

```swift
struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()

    var body: some View {
        VStack {
            Text(viewModel.phone)

            Button("Change") {
                viewModel.phone = "010"
            }
        }
    }
}
```

Now when `phone` changes:

```text
phone changed
→ @Published sends a change notification
→ ObservableObject notifies SwiftUI
→ SwiftUI recalculates body
→ The UI updates
```

So the correct mental model is:

```text
class only
= reference type, but no UI notification

ObservableObject
= allows SwiftUI to observe the object

@Published
= automatically sends a notification when this property changes

@StateObject / @ObservedObject
= connects the object to the SwiftUI View lifecycle
```

<details>
<summary>بالعربي</summary>

`class` لوحدها معناها إن الـ ViewModel reference type، لكن ده مش كفاية عشان SwiftUI تعمل update.

SwiftUI محتاجة إشعار إن في property اتغيرت جوه الـ ViewModel.

`ObservableObject` هو اللي بيخلي الـ object قابل للمراقبة من SwiftUI.

و `@Published` هي اللي بتبعت الإشعار تلقائيًا لما القيمة تتغير.

يعني:

```text
class لوحدها = reference فقط
ObservableObject = SwiftUI تقدر تراقبه
@Published = property تغييرها يبعت notification
```

</details>

---

## `@ObservedObject`

Use `@ObservedObject` when the object is created and owned somewhere else, usually by a parent view.

The current view does not own the object.

It only observes it.

Example:

```swift
final class LoginViewModel: ObservableObject {
    @Published var phone = ""
}
```

Parent:

```swift
struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()

    var body: some View {
        LoginFormView(viewModel: viewModel)
    }
}
```

Child:

```swift
struct LoginFormView: View {
    @ObservedObject var viewModel: LoginViewModel

    var body: some View {
        TextField("Phone", text: $viewModel.phone)
    }
}
```

Here:

```swift
@StateObject
```

is used in the owner view.

And:

```swift
@ObservedObject
```

is used in the child view that receives the object.

### Rule

```text
View creates the object
→ @StateObject

View receives the object
→ @ObservedObject
```

<details>
<summary>بالعربي</summary>

لو الـ ViewModel اتعمل جوه نفس الشاشة، استخدم `@StateObject`.

لو الـ ViewModel جاي من parent، استخدم `@ObservedObject`.

</details>

---

## `@Environment`

Use `@Environment` to read values from SwiftUI environment.

These values are usually provided by SwiftUI or by a parent view.

Common examples:

```swift
@Environment(\.dismiss) private var dismiss
@Environment(\.colorScheme) private var colorScheme
@Environment(\.layoutDirection) private var layoutDirection
@Environment(\.locale) private var locale
@Environment(\.openURL) private var openURL
@Environment(\.isEnabled) private var isEnabled
```

Example:

```swift
struct DetailsView: View {
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        Button("Close") {
            dismiss()
        }
    }
}
```

Example with color scheme:

```swift
struct CardView: View {
    @Environment(\.colorScheme) private var colorScheme

    var body: some View {
        Text("Hello")
            .padding()
            .background(colorScheme == .dark ? Color.black : Color.white)
    }
}
```

### When should you use `@Environment`?

Use it when the value is:

```text
A system/context value
Provided by SwiftUI or parent
Not owned by this view
Usually read-only or action-like
```

Examples:

```text
dismiss
color scheme
layout direction
locale
open URL
enabled/disabled state
```

<details>
<summary>بالعربي</summary>

`@Environment` للقيم العامة اللي SwiftUI أو parent حاططها في البيئة.

زي:

- أقفل الشاشة
- أعرف dark mode ولا light mode
- أعرف اتجاه اللغة RTL/LTR
- أفتح URL

</details>

---

## `@EnvironmentObject`

Use `@EnvironmentObject` when you have a shared `ObservableObject` injected into the view hierarchy.

It is useful when many views need access to the same object without passing it manually through every initializer.

Common examples:

```text
UserSession
AuthState
AppCoordinator
AppSettings
CartManager
LanguageManager
ThemeManager
```

Example:

```swift
final class UserSession: ObservableObject {
    @Published var isLoggedIn = false
    @Published var userName = ""

    func logout() {
        isLoggedIn = false
        userName = ""
    }
}
```

Inject it at a high level:

```swift
@main
struct MyApp: App {
    @StateObject private var session = UserSession()

    var body: some Scene {
        WindowGroup {
            RootView()
                .environmentObject(session)
        }
    }
}
```

Use it in any child view:

```swift
struct ProfileView: View {
    @EnvironmentObject private var session: UserSession

    var body: some View {
        VStack {
            Text(session.userName)

            Button("Logout") {
                session.logout()
            }
        }
    }
}
```

### Important

If you use:

```swift
@EnvironmentObject private var session: UserSession
```

then some parent view must inject it:

```swift
.environmentObject(session)
```

Otherwise, the app will crash at runtime.

### When should you use `@EnvironmentObject`?

Use it when the object is:

```text
Shared across many screens
Needed deeply in the view hierarchy
Owned by App or parent flow
ObservableObject
Long-lived
```

Good examples:

```swift
@EnvironmentObject var session: UserSession
@EnvironmentObject var appCoordinator: AppCoordinator
@EnvironmentObject var cartManager: CartManager
```

### When should you avoid it?

Do not use `@EnvironmentObject` just to avoid passing data.

Avoid this:

```swift
@EnvironmentObject var loginViewModel: LoginViewModel
```

if `LoginViewModel` is only used by the login screen.

Better:

```swift
@StateObject private var viewModel = LoginViewModel()
```

`@EnvironmentObject` creates a hidden dependency. The view initializer does not show that it needs the object.

So use it carefully for truly shared app-level or flow-level objects.

<details>
<summary>بالعربي</summary>

`@EnvironmentObject` مناسب للـ objects المشتركة على مستوى كبير.

زي:

- UserSession
- AppCoordinator
- CartManager
- AppSettings

لكنه مش مناسب لأي ViewModel خاص بشاشة واحدة.

</details>

---

## `@AppStorage`

`@AppStorage` is a SwiftUI property wrapper built on top of `UserDefaults`.

It lets you read and write simple persistent values.

Example:

```swift
@AppStorage("hasSeenOnboarding") private var hasSeenOnboarding = false
```

This means:

```text
Read the value from UserDefaults using this key.
If no value exists, use the default value.
When the value changes, save it to UserDefaults.
When the value changes, refresh the SwiftUI view.
```

Example:

```swift
struct RootView: View {
    @AppStorage("hasSeenOnboarding") private var hasSeenOnboarding = false

    var body: some View {
        if hasSeenOnboarding {
            LoginView()
        } else {
            OnboardingView()
        }
    }
}
```

Inside onboarding:

```swift
struct OnboardingView: View {
    @AppStorage("hasSeenOnboarding") private var hasSeenOnboarding = false

    var body: some View {
        Button("Get Started") {
            hasSeenOnboarding = true
        }
    }
}
```

When the button is tapped:

```text
hasSeenOnboarding becomes true
→ value is saved in UserDefaults
→ RootView updates
→ LoginView is shown
```

### Supported types

`@AppStorage` does not support arbitrary types. It only works with:

```text
Bool
Int
Double
String
URL
Data
RawRepresentable enums whose RawValue is one of the above
```

For a custom enum, conform it to `RawRepresentable`:

```swift
enum AppTheme: String {
    case light
    case dark
    case system
}

@AppStorage("appTheme") private var appTheme: AppTheme = .system
```

If you need to persist a struct or array, encode it to `Data` with `JSONEncoder` and store the `Data` — or use a different mechanism (file, Core Data, SwiftData).

<details>
<summary>بالعربي</summary>

`@AppStorage` بيدعم أنواع محددة فقط: `Bool`, `Int`, `Double`, `String`, `URL`, `Data`، و enums من نوع `RawRepresentable`.

لو محتاج تخزن struct أو array، حوّله لـ `Data` بـ `JSONEncoder` الأول.

</details>

### Good use cases

Use `@AppStorage` for simple, non-sensitive values:

```text
hasSeenOnboarding
selectedLanguage
selectedTheme
isBiometricEnabled
lastSelectedTab
isFirstTime
```

Example:

```swift
@AppStorage("selectedLanguage") private var selectedLanguage = "en"
@AppStorage("isBiometricEnabled") private var isBiometricEnabled = false
@AppStorage("lastSelectedTab") private var lastSelectedTab = 0
```

### Do not use `@AppStorage` for sensitive data

Avoid:

```swift
@AppStorage("accessToken") private var accessToken = ""
```

Use Keychain for:

```text
access token
refresh token
password
sensitive user data
```

### Difference between `@AppStorage` and custom UserDefaults wrappers

`@AppStorage`:

```text
UserDefaults + SwiftUI refresh
```

Custom UserDefaults wrappers:

```text
Organized access to UserDefaults
Useful in ViewModels, UIKit, Services, AppDelegate, Codable models
Not automatically reactive in SwiftUI unless you add observation manually
```

<details>
<summary>بالعربي</summary>

`@AppStorage` هو wrapper من SwiftUI فوق `UserDefaults`.

يعني:

```text
@AppStorage = UserDefaults + SwiftUI refresh
```

استخدمه مع القيم البسيطة وغير الحساسة.

أما token أو password أو أي بيانات حساسة، استخدم Keychain.

</details>

---

## Apartment Analogy

A simple analogy to remember the differences.

### `@State`

Something inside your own room.

You own it, and it belongs only to your room.

```swift
@State private var isPasswordVisible = false
```

### `@Binding`

A switch in your room controlling a light owned by another room.

You do not own the original value, but you can change it.

```swift
@Binding var isOn: Bool
```

### `@StateObject`

A device you bought and own in your apartment.

You created it, and you are responsible for its lifetime.

```swift
@StateObject private var viewModel = LoginViewModel()
```

### `@ObservedObject`

A device owned by someone else, but you can watch and use it.

```swift
@ObservedObject var viewModel: LoginViewModel
```

### `@Environment`

Building-wide rules or services.

For example:

```text
theme
language direction
dismiss action
open URL
```

```swift
@Environment(\.dismiss) private var dismiss
```

### `@EnvironmentObject`

A shared building system.

For example:

```text
internet
security system
central coordinator
shared session
```

```swift
@EnvironmentObject var session: UserSession
```

### `@AppStorage`

A notebook where you save simple preferences so they are still there tomorrow.

```swift
@AppStorage("hasSeenOnboarding") private var hasSeenOnboarding = false
```

<details>
<summary>بالعربي المختصر</summary>

```text
@State = حاجة ملك الشاشة نفسها
@Binding = وصلة لقيمة مملوكة للـ parent
@StateObject = object الشاشة هي اللي عملته وتملكه
@ObservedObject = object جاي من بره
@Environment = قيمة عامة من النظام أو parent
@EnvironmentObject = object مشترك على مستوى كبير
@AppStorage = قيمة بسيطة محفوظة في UserDefaults
```

</details>

---

## Quick Decision Guide

Use this guide when choosing a wrapper.

### Use `@State` when:

```text
The value is local to this View
The value is simple
The View owns it
```

Example:

```swift
@State private var searchText = ""
```

---

### Use `@Binding` when:

```text
Parent owns the value
Child needs to read and write it
```

Example:

```swift
@Binding var isSelected: Bool
```

---

### Use `@StateObject` when:

```text
This View creates the ObservableObject
The object should survive body refreshes
The View owns the object
```

Example:

```swift
@StateObject private var viewModel = LoginViewModel()
```

---

### Use `@ObservedObject` when:

```text
The object is passed from parent
This View does not own it
This View only observes it
```

Example:

```swift
@ObservedObject var viewModel: LoginViewModel
```

---

### Use `@Environment` when:

```text
The value comes from SwiftUI environment
The value is system/context-based
```

Example:

```swift
@Environment(\.dismiss) private var dismiss
```

---

### Use `@EnvironmentObject` when:

```text
The object is shared across many screens
The object is injected from a parent/root
The object is ObservableObject
```

Example:

```swift
@EnvironmentObject var session: UserSession
```

---

### Use `@AppStorage` when:

```text
The value is simple
The value should be saved in UserDefaults
The UI should update when it changes
```

Example:

```swift
@AppStorage("isFirstTime") private var isFirstTime = true
```

---

## Summary Table

| Wrapper | Type | Owner | Best For |
|---|---|---|---|
| `@State` | Value | Current View | Local UI state |
| `@Binding` | Binding to value | Parent View | Child editing parent state |
| `@StateObject` | `ObservableObject` | Current View | Screen ViewModel |
| `@ObservedObject` | `ObservableObject` | Parent View | Child using parent ViewModel |
| `@Environment` | Environment value | SwiftUI / Parent | `dismiss`, `colorScheme`, `locale` |
| `@EnvironmentObject` | Shared `ObservableObject` | Parent / App | `session`, `coordinator`, `settings` |
| `@AppStorage` | UserDefaults value | UserDefaults | Simple persistent preferences |

---

## Common Mistakes

### Mistake 1: Using a normal class without `ObservableObject`

Wrong:

```swift
final class LoginViewModel {
    var phone = ""
}
```

Correct:

```swift
final class LoginViewModel: ObservableObject {
    @Published var phone = ""
}
```

A plain `class` is a reference type but it does not notify SwiftUI when a property changes, so the UI will not update. See [`ObservableObject` and `@Published`](#observableobject-and-published) for the full explanation.

---

### Mistake 2: Using `@ObservedObject` when the View creates the object

Wrong:

```swift
struct LoginView: View {
    @ObservedObject var viewModel = LoginViewModel()

    var body: some View {
        Text(viewModel.phone)
    }
}
```

Correct:

```swift
struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()

    var body: some View {
        Text(viewModel.phone)
    }
}
```

Why?

```text
If the View creates and owns the object, use @StateObject.
```

---

### Mistake 3: Using `@EnvironmentObject` for a screen-specific ViewModel

Wrong:

```swift
struct LoginView: View {
    @EnvironmentObject var viewModel: LoginViewModel
}
```

Better:

```swift
struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()
}
```

Use `@EnvironmentObject` only when the object is shared across many screens.

---

### Mistake 4: Storing tokens in `@AppStorage`

Wrong:

```swift
@AppStorage("accessToken") private var accessToken = ""
```

Better:

```text
Use Keychain for sensitive data.
```

Use `@AppStorage` for simple non-sensitive values only.

---

### Mistake 5: Thinking `@State` makes value types reference types

Wrong idea:

```text
@State makes Int or String reference type.
```

Correct idea:

```text
@State keeps the value type as it is.
It only moves storage management to SwiftUI.
```

---

### Mistake 6: Passing everything with `@Binding`

Do not overuse `@Binding`.

Use `@Binding` only when the child really needs to mutate the parent state.

If the child only needs to display data, pass a normal value:

```swift
struct UserNameView: View {
    let name: String
}
```

Not:

```swift
struct UserNameView: View {
    @Binding var name: String
}
```

---

### Mistake 7: Using `@EnvironmentObject` to avoid clean dependency passing

`@EnvironmentObject` is powerful, but it hides dependencies.

This:

```swift
ProfileView()
```

does not show that `ProfileView` needs:

```swift
@EnvironmentObject var session: UserSession
```

So use it only for clear shared dependencies like:

```text
AppSession
AppCoordinator
AppSettings
CartManager
```

---

## Recommended Usage in Real Projects

A practical setup for real SwiftUI projects:

```text
Screen ViewModel
→ @StateObject

Child View using the same ViewModel
→ @ObservedObject

Reusable component editing parent value
→ @Binding

Local UI flag
→ @State

App coordinator/session/settings
→ @EnvironmentObject

Dismiss/theme/language/layout direction
→ @Environment

Simple persisted flag
→ @AppStorage

Token/sensitive data
→ Keychain
```

Example:

```swift
final class AppCoordinator: ObservableObject {
    @Published var route: AppRoute = .splash
}

final class LoginViewModel: ObservableObject {
    @Published var phone = ""
    @Published var password = ""
    @Published var isLoading = false
}
```

```swift
struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()
    @EnvironmentObject private var appCoordinator: AppCoordinator
    @AppStorage("hasSeenOnboarding") private var hasSeenOnboarding = false
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        LoginContentView(viewModel: viewModel)
    }
}
```

Child view:

```swift
struct LoginContentView: View {
    @ObservedObject var viewModel: LoginViewModel

    var body: some View {
        VStack {
            TextField("Phone", text: $viewModel.phone)
            SecureField("Password", text: $viewModel.password)
        }
    }
}
```

Reusable component:

```swift
struct PasswordVisibilityButton: View {
    @Binding var isVisible: Bool

    var body: some View {
        Button(isVisible ? "Hide" : "Show") {
            isVisible.toggle()
        }
    }
}
```

---

## Final Conclusion

The easiest way to choose the correct property wrapper is to ask four questions:

```text
1. Is it a value or an object?
2. Who owns it?
3. Should it persist after closing the app?
4. Should SwiftUI refresh when it changes?
```

### Final rules

```text
Local value owned by this View
→ @State

Value owned by parent and edited by child
→ @Binding

Object created and owned by this View
→ @StateObject

Object passed from parent
→ @ObservedObject

System or context value
→ @Environment

Shared app/flow object
→ @EnvironmentObject

Simple persistent UserDefaults value
→ @AppStorage
```

<details>
<summary>بالعربي</summary>

اختيار الـ wrapper الصح يبدأ من سؤال واحد:

```text
مين مالك الداتا؟
```

وبعدها اسأل:

```text
هل الداتا value ولا object؟
هل محتاجاها تفضل بعد قفل التطبيق؟
هل SwiftUI لازم تعمل refresh لما تتغير؟
```

</details>

---

## Author Note

This README is designed as a practical learning reference for SwiftUI state management.

It focuses on real-world usage, common mistakes, and choosing the right wrapper based on ownership and lifecycle.

**Intended audience:** iOS developers learning or refreshing their understanding of SwiftUI state.

**What this guide does *not* cover (yet):**

- The iOS 17+ **Observation framework** (`@Observable`, `@Bindable`, and the new `@Environment(MyType.self)` syntax) — coming in a separate article.
- Adjacent property wrappers like `@FocusState`, `@SceneStorage`, `@GestureState`, `@Namespace`, `@Query` (SwiftData), and `@FetchRequest` (Core Data).

<details>
<summary>بالعربي</summary>

الـ README ده مرجع عملي لـ state management في SwiftUI، ومناسب لأي iOS developer.

الإصدار الحالي مش بيغطي الـ Observation framework الجديد (`@Observable` و `@Bindable`) — هيكون في مقال منفصل.

</details>
