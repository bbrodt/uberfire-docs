###Place Manager and Place Requests

Our original UFTasks tutorial briefly mentioned the Place Manager and the concept of "places". We saw this service used in the **ShowcaseEntryPoint** class of our application to open different Perspectives. The Place Manager is used in general to open any kind of "managed window", and tracks which are currently active. The term "managed window" refers to any one of a number of different types of windows decorated with one of the annotations defined in the org.uberfire.client.annotations package; some of these you already know, e.g. WorkbenchScreen, WorkbenchEditor and Perspective. There are other types of managed windows, which are topics for future tutorials, but for now let's concentrate on the ones we know about.

The Place Manager keeps an internal lookup table of all currently open managed windows. The lookup key varies, depending on the type of window.

When your application wants to open a Screen, Perspective, or an Editor, it needs to construct a PlaceRequest. For Screens (and Perspectives and certain other types of managed windows), the PlaceRequest is simple: it only needs the Screen identifier, declared in the @WorkbenchScreen annotation as in, for example:

```
@WorkbenchScreen(identifier = "ProjectsPresenter")
```

This is because your application can only have one instance of a Screen type open at a time. The **PlaceRequest** is then simply: 

```
PlaceRequest pr = new DefaultPlaceRequest("ProjectsPresenter");
```

Workbench Editors, however can have multiple instances of the same type open at the same time, so the Place Manager uses a file **Path** used as input to the editor for the lookup table key. In this case we need to use a **PathPlaceRequest**, like so:

```
Path path = getFilePathToEdit();
PlaceRequest pr = new PathPlaceRequest(path);
```

We'll see how this works later.
