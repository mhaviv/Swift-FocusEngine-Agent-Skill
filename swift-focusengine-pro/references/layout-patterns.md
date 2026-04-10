# Common Layout Patterns and Focus (tvOS + macOS)

## Netflix/VOD Pattern: Table of Horizontal Collections

Each row is a table cell containing a horizontal collection view. This is the most common tvOS layout.

### SwiftUI

```swift
ScrollView(.vertical) {
    VStack(spacing: 40) {
        ForEach(categories) { category in
            VStack(alignment: .leading) {
                Text(category.title).font(.headline)
                ScrollView(.horizontal, showsIndicators: false) {
                    LazyHStack(spacing: 20) {
                        ForEach(category.items) { item in
                            Button { select(item) } label: {
                                PosterCard(item: item)
                            }
                            .buttonStyle(.card)
                        }
                    }
                }
                .focusSection()  // CRITICAL — prevents cross-row jumping
            }
        }
    }
}
```

### UIKit

```swift
// UITableViewController where each cell contains a UICollectionView
class CatalogTableViewCell: UITableViewCell {
    let collectionView: UICollectionView

    override init(style:, reuseIdentifier:) {
        // Setup horizontal flow layout
        let layout = UICollectionViewFlowLayout()
        layout.scrollDirection = .horizontal
        collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
        collectionView.remembersLastFocusedIndexPath = true
        super.init(style: style, reuseIdentifier: reuseIdentifier)
    }

    override var preferredFocusEnvironments: [UIFocusEnvironment] {
        return [collectionView]
    }
}
```

The table view naturally isolates rows — vertical focus moves between table cells, horizontal within collection views.

## Sidebar + Content Pattern

### SwiftUI (Fox Weather pattern)

```swift
struct SidebarContentView: View {
    @FocusState var focusedSection: Section?
    @State var isExpanded = false

    var body: some View {
        HStack(spacing: 0) {
            // Sidebar
            VStack {
                ForEach(sections) { section in
                    Button(section.title) { selectedSection = section }
                        .focused($focusedSection, equals: section)
                }
            }
            .frame(width: isExpanded ? 300 : 80)
            .focusSection()
            .onChange(of: focusedSection) { old, new in
                if old != nil && new == nil { isExpanded = false }
                if old == nil && new != nil { isExpanded = true }
            }

            // Content
            ContentView(section: selectedSection)
                .focusSection()
        }
        .onExitCommand {
            if !isExpanded { isExpanded = true }
        }
    }
}
```

Key patterns:
- Each pane gets `.focusSection()`
- Sidebar expand/collapse driven by focus entering/leaving
- `.onExitCommand` returns focus to sidebar instead of exiting app
- `isInitialLoad` guard to prevent sidebar expanding on first appearance

### UIKit

Use `UIFocusGuide` to bridge the gap between sidebar and content:

```swift
let sidebarContentGuide = UIFocusGuide()
view.addLayoutGuide(sidebarContentGuide)
// Position between sidebar and content
sidebarContentGuide.preferredFocusEnvironments = [contentView]

// Update direction based on where focus came from
override func didUpdateFocus(in context: UIFocusUpdateContext, ...) {
    if context.previouslyFocusedView?.isDescendant(of: sidebarView) == true {
        sidebarContentGuide.preferredFocusEnvironments = [contentView]
    } else {
        sidebarContentGuide.preferredFocusEnvironments = [sidebarView]
    }
}
```

## Tab Bar Pattern

### SwiftUI

```swift
TabView(selection: $selectedTab) {
    HomeView().tabItem { Label("Home", systemImage: "house") }.tag(Tab.home)
    ShowsView().tabItem { Label("Shows", systemImage: "tv") }.tag(Tab.shows)
}
```

For custom tab bar (Fox Weather SideTabBar pattern):
- Wrap tab buttons in `.focusSection()`
- Use `@FocusState` to track which tab is focused
- Expand/collapse on focus enter/leave
- Use `.onExitCommand` to bring focus back to tabs

### UIKit

Placeholder tabs (not yet implemented) should redirect focus back to the tab bar:

```swift
class PlaceholderTabViewController: UIViewController {
    private let focusGuide = UIFocusGuide()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.addLayoutGuide(focusGuide)
        // Constrain to fill view
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        if let tabBar = tabBarController?.tabBar {
            focusGuide.preferredFocusEnvironments = [tabBar]
        }
    }
}
```

## Hero + Catalog Pattern

Large hero image at top, catalog rows below. Hero collapses when catalog gets focus.

### Focus coordination

```swift
// Track catalog focus to drive hero collapse
override func didUpdateFocus(in context: UIFocusUpdateContext, ...) {
    let focusInCatalog = context.nextFocusedView?.isDescendant(of: catalogView) == true
    let focusWasInCatalog = context.previouslyFocusedView?.isDescendant(of: catalogView) == true

    if focusInCatalog && !focusWasInCatalog {
        collapseHero(animated: true)
    } else if !focusInCatalog && focusWasInCatalog {
        expandHero(animated: true)
    }
}
```

Block focus during hero animation:

```swift
override func shouldUpdateFocus(in context: UIFocusUpdateContext) -> Bool {
    return !isAnimatingHeroTransition
}
```

## Scroll + Arrow Button Pattern

Horizontal shelf with left/right arrow buttons (Fox Weather PersonalitiesShelfView):

```swift
@FocusState private var buttonFocus: ScrollDirection?

HStack {
    VStack {
        button(.reverse, isDisabled: atStart)
        button(.forward, isDisabled: atEnd)
    }
    .focusSection()  // Buttons get own section

    ScrollView(.horizontal) {
        LazyHStack { /* items */ }
    }
    .focusSection()  // Scroll content gets own section
}

func button(_ direction: ScrollDirection, isDisabled: Bool) -> some View {
    Button { scroll(direction) } label: { Image(systemName: "chevron.\(direction)") }
        .focused($buttonFocus, equals: direction)
        .allowsHitTesting(!isDisabled)  // NOT .disabled()
        .opacity(isDisabled ? 0.5 : 1.0)
}
```

When a button becomes disabled after scrolling to the end, auto-move focus to the other button:

```swift
if viewModel.isFullyScrolled(direction: direction) {
    buttonFocus = direction.opposite
}
```

## Split Detail Pattern

Master list on left, detail on right. tvOS NavigationSplitView or manual split:

```swift
NavigationSplitView {
    List(items, selection: $selectedItem) { item in
        Text(item.title)
    }
    .focusSection()
} detail: {
    DetailView(item: selectedItem)
        .focusSection()
}
```

For custom implementation, use two `.focusSection()` panes and handle `.onExitCommand` to return to the master list.

## macOS Layout Patterns

### Sidebar + Content (NavigationSplitView)

The standard macOS document/navigation pattern. Focus moves between sidebar and content via Tab or mouse click.

```swift
NavigationSplitView {
    List(items, selection: $selectedItem) { item in
        Text(item.title)
    }
    .focusSection()  // macOS 14+ — Tab switches between sidebar and content
} detail: {
    if let item = selectedItem {
        DetailView(item: item)
            .focusSection()
    }
}
```

AppKit equivalent uses `NSSplitViewController`:

```swift
class MainSplitViewController: NSSplitViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        // Tab moves between split panes automatically
        // Arrow keys navigate within each pane
    }

    // Focus the sidebar's first item on Cmd+1
    @IBAction func focusSidebar(_ sender: Any?) {
        let sidebarVC = splitViewItems[0].viewController
        view.window?.makeFirstResponder(sidebarVC.view)
    }
}
```

### Toolbar + Content

macOS apps with toolbar items above the main content area. Toolbar focus requires Full Keyboard Access or Ctrl+F5.

```swift
struct ContentView: View {
    @FocusState private var isContentFocused: Bool

    var body: some View {
        VStack {
            // Content area gets focus by default
            DocumentEditor()
                .focusable()
                .focused($isContentFocused)
        }
        .toolbar {
            ToolbarItem {
                TextField("Search", text: $search)
                // Search field is Tab-focusable by default
            }
        }
        .onAppear { isContentFocused = true }
    }
}
```

### Multi-Window Document App

Each window has independent focus state. Menu commands target the key window.

```swift
@main
struct MyApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: MyDocument()) { file in
            DocumentView(document: file.$document)
                .focusedSceneValue(\.activeDocument, file.document)
                // Each window publishes its document for menu commands
        }

        Settings {
            SettingsView()
            // Settings has its own focus scope — independent of documents
        }
    }
}

struct AppCommands: Commands {
    @FocusedValue(\.activeDocument) var document

    var body: some Commands {
        CommandGroup(after: .pasteboard) {
            Button("Format Selection") {
                document?.formatSelection()
            }
            .disabled(document == nil)
            .keyboardShortcut("f", modifiers: [.command, .shift])
        }
    }
}
```

### Preferences / Settings Window

```swift
Settings {
    TabView {
        GeneralSettingsView()
            .tabItem { Label("General", systemImage: "gear") }
        AdvancedSettingsView()
            .tabItem { Label("Advanced", systemImage: "gearshape.2") }
    }
    .frame(width: 450, height: 300)
}
// Tab key cycles through controls in the active settings tab
// Cmd+1/Cmd+2 switches between tabs
```

### Inspector Panel Pattern

Floating panel that doesn't steal focus from the main window:

```swift
// AppKit
let panel = NSPanel(contentRect: rect,
                    styleMask: [.titled, .closable, .utilityWindow],
                    backing: .buffered, defer: false)
panel.becomesKeyOnlyIfNeeded = true  // Don't steal focus
panel.isFloatingPanel = true          // Float above document windows
panel.orderFront(nil)                 // Show without making key

// SwiftUI — use Window scene
Window("Inspector", id: "inspector") {
    InspectorView()
}
.defaultSize(width: 250, height: 400)
```

### Source List + Editor + Inspector (Three-Column)

```swift
NavigationSplitView(columnVisibility: $columnVisibility) {
    // Source list (sidebar)
    List(projects, selection: $selectedProject) { project in
        Label(project.name, systemImage: "folder")
    }
    .focusSection()
} content: {
    // File list (middle)
    List(files, selection: $selectedFile) { file in
        Text(file.name)
    }
    .focusSection()
} detail: {
    // Editor (right)
    EditorView(file: selectedFile)
        .focusSection()
}
```

Tab moves between the three columns. Arrow keys navigate within each column. Each column maintains its own selection/focus independently.
