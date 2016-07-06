####Resource Types

A Resource Type simply identifies the content of a file, similar in concept to the Eclipse Content Types, except that Uberfire associates only the file extension with the Resource Type - there is no "peeking" inside the file to determine what's in it.

When defining a Workbench Editor, you must provide one or more Resource Types that the editor can handle. For example, a text editor may declare that it can be used for both a Text Resource Type and an XML Resource Type. The editor may also declare a "priority" to help Uberfire determine the best editor to use for a particular Resource Type. Thus, if your application contains both an advanced XML editor with syntax highlighting and tag completion and a bunch of other cool stuff, along with a plain text editor, Uberfire will choose the XML editor if it declares a higher priority in the **@WorkbenchEditor** annotation.

For our Task Editor we will create a new **ResourceType** to handle **Task** objects. The **ResourceType** definition looks like this:

_TaskResourceType.java_

```
package org.uberfire.client.editors;

import javax.enterprise.context.ApplicationScoped;

import org.uberfire.backend.vfs.Path;
import org.uberfire.client.workbench.type.ClientResourceType;

import com.google.gwt.user.client.ui.IsWidget;

@ApplicationScoped
public class TaskResourceType implements ClientResourceType {
 
    @Override
    public String getShortName() {
        return "task";
    }
 
    @Override
    public String getDescription() {
        return "TO-DO Task file";
    }
 
    @Override
    public String getPrefix() {
        return "";
    }
 
    @Override
    public String getSuffix() {
        return "task";
    }
 
    @Override
    public int getPriority() {
        return 0;
    }
 
    @Override
    public String getSimpleWildcardPattern() {
        return "*.task";
    }
 
    @Override
    public boolean accept(Path path) {
        return path.getFileName().endsWith( "." + getSuffix() );
    }
 
    @Override
    public IsWidget getIcon() {
        return null;
    }
 
}
```

This is pretty straightforward: it simply defines a file extension of "task" and provides some descriptive text and an icon for use with, for example, a file browser screen if one were provided. Now when we ask the Place Manager to open a VFS file with a ".task" extension, it will know to instantiate our Task Editor.
