# UITesting Workshop

## XCUIApplication
`XCUIApplication` is a proxy for your App (the App runs in a separate process):

* Use to launch, activate or terminate App. Only one instance can run at a time.
* Set launch arguments and launch environment.
* Root of all queries for elements.

https://developer.apple.com/documentation/xctest/xcuiapplication

```swift
let app = XCUIApplication()
app.launchArguments += ["-enableTestMode","1"]
app.launchEnvironment["DEBUG_LEVEL"] = "5"
app.launch()
```

## XCUIElementQuery
Querying the application proxy for elements:

* Application instance is at the root of the element tree
* Query returns a collection of elements
* Query by identifier (label, title, accessibility identifier), element type or with a predicate.

https://developer.apple.com/documentation/xctest/xcuielementquery

```swift
// See XCUIElement.ElementType for full list
.button 
.cell            // table or collection view cells
.collectionView
.image
.navigationBar
.scrollView
.staticText      // text labels
.switch
.tabBar
.table
.textField
```

* Query for descendant element:

```swift
// Find all buttons
let buttonQuery = app.descendants(matching: .button)
XCTAssertTrue(buttonQuery.count == 2)

// Find all text labels
let allLabels = app.descendants(matching: .staticText)
```

* When you only want a direct (child) descendant:
```swift
// Find nav bar button
let addButton = app.navigationBars.children(matching: .button)
```


## XCUIElement
You interact with and test properties of an `XCUIElement`:

* The element is the result of a query that matches one unique instance
* A number of methods will return a single element from a query
* Use Element when a query should only return one element (fails if there are multiple elements)
* Use firstMatch when you only care about finding the first element from many

https://developer.apple.com/documentation/xctest/xcuielement

* Getting an element from a query:
```swift
// Query for all buttons
let buttonQuery = app.descendants(matching: .button)

// Get element with subscript matching identifier
let stopButton = buttonQuery["stopButton"]

// Get element by index
let stopButton = buttonQuery.element(boundBy: 0)

// When query must only return a single unique element
// - will fail if matches multiple elements
let textField = app.textFields.element

// When you want only the first match
// - speeds up test by stopping query when match is found
let firstButton = buttonQuery.firstMatch
```

## UI Events
Once we find our element, we need to simulate user interactions. `XCUIElements` provides some APIs you can use to interact with `UIElement`:

* `tap()`
* `doubleTap()`
* `press(forDuration:, thenDragTo:)`
* `twoFingerTap()`
* `swipeUp(), swipeDown(), swipeLeft(), swipeRight()`
* `typeText("")`

An example of a tap API test

```swift
func testExample() throws {
 // UI tests must launch the application that they test.
 let app = XCUIApplication()
 app.launch()
 XCTAssertEqual(app.tables.cells.count, 6)
 let cell = app.tables.staticTexts["San Francisco"]
 cell.tap()
}
```

## Launch Argumetns

* Adding and arguments
```
app.launchArguments.append("--uitesting")
```

* Resetting your app’s state
```swift
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]?) -> Bool {
    if CommandLine.arguments.contains("--uitesting") {
        resetState()
    }

    // ...Finish setting up your app

    return true
}
```
* Executing different flows
```swift
func makeProfileViewController() -> UIViewController {
    if CommandLine.arguments.contains("-new-profile") {
        // If the "-new-profile" argument was passed, then return
        // an instance of the new implementation.
        return ProfileViewController()
    }

    // Fall back to the old implementation, which is what we
    // still use in production.
    return ProfileLegacyViewController()
}
```

## Creating extensions

* Verifying state in a UI test
```swift
XCTAssertTrue(app.isDisplayingOnboarding)
```

```swift
extension XCUIApplication {
    var isDisplayingOnboarding: Bool {
        return otherElements["onboardingView"].exists
    }
}
```

## Alerts

* System Alerts

To dismiss system alerts that might otherwise interrupt UI tests, add to setUp():
```swift
addUIInterruptionMonitor(withDescription: "System Dialog") { (alert) -> Bool in
   // Tap "Allow" button
   alert.buttons["Allow"].tap()
   return true
 }
 // Need to interact with App
 app.tap()
```

