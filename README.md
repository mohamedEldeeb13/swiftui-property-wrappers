# SwiftUI Property Wrappers Explained

A practical guide to `@State`, `@Binding`, `@StateObject`, `@ObservedObject`, `@Environment`, `@EnvironmentObject`, `@AppStorage`, and adjacent SwiftUI property wrappers like `@FocusState`, `@SceneStorage`, `@GestureState`, and `@Namespace` — plus the modern iOS 17+ **Observation framework** (`@Observable`).

<details>
<summary>بالعربي</summary>

<div dir="rtl">

شرح عملي ومبسط لأهم Property Wrappers في SwiftUI مع أمثلة وتشبيهات تساعدك تختار النوع الصح في كل حالة، بالإضافة إلى wrappers متخصصة زي `@FocusState`, `@SceneStorage`, `@GestureState`, و `@Namespace`، وكمان نظام الـ Observation الحديث في iOS 17+ (`@Observable`).

</div>

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
13. [The Modern Observation Framework (iOS 17+)](#the-modern-observation-framework-ios-17)
    - [The Old Way: `ObservableObject`](#the-old-way-observableobject)
    - [The New Way: `@Observable`](#the-new-way-observable)
    - [How Does the View Observe Changes Without `@Published`?](#how-does-the-view-observe-changes-without-published)
    - [Why `@State` Instead of `@StateObject`?](#why-state-instead-of-stateobject)
    - [`@State` with a Value Type vs a Reference Type](#state-with-a-value-type-vs-a-reference-type)
    - [Passing an `@Observable` ViewModel from a Parent](#passing-an-observable-viewmodel-from-a-parent)
    - [Computed Properties and Observation](#computed-properties-and-observation)
    - [`@MainActor` with `@Observable` ViewModels](#mainactor-with-observable-viewmodels)
    - [Old vs New Comparison](#old-vs-new-comparison)
    - [Is the Old Approach Deprecated?](#is-the-old-approach-deprecated)
14. [Adjacent SwiftUI Property Wrappers](#adjacent-swiftui-property-wrappers)
    - [`@FocusState`](#focusstate)
    - [`@SceneStorage`](#scenestorage)
    - [`@GestureState`](#gesturestate)
    - [`@Namespace`](#namespace)
    - [Adjacent Wrappers Summary Table](#adjacent-wrappers-summary-table)
    - [Adjacent Wrappers Mental Model](#adjacent-wrappers-mental-model)
15. [Apartment Analogy](#apartment-analogy)
16. [Quick Decision Guide](#quick-decision-guide)
17. [Summary Table](#summary-table)
18. [Common Mistakes](#common-mistakes)
19. [Recommended Usage in Real Projects](#recommended-usage-in-real-projects)
20. [Final Conclusion](#final-conclusion)

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

<div dir="rtl">

Property Wrapper يعني إن المتغير مش مجرد `var` عادي، لكن Swift بتضيف حواليه behavior معين.

مثال: `@State` بتخلي SwiftUI تراقب المتغير وتعمل update للـ UI لما قيمته تتغير.

</div>

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

<div dir="rtl">

SwiftUI مش بتخليك تغير الـ UI يدوي زي UIKit.

أنت بتغير الداتا، وSwiftUI تعيد بناء الـ View بناءً على الداتا الجديدة.

</div>

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

<div dir="rtl">

`@State` مش بتحول الـ `Int` أو `String` إلى reference type.

هي فقط بتخلي SwiftUI تدير تخزين القيمة وتعمل refresh للـ View لما القيمة تتغير.

</div>

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

Input focus or keyboard state
→ @FocusState

Temporary scene restoration state
→ @SceneStorage

Temporary gesture state
→ @GestureState

Shared animation identity
→ @Namespace
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

أهم سؤال:

مين مالك الداتا؟

الإجابة على السؤال ده غالبًا هتحدد الـ wrapper الصح.

</div>

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

<div dir="rtl">

استخدم `@State` لما القيمة تخص نفس الشاشة فقط.

مثال:

- إظهار أو إخفاء password
- فتح sheet
- search text
- selected tab

والقاعدة المهمة:

`@State` معناها إن الـ View دي هي المالكة للقيمة.

</div>

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

<div dir="rtl">

`@Binding` يعني إن القيمة مش مملوكة للـ child.

القيمة الأصلية موجودة في parent، والـ child واخد وصلة عليها يقدر يقرأ ويعدل.

`isOn` يعني القيمة نفسها.

`$isOn` يعني Binding على القيمة.

كمان ممكن تعمل `Binding` يدوي بـ `get` و `set` لما القيمة مش جاهزة كـ Binding أو محتاج تعدل عليها قبل ما تمررها للـ child.

</div>

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

<div dir="rtl">

لو الـ ViewModel فيه async، استخدم `@MainActor` عشان تضمن إن تحديث الـ `@Published` يحصل دايمًا على الـ main thread وما يجيش تحذير من SwiftUI.

</div>

</details>

<details>
<summary>بالعربي</summary>

<div dir="rtl">

استخدم `@StateObject` لما الـ View هي اللي بتعمل create للـ ViewModel.

مثال:

`LoginView` تعمل create لـ `LoginViewModel`.

هنا الأفضل:

```swift
@StateObject private var viewModel = LoginViewModel()
```

مش `@ObservedObject`.

</div>

</details>

> **iOS 17+:** If your minimum deployment target is iOS 17 or newer, prefer an `@Observable` class stored in `@State` instead of `@StateObject`. See [The Modern Observation Framework (iOS 17+)](#the-modern-observation-framework-ios-17).

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

<div dir="rtl">

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

</div>

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

<div dir="rtl">

لو الـ ViewModel اتعمل جوه نفس الشاشة، استخدم `@StateObject`.

لو الـ ViewModel جاي من parent، استخدم `@ObservedObject`.

</div>

</details>

> **iOS 17+:** With an `@Observable` class, the passed-down child usually needs only a plain property (often `let`) instead of `@ObservedObject`. See [Passing an `@Observable` ViewModel from a Parent](#passing-an-observable-viewmodel-from-a-parent).

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

<div dir="rtl">

`@Environment` للقيم العامة اللي SwiftUI أو parent حاططها في البيئة.

زي:

- أقفل الشاشة
- أعرف dark mode ولا light mode
- أعرف اتجاه اللغة RTL/LTR
- أفتح URL

</div>

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

<div dir="rtl">

`@EnvironmentObject` مناسب للـ objects المشتركة على مستوى كبير.

زي:

- UserSession
- AppCoordinator
- CartManager
- AppSettings

لكنه مش مناسب لأي ViewModel خاص بشاشة واحدة.

</div>

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

<div dir="rtl">

`@AppStorage` بيدعم أنواع محددة فقط: `Bool`, `Int`, `Double`, `String`, `URL`, `Data`، و enums من نوع `RawRepresentable`.

لو محتاج تخزن struct أو array، حوّله لـ `Data` بـ `JSONEncoder` الأول.

</div>

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

<div dir="rtl">

`@AppStorage` هو wrapper من SwiftUI فوق `UserDefaults`.

يعني:

```text
@AppStorage = UserDefaults + SwiftUI refresh
```

استخدمه مع القيم البسيطة وغير الحساسة.

أما token أو password أو أي بيانات حساسة، استخدم Keychain.

</div>

</details>

---

## The Modern Observation Framework (iOS 17+)

> **Availability:** iOS 17+ / macOS 14+ / Swift 5.9+

Starting with iOS 17 and Swift 5.9, Apple introduced a new system for observing data changes called **Observation**.

The new approach is based on the `@Observable` macro:

```swift
import Observation

@Observable
final class CounterViewModel {
    var count = 0
}
```

Instead of the old approach based on Combine:

```swift
import Combine

final class CounterViewModel: ObservableObject {
    @Published var count = 0
}
```

The core idea is that SwiftUI now knows exactly which properties were read inside a `body`, and re-evaluates the view only when those specific properties change.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

من iOS 17 وSwift 5.9، قدمت Apple نظام جديد لمراقبة البيانات اسمه **Observation** بيعتمد على الـ macro اسمه `@Observable` بدل `ObservableObject` و `@Published` القديمة.

الفكرة إن SwiftUI بقت تعرف بالظبط أي properties اتقرت جوه الـ `body`، وتعيد تحديث الـ View فقط لما الـ properties دي تتغير.

</div>

</details>

---

### The Old Way: `ObservableObject`

Before iOS 17, a ViewModel was usually written like this:

```swift
import SwiftUI
import Combine

@MainActor
final class CounterViewModel: ObservableObject {
    @Published var count = 0
    @Published var title = "Counter"

    func increment() {
        count += 1
    }
}
```

And in the view:

```swift
struct CounterView: View {
    @StateObject private var viewModel = CounterViewModel()

    var body: some View {
        VStack {
            Text(viewModel.title)
            Text("\(viewModel.count)")

            Button("Increase") {
                viewModel.increment()
            }
        }
    }
}
```

How it worked:

```text
@Published property changed
→ objectWillChange sends an event
→ @StateObject receives the event
→ SwiftUI recalculates body
```

The important limitation: the notification is usually at the level of the **whole object** (`CounterViewModel changed`), not a specific property. So changing `title` invalidates the view even if the body only reads `count`.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

في النظام القديم الـ ViewModel بيطابق `ObservableObject`، وكل property نراقبها بنحط عليها `@Published`. لما القيمة تتغير، `objectWillChange` بيبعت event والـ `@StateObject` بيستقبله وSwiftUI تعيد تقييم الـ `body`.

المشكلة إن الإشعار غالبًا على مستوى الـ object كله مش على مستوى property محددة.

</div>

</details>

---

### The New Way: `@Observable`

In iOS 17+ the same ViewModel becomes:

```swift
import Observation

@MainActor
@Observable
final class CounterViewModel {
    var count = 0
    var title = "Counter"
    var isLoading = false

    func increment() {
        count += 1
    }
}
```

And in the view, notice `@State` instead of `@StateObject`, and no `@Published`:

```swift
import SwiftUI

struct CounterView: View {
    @State private var viewModel = CounterViewModel()

    var body: some View {
        VStack {
            Text("\(viewModel.count)")

            Button("Increase") {
                viewModel.increment()
            }
        }
    }
}
```

Here the view reads `viewModel.count`, but it does **not** read `viewModel.title` or `viewModel.isLoading`. So SwiftUI records that this view depends only on `count`. If `title` or `isLoading` changes, this view is not re-evaluated, because it never read them inside `body`.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

في النظام الجديد بنكتب `@Observable` على الـ class، وبنستخدم `@State` بدل `@StateObject`، ومن غير `@Published` خالص.

الـ View اللي بتقرا `count` بس، SwiftUI بتسجّل إنها تعتمد على `count` فقط. لو `title` أو `isLoading` اتغيرت، الـ View مش هتتحدث لأنها ماقرتهاش جوه الـ `body`.

</div>

</details>

---

### How Does the View Observe Changes Without `@Published`?

`@Observable` is **not** a property wrapper. It is a **macro** that runs at compile time and automatically injects observation code into your class.

The generated code is roughly similar to this (illustrative, not the exact compiler output):

```swift
final class CounterViewModel: Observable {

    private let observationRegistrar = ObservationRegistrar()

    private var _count = 0

    var count: Int {
        get {
            observationRegistrar.access(self, keyPath: \.count)
            return _count
        }
        set {
            observationRegistrar.withMutation(of: self, keyPath: \.count) {
                _count = newValue
            }
        }
    }
}
```

When SwiftUI executes `Text("\(viewModel.count)")`, the `count` getter runs and registers that this view read this property:

```text
CounterView depends on:
- this ViewModel instance
- a property named count
```

When `viewModel.count += 1` runs, the setter notifies the Observation system, which finds the views that read `count` and invalidates them.

The full flow:

```text
SwiftUI executes body
→ body reads viewModel.count
→ Observation records the dependency
→ viewModel.count changes
→ Observation notifies SwiftUI
→ SwiftUI recalculates body
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@Observable` مش Property Wrapper، هي **Macro** بتشتغل وقت الـ compile وبتضيف كود مراقبة تلقائي للـ class.

لما SwiftUI تقرا الـ property بيشتغل الـ getter وبيسجّل إن الـ View قرت الـ property دي عن طريق `observationRegistrar.access`. ولما القيمة تتغير بيشتغل الـ setter وبيبلّغ نظام Observation عن طريق `withMutation`، فيلاقي الـ Views اللي قرت الـ property دي ويعملها invalidate.

</div>

</details>

---

### Why `@State` Instead of `@StateObject`?

`@StateObject` belongs to the old system built on `ObservableObject`, `objectWillChange`, and `@Published`.

An `@Observable` class does **not** conform to `ObservableObject` and has no `objectWillChange`, so you use `@State` to store it:

```swift
@State private var viewModel = CounterViewModel()
```

It is important to separate the two responsibilities:

```text
@State
→ holds the ViewModel instance
→ manages ownership and lifetime

@Observable
→ observes the internal properties
→ notifies SwiftUI when a property used in body changes
```

`@State` is **not** the thing observing `count` inside the ViewModel. The Observation system generated by `@Observable` does that.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@StateObject` مرتبطة بالنظام القديم (`ObservableObject` و `objectWillChange` و `@Published`). أما الـ class المعمول له `@Observable` فمابيطابقش `ObservableObject`، فبنستخدم `@State`.

مهم تفصل بين الدورين:

- `@State` بتحتفظ بالـ instance وتدير الـ ownership والـ lifetime.
- `@Observable` بتراقب الـ properties الداخلية وتبلّغ SwiftUI.

</div>

</details>

---

### `@State` with a Value Type vs a Reference Type

`@State` is often used with simple values, but its real meaning is "local state owned by the view that SwiftUI must keep alive for the view's identity." That works for both value types and `@Observable` reference types.

With a value type:

```swift
@State private var count = 0
```

Writing `count += 1` changes the stored value itself, so SwiftUI knows the view needs to update.

With an `@Observable` reference type:

```swift
@State private var viewModel = CounterViewModel()
```

Writing `viewModel.count += 1` does **not** change the reference stored in `@State` — it still points to the same instance. What changed is a property inside the instance.

```text
@State with Int
→ observes the change of the stored value itself.

@State with @Observable class
→ keeps the instance.
→ Observation tracks the internal properties.
```

Before Observation, changing a property on a plain class stored in `@State` was **not** enough to update the view, because the reference never changed — which is exactly why `ObservableObject` + `@Published` + `@StateObject` were required.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@State` مش مخصوصة للـ `String` و`Int` و`Bool` بس. معناها الحقيقي إنها state محلية تمتلكها الـ View وSwiftUI بتحافظ عليها طول عمر هوية الـ View.

- مع value type: تغيير القيمة نفسه بيعرّف SwiftUI تحدّث.
- مع `@Observable` class: الـ reference مابيتغيرش، `@State` بتحافظ على الـ instance، ونظام Observation بيراقب الـ properties جواها.

قبل Observation، تغيير property على class عادي جوه `@State` ماكانش كفاية عشان الـ reference مابيتغيرش، وعشان كده كنا محتاجين `ObservableObject` + `@Published` + `@StateObject`.

</div>

</details>

---

### Passing an `@Observable` ViewModel from a Parent

When the view **creates** the ViewModel, it owns it, so it uses `@State`:

```swift
struct CounterView: View {
    @State private var viewModel = CounterViewModel()

    var body: some View {
        Text("\(viewModel.count)")
    }
}
```

When the ViewModel is created by a parent and **passed down**, the child can use a plain property:

```swift
struct CounterDetailsView: View {
    let viewModel: CounterViewModel

    var body: some View {
        Text("\(viewModel.count)")
    }
}
```

Even though it is declared with `let`, the view still updates when `viewModel.count` changes. `let` only prevents reassigning the reference (`viewModel = anotherViewModel`); it does not prevent mutating properties inside the instance. As long as the class is `@Observable`, SwiftUI tracks the properties read inside `body`.

> This replaces the old `@ObservedObject` for the passed-down case.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

لو الـ View هي اللي بتعمل create للـ ViewModel، بتستخدم `@State`.

لو الـ ViewModel جاي من parent، الـ child ممكن يستخدم property عادية بـ `let`. رغم الـ `let`، الـ View هتتحدث لما `viewModel.count` تتغير، لأن `let` بتمنع تغيير الـ reference نفسه بس مش تغيير الـ properties جوه الـ instance. ده بيحل محل `@ObservedObject` القديمة في حالة التمرير.

</div>

</details>

---

### Computed Properties and Observation

Observation works through computed properties too. Whatever stored properties the computed property reads become dependencies of the view.

```swift
@Observable
final class LoginViewModel {
    var email = ""
    var password = ""

    var isLoginEnabled: Bool {
        !email.isEmpty && !password.isEmpty
    }
}
```

```swift
Text(viewModel.isLoginEnabled ? "Enabled" : "Disabled")
```

Reading `isLoginEnabled` internally reads `email` and `password`, so Observation registers the view as depending on both. When either changes, the view updates.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

الـ computed properties بتشتغل مع Observation. أي stored properties بتتقرا جوه الـ computed property بتبقى dependencies للـ View.

هنا قراءة `isLoginEnabled` بتقرا `email` و`password` جواها، فأي واحدة تتغير الـ View تتحدث.

</div>

</details>

---

### `@MainActor` with `@Observable` ViewModels

`@Observable` does **not** automatically make the ViewModel run on the main actor. For ViewModels that drive UI state, it is usually best to mark them with `@MainActor`:

```swift
@MainActor
@Observable
final class HomeViewModel {
    var items: [String] = []
    var isLoading = false
}
```

This ensures UI-related state (`items`, `isLoading`, `errorMessage`, `selectedItem`, ...) is accessed and mutated safely on the main actor.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@Observable` ماتعنيش إن الـ ViewModel بيشتغل تلقائيًا على الـ Main Actor. فالأفضل تحط `@MainActor` على الـ ViewModels اللي بتتحكم في UI state عشان تعديل الحالة يتم بأمان على الـ main actor.

</div>

</details>

---

### Old vs New Comparison

| Old System | New System |
|---|---|
| `ObservableObject` | `@Observable` |
| `@Published var count` | `var count` |
| `@StateObject` when the view creates the ViewModel | `@State` |
| `@ObservedObject` when the ViewModel is passed in | plain property (often `let`) |
| Based on Combine | Based on Observation |
| Sends `objectWillChange` | Tracks access to individual properties |
| Notification usually at object level | Tracking at property + instance level |

---

### Is the Old Approach Deprecated?

No. `ObservableObject`, `@Published`, `@StateObject`, and `@ObservedObject` still work, and are still required when your app supports iOS versions older than 17.

Use the old approach when your deployment target is below iOS 17:

```swift
@MainActor
final class HomeViewModel: ObservableObject {
    @Published var items: [String] = []
}
```

```swift
struct HomeView: View {
    @StateObject private var viewModel = HomeViewModel()

    var body: some View {
        Text("\(viewModel.items.count)")
    }
}
```

Use Observation when your minimum supported version is iOS 17 or newer:

```swift
@MainActor
@Observable
final class HomeViewModel {
    var items: [String] = []
}
```

```swift
struct HomeView: View {
    @State private var viewModel = HomeViewModel()

    var body: some View {
        Text("\(viewModel.items.count)")
    }
}
```

**Key sentence:** `@State` keeps the ViewModel alive, while `@Observable` makes SwiftUI listen to changes of the internal properties.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

الطريقة القديمة **مش** متلغية. `ObservableObject` و `@Published` و `@StateObject` و `@ObservedObject` لسه شغالين، ولسه مطلوبين لو التطبيق بيدعم إصدارات أقدم من iOS 17.

- deployment target أقل من iOS 17 → استخدم النظام القديم.
- أقل إصدار مدعوم iOS 17 أو أحدث → استخدم Observation.

أهم جملة: `@State` بتحافظ على عمر الـ ViewModel، و`@Observable` بتخلي SwiftUI تسمع تغييرات الـ properties الداخلية.

</div>

</details>

---

## Adjacent SwiftUI Property Wrappers

Besides the main SwiftUI property wrappers like `@State`, `@Binding`, `@StateObject`, `@ObservedObject`, `@Environment`, `@EnvironmentObject`, and `@AppStorage`, SwiftUI also provides some specialized property wrappers.

These wrappers are not usually used to manage screen ViewModels or app-wide state.

Instead, they solve specific UI-related problems such as:

```text
Input focus
Scene state restoration
Temporary gesture state
Shared animation identity
```

In this section, we will cover:

```swift
@FocusState
@SceneStorage
@GestureState
@Namespace
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

فيه Property Wrappers تانية في SwiftUI قريبة من فكرة الـ state management، لكنها مش بديل مباشر لـ `@StateObject` أو `@ObservedObject`.

الأنواع دي بتستخدم لحالات معينة زي:

```text
التحكم في الكيبورد والفوكس
حفظ حالة بسيطة للشاشة
حالة مؤقتة أثناء الـ gesture
ربط animations بين views مختلفة
```

</div>

</details>

---

## `@FocusState`

> **Availability:** iOS 15+ / macOS 12+ / watchOS 8+ / tvOS 15+

Use `@FocusState` when you need to track or control input focus in SwiftUI.

It is commonly used with:

```swift
TextField
SecureField
TextEditor
```

In UIKit, you usually control input focus using:

```swift
textField.becomeFirstResponder()
textField.resignFirstResponder()
```

In SwiftUI, `@FocusState` gives you a declarative way to control the same behavior.

### Example

```swift
struct LoginView: View {
    @State private var phone = ""
    @FocusState private var isPhoneFocused: Bool

    var body: some View {
        VStack(spacing: 16) {
            TextField("Phone", text: $phone)
                .focused($isPhoneFocused)

            Button("Focus Phone") {
                isPhoneFocused = true
            }

            Button("Dismiss Keyboard") {
                isPhoneFocused = false
            }
        }
    }
}
```

When `isPhoneFocused` becomes `true`, the `TextField` becomes focused and the keyboard appears.

When `isPhoneFocused` becomes `false`, the focus is removed and the keyboard is dismissed.

### Multiple fields example

For multiple input fields, using an enum is usually cleaner than using many Boolean values.

```swift
enum Field {
    case phone
    case password
}

struct LoginView: View {
    @State private var phone = ""
    @State private var password = ""

    @FocusState private var focusedField: Field?

    var body: some View {
        VStack(spacing: 16) {
            TextField("Phone", text: $phone)
                .focused($focusedField, equals: .phone)

            SecureField("Password", text: $password)
                .focused($focusedField, equals: .password)

            Button("Next") {
                focusedField = .password
            }

            Button("Done") {
                focusedField = nil
            }
        }
    }
}
```

### Supported value types

`@FocusState` is generic over its value type. The two valid shapes are:

```text
Bool                  — for a single field (focused or not)
Hashable (often enum) — for multiple fields, usually wrapped as Optional
```

So `@FocusState private var focusedField: Field?` works because `Field` is `Hashable`. A custom struct also works as long as it conforms to `Hashable`.

### When should you use `@FocusState`?

Use `@FocusState` when you need to:

```text
Open the keyboard
Dismiss the keyboard
Move focus from one field to another
Know which input field is currently active
```

### Summary

```text
@FocusState controls input focus in SwiftUI.
It is mostly used with text fields and keyboard handling.
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@FocusState` بتستخدم لما تكون عاوز تتحكم في الفوكس بتاع الـ input.

يعني تعرف أنهي `TextField` هو النشط حاليًا، أو تفتح الكيبورد، أو تقفل الكيبورد، أو تنقل المستخدم من field للتاني.

بدل ما في UIKit كنت بتستخدم:

```swift
becomeFirstResponder()
resignFirstResponder()
```

في SwiftUI بتستخدم `@FocusState`.

</div>

</details>

---

## `@SceneStorage`

> **Availability:** iOS 14+ / macOS 11+ / watchOS 7+ / tvOS 14+

Use `@SceneStorage` when you need SwiftUI to save and restore lightweight UI state for a specific scene.

A scene usually represents a window or an instance of your app UI.

`@SceneStorage` is similar to `@State`, but SwiftUI can try to restore its value when the scene is recreated.

### Example

```swift
struct SearchView: View {
    @SceneStorage("searchText") private var searchText = ""

    var body: some View {
        TextField("Search", text: $searchText)
    }
}
```

Here, `searchText` is UI state related to this scene.

SwiftUI may restore it if the scene is recreated.

### Another example

```swift
struct HomeView: View {
    @SceneStorage("selectedTab") private var selectedTab = 0

    var body: some View {
        TabView(selection: $selectedTab) {
            Text("Home")
                .tag(0)

            Text("Profile")
                .tag(1)
        }
    }
}
```

Here, `@SceneStorage` can remember the selected tab for this scene.

### Supported types

`@SceneStorage` accepts the same restricted set of types as `@AppStorage`:

```text
Bool
Int
Double
String
URL
Data
RawRepresentable enums whose RawValue is one of the above
```

If you need to persist a struct or array, encode it to `Data` with `JSONEncoder` first.

### Good use cases

Use `@SceneStorage` for lightweight UI state like:

```text
Search text
Selected tab
Selected item id
Draft UI input
Current screen state inside a scene
```

### `@SceneStorage` vs `@AppStorage`

`@AppStorage` has a longer lifetime than `@SceneStorage`.

`@AppStorage` stores values in `UserDefaults`.

That means the value is usually still available after closing and reopening the app.

Example:

```swift
@AppStorage("selectedLanguage") private var selectedLanguage = "en"
```

This is a good use case for `@AppStorage`, because the selected language is an app preference and should stay saved.

But `@SceneStorage` is different.

`@SceneStorage` is mainly used for scene state restoration, not permanent storage.

Example:

```swift
@SceneStorage("searchText") private var searchText = ""
```

This is good for temporary UI state, like restoring what the user was typing in a search field.

### Does `@SceneStorage` get cleared if the app is fully closed?

You should not treat `@SceneStorage` as long-term storage.

If the app goes to the background and comes back, or if SwiftUI recreates the scene, the value may be restored.

But if the app is fully closed, killed, or the system removes the saved scene state, the value may be lost.

So the practical rule is:

```text
Use @SceneStorage for temporary scene-based UI restoration.
Use @AppStorage for values that must stay saved after closing and reopening the app.
```

### Difference table

| Wrapper | Storage Lifetime | Best For |
|---|---|---|
| `@SceneStorage` | Temporary per-scene restoration | Search text, selected tab, selected item |
| `@AppStorage` | Longer persistent storage using UserDefaults | Language, theme, onboarding status, user preferences |

### Summary

```text
@SceneStorage is for restoring lightweight UI state for a specific scene.
It is not a replacement for @AppStorage.
If the value must stay saved after closing and reopening the app, use @AppStorage.
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@SceneStorage` بتستخدم لحفظ حالة بسيطة تخص scene معينة.

يعني مثلًا:

```text
search text
selected tab
selected item
```

لكن مهم جدًا تفهم الفرق بينها وبين `@AppStorage`.

`@AppStorage` عمر التخزين بتاعها أطول لأنها مبنية على `UserDefaults`.

يعني لو قفلت التطبيق وفتحته تاني، القيمة غالبًا هتفضل موجودة.

أما `@SceneStorage` فهي معمولة أكتر عشان state restoration.

يعني SwiftUI ممكن ترجع القيمة لو الـ scene اتعملها recreate، لكن متعتمدش عليها كـ storage دائم.

لو التطبيق اتقفل تمامًا، أو اتقتل، أو النظام مسح حالة الـ scene، القيمة ممكن تضيع.

القاعدة العملية:

```text
لو القيمة لازم تفضل بعد قفل وفتح التطبيق
→ استخدم @AppStorage

لو القيمة مجرد UI state مؤقت للـ scene
→ استخدم @SceneStorage
```

</div>

</details>

---

## `@GestureState`

> **Availability:** iOS 13+ / macOS 10.15+ / watchOS 6+ / tvOS 13+

Use `@GestureState` to store temporary state while a gesture is active.

The important point is:

```text
@GestureState automatically resets to its initial value when the gesture ends.
```

This makes it useful for gestures like:

```text
Drag
Long press
Magnification
Rotation
Gesture progress
```

### Example

```swift
struct DragExampleView: View {
    @GestureState private var dragOffset: CGSize = .zero

    var body: some View {
        Circle()
            .frame(width: 100, height: 100)
            .offset(dragOffset)
            .gesture(
                DragGesture()
                    .updating($dragOffset) { value, state, _ in
                        state = value.translation
                    }
            )
    }
}
```

Here, `dragOffset` changes while the user is dragging.

When the user releases the drag, `dragOffset` automatically returns to:

```swift
.zero
```

### `@GestureState` vs `@State`

Use `@GestureState` for temporary gesture values.

Use `@State` if you want to keep the final value after the gesture ends.

### Example using both

```swift
struct DraggableView: View {
    @State private var finalOffset: CGSize = .zero
    @GestureState private var dragOffset: CGSize = .zero

    var body: some View {
        Circle()
            .frame(width: 100, height: 100)
            .offset(
                width: finalOffset.width + dragOffset.width,
                height: finalOffset.height + dragOffset.height
            )
            .gesture(
                DragGesture()
                    .updating($dragOffset) { value, state, _ in
                        state = value.translation
                    }
                    .onEnded { value in
                        finalOffset.width += value.translation.width
                        finalOffset.height += value.translation.height
                    }
            )
    }
}
```

In this example:

```text
@GestureState stores the temporary drag movement.
@State stores the final position after the drag ends.
```

### Summary

```text
@GestureState stores temporary gesture state.
SwiftUI resets it automatically when the gesture ends.
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@GestureState` بتستخدم مع الـ gestures.

أهم فرق بينها وبين `@State` إن `@GestureState` بترجع للقيمة الأصلية تلقائيًا لما الـ gesture يخلص.

مثال:

لو المستخدم بيسحب view، ممكن تستخدم `@GestureState` للحركة المؤقتة أثناء السحب.

لكن لو عاوز تحفظ المكان النهائي بعد ما المستخدم يسيب السحب، استخدم `@State`.

```text
@GestureState = قيمة مؤقتة أثناء الـ gesture
@State = قيمة تفضل محفوظة بعد التغيير
```

</div>

</details>

---

## `@Namespace`

> **Availability:** iOS 14+ / macOS 11+ / watchOS 7+ / tvOS 14+ (same for `matchedGeometryEffect`)

Use `@Namespace` to create a shared animation namespace.

It is commonly used with:

```swift
matchedGeometryEffect
```

The main idea is that SwiftUI can connect two related views and animate smoothly between them.

### Example

```swift
struct NamespaceExampleView: View {
    @Namespace private var animation
    @State private var isExpanded = false

    var body: some View {
        VStack {
            if isExpanded {
                RoundedRectangle(cornerRadius: 20)
                    .matchedGeometryEffect(id: "card", in: animation)
                    .frame(width: 300, height: 300)
            } else {
                RoundedRectangle(cornerRadius: 20)
                    .matchedGeometryEffect(id: "card", in: animation)
                    .frame(width: 100, height: 100)
            }
        }
        .onTapGesture {
            withAnimation {
                isExpanded.toggle()
            }
        }
    }
}
```

Both views use the same:

```swift
.matchedGeometryEffect(id: "card", in: animation)
```

So SwiftUI understands that these two views represent the same visual element in different states.

Instead of removing one view and showing another suddenly, SwiftUI creates a smooth transition between them.

### When should you use `@Namespace`?

Use `@Namespace` when you need:

```text
Shared element animation
Smooth transition between related views
matchedGeometryEffect
Card expand/collapse animation
Image transition between list and details screen
```

### Summary

```text
@Namespace creates an animation namespace.
It is mainly used with matchedGeometryEffect.
It does not store app data or ViewModel state.
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

`@Namespace` بتستخدم غالبًا مع `matchedGeometryEffect`.

فكرتها إنها بتعمل مساحة مشتركة للـ animation، عشان SwiftUI تفهم إن view معينة و view تانية هما نفس العنصر بصريًا لكن في شكل أو مكان مختلف.

مثال:

كارت صغير في list ولما تضغط عليه يفتح كارت كبير في details.

باستخدام `@Namespace` و `matchedGeometryEffect`، SwiftUI تقدر تعمل transition ناعم بين الشكلين.

`@Namespace` مش لتخزين data، ومش للـ ViewModel.

هي للـ animations.

</div>

</details>

---

## Adjacent Wrappers Summary Table

| Wrapper | Purpose | Common Use |
|---|---|---|
| `@FocusState` | Tracks and controls input focus | Open/dismiss keyboard, move between text fields |
| `@SceneStorage` | Saves lightweight per-scene UI state | Restore selected tab, search text, selected item |
| `@GestureState` | Stores temporary gesture state | Drag, long press, magnification, gesture progress |
| `@Namespace` | Creates a shared animation namespace | `matchedGeometryEffect` animations |

---

## Adjacent Wrappers Mental Model

```text
@FocusState
→ Keyboard and input focus.

@SceneStorage
→ Temporary scene-based UI state restoration.

@GestureState
→ Temporary state while a gesture is active.

@Namespace
→ Shared identity for animations.

@AppStorage
→ Longer persistent storage using UserDefaults.
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

الخلاصة:

```text
@FocusState
= للتحكم في الفوكس والكيبورد

@SceneStorage
= لحفظ حالة UI بسيطة ومؤقتة للـ scene

@GestureState
= لحالة مؤقتة أثناء الـ gesture

@Namespace
= لربط animations بين views مختلفة

@AppStorage
= تخزين أطول باستخدام UserDefaults
```

</div>

</details>

---

## Apartment Analogy

A simple analogy to remember the differences between SwiftUI property wrappers.

Think of your app as a building, and each SwiftUI View as an apartment inside that building.

```text
@State
= Something inside your own apartment.
You own it, and it belongs only to this View.

Example:
isPasswordVisible, searchText, selectedTab
```

```text
@Binding
= A switch inside your apartment that controls something owned by another apartment.

You do not own the original value.
You only have a connection to it, and you can change it.

Example:
A child view changes a value owned by the parent view.
```

```text
@StateObject
= A device you bought and placed inside your apartment.

You created it, you own it, and it should stay alive as long as your apartment exists.

Example:
A screen creates its own ViewModel.
```

```text
@ObservedObject
= A device owned by another apartment, but you are allowed to watch and use it.

You did not create it.
You only observe it.

Example:
A child view receives a ViewModel from the parent.
```

```text
@Environment
= Building-wide settings or services.

These are values already available in the building environment.

Example:
theme, language direction, dismiss action, open URL
```

```text
@EnvironmentObject
= A shared system used by the whole building.

It is injected at a high level, and many apartments can access it.

Example:
UserSession, AppCoordinator, CartManager, AppSettings
```

```text
@AppStorage
= A notebook where you save simple preferences.

Even if you leave the apartment and come back later, the saved value is still there.

Example:
hasSeenOnboarding, selectedLanguage, selectedTheme
```

```text
@FocusState
= A spotlight inside your apartment that points at one device at a time.

You decide which device (TextField) gets attention, and you can turn the spotlight off.

Example:
focus the phone field, then move focus to the password field, then dismiss the keyboard
```

```text
@SceneStorage
= A sticky note inside your apartment for the current visit.

If you step out and come back, the note may still be there.
But if you fully move out (the app is killed), the note can disappear.

Example:
remember the search text or selected tab while the user navigates the app
```

```text
@GestureState
= Something you hold in your hand while doing an action.

The moment you let go, your hand is empty again.
The value resets automatically when the gesture ends.

Example:
the temporary offset while the user is dragging a card
```

```text
@Namespace
= A shared label between two pieces of furniture in different rooms.

The building uses the label to move smoothly between them — same identity, different shape or place.

Example:
a small card in a list smoothly expanding into a big card on the details screen
```

<details>
<summary>بالعربي المختصر</summary>

<div dir="rtl">

```text
@State = حاجة جوه شقتك وملك الشاشة نفسها
@Binding = مفتاح عندك بيتحكم في حاجة أصلها عند الـ parent
@StateObject = جهاز أنت اشتريته وحطيته في شقتك، يعني الشاشة عملته وتملكه
@ObservedObject = جهاز مش بتاعك، جاي من بره، وأنت بس بتراقبه
@Environment = إعدادات عامة في العمارة كلها، زي اللغة أو الثيم أو dismiss
@EnvironmentObject = نظام مشترك في العمارة، زي session أو coordinator
@AppStorage = نوتة بتخزن فيها إعدادات بسيطة وتفضل محفوظة بعدين
@FocusState = كشاف بيضوي على جهاز واحد في الأوضة، يعني بتحدد أنهي TextField شغال دلوقتي
@SceneStorage = ورقة لاصقة جوه الشقة للزيارة الحالية، لو خرجت ورجعت ممكن تلاقيها، لكن لو سبت الشقة خالص ممكن تروح
@GestureState = حاجة ماسكها في إيدك أثناء حركة، أول ما تسيبها إيدك ترجع فاضية تلقائي
@Namespace = لافتة مشتركة بين قطعتين أثاث في أوض مختلفة، عشان المبنى ينقل بينهم بشكل ناعم
```

</div>

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

### Use `@FocusState` when:

```text
You need to control input focus
You need to open or dismiss the keyboard
You need to move between text fields
```

Example:

```swift
@FocusState private var focusedField: Field?
```

---

### Use `@SceneStorage` when:

```text
The value is lightweight UI state
The value belongs to a specific scene
You want SwiftUI to restore it when the scene is recreated
```

Example:

```swift
@SceneStorage("searchText") private var searchText = ""
```

---

### Use `@GestureState` when:

```text
The value is temporary
The value exists only while a gesture is active
The value should reset automatically when the gesture ends
```

Example:

```swift
@GestureState private var dragOffset: CGSize = .zero
```

---

### Use `@Namespace` when:

```text
You need matchedGeometryEffect
You need a shared animation identity
You need a smooth transition between related views
```

Example:

```swift
@Namespace private var animation
```

---

## Summary Table

| Wrapper | Kind | Owner | Best For |
|---|---|---|---|
| `@State` | Value | Current View | Local UI state |
| `@Binding` | Binding | Parent View | Child editing parent state |
| `@StateObject` | Object | Current View | Screen ViewModel |
| `@ObservedObject` | Object | Parent View | Child using parent ViewModel |
| `@Environment` | Env value | SwiftUI / Parent | `dismiss`, `colorScheme`, `locale` |
| `@EnvironmentObject` | Env object | Parent / App | `session`, `coordinator`, `settings` |
| `@AppStorage` | Persisted value | UserDefaults | Simple persistent preferences |
| `@FocusState` | Focus state | Current View | Keyboard and input focus |
| `@SceneStorage` | Scene state | SwiftUI scene | Lightweight per-scene UI restoration |
| `@GestureState` | Gesture state | Current gesture | Drag, long press, gesture progress |
| `@Namespace` | Animation ID | Current View | `matchedGeometryEffect` animations |

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

### Mistake 8: Using `@State` for drag offsets instead of `@GestureState`

Wrong:

```swift
struct DragView: View {
    @State private var dragOffset: CGSize = .zero

    var body: some View {
        Circle()
            .offset(dragOffset)
            .gesture(
                DragGesture()
                    .onChanged { value in
                        dragOffset = value.translation
                    }
            )
    }
}
```

When the user lets go, `dragOffset` stays at the last drag value and the circle is stuck off-center until you manually reset it in `.onEnded`.

Correct:

```swift
@GestureState private var dragOffset: CGSize = .zero

// ...

.gesture(
    DragGesture()
        .updating($dragOffset) { value, state, _ in
            state = value.translation
        }
)
```

`@GestureState` resets to its initial value automatically when the gesture ends — no manual cleanup needed. See [`@GestureState`](#gesturestate) for the full pattern.

---

### Mistake 9: Treating `@SceneStorage` as permanent storage

Wrong assumption:

```swift
@SceneStorage("isLoggedIn") private var isLoggedIn = false
```

`@SceneStorage` is for scene-level UI restoration, not durable storage. If the app is killed or the system discards the saved scene state, the value can be lost — which means the user may suddenly appear logged out.

Correct:

```text
Use @AppStorage (or Keychain for sensitive data) for values that must survive app restarts.
Use @SceneStorage only for lightweight UI state like search text or selected tab.
```

See [`@SceneStorage` vs `@AppStorage`](#scenestorage-vs-appstorage) for the full comparison.

---

### Mistake 10: Many `Bool` `@FocusState` properties instead of one enum

Wrong:

```swift
@FocusState private var isPhoneFocused: Bool
@FocusState private var isPasswordFocused: Bool
@FocusState private var isEmailFocused: Bool
```

Now moving focus between fields means flipping multiple Bools, and it's easy to end up with two fields "focused" at once in your state, even though only one really is.

Better:

```swift
enum Field {
    case phone
    case password
    case email
}

@FocusState private var focusedField: Field?
```

One property tracks the currently focused field, and `nil` means "no focus / keyboard dismissed". Moving focus is just `focusedField = .password`.

---

### Mistake 11: Using `@StateObject` / `@ObservedObject` with an `@Observable` class

Wrong:

```swift
@Observable
final class CounterViewModel {
    var count = 0
}

struct CounterView: View {
    @StateObject private var viewModel = CounterViewModel() // ❌ won't compile
}
```

An `@Observable` class does not conform to `ObservableObject`, so `@StateObject` and `@ObservedObject` do not apply to it.

Correct:

```swift
struct CounterView: View {
    @State private var viewModel = CounterViewModel()
}
```

For a passed-down ViewModel, use a plain property instead of `@ObservedObject`. See [The Modern Observation Framework (iOS 17+)](#the-modern-observation-framework-ios-17).

---

## Recommended Usage in Real Projects

A practical setup for real SwiftUI projects:

```text
Screen ViewModel (iOS 16 and below)
→ ObservableObject + @StateObject

Screen ViewModel (iOS 17+)
→ @Observable + @State

Child View using the same ViewModel (iOS 16 and below)
→ @ObservedObject

Child View using the same ViewModel (iOS 17+)
→ plain property (often let)

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

Keyboard/input focus
→ @FocusState

Temporary scene restoration state
→ @SceneStorage

Temporary gesture state
→ @GestureState

Matched geometry animation
→ @Namespace

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

Keyboard/input focus
→ @FocusState

Temporary scene restoration state
→ @SceneStorage

Temporary gesture state
→ @GestureState

Shared animation identity
→ @Namespace
```

<details>
<summary>بالعربي</summary>

<div dir="rtl">

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

</div>

</details>

---

## Author Note

This README is designed as a practical learning reference for SwiftUI state management.

It focuses on real-world usage, common mistakes, and choosing the right wrapper based on ownership and lifecycle.

**Intended audience:** iOS developers learning or refreshing their understanding of SwiftUI state.

It also covers the iOS 17+ **Observation framework** (`@Observable` + `@State` for ViewModels) as the modern replacement for `ObservableObject` and `@Published`.

**What this guide does *not* cover (yet):**

- Deeper Observation topics like `@Bindable` and the new `@Environment(MyType.self)` syntax — coming in a separate article.

<details>
<summary>بالعربي</summary>

<div dir="rtl">

الـ README ده مرجع عملي لـ state management في SwiftUI، ومناسب لأي iOS developer.

الدليل ده كمان بيغطي الـ Observation framework بتاع iOS 17+ (`@Observable` مع `@State` للـ ViewModels) كبديل حديث لـ `ObservableObject` و `@Published`.

مواضيع Observation الأعمق زي `@Bindable` والـ `@Environment(MyType.self)` الجديدة هتكون في مقال منفصل.

</div>

</details>
