# [Eclipse Tutorial](https://www.tutorialspoint.com/eclipse/index.htm)

## Overview

### What is Eclipse

Eclipse can be used as an IDE for **any programming language** for which a plug-in is available.

The **Java Development Tools (JDT)** project provides a plug-in that allows Eclipse to be used as a Java IDE, **PyDev** is a plugin that allows Eclipse to be used as a Python IDE, **C/C++ Development Tools (CDT)** is a plug-in that allows Eclipse to be used for developing application using C/C++, the Eclipse Scala plug-in allows Eclipse to be used an IDE to develop Scala applications and PHPeclipse is a plug-in to eclipse that provides complete development tool for PHP.

### Licensing

Eclipse platform and other plug-ins from the Eclipse foundation is released under the Eclipse Public License (EPL). EPL ensures that Eclipse is **free to download and install**. It also allows Eclipse to be modified and distributed.

## Installation

### Downloading Eclipse

You can download eclipse from [http://www.eclipse.org/downloads/](http://www.eclipse.org/downloads/]). The download page lists a number of flavors of eclipse.

The capabilities of each packaging of eclipse are different. Java developers typically use **Eclipse Classic** or **Eclipse IDE for developing Java applications**.

### Installing Eclipse

### Launching Eclipse

When eclipse starts up for the first time it prompts you for the location of the workspace folder. **All your data will be stored in the workspace folder**. You can accept the default or choose a new location.

If you select "Use this as the default and do not ask again", this dialog box will not come up again. You can change this preference using the **Workspaces Preference Page**.

## Explore Windows

### Parts of an Eclipse Window

An eclipse **perspective** is the name given to an initial collection and arrangement of views and an editor area. The default perspective is called java. An eclipse window can have multiple perspectives open in it but **only one perspective can be active** at any point of time. A user can switch between open perspectives or open a new perspective. A **perspective controls what appears in some menus and tool bars**.

A perspective has only **one editor area** in which **multiple editors** can be open. The editor area is usually surrounded by **multiple views**. In general, editors are used to **edit the project data** and views are used to **view the project metadata**. For example the package explorer shows the java files in the project and the java editor is used to edit a java file.

The eclipse window can contain multiple editors and views but only one of them is active at any given point of time. The title bar of the active editor or view looks different from all the others.

The UI elements on the menu bar and tool bar represent commands that can be triggered by an end user.

### Using Multiple Windows

Multiple Eclipse Windows can be open at the same time. To open a new window, click on the Windows menu and select the New Window menu item.

Each window can have a different perspective open in them. For example you could open two Eclipse windows one in the Java perspective and the other in the Debug perspective. The window showing the Java perspective can be used for editing the java code and the window showing the debug perspective can be used for debugging the application being developed.

## Explore Menus

### Typical Eclipse Menus

Plug-ins can add new menus and menu items. For example when the java editor is open you will see the Source menu and when the XML editor is open, you will see the Design menu.

### Brief Description of Menus

## Explore Views

### About Views

Eclipse views allow users to see a **graphical representation of project metadata**. For example the project navigator view presents a graphical representation of the **folders** and **files** associated with a project and properties view presents a graphical representation of an element selected in another view or editor.

An eclipse perspective can show any number of **views** and **editors**. All editor instances appear in **a single editor area**, whereas views are placed inside **view folders**. A workbench window can display any number of view folders. Each view folder can display one or more views.

## Perspectives

### What is a Perspective

An eclipse perspective is the name given to an **initial collection and arrangement** of **views** and an **editor area**. The default perspective is called java. An eclipse window can have **multiple perspectives open** in it but **only one perspective is active** at any point of time. A user can switch between open perspectives or open a new perspective. The active perspective controls what appears in some menus and tool bars.

## Workspaces

### About Eclipse Workspace

The workspace has a hierarchical structure. **Projects** are at the top level of the hierarchy and inside them you can have **files** and **folders**. Plug-ins use an API provided by the resources plug-in to manage the resources in the workspace.

## Create Java Project

### Opening the New Java Project wizard

The New Java Project wizard can be used to create a new java project.

### Using the New Java Project wizard

The New Java Project Wizard has two pages. On the first page −

Enter the Project Name

Select the **Java Runtime Environment (JRE)** or leave it at the default

Select the Project Layout which determines whether there would be a **separate folder** for the **source codes** and **class files**. The recommended option is to **create separate folders for sources and class files**.

You can click on the Finish button to create the project or click on the Next button to change the java build settings.

On the second page you can **change the Java Build Settings** like setting the **Project dependency** (if there are multiple projects) and adding **additional jar files** to the build path.

### Viewing the Newly Created Project

## Create Java Package

### Opening the New Java Package wizard

### Using the New Java Package Wizard

Once the Java Package wizard comes up −

* Enter/confirm the source folder name.

* Enter the package name.

* Click on the Finish button.

### Viewing the Newly Created Package

The package explorer will show the newly created package under the source folder.

## Create Java Class

### Opening the New Java Class Wizard

Before bringing up the New Java Class wizard, if possible, **select the package** in which the class is to be created so that the wizard can **automatically fill in the package name** for you.

### Using the New Java Class Wizard

Once the java class wizard comes up −

* Ensure the **source folder** and **package** are correct.

* Enter the class name.

* Select the appropriate **class modifier**.

* Enter the **super class** name or click on the Browse button to search for an existing class.

* Click on the Add button to select the **interfaces** implemented by this class.

* Examine and modify the check boxes related to method stubs and comments.

* Click the Finish button.

### Viewing the Newly Created Java class

The newly created class should appear in the Package Explorer view and a **java editor instance** that allows you to modify the new class. It should appear in the editor area.

## Create Java Interface

### Opening the New Java Interface Wizard

Before bringing up the New Java Interface wizard, if possible, **select the package** in which the interface is to be created so that the wizard can **automatically fill in the package name** for you.

### Using the New Java Interface Wizard

Once the java interface wizard comes up −

* Ensure the **source folder** and **package** are correct.

* Enter the **interface name**.

* Click on the Add button to select the **extended interfaces**.

* Select the Generate comments check box if you like **comments** to be generated.

* Click on the Finish button.

### Viewing the Newly Created Java Interface

The newly created interface should appear in the Package Explorer view and a java editor instance that allows you to modify the new interface should appear in the editor area.

## Create XML File

### Using the New XML File wizard

Once the New XML File wizard comes up − 

* Enter or select the parent folder.

* Enter the name of the xml file.

* Click on the Next button to base the xml file on **DTD**, **XML Schema** or **XML template** else click on Finish.

### Viewing the Newly Created XML File

The newly created XML file should appear in the Package Explorer view and an XML editor instance that allows you to modify the newly created XML file should appear in the editor area.

The XML editor allows you to edit an XML file using either the **Design view** or **Source view**.

## Java Build Path

The Java build path is used while compiling a Java project to discover dependent classes . It is made up of the following items −

* Code in the **source folders**.

* **Jars** and **classes folder** associated with the project.

* **Classes** and **libraries** exported by projects **referenced** by this project.

The **java build path** can be seen and modified by using the **Java Build Path page** of the **Java Project properties dialog**.

To bring up the Java Project properties dialog box, right click on a Java Project in the Package Explorer view and select the Properties menu item. On the left hand side tree select Java Build Path.

A common requirement seen while developing java applications is to **add existing jars to the java build path**. This can be accomplished using the **Libraries tab**. In the Libraries tab, just click on **Add JARs** if the jar is already in the Eclipse workspace or click on **Add External JARs** if the jar is elsewhere in the file system.

## Run Configuration

### Creating and Using a Run Configuration

The **Run Configurations dialog** allows you create **multiple run configurations**. **Each** run configuration **can start an application**. The Run Configuration dialog can be invoked by selecting the **Run Configurations menu** item from the **Run menu**.

To create a run configuration for a Java application select Java Application from the list on the left hand side and click on the New button. In the dialog box that comes up in the main tab specify −

* A name for the **run configuration**.

* The name of a **Project**.

* The name of the main class.

In the arguments tab specify −

* Zero or more program arguments.

* Zero or more Virtual Machine arguments.

The Commons tab provides **common options** such as the ability to allocate a console for standard input and output.

To save the run configuration click on the Apply button and to launch the application click on the Run button.

## Running Program

### Running a Java Program

The quickest way to run a Java program is by using the Package Explorer view.

In the Package Explorer view −

* Right click on the **java class** that **contains the `main` method**.

* Select **Run As** → Java Application.

The same action can be performed using the Package Explorer view by **selecting the class** that **contains the `main` method** and clicking **`Alt + Shift + X, J`**.

Either actions mentioned above **create a new Run Configuration** and use it to start the Java application.

If a Run configuration has already been created you can use it to start the Java application by **selecting Run Configurations** from the **Run menu**, clicking on the name of the run configuration and then clicking on the Run button.

The Run item on the Run menu can be used to **restart** the java application that was previously started.

The shortcut key to launch the previously launched Java application is `Ctrl + F11`.

## Create Jar Files

### Opening the Jar File wizard

The Jar File wizard can be used to **export the content of a project** into a **jar file**. To bring up the Jar File wizard −

* In the Package Explorer select the **items that you want to export**. If you want to export **all the classes and resources** in the project just select the project.

* Click on the File menu and select Export.

* In the filter text box of the first page of the export wizard type in JAR.

* Under the Java category select JAR file.

* Click on Next.

### Using the Jar File wizard

In the JAR File Specification page −

* Enter the **JAR file name** and **folder**.

* The default is to **export only the classes**. To **export the source code** also, click on the Export Java source files and resources check box.

* Click on Next to change the JAR packaging options.

* Click on Next to change the JAR Manifest specification.

* Click on Finish

## Close Project

### Why Close a Project

An eclipse workspace can contain any number of projects. A project can be either in the **open state** or **closed state**.
Open projects −

* Consume memory.

* Take up **build time** especially when the Clean All Projects (Project → Clean all projects) with the Start a build immediately option is used.

### How to Close a Project

If a project is not under active development it can be closed. To close a project, from the Project select the Close Project menu item.

### Closed Project in Package Explorer

A closed project is visible in the Package Explorer view but its **contents cannot be edited** using the Eclipse user interface. Also, an open project **cannot have dependency on a closed project**. The Package Explorer view uses a different icon to represent a closed project.

## Reopen Project

### Reopening a Closed Project

To reopen a closed project, in the Package Explorer view, select the closed project and click on the Project menu and select Open Project.

Once the project is open its content can be edited using the Eclipse user interface.

## Build Project

### Building a Java Project

A project can have **zero or more builders** associated with it. **A java project is associated with a java builder**. To see the builders associated with a project −

* In the Package Explorer view right click on the project and select Properties.

* In the left hand side tree click Builders.

It's the java builder that **distinguishes a Java project** from other types of projects. By click on the New button you can associate the **Ant builder** with a java project. The java builder is responsible for **compiling** the java source code and **generating classes**.

The java builder is notified of changes to the resources in a workspace and can **automatically compile** java code. To disable automatic compilation deselect the **Build Automatically option** from the Project menu.

If automatic compilation is disabled then you can **explicitly build a project** by selecting the Build Project menu item on the Project menu. **The Build Project menu item is disabled if the Build Automatically menu item is selected**.

## Debug Configuration

### Creating and Using a Debug Configuration

An eclipse debug configuration is **similar to a run configuration** but it used to **start an application** in the **debug mode**. Because the application is started in the debug mode the users are prompted to switch to the **debug perspective**. The debug perspective offers a number of views that are suitable for debugging applications.

The **Debug Configuration dialog** can be invoked by selecting the **Debug Configurations menu** item from the **Run menu**.

To create a debug configuration for a Java application, select Java Application from the list on the left hand side and click on the New button. In the dialog box that comes up in the main tab specify −

* A name for the **debug configuration**.

* The name of a **Project**.

* The name of a **main class**.

In the arguments tab, specify −

* Zero or more **program arguments**.

* Zero or more **Virtual Machine arguments**.

To save the run configuration, click on the Apply button and to launch the application in the debug mode click on the Debug button.

## Debugging Program

### Debugging a Java Program

The quickest way to debug a Java program is to using the Package Explorer view. In the Package Explorer view −

* Right click on the java **class** that contains the **`main` method**.

* Select Debug As → Java Application.

The same action can be performed using the Package Explorer by selecting the **class** that **contains the `main` method** and clicking `Alt + Shift + D, J`.

Either actions mentioned above **create a new** Debug Configuration and **use it to** start the Java application.

If a Debug configuration has **already been created** you can use it to start the Java application by selecting **Debug Configurations** from the **Run menu**, clicking on the name of the debug configuration and then clicking on the **Debug button**.

The Debug menu item on the Run menu can be used to **restart** the java application that was previously started in the debug mode.

The shortcut key to launch the previously launched Java application in the debug mode is `F11`. When a java program is started in the debug mode, users are prompted to switch to the debug perspective. The debug perspective offers additional views that can be used to troubleshoot an application.

The java editor allows users to place **break points** in the java code. To set a break point, in the editor area right click on the marker bar and select Toggle Breakpoint.The java editor allows users to place break points in the java code. To set a break point, in the editor area right click on the marker bar and select Toggle Breakpoint.

## Preferences

## Content Assist

### Using Content Assist

Within an editor, content assist helps **reduce the characters typed** by providing a context sensitive list of possible completions to the characters already typed. The context assist can be invoked by **clicking `Ctrl + Space`**.

## Quick Fix

## Hover Help

## Search Menu

## Navigation

## Refactoring

## Add Bookmarks

## Task Management

## Install Plugins

## Code Templates

## Shortcuts

## Restart Option

## Tips & Tricks

## Web Browsers
