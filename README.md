# UITesting Workshop

## XCUIApplication
XCUIApplication is a proxy for your App (the App runs in a separate process):

* Use to launch, activate or terminate App. Only one instance can run at a time.
* Set launch arguments and launch environment.
* Root of all queries for elements.

```
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

```
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

```
// Find all buttons
let buttonQuery = app.descendants(matching: .button)
XCTAssertTrue(buttonQuery.count == 2)

// Find all text labels
let allLabels = app.descendants(matching: .staticText)
```

* When you only want a direct (child) descendant:
```
// Find nav bar button
let addButton = app.navigationBars.children(matching: .button)
```


## XCUIElement
You interact with and test properties of an XCUIElement:

* The element is the result of a query that matches one unique instance.
* A number of methods will return a single element from a query.
* Use Element when a query should only return one element (fails if there are multiple elements).
* Use firstMatch when you only care about finding the first element from many.

* Getting an element from a query:
```
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

## Launch Argumetns

* Adding and arguments
```
app.launchArguments.append("--uitesting")
```

* Resetting your app’s state
```
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
```
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
```
XCTAssertTrue(app.isDisplayingOnboarding)
```

```
extension XCUIApplication {
    var isDisplayingOnboarding: Bool {
        return otherElements["onboardingView"].exists
    }
}
```

## Alerts

* System Alerts
```
To dismiss system alerts that might otherwise interrupt UI tests, add to setUp():

addUIInterruptionMonitor(withDescription: "System Dialog") { (alert) -> Bool in
   // Tap "Allow" button
   alert.buttons["Allow"].tap()
   return true
 }
 // Need to interact with App
 app.tap()
```

