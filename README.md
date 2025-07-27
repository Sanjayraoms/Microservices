# Microservices
Efforts to learn Microservices and event driven architecture. I have followed this tutorial https://youtu.be/DgVjEo3OGBI?si=tTD-FwDGvQwgWP1P by Les Jackson

The motivation for this project is demonstration of asynchronous messaging between two services Platform service and Command service. Also demonstrating the synchronous sync up between two services using REST API and gRPC. 
Solution architecture is as below (copied from original youtube video)
<img width="1788" height="971" alt="image" src="https://github.com/user-attachments/assets/7f8c5fdd-3c2f-469b-9f0f-9357afdc3beb" />


In our case Platformservice is producer/publisher and Commandservice is consumer/subscriber. We will go through each service first and talk about the network infrastructure along the way

PlatformService Architecture
<img width="1788" height="971" alt="image" src="https://github.com/user-attachments/assets/68e751ec-fb6e-47f6-94b3-904d6f6eb61e" />

Platform service does basic CRUD operation for platforms. It has 3 API methods, it creates a platform, gets all platforms and gets platforms by ID. I am using controller based .NET Core REST API. Whenever a platform is created in platforms services, the created platforms are pushed to commands service synchronously using HTTP and it will also asynchronusly be pushed to message bus as shown in the solution architecture screenshot. 
Components used in Platform service
1. We have models for our internal representation of data.
2. DbContext to mediates these models down to SQL server.
3. Using repository pattern to abstract away the DbContext implementation. In this case IPlatformRepo is implemented by PlatformRepo which abstracts DbContext implementation. This service is registered in our program.cs by adding builder.Services.AddScoped<IPlatformRepo,PlatformRepo>().
4. Using Data Transfer Objects (DTOs) which is external representation of our internal models. Import auto-mapper and register in program.cs builder.Services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies()). Creating profiles by implementing AutoMapper.Profile
   eg : mapping : CreateMap<Platform,PlatformReadDto>(); usage : _mapper.Map<PlatformReadDto>(platformItem)
5. Synchronous data transfer to Command Service. Using HttpClient, post created platform to CommandService. In case of synchronous data transfer, platform service needs to have details about Commands Service. Platform service should wait for response from command service. This could lead to long dependency chains.
6. In case of asynchronous data transfer, platform service publishes platform created event to message bus. Here platform service do not even want to know about commannd service. I am using RabbitMQ client here as message bus. Below screenshot gives details on connecting to RabbitMQ message bus. We will talk about ports at the end of this readme. 
<img width="1021" height="448" alt="image" src="https://github.com/user-attachments/assets/a1b4633e-82b8-415b-b17c-aa84440163a2" />
I am using Fanout exchange type, where message is delivered to all queues that are bound to the exchange. It ignores the routing keys as we are doing broadcast messaging here.
<img width="1853" height="742" alt="image" src="https://github.com/user-attachments/assets/b50686dd-8281-4683-b123-cbe3b16253b7" />

CommandService Architecture
<img width="1895" height="1053" alt="image" src="https://github.com/user-attachments/assets/6b99f330-db6f-41b2-bec1-b8c9e4ed2c39" />

Command service API retrieves all the commands associated to given platform id, and gets specific command for the given platformId and CommandId. It also creates command for the given platform Id only if platform exists.
Components used in commands service
1. Models, DbContext, Repository and DTOs remains same as Platform service.
2. In command service, we have MessageBusSubscriber service running as background service. It is registered as a hosted service in program.cs as builder.Services.AddHostedService<MessageBusSubscriber>();. Code for initializing the Message subscriber is as below, which is similar to platform service
<img width="1526" height="635" alt="image" src="https://github.com/user-attachments/assets/5057cef4-2478-439c-aac2-8d619b14502f" />
3. As message bus service is implementing Background service, it needs to implement ExecuteAsync function. The MessageBusSubscriber is continuously liatening to the channel and once the event is received EventProcessor processes the events from here
<img width="848" height="435" alt="image" src="https://github.com/user-attachments/assets/ba3e4c4e-b6d9-4dea-a245-c93b60e7b64a" />
4. Event processor creates the platform in Commad service side
<img width="1198" height="662" alt="image" src="https://github.com/user-attachments/assets/c4608a17-ccd4-4459-81e9-060a2eb58315" />

...to be continued to explain docker and kubernetes orchestration








