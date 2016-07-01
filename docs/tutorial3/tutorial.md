##Tutorial 3: Working With Widgets

In this tutorial we will explore some of the GWT widgets by extending the functionality of our UFTasks application.
We will be adding an editing screen by duplicating the Presenter-View pattern already established by the NewFolder and NewProject Popup dialogs.

We will discover how widgets can be created, both in the HTML template file and from our Uberfire View class, and how to transfer data to and from these widgets.

We will also need to extend our UFTasks application model to take advantage of these widgets. Specifically, we will be adding the following two new attributes to our **Task** model object:

- **notes** a string field that can be used to describe the task in more detail
- **priority** an integer value representing the task priority, with 0 being low, and positive numbers being higher priorities
- **dueDate** a **java.util.Date** object indicating when the Task is due
- **id** a string value that uniquely identifies this Task object. This is used to locate a specific Task in the model tree after it has been changed by our editing screen.