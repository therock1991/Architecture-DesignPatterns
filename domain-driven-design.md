# DDD Pattern 
- **Ubiquitous language**
  - Ubiquitous language is a model that acts as a universal language to help communication between software developers and domain experts.
  - Collaborating, learning, and defining a model brings a lot of initial communication barriers between software specialists and domain experts. So evolving domain model with practicing the same type of communications (discussions, writings, and in diagrams) within a context is paramount for successful implementations, and that sort of conversation is called Ubiquitous Language. It is structured around the domain model and extensively used by all the team members within a bounded context. It should be the medium or mode to connect all the activities of the team within the development of software.

The design team can establish deep understanding and connecting domain jargons and software entities with Ubiquitous language to keep discovering and evolving their domain models.
- **Domain Layer**
  - This layer is the heart of business software.
    - Representing concepts of the business, information about the business situation, and business rules.
    - State that reflects the business situation is controlled and used here even though the technical details of storing it are delegated to the infrastructure
  - Following the Persistence **Ignorance and the Infrastructure Ignorance** principles, this layer must completely ignore data persistence details. 
    - These persistence tasks should be performed by the infrastructure layer.
    - Therefore, this layer should not take direct dependencies on the infrastructure, which means that an important rule is that your domain model entity classes should be POCOs.
    - Domain entities should not have any direct dependency (like deriving from a base class) on any data access infrastructure framework like Entity Framework or NHibernate
    - Ideally, your domain entities should not derive from or implement any type defined in any infrastructure framework.
    - Most modern ORM frameworks like Entity Framework Core allow this approach, so that your domain model classes are not coupled to the infrastructure.
    -  For example
      - As part of an order entity class you must have business logic and operations implemented as methods for tasks such as adding an order item, data validation, and total calculation.
      - The entity's methods take care of the invariants and rules of the entity instead of having those rules spread across the application layer.
  - **Aggregate Roots**
    - Aggregate roots as consistency boundaries.
    -   When I interact with an Aggregate, its invariants must always be satisfied
    - The root Entity has global identity and is ultimately responsible for checking invariants
    - Root Entities have global identity.  Entities inside the boundary have local identity, unique only within the Aggregate.
    - Nothing outside the Aggregate boundary can hold a reference to anything inside, except to the root Entity.  The root Entity can hand references to the internal Entities to other objects, but they can only use them transiently (within a single method or block).
    - Only Aggregate Roots can be obtained directly with **database queries**.  Everything else must be done through traversal.
    - Objects within the Aggregate can hold references to other Aggregate roots.
    - A delete operation must remove everything within the Aggregate boundary all at once
  - When a change to any object within the Aggregate boundary is committed, all invariants of the whole Aggregate must be satisfied.
- **Domain events**
  - Intro
    - An event is something that has happened in the past.
    - A domain event is, something that happened in the domain that you want other parts of the same domain (in-process) to be aware of. The notified parts usually react somehow to the events.
  - What used for?
    - Use domain events to explicitly implement side effects of changes within your domain.
    - Use domain events to explicitly implement side effects across multiple aggregates
    - Optionally, for better scalability and less impact in database locks, use eventual consistency between aggregates within the same domain.
  - For example
    - If you're just using Entity Framework and there has to be a reaction to some event, you would probably code whatever you need close to what triggers the event.
    - So the rule gets coupled, implicitly, to the code, and you have to look into the code to, hopefully, realize the rule is implemented there.
  - On the other hand, using domain events makes the concept explicit, because there is a DomainEvent and at least one DomainEventHandler involved.  
    - For example, in the eShopOnContainers application, when an order is created, the user becomes a buyer, so an OrderStartedDomainEvent is raised and handled in the ValidateOrAddBuyerAggregateWhenOrderStartedDomainEventHandler, so the underlying concept is evident.
    - In short, domain events help you to express, explicitly, the domain rules, based in the ubiquitous language provided by the domain experts. Domain events also enable a better separation of concerns among classes within the same domain.
    - It's important to ensure that, just like a database transaction, either all the operations related to a domain event finish successfully or none of them do.
  - Domain events are similar to messaging-style events with one important difference.
    - With real messaging, message queuing, message brokers, or a service bus using AMQP, a message is always sent asynchronously and communicated across processes and machines. This is useful for integrating multiple Bounded Contexts, microservices, or even different applications.
    - However, with domain events, you want to raise an event from the domain operation you are currently running, but you want any side effects to occur within the same domain.
    - The domain events and their side effects (the actions triggered afterwards that are managed by event handlers) should occur almost immediately, usually in-process, and within the same domain. Thus,
      - Domain events could be synchronous or asynchronous.
      - Integration events, however, should always be asynchronous.
  - Domain events as a preferred way to trigger side effects across multiple aggregates within the same domain
    - It is **important** to understand that this **event-based communication** is not implemented directly within the aggregates; you need to implement domain event handlers.
    - If executing a command related to one aggregate instance requires additional domain rules to be run on one or more additional aggregates, you should design and implement those side effects to be triggered by domain events.
    - When the user initiates an order, the **Order Aggregate** sends an **OrderStarted** domain event. The OrderStarted domain event is handled by the Buyer Aggregate to create a Buyer object in the ordering microservice, based on the original user info from the **identity microservice** (with information provided in the CreateOrder command).
    - Alternately, you can have the aggregate root subscribed for events raised by members of its aggregates (child entities).
      - For instance, each OrderItem child entity can raise an event when the item price is higher than a specific amount, or when the product item amount is too high. The aggregate root can then receive those events and perform a global calculation or aggregation.
    - Handling the domain events is an application concern. The domain model layer should only focus on the domain logic—things that a domain expert would understand, not application infrastructure like handlers and side-effect persistence actions using repositories. Therefore, the application layer level is where you should have domain event handlers triggering actions when a domain event is raised.
    - If you use domain events, you can create a fine-grained and decoupled implementation by segregating responsibilities using this approach:
      - 1. Send a command (for example, CreateOrder).
      - 2. Receive the command in a **command handler**.
        - Execute a single aggregate's transaction.
        - (Optional) Raise domain events for side effects (for example, OrderStartedDomainEvent).
      - 3. Handle domain events (within the current process) that will execute an open number of side effects in **multiple aggregates or application actions**. For example:
        - Verify or create buyer and payment method.
        - Create and send a related **integration event** to the event bus to propagate states across microservices or trigger external actions like sending an email to the buyer.
        - Handle other side effects.
    - The important difference is that:
      - A command should be processed only **once**.
      - A domain event could be processed **zero or n times**, because it can be received by multiple receivers or event handlers with a different purpose for each handler.
    - Having an open number of handlers per domain event allows you to add as many domain rules as needed, without affecting current code. For instance, implementing the following business rule might be as easy as adding a few event handlers (or even just one):
      - When the total amount purchased by a customer in the store, across any number of orders, exceeds $6,000, apply a 10% off discount to every new order and notify the customer with an email about that discount for future orders.
    - In terms of the ubiquitous language of the domain, since an event is something that happened in the past, the class name of the event should be represented as a **past-tense verb**, like OrderStartedDomainEvent or OrderShippedDomainEvent. That's how the domain event is implemented in the ordering microservice in eShopOnContainers.
      ```csharp
      public class OrderStartedDomainEvent : INotification
      {
          public string UserId { get; }
          public int CardTypeId { get; }
          public string CardNumber { get; }
          public string CardSecurityNumber { get; }
          public string CardHolderName { get; }
          public DateTime CardExpiration { get; }
          public Order Order { get; }

          public OrderStartedDomainEvent(Order order,
                                        int cardTypeId, string cardNumber,
                                        string cardSecurityNumber, string cardHolderName,
                                        DateTime cardExpiration)
          {
              Order = order;
              CardTypeId = cardTypeId;
              CardNumber = cardNumber;
              CardSecurityNumber = cardSecurityNumber;
              CardHolderName = cardHolderName;
              CardExpiration = cardExpiration;
          }
      }
      ```
    - An important characteristic of events is that since an event is something that happened in the **past**, it should **not change**. Therefore, it must be an **immutable** class. You can see in the previous code that the properties are **read-only**. There's no way to update the object, you can only set values when you create it.
    - It's important to highlight here that if **domain events** were to be handled **asynchronously**, using a queue that required serializing and deserializing the event objects, the properties would have to be "private set" instead of read-only, so the deserializer would be able to assign the values upon dequeuing. This is not an issue in the Ordering microservice, as the domain event pub/sub is implemented synchronously using MediatR.
- **Domain events versus integration events**
  - Domain and integration events are the same thing: notifications about something that just happened.
    - However, their implementation must be different.
    - **Domain events** are just messages pushed to a domain event dispatcher, which could be implemented as an in-memory mediator based on an IoC container or any other method.
    - **Integration events** is to propagate **committed transactions** and updates to additional **subsystems**, whether they are other **microservices, Bounded Contexts or even external applications**. Hence, they should occur **only** if the entity is successfully **persisted**, otherwise it's as if the entire operation never happened.
      -  Integration events must be based on **asynchronous communication** between multiple microservices (other Bounded Contexts) or even external systems/applications.
      - The **event bus** interface needs some infrastructure that allows **inter-process and distributed communication** between potentially remote services. It can be based on a commercial **service bus, queues, a shared database** used as a mailbox, or any other distributed and ideally push based messaging system.
- **Raise domain events**
  - when the domain events class is static, it also dispatches to handlers immediately. 
  - This makes testing and debugging more difficult, because the event handlers with side-effects logic are executed immediately after the event is raised
  - When you are testing and debugging, you want to focus on and just what is happening in the current aggregate classes; you do not want to suddenly be redirected to other event handlers for side effects related to other aggregates or application logic. This is why other approaches have evolved, as explained in the next section.
- **The deferred approach to raise and dispatch events**
  - Instead of dispatching to a domain event handler immediately, a better approach is to add the domain events to a collection and then to dispatch those domain events right before or right after committing the transaction (as with SaveChanges in EF)
  - Deciding if you send the domain events right before or right after committing the transaction is important, since it determines whether you will include the side effects as part of the **same transaction or in different transactions**. In the latter case, you need to deal with **eventual consistency** across multiple **aggregates**
    ```csharp
    // EF Core DbContext
    public class OrderingContext : DbContext, IUnitOfWork
    {
        // ...
        public async Task<bool> SaveEntitiesAsync(CancellationToken cancellationToken = default(CancellationToken))
        {
            // Dispatch Domain Events collection.
            // Choices:
            // A) Right BEFORE committing data (EF SaveChanges) into the DB. This makes
            // a single transaction including side effects from the domain event
            // handlers that are using the same DbContext with Scope lifetime
            // B) Right AFTER committing data (EF SaveChanges) into the DB. This makes
            // multiple transactions. You will need to handle eventual consistency and
            // compensatory actions in case of failures.
            await _mediator.DispatchDomainEventsAsync(this);

            // After this line runs, all the changes (from the Command Handler and Domain
            // event handlers) performed through the DbContext will be committed
            var result = await base.SaveChangesAsync();
        }
    }
    ```
  - Be aware that transactional boundaries come into significant play here.
    - If your unit of work and transaction can span more than one aggregate (as when using EF Core and a relational database), this can work well. 
    - But if the transaction cannot span aggregates, such as when you are using a NoSQL database like Azure CosmosDB, you have to implement additional steps to achieve consistency. This is another reason why persistence ignorance is not universal; it depends on the storage system you use.
- **Single transaction across aggregates versus eventual consistency across aggregates**
  - A single transaction across aggregates 
    - Any rule that spans Aggregates will not be expected to be up-to-date at all times. Through event processing, batch processing, or other update mechanisms, other dependencies can be resolved within some specific time
  - An eventual consistency across those aggregates
    - Thus, if executing a command on one aggregate instance requires that additional business rules execute on one or more aggregates, use eventual consistency, There is a practical way to support eventual consistency in a DDD model.
    - An aggregate method publishes a domain event that is in time delivered to one or more asynchronous subscribers.
  - This rationale is based on embracing fine-grained transactions instead of transactions spanning many aggregates or entities.
  - The idea is that in the **second case**, the number of database **locks** will be substantial in **large-scale** applications with high scalability needs.
  -  the fact that highly scalable applications need **not** have instant transactional consistency between multiple aggregates helps with accepting the concept of eventual consistency
  - If you dispatch the domain events right before committing the original transaction
    - It is because you want the side effects of those events to be included in the same transaction.
    - For example, if the EF DbContext SaveChanges method fails, the transaction will roll back all changes, including the result of any side effect operations implemented by the related domain event handlers.
    - This is because the DbContext life scope is by default defined as "scoped." Therefore, the DbContext object is shared across multiple repository objects being instantiated within the same scope or object graph. This coincides with the HttpRequest scope when developing Web API or MVC apps.
  - Consider that if you commit changes to the original aggregate and afterwards, when the events are being dispatched, if there is an issue and the event handlers cannot commit their side effects, you will have inconsistencies between aggregates.
    - A way to allow compensatory actions would be to store the domain events in **additional database tables** so they can be part of the original transaction. 
    - Afterwards, you could have a batch process that detects inconsistencies and runs compensatory actions by comparing the list of events with the current state of the aggregates
  - Raising the events **before** committing, so you use a single transaction—is the simplest approach when using EF Core and a relational database. It is easier to implement and valid in many business cases.
  - For better scalability and less impact on database locks, use eventual consistency between aggregates within the same domain.
  - The reference app uses MediatR to propagate domain events synchronously across aggregates, within a single transaction.
  - However, you could also use some AMQP implementation like **RabbitMQ or Azure Service Bus** to propagate domain events asynchronously, using **eventual consistency** but, as mentioned above, you have to consider the need for **compensatory** actions in case of **failures**.
- **Domain events can generate integration events to be published outside of the microservice boundaries**
  - It's important to mention that you might sometimes want to propagate events across multiple microservices.
  - That propagation is an integration event, and it could be published through an event bus from any specific domain event handler.
- **Services**:
  - A good Service has these characteristics:
    - The operation relates to a domain concept that is not a natural part of an Entity or Value Object
    - The interface is defined in terms of other elements in the domain model
    - The operation is stateless
  - But Services exist in most layers of the DDD layered architecture:
    - **Application Services or Application Layer**
      - Defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems
      - The tasks this layer is responsible for are meaningful to the business or necessary for interaction with the application layers of other systems
      - This layer is kept thin.
        - It does not contain **business rules or knowledge**, but only coordinates tasks and delegates work to collaborations of domain objects in the next layer down
        - It does not have **state** reflecting the business situation, but it can have state that reflects the progress of a task for the user or the program.
        - The application layer must only coordinate tasks and must not hold or define any domain state (domain model).
        - It **delegates** the execution of business rules to the domain model classes themselves (aggregate roots and domain entities), which will ultimately update the data within those domain entities.
      - What used for?
        - The microservice's interaction
        - remote network access
        - external Web APIs used from the UI or client apps.
        - Infra services 
        - Other App Services
      - Are the interface used by the outside world, where the outside world can’t communicate via our Entity objects, but may have other representations of them.
      - Application Services could map outside messages to internal operations and processes, **communicating with services in the Domain and Infrastructure layers** to provide cohesive operations for outside clients.
      - **Messaging patterns** tend to rule Application Services, as the other service layers don’t have a reference back out to the Application Services.
      - **Business rules** are not allowed in an Application Service, those belong in the Domain layer.
    - **Domain services**
      - Are the coordinators, allowing higher level functionality between many different smaller parts.  These would include things like OrderProcessor, ProductFinder, FundsTransferService, and so on.
      - Since Domain Services are first-class citizens of our domain model, their names and usages should be part of the Ubiquitous Language.
      -  Meanings and responsibilities should make sense to the stakeholders or domain experts.
      - In many cases, the software we write is replacing or supplementing a human’s job, such as Order Processor, so it’s often we find inspiration in the existing business process for names and responsibilities.  Where an existing name doesn’t fit, we dive into the domain to try and surface a hidden concept with the domain expert, which might have existed but didn’t have a name.
      - Domain Services (or just Services in DDD) is used to perform domain operations and business rules. In his DDD book, Eric Evans describes a good Service in three characteristics:
        - The operation relates to a domain concept that is not a natural part of an Entity or Value Object.
        - The interface is defined in terms of other elements of the domain model.
        - The operation is stateless.
      - Unlike Application Services which get/return Data Transfer Objects, a Domain Service gets/returns domain objects (like entities or value types).
      - A Domain Service can be used by Application Services and other Domain Services, but not directly by the presentation layer (application services are for that).
      - Domain Service allows you to capture logic that doesn't belong in the Domain Entity.
      - Domain Service allows you to orchestrate between different Domain Entities.      
      - Tips:
        - Don't create too many Domain Services, most of the logic should reside in the domain entities, event handlers, etc. 
        - It's a great place for calculation and validation as it can access entities, and other kind of objects (e.g. Settings) that are not available via the entity graph.
        - Methods should return primitive types, custom enums are fine too.

        ```csharp
        public interface ITaskManager : IDomainService
        {
            void AssignTaskToPerson(Task task, Person person);
        }
        ```
      - As you can see, the TaskManager service works with domain objects: a Task and a Person. There are some conventions when naming domain services. It can be TaskManager, TaskService or TaskDomainService...
        ```csharp
        public class TaskManager : DomainService, ITaskManager
        {
            public const int MaxActiveTaskCountForAPerson = 3;

            private readonly ITaskRepository _taskRepository;

            public TaskManager(ITaskRepository taskRepository)
            {
                _taskRepository = taskRepository;
            }

            public void AssignTaskToPerson(Task task, Person person)
            {
                if (task.AssignedPersonId == person.Id)
                {
                    return;
                }

                if (task.State != TaskState.Active)
                {
                    throw new ApplicationException("Can not assign a task to a person when task is not active!");
                }

                if (HasPersonMaximumAssignedTask(person))
                {
                    throw new UserFriendlyException(L("MaxPersonTaskLimitMessage", person.Name));
                }

                task.AssignedPersonId = person.Id;
            }

            private bool HasPersonMaximumAssignedTask(Person person)
            {
                var assignedTaskCount = _taskRepository.Count(t => t.State == TaskState.Active && t.AssignedPersonId == person.Id);
                return assignedTaskCount >= MaxActiveTaskCountForAPerson;
            }
        }
        ```
      - We have two business rules here:
        - A task should be in an Active state in order for it to be assigned to a new Person.
        - A person can have a maximum of 3 active tasks.
      - Wondering why we throw an ApplicationException for the first check and UserFriendlyException for the second check (see exception handling)? This is not related to domain services at all. This is just an example, it's completely up to you. The user interface must check a task's state and should not allow us to assign it to a person. This is an application-level error and we may want to hide it from user.
      - Using the Domain Service from an Application Service or Command Handler
        ```csharp
        public class TaskAppService : ApplicationService, ITaskAppService
        {
            private readonly IRepository<Task, long> _taskRepository;
            private readonly IRepository<Person> _personRepository;
            private readonly ITaskManager _taskManager;

            public TaskAppService(IRepository<Task, long> taskRepository, IRepository<Person> personRepository, ITaskManager taskManager)
            {
                _taskRepository = taskRepository;
                _personRepository = personRepository;
                _taskManager = taskManager;
            }

            public void AssignTaskToPerson(AssignTaskToPersonInput input)
            {
                var task = _taskRepository.Get(input.TaskId);
                var person = _personRepository.Get(input.PersonId);

                _taskManager.AssignTaskToPerson(task, person);
            }
        }
        ```
      - The Task Application Service uses a given DTO (input), then uses repositories to retrieve that related task and person. Finally, it passes them to the Task Manager (the domain service).
      - Why not use only the Application Services?
        - You may wonder why the application service itself does not implenent the logic contained in the domain service.
        - We can simply say that it's not an application service task. Because it's not a use-case, instead, it's a business operation, we may end up using the same 'assign a task to a user' domain logic in a different use-case. Say that we have another screen to somehow update the task. This updating can include assigning the task to another person. We can use the same domain logic there. We may also have 2 different UIs (one mobile application and one web application) that share the same domain or we may have a web API for remote clients that includes a task-assigning operation.
        - If your domain is simple, will only have one UI, and assigning a task to a person can be done at just a single point, then you may consider skipping domain services and implementing the logic in your application service. This is not the best practice for DDD, but ASP.NET Boilerplate does not force you to use such a design.
    - **Infrastructure Service**
      - Its initially held in domain entities (in memory) is persisted in databases or another persistent store. An example is using Entity Framework Core code to implement the Repository pattern classes that use a DBContext to persist data in a relational database.
      - The infrastructure layer must not "contaminate" the domain model layer. You must keep the domain model entity classes agnostic from the infrastructure that you use to persist data (EF or any other framework) by not taking hard dependencies on frameworks
      - Would be something like our IEmailSender, that communicates directly with external resources, such as the file system, registry, SMTP, database, etc.
      - Something like NHibernate would show up in the Infrastructure.
- **Repositories**
  - Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.
  - Repositories, in practice, are used to perform database operations for domain objects (Entity and Value types). Generally, a separate repository is used for each Entity (or Aggregate Root).
  - With EF Core code or any other infrastructure dependencies and code (Linq, SQL, etc.), must not be implemented within the domain model
  - For example, the following example with the IOrderRepository interface defines what operations the OrderRepository class will need to implement at the infrastructure layer. In the current implementation of the application, the code just needs to add or update orders to the database, since queries are split following the simplified CQRS approach.
    ```csharp
    // Defined at IOrderRepository.cs
    public interface IOrderRepository : IRepository<Order>
    {
        Order Add(Order order);

        void Update(Order order);

        Task<Order> GetAsync(int orderId);
    }

    // Defined at IRepository.cs (Part of the Domain Seedwork)
    public interface IRepository<T> where T : IAggregateRoot
    {
        IUnitOfWork UnitOfWork { get; }
    }
    ```
  - **Define one repository per aggregate**
    - For each aggregate or aggregate root, you should create one repository class.
    - To achieve the goal of the aggregate root to maintain transactional consistency between all the objects within the aggregate, you should **never** create a repository for each table in the database.
    - the only channel you should use to update the database should be the repositories
    - This is because they have a one-to-one relationship with the aggregate root, which controls the aggregate's invariants and transactional consistency-
    - It's okay to query the database through other channels (as you can do following a CQRS approach), because queries don't change the state of the database
    - However, the transactional area (that is, the updates) must always be controlled by the repositories and the aggregate roots.
    - A repository allows you to populate data in memory that comes from the database in the form of the domain entities. Once the entities are in memory, they can be changed and then persisted back to the database through transactions.
    - if you're using the CQS/CQRS architectural pattern
      - the initial queries are performed by side queries out of the domain model, performed by simple SQL statements using Dapper.
      - This approach is much more flexible than repositories because you can query and join any tables you need, and these queries aren't restricted by rules from the aggregates. That data goes to the presentation layer or client app.
    - Flows:
      - 1. If the user makes changes, the data to be updated comes from the client app or presentation layer to the application layer (such as a Web API service).
      - 2. When you receive a command in a command handler, you use repositories to get the data you want to update from the database.
      - 3. You update it in memory with the data passed with the commands, and you then add or update the data (domain entities) in the database through a transaction.
  - **The Repository pattern makes it easier to test your application logic**
    - The Repository pattern allows you to easily test your application with unit tests. Remember that unit tests only test your code, not infrastructure, so the repository abstractions make it easier to achieve that goal.
    -  It's recommended that you define and place the repository interfaces in the domain model layer so the application layer, such as your Web API microservice, doesn't depend directly on the infrastructure layer where you've implemented the actual repository classes
    -  By doing this and using Dependency Injection in the controllers of your Web API, you can implement mock repositories that return fake data instead of data from the database. This decoupled approach allows you to create and run unit tests that focus the logic of your application without requiring connectivity to the database.
    - Connections to databases can fail and, more importantly, running hundreds of tests against a database is bad for two reasons.
      - First, it can take a long time because of the large number of tests.
      - Second, the database records might change and impact the results of your tests, so that they might not be consistent. Testing against the database isn't a unit test but an integration test. You should have many unit tests running fast, but fewer integration tests against the databases.
    - In terms of separation of concerns for unit tests, your logic operates on domain entities in memory. It assumes the repository class has delivered those. Once your logic modifies the domain entities, it assumes the repository class will store them correctly. The important point here is to create unit tests against your domain model and its domain logic. Aggregate roots are the main consistency boundaries in DDD.
  - **The difference between the Repository pattern and the legacy Data Access class (DAL class) pattern**
    - A **data access object** directly performs data access and persistence operations against storage.
    - A **repository** marks the data with the operations you want to perform in the memory of a unit of work object (as in EF when using the DbContext class), but these updates aren't performed immediately to the database.
    - A **unit of work is** referred to as a single transaction that involves multiple **insert, update, or delete** operations.
      - In simple terms, it means that for a specific user action, such as a registration on a website, all the insert, update, and delete operations are handled in a single transaction.
      - This is more efficient than handling multiple database transactions in a chattier way.
      - These multiple persistence operations are performed later in a single action when your code from the application layer commands it.
      - The decision about applying the in-memory changes to the actual database storage is typically based on the Unit of Work pattern. In EF, the Unit of Work pattern is implemented as the **DbContext**.
      - This pattern or way of applying operations against the storage can increase application **performance** and reduce the possibility of **inconsistencies**. 
      - It also reduces **transaction blocking** in the database tables, because all the intended operations are committed as part of one transaction.
      - Therefore, the selected ORM can **optimize** the execution against the database by grouping several update actions within the same transaction, **as opposed** to many small and separate transaction executions.
- **Infrastructure persistence layer with Entity Framework Core**
  - We can now have read-only access to collections by using a public property typed as **IReadOnlyCollection<T>**, which is backed by a private field member for the collection (like a List<T>) in your entity that relies on EF for persistence
  - Previous versions of Entity Framework required collection properties to support **ICollection<T>**, which meant that any developer using the parent entity class could add or remove items through its property collections. That possibility would be **against** the recommended patterns in DDD.
  ```csharp
  public class Order : Entity
  {
      // Using private fields, allowed since EF Core 1.1
      private DateTime _orderDate;
      // Other fields ...

      private readonly List<OrderItem> _orderItems;
      public IReadOnlyCollection<OrderItem> OrderItems => _orderItems;

      protected Order() { }

      public Order(int buyerId, int paymentMethodId, Address address)
      {
          // Initializations ...
      }

      public void AddOrderItem(int productId, string productName,
                              decimal unitPrice, decimal discount,
                              string pictureUrl, int units = 1)
      {
          // Validation logic...

          var orderItem = new OrderItem(productId, productName,
                                        unitPrice, discount,
                                        pictureUrl, units);
          _orderItems.Add(orderItem);
      }
  }
  ```
  - The OrderItems property can only be accessed as read-only using IReadOnlyCollection<OrderItem>. This type is read-only so it is protected against regular external updates.
  - EF Core provides a way to map the domain model to the physical database without "contaminating" the domain model. It is pure .NET POCO code, because the mapping action is implemented in the persistence layer. In that mapping action, you need to configure the fields-to-database mapping. In the following example of the OnModelCreating method from OrderingContext and the OrderEntityTypeConfiguration class, the call to SetPropertyAccessMode tells EF Core to access the OrderItems property through its field.
    ```csharp
    // At OrderingContext.cs from eShopOnContainers
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      // ...
      modelBuilder.ApplyConfiguration(new OrderEntityTypeConfiguration());
      // Other entities' configuration ...
    }

    // At OrderEntityTypeConfiguration.cs from eShopOnContainers
    class OrderEntityTypeConfiguration : IEntityTypeConfiguration<Order>
    {
        public void Configure(EntityTypeBuilder<Order> orderConfiguration)
        {
            orderConfiguration.ToTable("orders", OrderingContext.DEFAULT_SCHEMA);
            // Other configuration

            var navigation =
                  orderConfiguration.Metadata.FindNavigation(nameof(Order.OrderItems));

            //EF access the OrderItem collection property through its backing field
            navigation.SetPropertyAccessMode(PropertyAccessMode.Field);

            // Other configuration
        }
    }
    ```
  - The EF DbContext comes through the constructor through Dependency Injection. It is shared between multiple repositories within the same HTTP request scope, thanks to its default lifetime **(ServiceLifetime.Scoped)** in the IoC container (which can also be explicitly set with services.**AddDbContext<>**).
**Methods to implement in a repository (updates or transactions versus queries)**
  - Within each repository class, you should put the persistence methods that update the state of entities contained by its related aggregate. Remember there is one-to-one relationship between an aggregate and its related repository.
  - Consider that **an aggregate root entity object might have embedded child entities** within its EF graph. For example, a buyer might have multiple payment methods as related child entities.
  - Since the approach for the ordering microservice in eShopOnContainers is also based on CQS/CQRS, most of the queries are **not** implemented in custom **repositories**
  - Most of the custom repositories suggested by this guide have several **update or transactional methods** but just the **query** methods needed to get data to be updated
  -  For example, the BuyerRepository repository implements a FindAsync method, because the application needs to know whether a particular buyer exists before creating a new buyer related to the order.
**EF DbContext and IUnitOfWork instance lifetime in your IoC container**
  - The **DbContext** object (exposed as an IUnitOfWork object) should be **shared** among multiple repositories within the same **HTTP** request scope
  - . For example, this is true when the operation being executed must deal with multiple aggregates, or simply because you are using multiple repository instances.
  - It is also important to mention that the **IUnitOfWork** interface is part of your **domain** layer, **not** an EF Core type.
  - In order to do that, the instance of the DbContext object has to have its service lifetime set to **ServiceLifetime.Scoped**. This is the default lifetime when registering a DbContext with services.AddDbContext in your IoC container from the ConfigureServices method of the Startup.cs file in your ASP.NET Core Web API project.
    ```csharp
    public IServiceProvider ConfigureServices(IServiceCollection services)
    {
        // Add framework services.
        services.AddMvc(options =>
        {
            options.Filters.Add(typeof(HttpGlobalExceptionFilter));
        }).AddControllersAsServices();

        services.AddEntityFrameworkSqlServer()
          .AddDbContext<OrderingContext>(options =>
          {
              options.UseSqlServer(Configuration["ConnectionString"],
                                  sqlOptions => sqlOptions.MigrationsAssembly(typeof(Startup).GetTypeInfo().
                                                                                        Assembly.GetName().Name));
          },
          ServiceLifetime.Scoped // Note that Scoped is the default choice
                                // in AddDbContext. It is shown here only for
                                // pedagogic purposes.
          );
    }
    ```
- **The repository instance lifetime in your IoC container**
  - Repository's **lifetime** should usually be set as **scoped** (InstancePerLifetimeScope in Autofac).
  - It could also be transient (InstancePerDependency in Autofac), but your service will be more efficient in regards **memory** when using the scoped lifetime.
  - Using the **singleton** lifetime for the repository could cause you **serious concurrency problems** when your DbContext is set to scoped (InstancePerLifetimeScope) lifetime (the default lifetimes for a DBContext).
- **Fluent API and the OnModelCreating method**
  - As mentioned, in order to change conventions and mappings, you can use the OnModelCreating method in the DbContext class.
  - The ordering microservice in eShopOnContainers implements explicit mapping and configuration, when needed, as shown in the following code.
    ```csharp
    // At OrderingContext.cs from eShopOnContainers
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      // ...
      modelBuilder.ApplyConfiguration(new OrderEntityTypeConfiguration());
      // Other entities' configuration ...
    }

    // At OrderEntityTypeConfiguration.cs from eShopOnContainers
    class OrderEntityTypeConfiguration : IEntityTypeConfiguration<Order>
    {
        public void Configure(EntityTypeBuilder<Order> orderConfiguration)
        {
            orderConfiguration.ToTable("orders", OrderingContext.DEFAULT_SCHEMA);

            orderConfiguration.HasKey(o => o.Id);

            orderConfiguration.Ignore(b => b.DomainEvents);

            orderConfiguration.Property(o => o.Id)
                .UseHiLo("orderseq", OrderingContext.DEFAULT_SCHEMA);

            //Address value object persisted as owned entity type supported since EF Core 2.0
            orderConfiguration
                .OwnsOne(o => o.Address, a =>
                {
                    a.WithOwner();
                });

            orderConfiguration
                .Property<int?>("_buyerId")
                .UsePropertyAccessMode(PropertyAccessMode.Field)
                .HasColumnName("BuyerId")
                .IsRequired(false);

            orderConfiguration
                .Property<DateTime>("_orderDate")
                .UsePropertyAccessMode(PropertyAccessMode.Field)
                .HasColumnName("OrderDate")
                .IsRequired();

            orderConfiguration
                .Property<int>("_orderStatusId")
                .UsePropertyAccessMode(PropertyAccessMode.Field)
                .HasColumnName("OrderStatusId")
                .IsRequired();

            orderConfiguration
                .Property<int?>("_paymentMethodId")
                .UsePropertyAccessMode(PropertyAccessMode.Field)
                .HasColumnName("PaymentMethodId")
                .IsRequired(false);

            orderConfiguration.Property<string>("Description").IsRequired(false);

            var navigation = orderConfiguration.Metadata.FindNavigation(nameof(Order.OrderItems));

            // DDD Patterns comment:
            //Set as field (New since EF 1.1) to access the OrderItem collection property through its field
            navigation.SetPropertyAccessMode(PropertyAccessMode.Field);

            orderConfiguration.HasOne<PaymentMethod>()
                .WithMany()
                .HasForeignKey("_paymentMethodId")
                .IsRequired(false)
                .OnDelete(DeleteBehavior.Restrict);

            orderConfiguration.HasOne<Buyer>()
                .WithMany()
                .IsRequired(false)
                .HasForeignKey("_buyerId");

            orderConfiguration.HasOne(o => o.OrderStatus)
                .WithMany()
                .HasForeignKey("_orderStatusId");
        }
    }
    ```
- **The Hi/Lo algorithm in EF Core**
  - An interesting aspect of code in the preceding example is that it uses the Hi/Lo algorithm as the key generation strategy.
  - The Hi/Lo algorithm is useful when you need **unique keys** before **committing changes**.
  - As a summary, the Hi-Lo algorithm assigns unique identifiers to table rows while not depending on storing the row in the database immediately. This lets you start using the identifiers right away, as happens with regular sequential database IDs.
  - The Hi/Lo algorithm describes a mechanism for getting a batch of unique IDs from a related database sequence. These IDs are safe to use because the database guarantees the uniqueness, so there will be no collisions between users. This algorithm is interesting for these reasons:
    - It does not break the Unit of Work pattern.
    - It gets sequence IDs in batches, to minimize round trips to the database.
    - It generates a human readable identifier, unlike techniques that use GUIDs.
- **Query Specification pattern**
  - the Query Specification pattern is a Domain-Driven Design pattern designed as the place where you can put the definition of a query with optional sorting and paging logic.
  - The Query Specification pattern defines a query in an object. For example, in order to encapsulate a paged query that searches for some products you can create a PagedProduct specification that takes the necessary input parameters (pageNumber, pageSize, filter, etc.)
    ```csharp
    // GENERIC SPECIFICATION INTERFACE
    // https://github.com/dotnet-architecture/eShopOnWeb

    public interface ISpecification<T>
    {
        Expression<Func<T, bool>> Criteria { get; }
        List<Expression<Func<T, object>>> Includes { get; }
        List<string> IncludeStrings { get; }
    }
    // GENERIC SPECIFICATION IMPLEMENTATION (BASE CLASS)
    // https://github.com/dotnet-architecture/eShopOnWeb

    public abstract class BaseSpecification<T> : ISpecification<T>
    {
        public BaseSpecification(Expression<Func<T, bool>> criteria)
        {
            Criteria = criteria;
        }
        public Expression<Func<T, bool>> Criteria { get; }

        public List<Expression<Func<T, object>>> Includes { get; } =
                                              new List<Expression<Func<T, object>>>();

        public List<string> IncludeStrings { get; } = new List<string>();

        protected virtual void AddInclude(Expression<Func<T, object>> includeExpression)
        {
            Includes.Add(includeExpression);
        }

        // string-based includes allow for including children of children
        // e.g. Basket.Items.Product
        protected virtual void AddInclude(string includeString)
        {
            IncludeStrings.Add(includeString);
        }
    }
    // SAMPLE QUERY SPECIFICATION IMPLEMENTATION

    public class BasketWithItemsSpecification : BaseSpecification<Basket>
    {
        public BasketWithItemsSpecification(int basketId)
            : base(b => b.Id == basketId)
        {
            AddInclude(b => b.Items);
        }

        public BasketWithItemsSpecification(string buyerId)
            : base(b => b.BuyerId == buyerId)
        {
            AddInclude(b => b.Items);
        }
    }
    // GENERIC EF REPOSITORY WITH SPECIFICATION
    // https://github.com/dotnet-architecture/eShopOnWeb

    public IEnumerable<T> List(ISpecification<T> spec)
    {
        // fetch a Queryable that includes all expression-based includes
        var queryableResultWithIncludes = spec.Includes
            .Aggregate(_dbContext.Set<T>().AsQueryable(),
                (current, include) => current.Include(include));

        // modify the IQueryable to include any string-based include statements
        var secondaryResult = spec.IncludeStrings
            .Aggregate(queryableResultWithIncludes,
                (current, include) => current.Include(include));

        // return the result of the query using the specification's criteria expression
        return secondaryResult
                        .Where(spec.Criteria)
                        .AsEnumerable();
    }
    ```
- **Use NoSQL databases as a persistence infrastructure(Not in scope of LB Project**
  - When you use a document-oriented database, you implement an aggregate as a single document, serialized in JSON or another format.
  - However, the use of the database is transparent from a domain model code point of view.
  - When using a NoSQL database, you still are using entity classes and aggregate root classes, but with more flexibility than when using EF Core because the persistence is not relational.
  - For example, 
    - In a document-oriented database, it is okay for an aggregate root to have multiple child collection properties.
    - In a relational database, querying multiple child collection properties is not easily optimized, because you get a UNION ALL SQL statement back from EF
    - Having the same domain model for relational databases or NoSQL databases is not simple, and you should not try to do it. You really have to design your model with an understanding of how the data is going to be used in each particular database.
    - A benefit when using NoSQL databases is that the entities are more denormalized, so you do not set a table mapping. Your domain model can be more flexible than when using a relational database
    - When you design your domain model based on aggregates, moving to NoSQL and document-oriented databases might be even easier than using a relational database, because the aggregates you design are similar to serialized documents in a document-oriented database. Then you can include in those "bags" all the information you might need for that aggregate.
    ```json
    {
      "id": "2017001",
      "orderDate": "2/25/2017",
      "buyerId": "1234567",
      "address": [
          {
          "street": "100 One Microsoft Way",
          "city": "Redmond",
          "state": "WA",
          "zip": "98052",
          "country": "U.S."
          }
      ],
      "orderItems": [
          {"id": 20170011, "productId": "123456", "productName": ".NET T-Shirt",
          "unitPrice": 25, "units": 2, "discount": 0},
          {"id": 20170012, "productId": "123457", "productName": ".NET Mug",
          "unitPrice": 15, "units": 1, "discount": 0}
      ]
    }
    ```
    ```csharp
    // C# EXAMPLE OF AN ORDER AGGREGATE BEING PERSISTED WITH AZURE COSMOS DB API
    // *** Domain Model Code ***
    // Aggregate: Create an Order object with its child entities and/or value objects.
    // Then, use AggregateRoot's methods to add the nested objects so invariants and
    // logic is consistent across the nested properties (value objects and entities).

    Order orderAggregate = new Order
    {
        Id = "2017001",
        OrderDate = new DateTime(2005, 7, 1),
        BuyerId = "1234567",
        PurchaseOrderNumber = "PO18009186470"
    }

    Address address = new Address
    {
        Street = "100 One Microsoft Way",
        City = "Redmond",
        State = "WA",
        Zip = "98052",
        Country = "U.S."
    }

    orderAggregate.UpdateAddress(address);

    OrderItem orderItem1 = new OrderItem
    {
        Id = 20170011,
        ProductId = "123456",
        ProductName = ".NET T-Shirt",
        UnitPrice = 25,
        Units = 2,
        Discount = 0;
    };

    //Using methods with domain logic within the entity. No anemic-domain model
    orderAggregate.AddOrderItem(orderItem1);
    // *** End of Domain Model Code ***

    // *** Infrastructure Code using Cosmos DB Client API ***
    Uri collectionUri = UriFactory.CreateDocumentCollectionUri(databaseName,
        collectionName);

    await client.CreateDocumentAsync(collectionUri, orderAggregate);

    // As your app evolves, let's say your object has a new schema. You can insert
    // OrderV2 objects without any changes to the database tier.
    Order2 newOrder = GetOrderV2Sample("IdForSalesOrder2");
    await client.CreateDocumentAsync(collectionUri, newOrder);
    ```
  - You can see that the way you work with your domain model can be similar to the way you use it in your domain model layer when the infrastructure is EF. You still use the same aggregate root methods to ensure consistency, invariants, and validations within the aggregate.
  - However, when you persist your model into the NoSQL database, the code and API change dramatically compared to EF Core code or any other code related to relational databases.
- **Seedwork or SharedKernel (reusable base classes and interfaces for your domain model)**
  - This folder contains custom base classes that you can use as a base for your domain entities and value objects
  - Use these base classes so you don't have redundant code in each domain's object class.
  - It's called SeedWork because the folder contains just a small subset of reusable classes that cannot really be considered a framework
  - Few custom base classes like 
    - Entity
    - ValueObject
    - Enumeration
    - plus a few interfaces.
    ```csharp
    // COMPATIBLE WITH ENTITY FRAMEWORK CORE (1.1 and later)
    public abstract class Entity
    {
        int? _requestedHashCode;
        int _Id;
        private List<INotification> _domainEvents;
        public virtual int Id
        {
            get
            {
                return _Id;
            }
            protected set
            {
                _Id = value;
            }
        }

        public List<INotification> DomainEvents => _domainEvents;
        public void AddDomainEvent(INotification eventItem)
        {
            _domainEvents = _domainEvents ?? new List<INotification>();
            _domainEvents.Add(eventItem);
        }
        public void RemoveDomainEvent(INotification eventItem)
        {
            if (_domainEvents is null) return;
            _domainEvents.Remove(eventItem);
        }

        public bool IsTransient()
        {
            return this.Id == default(Int32);
        }

        public override bool Equals(object obj)
        {
            if (obj == null || !(obj is Entity))
                return false;
            if (Object.ReferenceEquals(this, obj))
                return true;
            if (this.GetType() != obj.GetType())
                return false;
            Entity item = (Entity)obj;
            if (item.IsTransient() || this.IsTransient())
                return false;
            else
                return item.Id == this.Id;
        }

        public override int GetHashCode()
        {
            if (!IsTransient())
            {
                if (!_requestedHashCode.HasValue)
                    _requestedHashCode = this.Id.GetHashCode() ^ 31;
                // XOR for random distribution. See:
                // https://docs.microsoft.com/archive/blogs/ericlippert/guidelines-and-rules-for-gethashcode
                return _requestedHashCode.Value;
            }
            else
                return base.GetHashCode();
        }
        public static bool operator ==(Entity left, Entity right)
        {
            if (Object.Equals(left, null))
                return (Object.Equals(right, null));
            else
                return left.Equals(right);
        }
        public static bool operator !=(Entity left, Entity right)
        {
            return !(left == right);
        }
    }
    ```
- **Value Objects**
  - They have no identity.
  - They are immutable.
  - A value object can reference other entities.
  - For example, in an application that generates a route that describes how to get from one point to another, that route would be a value object. It would be a snapshot of points on a specific route, but this suggested route would not have an identity, even though internally it might refer to entities like City, Road, etc.
    ```csharp
    public class Address : ValueObject
    {
        public String Street { get; private set; }
        public String City { get; private set; }
        public String State { get; private set; }
        public String Country { get; private set; }
        public String ZipCode { get; private set; }

        public Address() { }

        public Address(string street, string city, string state, string country, string zipcode)
        {
            Street = street;
            City = city;
            State = state;
            Country = country;
            ZipCode = zipcode;
        }

        protected override IEnumerable<object> GetEqualityComponents()
        {
            // Using a yield return statement to return each element one at a time
            yield return Street;
            yield return City;
            yield return State;
            yield return Country;
            yield return ZipCode;
        }
    }
    ```
- **Use enumeration classes instead of enum types**
  - Enumerations (or enum types for short) are a thin language wrapper around an integral type.
  - You might want to limit their use to when you are storing one value from a closed set of values.
  - Classification based on sizes (small, medium, large) is a good example.
  - Using enums for control flow or more robust abstractions can be a code smell.
  - This type of usage leads to fragile code with many control flow statements checking values of the enum.
  - Instead, you can create Enumeration classes that enable all the rich features of an object-oriented language.
    - However, this isn't a critical topic and in many cases, for simplicity, you can still use regular enum types if that's your preference. Anyway, the use of enumeration classes is more related to business-related concepts.
    ```csharp
    public abstract class Enumeration : IComparable
    {
        public string Name { get; private set; }

        public int Id { get; private set; }

        protected Enumeration(int id, string name)
        {
            Id = id;
            Name = name;
        }

        public override string ToString() => Name;

        public static IEnumerable<T> GetAll<T>() where T : Enumeration
        {
            var fields = typeof(T).GetFields(BindingFlags.Public |
                                            BindingFlags.Static |
                                            BindingFlags.DeclaredOnly);

            return fields.Select(f => f.GetValue(null)).Cast<T>();
        }

        public override bool Equals(object obj)
        {
            var otherValue = obj as Enumeration;

            if (otherValue == null)
                return false;

            var typeMatches = GetType().Equals(obj.GetType());
            var valueMatches = Id.Equals(otherValue.Id);

            return typeMatches && valueMatches;
        }

        public int CompareTo(object other) => Id.CompareTo(((Enumeration)other).Id);

        // Other utility methods ...
    }
    // You can use this class as a type in any entity or value object, as for the following CardType : Enumeration class:
    public class CardType : Enumeration
    {
        public static readonly CardType Amex = new CardType(1, "Amex");
        public static readonly CardType Visa = new CardType(2, "Visa");
        public static readonly CardType MasterCard = new CardType(3, "MasterCard");

        public CardType(int id, string name)
            : base(id, name)
        {
        }
    }
    ```
- **Validation in Domain Model**
  - **Definition**
    - Validation rules can be thought as invariants.
    - The main responsibility of an aggregate is to enforce invariants across state changes for all the entities within that aggregate.
    - Domain entities should always be **valid** entities.
    - There are a certain number of invariants for an object that should always be **true**.
    - For example, an order item object always has to have a quantity that must be a positive integer, plus an article name and price.
    - Therefore, invariants enforcement is the responsibility of the domain entities (especially of the aggregate root) and an entity object should not be able to exist without being valid.
    - Invariant rules are simply expressed as contracts, and **exceptions or notifications** are raised when they are **violated**.
    - Let's propose we now have a SendUserCreationEmailService that takes a UserProfile ...
      - How can we rationalize in that service that Name is not null? Do we check it again? Or more likely ...
      - You just don't bother to check and "hope for the best"—you hope that someone bothered to validate it before sending it to you.
      - Of course, using TDD one of the first tests we should be writing is that if I send a customer with a null name that it should raise an error.
      - But once we start writing these kinds of tests over and over again we realize ... "wait if we never allowed name to become null we wouldn't have all of these tests".
    - **Two-step validation**
      - Use field-level validation on your command Data Transfer Objects (DTOs) and domain-level validation inside your entities. You can do this by returning a result object instead of exceptions in order to make it easier to deal with the validation errors.
      - Using field validation with data annotations, for example, you do not duplicate the validation definition. The execution, though, can be both server-side and client-side in the case of DTOs (commands and ViewModels, for instance).
    - **Implementation**
      - Validations are usually implemented in domain entity:
      - **constructors**
      - **Methods** that can update the entity.
      - There are multiple ways to implement validations such as:
        - Verifying data and raising exceptions if the validation fails.
        - **Specification pattern** for validations
        - **Notification pattern** to return a collection of errors instead of returning an exception for each validation as it occurs.
    - **Validate conditions and throw exceptions**
      - The following code example shows the simplest approach to validation in a domain entity by raising an exception. 
        ```csharp
        public void SetAddress(Address address)
        {
            _shippingAddress = address?? throw new ArgumentNullException(nameof(address));
        }
        public void SetAddress(string line1, string line2,
            string city, string state, int zip)
        {
            _shippingAddress.line1 = line1 ?? throw new ...
            _shippingAddress.line2 = line2;
            _shippingAddress.city = city ?? throw new ...
            _shippingAddress.state = (IsValid(state) ? state : throw new …);
        }
        ```
      - Data annotations and the IValidatableObject interface can still be used for model validation during model binding, prior to the controller's actions invocation as usual, but that model is meant to be a ViewModel or DTO and that's an MVC or API concern **not a domain model concern**.
- **Client-side validation**
  - Even when the source of truth is the domain model and ultimately you must have validation at the domain model level, validation can still be handled at both the domain model level (server side) and the UI (client side).
  - Client-side validation is a great convenience for users. It saves time they would otherwise spend waiting for a round trip to the server that might return validation errors
  -  In business terms, even a few fractions of seconds multiplied hundreds of times each day adds up to a lot of time, expense, and frustration. Straightforward and immediate validation enables users to work more efficiently and produce better quality input and output.
  - Just as the view model and the domain model are different
    - View model validation and domain model validation might be similar but serve a different purpose.
    - If you are concerned about DRY (the Don't Repeat Yourself principle), consider that in this case code reuse might also mean coupling, and in enterprise applications it is more important not to couple the server side to the client side than to follow the DRY principle.
  - Even when using client-side validation
    - You should always validate your commands or input DTOs in server code, because the **server APIs** are a **possible attack vector**.
    - Usually, doing **both** is your *best bet because if you have a client application, from a UX perspective, it is best to be proactive and not allow the user to enter invalid information.
    - Therefore, in client-side code you typically validate the ViewModels. You could also validate the client output DTOs or commands before you send them to the services.
    - The implementation of client-side validation depends on what kind of client application you are building.
      - **MVC Web** application will be different if you are validating data ith most of the code in .NET
      - **SPA Web** application with that validation being coded in JavaScript or TypeScript, or a mobile app coded with Xamarin and C#.
- **Warnings**
  - For example, following DDD patterns, you should not do the following from any command handler method or application layer class (actually, it should be impossible for you to do so):
    ```csharp
    // WRONG ACCORDING TO DDD PATTERNS – CODE AT THE APPLICATION LAYER OR
    // COMMAND HANDLERS
    // Code in command handler methods or Web API controllers
    //... (WRONG) Some code with business logic out of the domain classes ...
    OrderItem myNewOrderItem = new OrderItem(orderId, productId, productName,
        pictureUrl, unitPrice, discount, units);

    //... (WRONG) Accessing the OrderItems collection directly from the application layer // or command handlers
    myOrder.OrderItems.Add(myNewOrderItem);
    //...
    ```
    - In this case, the Add method is purely an operation to add data, with direct access to the OrderItems collection. Therefore, most of the domain logic, rules, or validations related to that operation with the child entities will be spread across the application layer (command handlers and Web API controllers).
    - If you go around the aggregate root, the aggregate root cannot guarantee its invariants, its validity, or its consistency. Eventually you will have spaghetti code or transactional script code.
  - To follow DDD patterns, entities must not have public setters in any entity property.
  - Changes in an entity should be driven by explicit methods with explicit ubiquitous language about the change they are performing in the entity.
  - Furthermore, collections within the entity (like the order items) should be read-only properties (the AsReadOnly method explained later). You should be able to update it only from within the aggregate root class methods or the child entity methods.
  - The following code snippet shows the proper way to code the task of adding an OrderItem object to the Order aggregate.
    ```csharp
    class OrderAggregate {
      // RIGHT ACCORDING TO DDD--CODE AT THE APPLICATION LAYER OR COMMAND HANDLERS
      // The code in command handlers or WebAPI controllers, related only to application stuff
      // There is NO code here related to OrderItem object's business logic
      myOrder.AddOrderItem(productId, productName, pictureUrl, unitPrice, discount, units);

      // The code related to OrderItem params validations or domain rules should
      // be WITHIN the AddOrderItem method.

      //...
    }
    ```
  - In this snippet, most of the validations or logic related to the creation of an OrderItem object will be under the control of the Order aggregate root—in the AddOrderItem method—especially validations and logic related to other elements in the aggregate.
  - For instance, you might get the same product item as the result of multiple calls to AddOrderItem.
  - In that method, you could examine the product items and consolidate the same product items into a single OrderItem object with several units. 
  - Additionally, if there are different discount amounts but the product ID is the same, you would likely apply the higher discount. This principle applies to any other domain logic for the OrderItem object.
  - In addition, the new OrderItem(params) operation will also be controlled and performed by the AddOrderItem method from the Order aggregate root.
  - Therefore, most of the **logic or validations** related to that operation (especially anything that impacts the consistency between other child entities) will be in a single place within the aggregate root. That is the ultimate purpose of the aggregate root pattern.

## Reference
- Services in Domain-Driven Design: https://lostechies.com/jimmybogard/2008/08/21/services-in-domain-driven-design/
- Repositories in DDD https://lostechies.com/jimmybogard/2009/09/03/ddd-repository-implementation-patterns/
- Event Sourcing: https://martinfowler.com/eaaDev/EventSourcing.html
- Domain Events: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation
- Use NoSQL databases as a persistence infrastructure: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure
- Implement the microservice application layer using the Web API: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-application-layer-implementation-web-api
- Repository: https://aspnetboilerplate.com/Pages/Documents/Repositories
- Domain Service: https://aspnetboilerplate.com/Pages/Documents/Domain-Services