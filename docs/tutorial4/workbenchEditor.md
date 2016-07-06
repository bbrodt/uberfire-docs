###The Workbench Editor

A WorkbenchEditor is constructed the same as any of our other Presenter classes, except it uses the **@WorkbenchEditor** annotation:

```
@WorkbenchEditor(identifier = "TaskEditor", supportedTypes = { TaskResourceType.class })
```

The "supportedTypes" attribute tells the Place Manager the file types (actually the filename extensions) to associate with this editor. We will see how this is implemented when we talk about Resource Types later in this tutorial.

An Editor also needs to provide some additional UI and lifecycle bits, which are identified by annotations. These are:

- **@PostConstruct** - method that is called immediately after construction of the editor Presenter class to initialize the editor's View class
- **@OnStartup** - startup method that is called after the @PostConstruct method, which is responsible for initializing internal data structures and loading the editor content
- **@WorkbenchPartTitle** - a method that returns the title text for the editor
- **@WorkbenchPartTitleDecoration** - optional method that returns the editor title widget
- **@WorkbenchPartView** - method that returns the View for the editor
- **@WorkbenchMenu** - method that returns an editor-specific menu, which may be null

We will see these later when we start writing some code.