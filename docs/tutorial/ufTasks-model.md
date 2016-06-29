###The Model

Arguably, the most important design task when creating any application is defining the data structures and relationships
that represent the objects and concepts in the real world that we are trying to model.
Since even minor model changes can affect the entire application, it's important to get this right early in the design process.

For our UFTasks application, we want to be able to categorize our TO-DO list tasks by "Projects";
a Project in our case will simply have a descriptive name, for example "Household chores" or "My First Uberfire Application".
We also want to further organize our tasks by "Folders", which groups the Tasks by topics.
In our application, Folders have only a descriptive name.

From this design requirement, our model becomes evident: essentially we have a tree structure with Projects at the top, Folders as child nodes of Projects and Tasks as child nodes of Folders.

We will create all of our model classes in the org.uberfire.shared.model package in the uftasks-webapp project.

Let's use a generic class called TreeNode which we can extend for Projects, Folders and Tasks.

```
public class TreeNode<PARENT extends TreeNode, CHILD extends TreeNode> {
    private PARENT parent;
    private List<CHILD> children;

    public TreeNode() {
        parent = null;
    }

    public String getName() {
        return "noname";
    }

    public PARENT getParent() {
        return (PARENT) parent;
    }

    public void setParent(PARENT parent) {
        this.parent = parent;
    }

    public List<CHILD> getChildren() {
        if (children == null)
            children = new ArrayList<CHILD>();
        return children;
    }

    public void addChild(CHILD child) {
        getChildren().add(child);
        child.setParent(this);
    }
}
```

The generic type parameters PARENT and CHILD define the parent and child class types of this node type. This will make more sense when we look at the Project class:

```
public class Project extends TreeNode<TasksRoot, Folder> {

    private final String name;
    private boolean selected;

    public Project(String name) {
        this.name = name;
        this.selected = false;
    }

    public String getName() {
        return name;
    }

    public boolean isSelected() {
        return selected;
    }

    public void setSelected(boolean selected) {
        this.selected = selected;
    }
}
```

A Project has a name and a "selected" flag indicating if it is currently the selected or "active" project. Here, the PARENT is a TasksRoot type, and the CHILD is a Folder type.

TasksRoot is the root of our tree and looks like this:

```
public class TasksRoot extends TreeNode<TreeNode,Project>{
    
    public TasksRoot() {
    }
    
}
```

There's not much in here because all of the tree navigation functionality is in the TreeNode base class.
Note that since the root has no PARENT type, we will just use TreeNode as a place holder; its parent will be null.

The Folder node looks like this:

```
public class Folder extends TreeNode<Project, Task> {

    private final String name;

    public Folder(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

As expected, the PARENT is a Project and the CHILD is a Task type.

Finally, the Task class:

```
public class Task extends TreeNode<Folder, TreeNode> {
    private String name;
    private boolean done;

    public Task(String name) {
        this.name = name;
        this.done = false;
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
}
```

Again, no surprises here except this is a "leaf" node in the tree so it has no children and we'll use TreeNode as the CHILD type parameter.