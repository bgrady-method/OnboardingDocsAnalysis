Account

A company or organization that is either in trial, or has purchased a license to our platform.



Action



A basic building block of our no-code editor. It is any unit that allows us to define logic to execute. ie. Send Email, Update Controls On Screen, Save All etc. See our external documentation for more detail



Action Set

A collection of actions that allow us to define complicated business logic by grouping actions. Action sets are associated with control actions, Screens, and Apps (in the form of App Routines)



Action Editor

This is the screen that is used to edit action sets. Although they look similar, the action editor used in screen Action Sets is not the same as the Action Editor used to maintain App Routines. 



App

Apps are collections of Runtime Screens and App Routines that are grouped together. When they are installed from Method, they are called Stock Apps, and when they are created, they are Custom Apps.



App builder

The new editor visual IDE where you assemble screens, define data models, and wire up logic without coding.



App Routine

An Action Set that can be defined once at the app level, and reused across screens. It comes with a few limitations however – it doesn’t have access to the screen context, so it can’t update controls or show messages, and it can’t call other App Routines. 



App Ribbon

A way for Method Apps "talk" to one another — they display related information from a different app and serve as a shortcut directly to that information. Our external documentation has some examples



Bound

A component whose value is directly linked to a data field or query—reads from and writes to the database automatically.



Black box components

A self‑contained widget (e.g., chart, checkout module) whose internal logic is part of the software implementation, not the configuration in our screen designer/action editor.Black boB



Component

A reusable UI element (button, input, grid, modal) that you configure and place on screens.



Contact

An important Table for associating a person with a lead or a customer within an account. (See Table)



Control

The general term used to describe things that can be dragged onto the designer canvas: includes objects, fields, and containers.



Customization

Customization refers to modifications made to a screen in the backend by administrators or customizers. These changes are typically applied at the company level and affect how all users see the screen.  Happens within the "app builder" and was previously done in the "designer". The platform is in a transition from designer to app builder.

Standard views are an example of customizations. These are built-in views created by customizers in the app builder and cannot be altered by regular users on the runtime screen. Even if standard views have underlying filters, these are not exposed to regular users.

Customizers have permissions beyond regular users, such as the ability to modify the screen for the entire company. They can build and define the base structure and components of a screen.



Configuration

Configuration refers to the ability of a regular user to modify certain aspects of a screen at runtime to personalize their view. These changes are typically user-specific and do not affect how other users see the same screen.

Examples of configurations include:

◦ Hiding or showing columns in a grid.

◦ Sorting columns.

◦ Filtering data.

◦ Setting the number of records displayed per page on a grid.

Configurations are done on the runtime screen. When a user makes a configuration change on a grid (e.g., reordering columns), this is considered a configuration, not a customization.

Users generally have the ability to configure screens by default, but administrators can control whether users have permission to perform certain configurations. For example, the ability to hide or show columns can be enabled or disabled.

Changes made through configuration are generally saved at the user level.



Custom Views

Custom Views allow regular users to take an existing view (often a standard view that has been customized) and modify it according to their needs, saving it as a personalized view.

Users can set filters, sort orders, and choose which columns are visible in their custom view.

Custom views can be either personal (only visible to the user who created them) or shared with everyone.

Unlike standard views (which are customizations), custom views (which are configurations) can be further edited and modified by the user on the runtime screen.

The functionality for custom views with editable grids was still under development at the time of the conversation, which was a reason why editable grids hadn't been fully rolled out.



Customer

Our customers' customers. A customer is anyone who makes payments to accounts for goods and services. 


Note: This is not how we refer to our customers



Designer

A mode in Method that allows users to manipulate elements of a screen and even build a screen from the ground up.  



Diffs

The “difference” view showing changes between two versions or branches—e.g., what was added, removed, or modified.



Data models

Schemas, relationships, and rules that define how objects relate and enforce data integrity.



EDA

Event‑Driven Architecture: a design pattern where logic responds to streams of events (user or system) rather than linear code paths.



Event

A detectable occurrence or change in system state that triggers an Action set. ie. Click Event (user clicks a button), Change Event (user updates a text box), Load Event (user navigates to a screen).



Field

Defines what type of information is stored for each record. There are many different field types you will use in Method.



Forking

Creating a divergent copy (branch) of an app or component for experimentation, which can lead to version drift and merge conflicts.



Grid

Unique Control used to display information in a specific order. Similar to grids in Excel spreadsheets. (See Control for more information.)



Grid Views

This is the set of filters, sort orders and display columns that are defined for displaying a grid. When the view is defined within the Grid Builder or the Designer, it is called a Standard View. When the view is created by using the runtime screen, these are called “Custom Views”

❌

Platform solution

The core no‑code environment or suite of services (e.g., auth, hosting, integrations) that underpins all custom screens—like the “OS” your app runs on.



Runtime Screens (Also abbreviated to “Screens”)

The “runtime screen” is the live, end‑user interface you see when the app is running. It’s built and customized in the no‑code App Builder rather than coded by developers. In other words, it’s the actual screen your users interact with, configured on the fly via the no‑code builder’s WYSIWYG editor rather than pre‑rendered in static code.



Stock App

A “stock app” is a pre‑built, out‑of‑the‑box application template provided by the platform (e.g. contacts, invoices, reports). These stock apps come with default screens and components that you customize or configure in the no‑code builder to suit your needs, instead of building an app entirely from scratch.



 System Page

“System pages” are the core functional screens implemented in true code and maintained in Storybook as the source‑of‑truth components. To make these pages customizable at runtime, their Storybook components must be ported into the no‑code builder so designers and admins can drag‑and‑drop and configure them in the App Builder.



Sync widget

A UI element that manages scheduled or real‑time synchronization between the front end and external data sources or APIs.



Screen GUID

A system‑generated global unique identifier (e.g., screen_3f5a6b2d) for each screen, used for versioning and analytics.



Screen designer

The old editor



Table

A table is a stored collection of related data. For example, Method has a table called the contacts table which stores all the contacts.



Tenant

A sub-organization that exists under the parent account. 

Eg. No-Frills would be a Tenant of Loblaws



User

These are the people who have access to accounts. 



Fields

 Individual attributes on an object (e.g., status, due_date) that hold specific data types.



Bound

A component whose value is directly linked to a data field or query—reads from and writes to the database automatically.



Unbound

A component not tied to a data source; its value must be set/read via actions rather than automatically persisted.



Manual

A process or update triggered explicitly by a user action (e.g., clicking “Save”) rather than running automatically.Man



Mapped

Linking one field or object to another (e.g., mapping a “Customer ID” dropdown to the underlying record’s ID).



Record

A single entry (row) in an object or table (e.g., one Customer record).



Record ID

The unique key for a record (often auto‑generated, e.g., cus_01A23B4C).



Tables and fields

The platform’s database analog: tables ≈ objects, fields ≈ columns.



App Builder

The new editor visual IDE where you assemble screens, define data models, and wire up logic without coding.



WYSIWYG

“What You See Is What You Get”: an editor mode where the on‑canvas design matches exactly what end users will experience.

