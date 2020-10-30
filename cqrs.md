# CQRS Pattern 
- **a) Intro**
  - What is CQS?
    - There are 2 basic concepts Of **CQS**:
      - **Queries**: These return a result and do not change the state of the system, and they are free of side effects.
      - **Commands**: These change the state of a system.
  - What is CQRS ?
    - Based on CQS with pattern based on **commands** and **events** plus optionally on **asynchronous 
   messages**
    - More advanced scenarios, like having a **different** physical database for **reads (queries)** than for **writes (updates)**
    - A more **evolved** CQRS system might implement **Event-Sourcing** (ES) for your updates database, so you would only store events in the domain model instead of storing the current-state data
    - By separate models we most commonly mean different object models, probably running in different logical processes, perhaps on separate hardware.
    - A web example would see a user looking at a web page that's rendered using the query model. If they initiate a change that change is routed to the separate command model for processing, the resulting change is communicated to the query model to render the updated state.
    - There's room for considerable variation here. The in-memory models may share the same database, in which case the database acts as the communication between the two models. However they may also use separate databases, effectively making the query-side's database into a real-time ReportingDatabase. In this case there needs to be some communication mechanism between the two models or their databases.
    - Each layer(Queries an Commands) has its **own data model** (note that we say model, not necessarily a different database) 
    - The two layers can be within the same tier or microservice, as in the example (ordering microservice) used for this guide.
    - Or they could be implemented on different microservices or processes so they can be optimized and scaled out separately without affecting one another.
- **b) Benefits**:
  - Allows you to separate the load from reads and writes allowing you to scale each independently.
  - As we move away from a single representation that we interact with via CRUD, we can easily move to a task-based UI.
  - CQRS fits well with event-based programming models. It's common to see CQRS system split into separate services communicating with Event Collaboration. This allows these services to easily take advantage of Event Sourcing.
  - For many domains, much of the logic is needed when you're updating, so it may make sense to use EagerReadDerivation to simplify your query-side models.
  - If the write model generates events for all updates, you can structure read models as EventPosters, allowing them to be MemoryImages and thus avoiding a lot of database interactions.
  - Optimized data schemas. The read side can use a schema that is optimized for queries, while the write side uses a schema that is optimized for updates.
  - Security. It's easier to ensure that only the right domain entities are performing writes on the data.
  - Separation of concerns. Segregating the read and write sides can result in models that are more maintainable and flexible. Most of the complex business logic goes into the write model. The read model can be relatively simple.
  - Simpler queries. By storing a materialized view in the read database, the application can avoid complex joins when querying.
- **c) Challenges**
  - Complexity. The basic idea of CQRS is simple. But it can lead to a more complex application design, especially if they include the Event Sourcing pattern.
  - Messaging. Although CQRS does not require messaging, it's common to use messaging to process commands and publish update events. In that case, the application must handle message failures or duplicate messages.
  - Eventual consistency. If you separate the read and write databases, the read data may be stale. The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data.
- **d) Read: Implement reads/queries in a CQRS microservice**
  - Approaches:
    - ORM like EF Core
    - AutoMapper projections
    - stored procedures
    - views
    - materialized views
    - a micro ORM - Dapper.
  - Querying the database with a Micro-ORM like Dapper, returning dynamic ViewModels
  - It should be separated from commands
  - The goal is to have more flexibility in the queries instead of limiting the queries with constraints from DDD patterns like aggregates.
  - The query definitions query the database and return a dynamic ViewModel or DTOs(Data Tranfer Objects ) built on the fly for each query. Since the queries are idempotent, they won't change the data no matter how many times you run a query.
  - Therefore, you don't need to be restricted by any DDD pattern used in the transactional side, like aggregates and other patterns, and that is why queries are separated from the **transactional area**
  -  You query the database for the data that the UI needs and return a dynamic ViewModel that does not need to be statically defined anywhere (no classes for the ViewModels) except in the SQL statements themselves.
  - The returned data **(ViewModel)** can be the result of joining data from multiple entities or tables in the database, or even across multiple aggregates defined in the domain model for the transactional area.
  - In this case, because you are creating queries independent of the domain model, the aggregates boundaries and constraints are ignored and you're free to query any table and column you might need. 
  - The ViewModels can be static types defined in classes (as is implemented in the ordering microservice). Or they can be created dynamically based on the queries performed, which is very agile for developers.
  - Can use dynamic ViewModels or static ViewModels
    - Dynamic:
      - Pros: This approach reduces the need to modify static ViewModel classes whenever you update the SQL sentence of a query, making this design approach agile when coding, straightforward, and quick to evolve in regard to future changes.
      - Cons: In the long term, dynamic types can negatively impact the clarity and the compatibility of a service with client apps. In addition, middleware software like Swashbuckle cannot provide the same level of documentation on returned types if using dynamic types.
    - Static:
      - Pros:
        - Having static, predefined ViewModel classes, like "contracts" based on explicit DTO classes, is definitely better for public APIs but also for long-term microservices, even if they are only used by the same application.
        - Predefined DTO classes allow you to offer richer information from Swagger. That improves the API documentation and compatibility when consuming an API.
      - Cons: As mentioned earlier, when updating the code, it takes some more steps to update the DTO classes.
  - Describe response types of Web APIs
    - Without proper documentation in the Swagger UI, the consumer lacks knowledge of what types are being returned or what HTTP codes can be returned.
    - That problem is fixed by adding the **Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute**, so Swashbuckle can generate richer information about the API return model and values
    - When using the **ProducesResponseType** attribute, you can also specify what is the expected outcome in regards possible HTTP errors/codes, like 200, 400, etc.
- **e) Write: Domain Command Patterns - Handlers**
  - Using commands to update data
  - Using DDD patterns for the transactional domain 
  - Trigger transactions and data updates, change state in the system
  - Commands should be task based, rather than data centric. ("Book hotel room", not "set ReservationStatus to Reserved").
  - Commands may be placed on a queue for asynchronous processing, rather than being processed synchronously.
  - If separate read and write databases are used
    - they must be kept in sync. 
    - Typically this is accomplished by having the write model publish an event whenever it updates the database.
    - Updating the database and publishing the event must occur in a single transaction.
    - The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.
    - Using multiple read-only replicas can increase query performance, especially in distributed scenarios where read-only replicas are located close to the application instances.
  - **The command class?**
    - A command is a request for the system to perform an action that changes the state of the system. Commands are imperative, and should be processed just once.
    - Since commands are imperatives, they are typically named with a verb in the imperative mood (for example, "create" or "update"), and they might include the aggregate type, such as CreateOrderCommand. Unlike an event, a command is not a fact from the past; it is only a request, and thus may be refused.
    - Commands can originate from the UI as a result of a user initiating a request, or from a process manager when the process manager is directing an aggregate to perform an action.
    - An important characteristic of a command is that it should be processed **just once** by a single receiver.
    - This is because a command is a single action or transaction you want to perform in the application. For example, the same order creation command should not be processed more than once. This is an important difference between commands and events.
    - **Events** may be processed multiple times, because many systems or microservices might be interested in the event.
    - In addition, it is important that a command be processed only once in case the command is not idempotent. A command is idempotent if it can be executed multiple times without changing the result, either because of the nature of the command, or because of the way the system handles the command.
    - It is a good practice to make your commands and updates **idempotent** when it makes sense under your domain's business rules and invariants. 
    - For instance,
      - to use the same example, if for any reason (retry logic, hacking, etc.) 
      - the same CreateOrder command reaches your system multiple times, 
      - you should be able to identify it and ensure that you **do not create multiple orders**. 
      - To do so, you need to attach some kind of identity in the operations and identify whether the command or update was already processed.
    - Note that it is recommended to implement immutable Commands
      ```csharp
      // DDD and CQRS patterns comment: Note that it is recommended to implement immutable Commands
      // In this case, its immutability is achieved by having all the setters as private
      // plus only being able to update the data just once, when creating the object through its constructor.
      // References on Immutable Commands:
      // http://cqrs.nu/Faq
      // https://docs.spine3.org/motivation/immutability.html
      // http://blog.gauffin.org/2012/06/griffin-container-introducing-command-support/
      // https://docs.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/how-to-implement-a-lightweight-class-with-auto-implemented-properties

      [DataContract]
      public class CreateOrderCommand
          : IRequest<bool>
      {
          [DataMember]
          private readonly List<OrderItemDTO> _orderItems;

          [DataMember]
          public string UserId { get; private set; }

          [DataMember]
          public string UserName { get; private set; }

          [DataMember]
          public string City { get; private set; }

          [DataMember]
          public string Street { get; private set; }

          [DataMember]
          public string State { get; private set; }

          [DataMember]
          public string Country { get; private set; }

          [DataMember]
          public string ZipCode { get; private set; }

          [DataMember]
          public string CardNumber { get; private set; }

          [DataMember]
          public string CardHolderName { get; private set; }

          [DataMember]
          public DateTime CardExpiration { get; private set; }

          [DataMember]
          public string CardSecurityNumber { get; private set; }

          [DataMember]
          public int CardTypeId { get; private set; }

          [DataMember]
          public IEnumerable<OrderItemDTO> OrderItems => _orderItems;

          public CreateOrderCommand()
          {
              _orderItems = new List<OrderItemDTO>();
          }

          public CreateOrderCommand(List<BasketItem> basketItems, string userId, string userName, string city, string street, string state, string country, string zipcode,
              string cardNumber, string cardHolderName, DateTime cardExpiration,
              string cardSecurityNumber, int cardTypeId) : this()
          {
              _orderItems = basketItems.ToOrderItemsDTO().ToList();
              UserId = userId;
              UserName = userName;
              City = city;
              Street = street;
              State = state;
              Country = country;
              ZipCode = zipcode;
              CardNumber = cardNumber;
              CardHolderName = cardHolderName;
              CardExpiration = cardExpiration;
              CardSecurityNumber = cardSecurityNumber;
              CardTypeId = cardTypeId;
              CardExpiration = cardExpiration;
          }


          public class OrderItemDTO
          {
              public int ProductId { get; set; }

              public string ProductName { get; set; }

              public decimal UnitPrice { get; set; }

              public decimal Discount { get; set; }

              public int Units { get; set; }

              public string PictureUrl { get; set; }
          }
      }
      ```
    - Basically, the command class contains all the data you need for performing a **business transaction** by using the domain model objects.
    - Thus, commands are simply data structures that contain **read-only** data, and **no behavior**. The command's name indicates its purpose. In many languages like C#, commands are represented as classes, but they are not true classes in the real object-oriented sense.
    - As an additional characteristic, commands are **immutable**, because the expected usage is that they are processed directly by the domain model. They do not need to change during their projected lifetime. In a C# class, immutability can be achieved by not having any setters or other methods that change the internal state.
    - Keep in mind that if you intend or expect commands to go through a **serializing/deserializing** process, the properties must have a private setter, and the **[DataMember] (or [JsonProperty]) attribute**.
    - Otherwise, the deserializer won't be able to reconstruct the object at the destination with the required values. You can also use truly read-only properties if the class has a constructor with parameters for all properties, with the usual camelCase naming convention, and annotate the constructor as [JsonConstructor]. However, this option requires more code.
  - **The Command handler class?**
    - You should implement a specific command handler class for each command.
    - That is how the pattern works, and it's where you'll use the command object, the domain objects, and the infrastructure repository objects. 
    - The command handler is in fact the heart of the application layer in terms of **CQRS and DDD**.
    - However, all the **domain logic** should be contained in the **domain classes—within the aggregate roots (root entities), child entities, or domain services**, but not within the command handler, which is a class from the application layer.
    - A command handler receives a command and obtains a result from the aggregate that is used. The result should be either successful execution of the command, or an exception. In the case of an exception, the system state should be unchanged.
    - The command handler usually takes the following steps:
      - It receives the command object, like a DTO (from the mediator or other infrastructure object).
      - It validates that the command is valid (if not validated by the mediator).
      - It instantiates the aggregate root instance that is the target of the current command.
      - It executes the method on the aggregate root instance, getting the required data from the command.
      - It persists the new state of the aggregate to its related database. This last operation is the actual transaction.
    - Typically, a command handler deals with a single aggregate driven by its aggregate root (root entity). If multiple aggregates should be **impacted** by the reception of a single command, you could use **domain events** to propagate states or actions across multiple aggregates.
    - The **important point** here is that when a command is being processed: 
      - All the domain logic should be inside the domain model (the aggregates), fully encapsulated and ready for unit testing.
      - The command handler just acts as a way to get the domain model from the database, and as the final step, to tell the infrastructure layer (repositories) to persist the changes when the model is changed.
      - The advantage of this approach is that you can **refactor the domain logic** in an isolated, fully encapsulated, rich, behavioral domain model without changing code in the application or infrastructure layers, which are the plumbing level (command handlers, Web API, repositories, etc.).
    - When command handlers get complex, with too much logic, that can be a **code smell**. Review them, and if you find domain logic, refactor the code to **move** that domain behavior to the methods of the domain objects (the aggregate root and child entity).
      ```csharp
      public class CreateOrderCommandHandler
        : IRequestHandler<CreateOrderCommand, bool>
      {
          private readonly IOrderRepository _orderRepository;
          private readonly IIdentityService _identityService;
          private readonly IMediator _mediator;
          private readonly IOrderingIntegrationEventService _orderingIntegrationEventService;
          private readonly ILogger<CreateOrderCommandHandler> _logger;

          // Using DI to inject infrastructure persistence Repositories
          public CreateOrderCommandHandler(IMediator mediator,
              IOrderingIntegrationEventService orderingIntegrationEventService,
              IOrderRepository orderRepository,
              IIdentityService identityService,
              ILogger<CreateOrderCommandHandler> logger)
          {
              _orderRepository = orderRepository ?? throw new ArgumentNullException(nameof(orderRepository));
              _identityService = identityService ?? throw new ArgumentNullException(nameof(identityService));
              _mediator = mediator ?? throw new ArgumentNullException(nameof(mediator));
              _orderingIntegrationEventService = orderingIntegrationEventService ?? throw new ArgumentNullException(nameof(orderingIntegrationEventService));
              _logger = logger ?? throw new ArgumentNullException(nameof(logger));
          }

          public async Task<bool> Handle(CreateOrderCommand message, CancellationToken cancellationToken)
          {
              // Add Integration event to clean the basket
              var orderStartedIntegrationEvent = new OrderStartedIntegrationEvent(message.UserId);
              await _orderingIntegrationEventService.AddAndSaveEventAsync(orderStartedIntegrationEvent);

              // Add/Update the Buyer AggregateRoot
              // DDD patterns comment: Add child entities and value-objects through the Order Aggregate-Root
              // methods and constructor so validations, invariants and business logic
              // make sure that consistency is preserved across the whole aggregate
              var address = new Address(message.Street, message.City, message.State, message.Country, message.ZipCode);
              var order = new Order(message.UserId, message.UserName, address, message.CardTypeId, message.CardNumber, message.CardSecurityNumber, message.CardHolderName, message.CardExpiration);

              foreach (var item in message.OrderItems)
              {
                  order.AddOrderItem(item.ProductId, item.ProductName, item.UnitPrice, item.Discount, item.PictureUrl, item.Units);
              }

              _logger.LogInformation("----- Creating Order - Order: {@Order}", order);

              _orderRepository.Add(order);

              return await _orderRepository.UnitOfWork
                  .SaveEntitiesAsync(cancellationToken);
          }
      }
      ```
    - These are additional steps a command handler should take:
      - Use the command's data to operate with the aggregate root's methods and behavior.
      - Internally within the domain objects, raise domain events while the transaction is executed, but that is transparent from a command handler point of view.
      - If the aggregate's operation result is successful and after the transaction is finished, raise integration events. (These might also be raised by infrastructure classes like repositories.)
    - The reason that using the **Mediator** pattern makes sense is that in enterprise applications, the processing requests can get complicated.
      - You want to be able to add an open number of **cross-cutting concerns** like **logging, validations, audit, and security**. In these cases, you can rely on a **mediator pipeline** (see Mediator pattern) to provide a means for these extra behaviors or cross-cutting concerns.
    - A **mediator** is an object that encapsulates the "how" of this process: it coordinates execution based on state, the way a command handler is invoked, or the payload you provide to the handler. With a mediator component, you can apply cross-cutting concerns in a centralized and transparent way by applying decorators (or pipeline behaviors since MediatR 3).
  - **Use message queues (out-of-proc) in the command's pipeline**
    - Command's pipeline can also be handled by a high availability message queue to deliver the commands to the appropriate handler.
    - Using message queues to accept the commands can further complicate your command's pipeline, because you will probably need to split the pipeline into two processes connected through the external message queue. 
    - Still, it should be used if you need to have improved scalability and performance based on asynchronous messaging.
    - That is a great benefit of queues: the message queue can act as a buffer in cases when hyper scalability is needed, such as for stocks or any other scenario with a high volume of ingress data.
    - However, because of the asynchronous nature of message queues, you need to figure out how to communicate with the client application about the success or failure of the command's process. As a rule, you should never use **"fire and forget"** commands. Every business application needs to know if a command was processed successfully, or at least validated and accepted.
    - Asynchronous commands greatly increase the complexity of a system, because there is no simple way to indicate failures. Therefore, asynchronous commands are not recommended other than when scaling requirements are needed or in special cases when communicating the internal microservices through messaging. In those cases, you must design a separate reporting and **recovery system for failures**.
  - **Implement idempotent Commands**
    - More advanced example than the above is submitting a CreateOrderCommand object from the Ordering microservice.
    - But since the Ordering business process is a bit more complex and, in our case, it actually starts in the Basket microservice, this action of submitting the CreateOrderCommand object is performed from an integration-event handler named UserCheckoutAcceptedIntegrationEventHandler instead of a simple WebAPI controller called from the client App as in the previous simpler example.
      ```csharp
      var createOrderCommand = new CreateOrderCommand(eventMsg.Basket.Items,
                                                eventMsg.UserId, eventMsg.City,
                                                eventMsg.Street, eventMsg.State,
                                                eventMsg.Country, eventMsg.ZipCode,
                                                eventMsg.CardNumber,
                                                eventMsg.CardHolderName,
                                                eventMsg.CardExpiration,
                                                eventMsg.CardSecurityNumber,
                                                eventMsg.CardTypeId);

      var requestCreateOrder = new IdentifiedCommand<CreateOrderCommand,bool>(createOrderCommand,
                                                                              eventMsg.RequestId);
      result = await _mediator.Send(requestCreateOrder);
      ```
    - However, this case is also slightly more advanced because we're also implementing idempotent commands.
    - The CreateOrderCommand process should be idempotent, so if the same message comes duplicated through the network, because of any reason, like retries, the same business order will be processed just once.
    - This is implemented by wrapping the business command (in this case CreateOrderCommand) and embedding it into a generic IdentifiedCommand, which is tracked by an ID of every message coming through the network that has to be idempotent.
      ```csharp
      public class IdentifiedCommand<T, R> : IRequest<R>
          where T : IRequest<R>
      {
          public T Command { get; }
          public Guid Id { get; }
          public IdentifiedCommand(T command, Guid id)
          {
              Command = command;
              Id = id;
          }
      }
      ```
    - Then the CommandHandler for the **IdentifiedCommand** named **IdentifiedCommandHandler.cs** will basically check if the ID coming as part of the message already exists in a table. If it already exists, that command won't be processed again, so it behaves as an idempotent command. That infrastructure code is performed by the _requestManager.ExistAsync method call below.
      ```csharp
      // IdentifiedCommandHandler.cs
      public class IdentifiedCommandHandler<T, R> : IRequestHandler<IdentifiedCommand<T, R>, R>
              where T : IRequest<R>
      {
          private readonly IMediator _mediator;
          private readonly IRequestManager _requestManager;
          private readonly ILogger<IdentifiedCommandHandler<T, R>> _logger;

          public IdentifiedCommandHandler(
              IMediator mediator,
              IRequestManager requestManager,
              ILogger<IdentifiedCommandHandler<T, R>> logger)
          {
              _mediator = mediator;
              _requestManager = requestManager;
              _logger = logger ?? throw new System.ArgumentNullException(nameof(logger));
          }

          /// <summary>
          /// Creates the result value to return if a previous request was found
          /// </summary>
          /// <returns></returns>
          protected virtual R CreateResultForDuplicateRequest()
          {
              return default(R);
          }

          /// <summary>
          /// This method handles the command. It just ensures that no other request exists with the same ID, and if this is the case
          /// just enqueues the original inner command.
          /// </summary>
          /// <param name="message">IdentifiedCommand which contains both original command & request ID</param>
          /// <returns>Return value of inner command or default value if request same ID was found</returns>
          public async Task<R> Handle(IdentifiedCommand<T, R> message, CancellationToken cancellationToken)
          {
              var alreadyExists = await _requestManager.ExistAsync(message.Id);
              if (alreadyExists)
              {
                  return CreateResultForDuplicateRequest();
              }
              else
              {
                  await _requestManager.CreateRequestForCommandAsync<T>(message.Id);
                  try
                  {
                      var command = message.Command;
                      var commandName = command.GetGenericTypeName();
                      var idProperty = string.Empty;
                      var commandId = string.Empty;

                      switch (command)
                      {
                          case CreateOrderCommand createOrderCommand:
                              idProperty = nameof(createOrderCommand.UserId);
                              commandId = createOrderCommand.UserId;
                              break;

                          case CancelOrderCommand cancelOrderCommand:
                              idProperty = nameof(cancelOrderCommand.OrderNumber);
                              commandId = $"{cancelOrderCommand.OrderNumber}";
                              break;

                          case ShipOrderCommand shipOrderCommand:
                              idProperty = nameof(shipOrderCommand.OrderNumber);
                              commandId = $"{shipOrderCommand.OrderNumber}";
                              break;

                          default:
                              idProperty = "Id?";
                              commandId = "n/a";
                              break;
                      }

                      _logger.LogInformation(
                          "----- Sending command: {CommandName} - {IdProperty}: {CommandId} ({@Command})",
                          commandName,
                          idProperty,
                          commandId,
                          command);

                      // Send the embeded business command to mediator so it runs its related CommandHandler
                      var result = await _mediator.Send(command, cancellationToken);

                      _logger.LogInformation(
                          "----- Command result: {@Result} - {CommandName} - {IdProperty}: {CommandId} ({@Command})",
                          result,
                          commandName,
                          idProperty,
                          commandId,
                          command);

                      return result;
                  }
                  catch
                  {
                      return default(R);
                  }
              }
          }
      }
      ```
    - Since the IdentifiedCommand acts like a business command's envelope, when the business command needs to be processed because it is not a repeated ID, then it takes that inner business command and resubmits it to Mediator, as in the last part of the code shown above when running _mediator.Send(message.Command), from the IdentifiedCommandHandler.cs.
  - **Apply cross-cutting concerns when processing commands with the Behaviors in MediatR**
    - There is one more thing: being able to apply cross-cutting concerns to the mediator pipeline. You can also see at the end of the Autofac registration module code how it registers a behavior type, specifically, a custom LoggingBehavior class and a ValidatorBehavior class. But you could add other custom behaviors, too.
    ```csharp
    public class MediatorModule : Autofac.Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            builder.RegisterAssemblyTypes(typeof(IMediator).GetTypeInfo().Assembly)
                .AsImplementedInterfaces();

            // Register all the Command classes (they implement IRequestHandler)
            // in assembly holding the Commands
            builder.RegisterAssemblyTypes(
                                  typeof(CreateOrderCommand).GetTypeInfo().Assembly).
                                      AsClosedTypesOf(typeof(IRequestHandler<,>));
            // Other types registration
            //...
            builder.RegisterGeneric(typeof(LoggingBehavior<,>)).
                                                      As(typeof(IPipelineBehavior<,>));
            builder.RegisterGeneric(typeof(ValidatorBehavior<,>)).
                                                      As(typeof(IPipelineBehavior<,>));
        }
    }
    ```
  - That LoggingBehavior class can be implemented as the following code, which logs information about the command handler being executed and whether it was successful or not.
    ```csharp
    public class LoggingBehavior<TRequest, TResponse>
         : IPipelineBehavior<TRequest, TResponse>
    {
        private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;
        public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger) =>
                                                                      _logger = logger;

        public async Task<TResponse> Handle(TRequest request,
                                            RequestHandlerDelegate<TResponse> next)
        {
            _logger.LogInformation($"Handling {typeof(TRequest).Name}");
            var response = await next();
            _logger.LogInformation($"Handled {typeof(TResponse).Name}");
            return response;
        }
    }
    ```
  - The eShopOnContainers ordering microservice also applies a second behavior for basic validations, the **ValidatorBehavior** class that relies on the **FluentValidation library**, as shown in the following code:
    ```csharp
    public class ValidatorBehavior<TRequest, TResponse>
         : IPipelineBehavior<TRequest, TResponse>
    {
        private readonly IValidator<TRequest>[] _validators;
        public ValidatorBehavior(IValidator<TRequest>[] validators) =>
                                                            _validators = validators;

        public async Task<TResponse> Handle(TRequest request,
                                            RequestHandlerDelegate<TResponse> next)
        {
            var failures = _validators
                .Select(v => v.Validate(request))
                .SelectMany(result => result.Errors)
                .Where(error => error != null)
                .ToList();

            if (failures.Any())
            {
                throw new OrderingDomainException(
                    $"Command Validation Errors for type {typeof(TRequest).Name}",
                            new ValidationException("Validation exception", failures));
            }

            var response = await next();
            return response;
        }
    }
    ```
  - Here the behavior is raising an exception if validation fails, but you could also return a result object, containing the command result if it succeeded or the validation messages in case it didn't. This would probably make it easier to display validation results to the user.
- **e) Event Sourcing(Not in scope for LB)**
  - **Intro:**
    - Event Sourcing ensures that all changes to application state are stored as a sequence of events
    - Not just can we query these events, we can also use the event log to reconstruct past states, and as a foundation to automatically adjust the state to cope with retroactive changes.
  - **How it works?**
    - The fundamental idea of **Event Sourcing** is that of ensuring every change to the **state** of an application is captured in an event object
    - Then that these event objects are themselves stored in the sequence they were applied for the same lifetime as the application state itself.
  - **Benefits?**
    - Guarantee that all changes to the domain objects are initiated by the event objects
    - **Complete Rebuild**: We can discard the application state completely and rebuild it by re-running the events from the event log on an empty application.
    - **Temporal Query**: We can determine the application state at any point in time. Notionally we do this by starting with a blank state and rerunning the events up to a particular time or event. We can take this further by considering multiple time-lines (analogous to branching in a version control system).
    - **Event Replay**: 
      - If we find a past event was incorrect, we can compute the consequences by reversing it and later events and then replaying the new event and later events.
      - Or indeed by throwing away the application state and replaying all events with the correct event in sequence.
      - The same technique can handle events received in the wrong sequence - a common problem with systems that communicate with asynchronous messaging.
    - **Revert Events**:
      - Reversal is the most straightforward when the event is cast in the form of a difference
      - An example of this
        - would be "add $10 to Martin's account" as opposed to "set Martin's account to $110".
        - In the former case I can reverse by just subtracting $10, but in the latter case I don't have enough information to recreate the past value of the account.
      - It's worth remembering that all the capabilities of reversing events can be done instead by reverting to a past snapshot and replaying the event stream. 
    - **External Updates**
      - Problems:
        - One of the tricky elements to Event Sourcing is how to deal with external systems that don't follow this approach
        - You get problems when you are sending modifier messages to external systems and when you are receiving queries from other systems.
        - Many of the advantages of Event Sourcing stem from the ability to replay events at will, but if these events cause update messages to be sent to external systems, then things will go wrong because those external systems don't know the difference between real processing and replays.
      - Resolve:
        - To handle this you'll need to wrap any external systems with a **Gateway**.
        - The gateway should handle that distinction by having a reference to the event processor and checking the whether it's in replay mode before passing the external call off to the outside world. 
  - What used for?
    - Version control system
    - Tracking system (Ships, Orders, Deliveries..)
    - Banking system
    - Audit log
  - Example: Tracking Ships (C#)
    - Cargo -> Ship -> Port
    - There are four kinds of events that affect the model:
      - Arrival: ship arrives at a port
      - Departure: ship leaves a port
      - Load: cargo is loaded on a ship
      - Unload: cargo is unloaded from a ship
    - class Tester
      ```csharp
      Ship kr;
      Port sfo, la, yyv;
      Cargo refact;
      EventProcessor eProc;

      [SetUp]
      public void SetUp() {
        eProc = new EventProcessor(); 
        refact = new Cargo ("Refactoring");
        kr = new Ship("King Roy");
        sfo = new Port("San Francisco", Country.US);
        la = new Port("Los Angeles", Country.US);
        yyv = new Port("Vancouver", Country.CANADA) ;
      }
      [Test]
      public void ArrivalSetsShipsLocation() {
        ArrivalEvent ev = new ArrivalEvent(new DateTime(2005,11,1), sfo, kr);
        eProc.Process(ev);
        Assert.AreEqual(sfo, kr.Port);
      }
      [Test]
      public void DeparturePutsShipOutToSea() {
        eProc.Process(new ArrivalEvent(new DateTime(2005,10,1), la, kr));
        eProc.Process(new ArrivalEvent(new DateTime(2005,11,1), sfo, kr));
        eProc.Process(new DepartureEvent(new DateTime(2005,11,1), sfo, kr));
        Assert.AreEqual(Port.AT_SEA, kr.Port);    
      }
      ```
    - To make these tests work we just need the arrival and departure events. The event processor is very simple.
      ```csharp
      class EventProcessor{}

      IList log = new ArrayList();  
      public void Process(DomainEvent e) {
        e.Process();
        log.Add(e);
      }
      ```
    - Each event has a process method.
      ```csharp
      class DomainEvent{}

      DateTime _recorded, _occurred;
      internal DomainEvent (DateTime occurred) {
        this._occurred = occurred;
        this._recorded = DateTime.Now;
      }
      abstract internal void Process();
      ```
    - The arrival event simply captures the data and has a process method that simply forwards the event to an appropriate domain object.
      ```csharp
      class DepartureEvent{}

      Port _port;
      Ship _ship;
      internal Port Port  {get { return _port; }}
      internal Ship Ship  {get { return _ship; }}
      internal DepartureEvent(DateTime time, Port port, Ship ship) : base (time)  {
        this._port = port;
        this._ship = ship;
      }
      internal override void Process() {
        Ship.HandleDeparture(this);
      }
      ```
    - So here the event just does the processing selection logic. The processing domain logic is done by the ship.
      ```csharp
      class Ship{}

      public Port Port;
      public void HandleDeparture(DepartureEvent ev) {
        Port = Port.AT_SEA;
      }
      ```
- **f) When to use CQRS pattern**
  - Collaborative domains where many users access the same data in parallel. CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level, and conflicts that do arise can be merged by the command.
  - Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models. The write model has a full command-processing stack with business logic, input validation, and business validation. The write model may treat a set of associated objects as a single unit for data changes (an aggregate, in DDD terminology) and ensure that these objects are always in a consistent state. The read model has no business logic or validation stack, and just returns a DTO for use in a view model. The read model is eventually consistent with the write model.
  - Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the number of reads is much greater than the number of writes. In this scenario, you can scale out the read model, but run the write model on just a few instances. A small number of write model instances also helps to minimize the occurrence of merge conflicts.
  - Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.
  - Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where **business rules change regularly**.
  - Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.
- **g) - This pattern isn't recommended when:**
  - The domain or the business rules are simple.
  - A simple CRUD-style user interface and data access operations are sufficient.

## Reference

- Implement reads/queries in a CQRS microservice: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/cqrs-microservice-reads
- CQRS: https://martinfowler.com/bliki/CQRS.html