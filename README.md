# Mermaid Class Diagram Styles (Dark Theme)

Source: [mermaid.ai — Class diagrams, Styling / Classes / Define Namespace](https://mermaid.ai/open-source/syntax/classDiagram.html). Colors are tuned for a dark rendering background (dark fill, light text, saturated stroke) rather than the light-theme defaults in the upstream docs.

## Class Diagrams

```mermaid
%%{init: {'themeVariables': {'lineColor': '#8b949e'}}}%%
classDiagram
    namespace Api {
        class BaseController {
            #logger : ILogger
        }
        class OrderController {
            -orderService : OrderService
            +submit(order) bool
        }
    }
    namespace Domain {
        class OrderService {
            -repository : IOrderRepository
            ~cache : OrderCache
            +placeOrder(order) bool
            +getInstance() OrderService$
        }
        class OrderLineItem:::memberChanged {
            -sku : string
            +getSubtotal() decimal
            +[add] getTax() decimal
        }
        class IOrderRepository {
            +save(order) bool
        }
        class OrderExportJob:::added {
            +run()
        }
    }
    namespace Infrastructure {
        class SqlOrderRepository:::memberChanged {
            -connectionString : string
            +save(order) bool
            -[rem] legacyConnectionString : string
        }
        class LegacyOrderQueue:::removed {
            -queueName : string
        }
    }

    OrderController --|> BaseController
    OrderController *-- OrderLineItem
    OrderService o-- IOrderRepository
    SqlOrderRepository ..|> IOrderRepository
    OrderController ..> OrderService

    note for OrderService "Coordinates order use cases; delegates persistence to IOrderRepository"

    classDef default fill:#2a2a2a,stroke:#8b949e,color:#c9d1d9,stroke-width:2px
    classDef added stroke:#4a7a5a,stroke-width:2px
    classDef removed stroke:#8a4a4a,stroke-width:2px
    classDef memberChanged stroke:#8b949e,stroke-width:2px,stroke-dasharray: 4 3
```

#### Classes

`classDef default fill:...` defines a **class of styles** applied to every node that has no explicit `:::name`, instead of repeating a `style NodeId fill:...` line per node. Attach a different class to any one node with the `:::name` shorthand, or the quoted form `cssClass "NodeId" name` when the id needs quoting.

## Notes

- Member visibility markers: `+` public (`OrderController.submit`), `-` private (`OrderService.repository`), `#` protected (`BaseController.logger`), `~` internal/package (`OrderService.cache`). A trailing `$` marks a member static (`OrderService.getInstance()$`).
- `classDef`/`:::` only styles whole classes, not individual members — there's no per-line color selector. To flag a single added/deleted field or method inside an otherwise-unstyled class, prefix its name with literal `[add]`/`[rem]` text, e.g. `+[add] getTax() decimal` (`OrderLineItem`) and `-[rem] legacyConnectionString : string` (`SqlOrderRepository`).
- Border style tells added/deleted apart at the class level: a **solid** green (`added`) or red (`removed`) border marks a class that is *entirely* new or deleted (`OrderExportJob`, `LegacyOrderQueue`); a **dashed gray** border (`memberChanged`) marks a class that only has an `[add]`/`[rem]` *member* inside it (`OrderLineItem`, `SqlOrderRepository`) — the class itself isn't new or removed, so it keeps the default gray color.
- Namespaces (`namespace Name { ... }`) group classes visually and logically; they cannot be styled directly with `style`/`classDef` — only the classes inside them can.
- Relationship arrows are not `classDef`-able (`linkStyle`, which works in flowcharts, raises a parse error in `classDiagram`). Set arrow/line color for the whole diagram via the `%%{init: {'themeVariables': {'lineColor': '#8b949e'}}}%%` directive on the first line, before `classDiagram` — verified against `mmdc` 11.16.0. `#8b949e` matches the `classDef default` stroke so arrows blend with the class borders instead of standing out as an unrelated accent color.
- **Inheritance** (`--|>`): the child class carries the same operations/attributes as its parent plus its own extras. `OrderController --|> BaseController`.
- **Aggregation** (`o--`): the container holds instances of another class, but those instances outlive the container — hollow diamond at the container. `OrderService o-- IOrderRepository`. Used for scoped/singleton dependencies injected via DI, since the DI container controls their lifetime.
- **Dependency** (`..>`): a dashed line — changes in one class ripple into another without an ownership relationship. `OrderController ..> OrderService`.
- **Interface implementation** (`..|>`): a class implements an interface. `SqlOrderRepository ..|> IOrderRepository`. The class name (`I`-prefix) already signals the role — no `<<Interface>>`/`<<Abstract>>` stereotype text is added.
- **Composition** (`*--`): like aggregation, but the contained instances are deleted along with the container — solid diamond at the container. `OrderController *-- OrderLineItem`. Used for transient dependencies injected via DI (the DI container controls their lifetime), and for instances created with `new` inside the container class.
- `note for ClassName "..."` attaches a free-text note to one class (`OrderService` above) — useful for a short explanation that doesn't belong in the class body.
- `classDef added`/`classDef removed` only override `stroke`, so the change status shows as a green or red **border** on top of the same gray fill/text as every other class. Colors are deliberately muted (`#4a7a5a`, `#8a4a4a`) rather than saturated, to stay legible without drawing more attention than the class names themselves.

## Sequence Diagrams

Source: [mermaid.ai — Sequence diagrams](https://mermaid.ai/open-source/syntax/sequenceDiagram.html).

```mermaid
%%{init: {'themeVariables': {
    'lineColor': '#8b949e',
    'actorBkg': '#2a2a2a', 'actorBorder': '#8b949e', 'actorTextColor': '#c9d1d9', 'actorLineColor': '#8b949e',
    'signalColor': '#8b949e', 'signalTextColor': '#c9d1d9',
    'labelBoxBkgColor': '#2a2a2a', 'labelBoxBorderColor': '#8b949e', 'labelTextColor': '#c9d1d9',
    'loopTextColor': '#c9d1d9',
    'noteBkgColor': '#2a2a2a', 'noteBorderColor': '#8b949e', 'noteTextColor': '#c9d1d9',
    'activationBorderColor': '#8b949e', 'activationBkgColor': '#2a2a2a',
    'sequenceNumberColor': '#c9d1d9'
}}}%%
sequenceDiagram
    actor User
    participant Api as OrderController
    participant Svc as OrderService
    participant Repo as IOrderRepository
    participant Queue as OrderExportJob

    User->>Api: submit(order)
    activate Api
    Api->>Svc: placeOrder(order)
    activate Svc
    Svc->>Repo: save(order)
    activate Repo
    Repo-->>Svc: bool
    deactivate Repo
    alt order valid
        Svc->>Queue: run()
        Queue-->>Svc: ack
    else order invalid
        Svc-->>Api: throws ValidationError
    end
    Svc-->>Api: bool
    deactivate Svc
    Api-->>User: 200 OK
    deactivate Api

    note over Svc,Repo: persistence is transactional
```

#### Notes

- `actor`/`participant` both declare a lifeline; `actor` renders as a stick figure and is reserved for the human/external initiator (`User`), while `participant` renders as a box and is used for system components. The `as` alias (`participant Api as OrderController`) keeps arrow lines short while still labeling the box with the real class name.
- Solid arrow with filled head (`->>`) is a synchronous call/request; dashed arrow with filled head (`-->>`) is the matching return/response. Pair every `->>` with a `-->>` from the same target so the diagram reads as request/response, not fire-and-forget.
- `activate`/`deactivate` (or the `+`/`-` shorthand on the arrow, e.g. `Api->>+Svc:`) draws the vertical activation bar showing how long a participant is doing work — nest them to show a call chain still on the stack (`Api` stays active while it waits on `Svc`, which stays active while it waits on `Repo`).
- `alt`/`else`/`end` branches the diagram for mutually exclusive outcomes (validation success vs. failure) — same semantics as an `if/else`, rendered as a labeled box split by a dashed line. Use `opt`/`end` instead when there's only one conditional branch with no alternative.
- `note over A,B: text` spans a free-text note across two lifelines (`Svc` and `Repo`) to call out a cross-cutting concern (transactionality) that doesn't belong on a single arrow.
- Sequence diagrams have no `classDef`/`:::` styling mechanism, so matching the class diagram's dark palette requires per-property `themeVariables` instead: `actorBkg`/`labelBoxBkgColor`/`noteBkgColor`/`activationBkgColor` all use the same `#2a2a2a` fill, `actorBorder`/`labelBoxBorderColor`/`noteBorderColor`/`activationBorderColor`/`signalColor` all use the same `#8b949e` stroke, and `actorTextColor`/`labelTextColor`/`noteTextColor`/`signalTextColor`/`loopTextColor`/`sequenceNumberColor` all use the same `#c9d1d9` text color as `classDef default` in the class diagram.
- `lineColor` is kept alongside the new variables for consistency with the class/flowchart diagrams, though `signalColor` is what actually controls arrow color in a sequence diagram.

## Block Diagrams (Flowcharts)

Source: [mermaid.ai — Flowcharts](https://mermaid.ai/open-source/syntax/flowchart.html).

```mermaid
%%{init: {'themeVariables': {'lineColor': '#8b949e'}}}%%
flowchart TD
    User(["User"])
    Api["OrderController"]
    Svc["OrderService"]
    Repo[("IOrderRepository")]
    Queue["OrderExportJob"]:::added
    Legacy["LegacyOrderQueue"]:::removed

    User --> Api
    Api --> Svc
    Svc --> Repo
    Svc -- valid order --> Queue
    Svc -. deprecated .-> Legacy

    subgraph Infrastructure
        Repo
        Legacy
    end

    classDef default fill:#2a2a2a,stroke:#8b949e,color:#c9d1d9,stroke-width:2px
    classDef added stroke:#4a7a5a,stroke-width:2px
    classDef removed stroke:#8a4a4a,stroke-width:2px
```

#### Notes

- Node shape signals role, not just style: `(["..."])` stadium shape for an external actor (`User`), plain `["..."]` rectangle for a process/component (`Api`, `Svc`), and `[("...")]` cylinder for a data store/repository (`Repo`) — matching the ER-diagram convention of "cylinder = storage".
- `flowchart TD` lays the graph out top-down; `flowchart LR` (left-to-right) reads better for pipelines with many parallel branches. Pick the direction that keeps the diagram narrower than it is tall (or vice versa) for the surrounding page layout.
- Solid arrow (`-->`) is the default/normal flow edge; a labeled solid arrow (`-- text -->`) documents the condition under which that edge is taken (`Svc -- valid order --> Queue`); a dotted arrow (`-.text.->`) marks a deprecated or exceptional path (`Svc -. deprecated .-> Legacy`) — same visual vocabulary as dashed vs. solid relationship lines in the class diagram.
- `subgraph Name ... end` groups existing nodes into a labeled box without redeclaring them — same purpose as `namespace` in class diagrams, but flowchart `subgraph`s can also be targets of edges (an arrow can point at the whole box instead of one node inside it).
- `classDef`/`:::` styling works identically to class diagrams here: `added`/`removed` classes reuse the same muted green/red stroke convention to flag new (`Queue`) or deleted (`Legacy`) nodes at a glance.
- The same dark-theme `classDef default` and `%%{init: ...}%%` line-color directive are reused verbatim from the class-diagram section so all three diagram types render consistently on a dark background.

## C4 Container diagram (C4Container)

Source: [mermaid.ai — C4 Diagrams](https://mermaid.ai/open-source/syntax/c4.html). C4 is Mermaid's **experimental** diagram type for system-context/container/component/deployment views; use it for the "solution diagram" level of a design instead of a flowchart.

```mermaid
C4Container
    title Container diagram for Order Management System

    Person(customer, "Customer", "Places and tracks orders")
    System_Ext(email_system, "E-Mail System", "Sends order confirmation e-mails")
    System_Ext(mainframe, "Mainframe Banking System", "Processes payments")

    Container_Boundary(c1, "Order Management System") {
        Container(api, "OrderController", "ASP.NET Core", "Accepts and validates order submissions")
        Container(svc, "OrderService", "C# / .NET", "Coordinates order placement and persistence")
        ContainerDb(db, "Order Database", "SQL Server", "Stores orders and line items")
        ContainerQueue(queue, "OrderExportJob", "Azure Service Bus", "Publishes fulfilled orders downstream")
    }

    Rel(customer, api, "Submits order", "HTTPS")
    Rel(api, svc, "Places order via")
    Rel(svc, db, "Reads from and writes to", "EF Core")
    Rel(svc, queue, "Publishes to", "async")
    Rel(svc, mainframe, "Charges payment via", "sync/async, HTTPS")
    Rel(svc, email_system, "Sends confirmation via", "SMTP")

    UpdateElementStyle(customer, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#4a5a8a")
    UpdateElementStyle(api, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(svc, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(db, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(queue, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(email_system, $fontColor="#c9d1d9", $bgColor="#1a1a1a", $borderColor="#8b949e")
    UpdateElementStyle(mainframe, $fontColor="#c9d1d9", $bgColor="#1a1a1a", $borderColor="#8b949e")
    UpdateRelStyle(customer, api, $textColor="#c9d1d9", $lineColor="#8b949e", $offsetY="-10")
    UpdateRelStyle(svc, mainframe, $textColor="#c9d1d9", $lineColor="#8b949e", $offsetY="20", $offsetX="-30")
```

#### Notes

- C4 has five diagram kinds selected by the first keyword: `C4Context` (system context — actors and systems only), `C4Container` (adds the containers inside one system, used above), `C4Component` (adds the components inside one container), `C4Dynamic` (numbered interaction steps, like a simplified sequence diagram), and `C4Deployment` (nests `Deployment_Node`s to show infrastructure placement). Pick the shallowest kind that answers the question the diagram needs to answer — don't drop to `C4Component` just to show one extra box.
- Element macros encode both role and internal/external ownership in the name itself: `Person`/`Person_Ext`, `System`/`System_Ext`, `SystemDb`/`SystemDb_Ext`, `Container`/`Container_Ext`, `ContainerDb`/`ContainerDb_Ext`, `ContainerQueue`/`ContainerQueue_Ext`, and their `Component*` equivalents inside `C4Component`. The `_Ext` suffix marks something outside the team's control (`System_Ext(email_system, ...)`, `System_Ext(mainframe, ...)`) — same distinction the flowchart section draws with node shape, but here it's carried by the macro name instead.
- `Person` shape rendering is renderer-version-dependent: `mmdc` 11.16.0 draws a head-and-body icon above the label, but other renderers (e.g. the VS Code Markdown preview's bundled Mermaid) fall back to the same plain rounded rectangle used for `System`/`Container`. Don't rely on shape alone to tell `Person` apart when the design doc might be viewed in either renderer — `customer`'s border uses a separate muted blue (`#4a5a8a`) instead of the shared `#8b949e`, so the human actor stays distinguishable by color even where the icon doesn't render.
- `Container_Boundary(id, "label") { ... }` groups the containers owned by one system, same role as flowchart `subgraph` or class-diagram `namespace`; `System_Boundary` and the generic `Boundary(id, "label", "type")` exist for the equivalent grouping at the `C4Context`/custom level.
- `Rel(from, to, label, ?technology)` is the base relationship; `BiRel` draws it bidirectional, and `Rel_Back`/`Rel_U`/`Rel_D`/`Rel_L`/`Rel_R` reverse the arrowhead or bias the layout direction without changing the semantics — layout in C4 is controlled by statement order and these directional hints, not an automatic layout engine.
- **C4 has no `classDef`/`:::` and no diagram-specific `themeVariables`** (confirmed by the upstream docs: "C4 diagram is fixed style, such as css color, so different css is not provided under different skins") — this is the one diagram type in this file that can't be palette-matched with an `%%{init: ...}%%` line-color directive the way class/sequence/flowchart diagrams are. The only per-element/per-relationship overrides available are `UpdateElementStyle(id, $fontColor=..., $bgColor=..., $borderColor=...)` and `UpdateRelStyle(from, to, $textColor=..., $lineColor=..., $offsetX=..., $offsetY=...)` — apply them individually to every element/relationship that needs to match the dark palette (`customer`, `api`, `svc`, `db`, `queue` above), there is no `default` class to set once.
- External elements (`System_Ext`, `Container_Ext`, `ContainerDb_Ext`, `ContainerQueue_Ext`) get a darker `$bgColor="#1a1a1a"` than owned elements' `#2a2a2a` (`email_system`, `mainframe` above), same `$borderColor="#8b949e"` and `$fontColor="#c9d1d9"` otherwise — the darker fill is the only signal distinguishing "outside the team's control" once the `_Ext` suffix itself renders identically to its owned counterpart.
- `$offsetX`/`$offsetY` on `UpdateRelStyle` nudge a relationship's label off the line when it would otherwise collide with a box or another label (`svc, mainframe` above) — there is no automatic label-collision avoidance in C4, unlike flowchart/sequence diagrams.

## C4 Deployment view, modeled as C4Container

Source: [mermaid.ai — C4 Diagrams](https://mermaid.ai/open-source/syntax/c4.html); the classic C4-PlantUML "Internet Banking System" deployment example, ported from `C4Deployment` to `C4Container`. Each top-level `Deployment_Node` (mobile device, customer's computer, Big Bank plc data center) becomes a `Container_Boundary`, and every nested `Deployment_Node` beneath it (web browser, server host, Apache Tomcat, Oracle instance) becomes a generic `Boundary` nested inside — `C4Container` has no infrastructure-placement macro of its own, so `Boundary` is reused purely as a grouping label instead.

```mermaid
C4Container
    title Deployment view as C4Container (Internet Banking System)

    Person(customer, "Personal Banking Customer", "A customer of the bank, with personal bank accounts.")
    System_Ext(mainframe, "Mainframe Banking System", "Stores all of the core banking information about customers, accounts, transactions, etc.")

    Container_Boundary(mob, "Customer's mobile device", "Apple iOS or Android") {
        Container(mobile, "Mobile App", "Xamarin", "Provides a limited subset of the Internet Banking functionality to customers via their mobile device.")
    }

    Container_Boundary(comp, "Customer's computer", "Microsoft Windows or Apple macOS") {
        Boundary(browser, "Web Browser", "Google Chrome, Mozilla Firefox, Apple Safari or Microsoft Edge") {
            Container(spa, "Single Page Application", "JavaScript and Angular", "Provides all of the Internet Banking functionality to customers via their web browser.")
        }
    }

    Container_Boundary(plc, "Big Bank plc", "Big Bank plc data center") {
        Boundary(dn, "bigbank-api*** x8", "Ubuntu 16.04 LTS") {
            Boundary(apache, "Apache Tomcat", "Apache Tomcat 8.x") {
                Container(api, "API Application", "Java and Spring MVC", "Provides Internet Banking functionality via a JSON/HTTPS API.")
            }
        }
        Boundary(bb2, "bigbank-web*** x4", "Ubuntu 16.04 LTS") {
            Boundary(apache2, "Apache Tomcat", "Apache Tomcat 8.x") {
                Container(web, "Web Application", "Java and Spring MVC", "Delivers the static content and the Internet Banking single page application.")
            }
        }
        Boundary(bigbankdb01, "bigbank-db01", "Ubuntu 16.04 LTS") {
            Boundary(oracle, "Oracle - Primary", "Oracle 12c") {
                ContainerDb(db, "Database", "Relational Database Schema", "Stores user registration information, hashed authentication credentials, access logs, etc.")
            }
        }
        Boundary(bigbankdb02, "bigbank-db02", "Ubuntu 16.04 LTS") {
            Boundary(oracle2, "Oracle - Secondary", "Oracle 12c") {
                ContainerDb(db2, "Database", "Relational Database Schema", "Stores user registration information, hashed authentication credentials, access logs, etc.")
            }
        }
    }

    Rel(customer, mobile, "Uses")
    Rel(customer, spa, "Uses")
    Rel(mobile, api, "Makes API calls to", "json/HTTPS")
    Rel(spa, api, "Makes API calls to", "json/HTTPS")
    Rel_U(web, spa, "Delivers to the customer's web browser")
    Rel(api, db, "Reads from and writes to", "JDBC")
    Rel(api, db2, "Reads from and writes to", "JDBC")
    Rel_R(db, db2, "Replicates data to")
    Rel(api, mainframe, "Makes API calls to", "XML/HTTPS")

    UpdateElementStyle(customer, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#4a5a8a")
    UpdateElementStyle(mobile, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#4a7a5a")
    UpdateElementStyle(spa, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(api, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(web, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(db, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(db2, $fontColor="#c9d1d9", $bgColor="#2a2a2a", $borderColor="#8b949e")
    UpdateElementStyle(mainframe, $fontColor="#c9d1d9", $bgColor="#1a1a1a", $borderColor="#8b949e")

    UpdateRelStyle(customer, mobile, $textColor="#c9d1d9", $lineColor="#8b949e")
    UpdateRelStyle(customer, spa, $textColor="#c9d1d9", $lineColor="#8b949e")
    UpdateRelStyle(mobile, api, $textColor="#c9d1d9", $lineColor="#8b949e")
    UpdateRelStyle(spa, api, $textColor="#c9d1d9", $lineColor="#8b949e", $offsetY="-40")
    UpdateRelStyle(web, spa, $textColor="#c9d1d9", $lineColor="#8b949e", $offsetY="-40")
    UpdateRelStyle(api, db, $textColor="#c9d1d9", $lineColor="#8b949e", $offsetY="-20", $offsetX="5")
    UpdateRelStyle(api, db2, $textColor="#c9d1d9", $lineColor="#8b949e", $offsetX="-40", $offsetY="-20")
    UpdateRelStyle(db, db2, $textColor="#c9d1d9", $lineColor="#8b949e", $offsetY="-10")
    UpdateRelStyle(api, mainframe, $textColor="#c9d1d9", $lineColor="#8b949e")
```

#### Notes

- `Container_Boundary` is only used for the three **top-level** `Deployment_Node`s (`mob`, `comp`, `plc`); every nested `Deployment_Node` beneath them uses the generic `Boundary(id, label, ?type)` macro instead, since `C4Container` has no second boundary macro dedicated to sub-groupings the way `Deployment_Node` can nest inside `Deployment_Node` in `C4Deployment`.
- The `type` parameter (`Deployment_Node`'s second string argument, e.g. `"Ubuntu 16.04 LTS"`, `"Apache Tomcat 8.x"`) maps directly onto `Boundary`'s optional third argument, so no infrastructure detail is lost in the port — it just renders as a boundary subtitle instead of a deployment-node subtitle.
- Colors match this file's C4Container section exactly (`$fontColor="#c9d1d9"`, `$bgColor="#2a2a2a"`, `$borderColor="#8b949e"` on every owned `Container`/`ContainerDb`; `$textColor="#c9d1d9"`, `$lineColor="#8b949e"` on every `Rel`), with the original `$offsetX`/`$offsetY` label-nudges preserved on top since C4's `UpdateRelStyle` combines both sets of parameters on one call.
- `customer` (`Person`) gets the same muted blue `$borderColor="#4a5a8a"` as the `C4Container` section's `customer` above, so `Person` elements stay visually consistent across every C4 diagram in this file; `mainframe` (`System_Ext`) gets the same darker `$bgColor="#1a1a1a"` external styling introduced above, keeping it visually distinct from the owned `Container`/`ContainerDb` nodes' `#2a2a2a` fill.
- `mobile` is marked as a newly added container by giving it the `$borderColor="#4a7a5a"` green border instead of the shared `#8b949e` gray — the same muted green the class/flowchart `classDef added` convention uses to flag brand-new nodes. C4's `UpdateElementStyle` has no `:::`/`classDef` class mechanism, so the border color has to be set per-element rather than by attaching an `added` class once.
- This diagram trades `C4Deployment`'s infrastructure-placement semantics for staying inside one diagram type: there's no way to say "this is a deployment node, not a logical grouping" once `Boundary` is reused for physical hosts, and nesting five levels deep (`plc` → `dn` → `apache` → `api`) is visually busier than the flatter groupings `C4Container` diagrams normally have.
