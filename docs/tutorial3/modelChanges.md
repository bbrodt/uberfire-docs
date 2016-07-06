###Model Changes

With the new requirements, we will have to make some changes in the model. First, the **Task** object needs to be extended with the new attributes. We have also added a new **set()** method which simply copies the values from another **Task** object.

_Task.java_
```
@Portable
public class Task extends TreeNode<Folder, TreeNode> {
    private String name;
    private boolean done;
    private int priority;
    private Date dueDate;
    private String id;

    public Task(@MapsTo("name") String name) {
        this.name = name;
        this.done = false;
        priority = 0;
        dueDate = new Date();
        
        // Yes we should probably use a UUID here to ensure uniqueness,
        // but this is good enough for our purposes...
        this.id = Long.toString(System.currentTimeMillis());
    }
    
    public Task(Task that) {
        set(that);
    }
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    
    public boolean isDone() {
        return done;
    }

    public void setDone(boolean done) {
        this.done = done;
    }

    public int getPriority() {
        return priority;
    }

    public void setPriority(int priority) {
        this.priority = priority;
    }

    public Date getDueDate() {
        return dueDate;
    }

    public void setDueDate(Date dueDate) {
        this.dueDate = dueDate;
    }

    public String getId() {
        return id;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Task) {
            Task that = (Task)obj;
            if (!this.getName().equals(that.getName()))
                return false;
            if (this.isDone() != that.isDone())
                return false;
            if (this.getPriority() != that.getPriority())
                return false;
            if (!this.getDueDate().equals(that.getDueDate()))
                return false;
            return true;
        }
        return super.equals(obj);
    }

    public void set(Task that) {
        this.name = that.name;
        this.done = that.done;
        this.priority = that.priority;
        this.dueDate = that.dueDate;
        this.id = that.id;
    }
}
```

We will also add a support method to our **TasksRoot** object for locating a **Task** by its ID:

_TasksRoot.java_
```
@Portable
public class TasksRoot extends TreeNode<TreeNode,Project>{

    public Task getTask(String id) {
        for (Project p : getChildren()) {
            for (Folder f : p.getChildren()) {
                for (Task t : f.getChildren()) {
                    if (t.getId().equals(id)) {
                        return t;
                    }
                }
            }
        }
        return null;
    }
}
```
