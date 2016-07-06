###The Presenter

In Tutorial 3, we saw that the **TasksPresenter#showTaskEditor()** method was responsible for displaying a Popup dialog that contained our task editing Screen. We will still use this method, but replace it with a Place Manager request to open our **WorkbenchEditor**:

_TasksPresenter.java_
```
    public void showTaskEditor(final Task task) {
	    // get the ".task" file name from UFTasksService
        ufTasksService.call(new RemoteCallback<String>() {
            @Override
            public void callback(final String response) {
                if (response!=null) {
                    String filename = response.replaceFirst(".*/", "");
                    Path path = PathFactory.newPath(filename, response);
                    PathPlaceRequest place = new PathPlaceRequest(path);
                    
                    // add the Task fields to Place Request
                    place.addParameter("id", task.getId());
                    place.addParameter("name", task.getName());
                    place.addParameter("dueDate", DateFormat.getDateInstance().format(task.getDueDate()));
                    place.addParameter("priority", ""+task.getPriority());
                    placeManager.goTo(place);
                }
                else 
                    GWT.log("UFTasksService is unable to load tasks file");
            }
        }).getFilePath(user.getIdentifier(), task);
    }

```

The Place Manager provides a simple mechanism for passing information to a WorkbenchEditor, using a list of a name/value pairs. Unfortunately, only String values are supported, so we have to convert some of our Task fields (**dueDate** and **priority**) to String. These must then be parsed in the WorkbenchEditor into their proper objects.

The Task Editor Presenter (our **@WorkbenchEditor**) class must provide a new lifecycle method identified by the **@OnStartup** annotation. This method is responsible for loading the data for the **Task** being edited.

We'll create this class in a new package, org.uberfire.client.editors:

_TaskEditorPresenter.java_
```
package org.uberfire.client.editors;

import javax.annotation.PostConstruct;
import javax.enterprise.context.Dependent;
import javax.enterprise.event.Event;
import javax.inject.Inject;

import org.jboss.errai.common.client.api.Caller;
import org.jboss.errai.common.client.api.ErrorCallback;
import org.jboss.errai.common.client.api.RemoteCallback;
import org.uberfire.backend.vfs.ObservablePath;
import org.uberfire.backend.vfs.Path;
import org.uberfire.backend.vfs.VFSService;
import org.uberfire.client.annotations.WorkbenchEditor;
import org.uberfire.client.annotations.WorkbenchMenu;
import org.uberfire.client.annotations.WorkbenchPartTitle;
import org.uberfire.client.annotations.WorkbenchPartTitleDecoration;
import org.uberfire.client.annotations.WorkbenchPartView;
import org.uberfire.client.mvp.UberView;
import org.uberfire.client.screens.TasksPresenter;
import org.uberfire.component.model.TaskWithNotes;
import org.uberfire.lifecycle.OnStartup;
import org.uberfire.mvp.PlaceRequest;
import org.uberfire.mvp.impl.PathPlaceRequest;
import org.uberfire.shared.events.TaskChangedEvent;
import org.uberfire.workbench.model.menu.Menus;

import com.google.gwt.core.shared.GWT;
import com.google.gwt.user.client.ui.IsWidget;

@Dependent
@WorkbenchEditor(identifier = "TaskEditor", supportedTypes = { TaskResourceType.class })
public class TaskEditorPresenter {

    public interface View extends UberView<TaskEditorPresenter> {
        IsWidget getTitleWidget();
        void setContent(final TaskWithNotes content);
        TaskWithNotes getContent();
        boolean isDirty();
    }

    @Inject
    private View view;

    @Inject
    private Event<TaskChangedEvent> taskChangedEvent;

    @Inject
    private Caller<VFSService> vfsServices;
    
    @Inject
    private TasksPresenter tasksPresenter;
    
    // the task being edited
    private TaskWithNotes taskWithNotes;

    private PathPlaceRequest placeRequest;
    
    @PostConstruct
    public void setup() {
        view.init( this );
    }

    @OnStartup
    public void onStartup( final ObservablePath path,
                           final PlaceRequest place ) {
        placeRequest = (PathPlaceRequest) place;
        // Create a Task object and populate it with
        // the parameters passed in by the TasksPresenter 
        Task task = new Task(place.getParameter("name", ""));
        try {
            task.setId(place.getParameter("id", ""));
            task.setDueDate(
                    DateFormat.getDateInstance().parse(
                    place.getParameter("dueDate", "")));
            task.setPriority(Integer.parseInt(place.getParameter("priority", "")));
        }
        catch (Exception e) {
        }
        taskWithNotes = new TaskWithNotes(task);
        loadTaskNotes(path);
    }
    
    private void loadTaskNotes(ObservablePath path) {
        vfsServices.call(
            new RemoteCallback<String>() {
                @Override
                public void callback(String response) {
                    if ( response != null ) {
                        taskWithNotes.setNotes(response);
                    }
                    view.setContent(taskWithNotes);
                }
            }, new ErrorCallback<Object>() {
    
                @Override
                public boolean error(Object message, Throwable throwable) {
                    view.setContent(taskWithNotes);
                    return false;
                }
            }
        ).readAllString( path );
    }
    
    private void saveTaskNotes(ObservablePath path, String notes) {
        vfsServices.call(
                new RemoteCallback<Path>() {
                    @Override
                    public void callback(Path response) {
                        if ( response != null ) {
                            GWT.log("saveTaskNotes SUCCEEDED: "+response);
                        }
                    }
                }, new ErrorCallback<Object>() {
        
                    @Override
                    public boolean error(Object message, Throwable throwable) {
                        GWT.log("saveTaskNotes FAILED: "+message);
                        return false;
                    }
                }
            ).write(path, notes);
    }
    
    @WorkbenchPartTitle
    public String getTitleText() {
        return "Task Editor";
    }

    @WorkbenchPartTitleDecoration
    public IsWidget getTitle() {
        return view.getTitleWidget();
    }

    @WorkbenchPartView
    public IsWidget asWidget() {
        return view;
    }

    @WorkbenchMenu
    public Menus getMenus() {
        return null;
    }

    public void save() {
        if (view.isDirty()) {
            TaskWithNotes changedTask = view.getContent();
            if (!taskWithNotes.getNotes().equals(changedTask.getNotes()))
                saveTaskNotes(placeRequest.getPath(), changedTask.getNotes());
            if (!changedTask.asTask().equals(taskWithNotes.asTask()))
                taskChangedEvent.fire(new TaskChangedEvent(changedTask));
        }
    }
    
    public void close() {
        tasksPresenter.closeTaskEditor(placeRequest);
    }
}
```

Note here that our Presenter needs to save the **placeRequest** because we'll need that again when it comes time to close the editor. Also, we can see where the **Task** field values are reconstructed from the Place Manager parameters that were passed by the **TasksPresenter** Screen.

As before, when the user clicks the "OK" button, our editor checks to see if anything in the **Task** has changed and, if so, fires the **TaskChangedEvent**.

Finally, we ask the **TasksPresenter** to close the editor, using the original **placeRequest** that was used to open it.
