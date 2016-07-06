###More Model Changes!

In order to better illustrate the use of **PathPlaceRequest**s, we will extend our model just a bit by adding a **notes** String attribute which the user can edit inside our Task Editor. However, we don't want to incur the penalty of having to marshal this potentially large amount of data across the wire for every task displayed by the TasksPresenter/TasksView list. Instead we will only load the **notes** data when the user opens the Task Editor. This is accomplished using a separate text file associated with a Task instance. Thus, when we issue a Place Request with this file name, Place Manager will locate and open our Editor and associate this **Path** with the editor instance.

To accomplish this, we'll create a wrapper class which extends our **Task** object with the **notes** field. This is for client-side consumption only, so there's no need to provide any kind of marshalling support as we did with the other Model objects.

_TaskWithNotes.java_
```
package org.uberfire.component.model;

public class TaskWithNotes extends Task {

    private String notes = "";
     
    public TaskWithNotes(Task that) {
        super(that);
    }

    public void setNotes(String notes) {
        this.notes = notes;
    }
    
    public String getNotes() {
        return notes;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof TaskWithNotes) {
            TaskWithNotes other = (TaskWithNotes)obj;
            if (!this.getNotes().equals(other.getNotes()))
                return false;
        }
        return super.equals(obj);
    }
    
    public Task asTask() {
        return new Task(this);
    }
}
```

The text file path for our "task notes" is constructed using the User ID and Task **id** field suffixed with a ".task" file extension. Recall that our model is being serialized by the UFTasksService, so we're just going to add this file path construction to this service, since it already knows the location of the "uftasks" repository. So let's extend the UFTasksService interface and implementation class:

_UFTasksService.java_
```
public interface UFTasksService {
[...]
    String getFilePath(String userId, Task task);
}
```

_UFTasksServiceImpl.java_
```
public interface UFTasksServiceImpl {
[...]
    @Override
    public String getFilePath(String userId, Task task) {
        return DEFAULT_URI + "/" + userId + "/" + task.getId() + ".task";
    }
}
```