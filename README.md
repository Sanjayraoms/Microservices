# Microservices
Efforts to learn Microservices and event driven architecture. I have followed this tutorial https://youtu.be/DgVjEo3OGBI?si=tTD-FwDGvQwgWP1P 

The motivation for this project is demonstration of asynchronous messaging between two services Platform service and Command service. Also demonstrating the synchronous sync up between two services using REST API and gRPC. 
Solution architecture is as below (copied from original youtube video)
<img width="1788" height="971" alt="image" src="https://github.com/user-attachments/assets/7f8c5fdd-3c2f-469b-9f0f-9357afdc3beb" />


In our case Platformservice is producer/publisher and Commandservice is consumer/subscriber. We will go through each service first and talk about the network infrastructure along the way

PlatformService
1. Platform service does basic CRUD operation for platforms. It has 3 API methods, it creates a platform, gets all platforms and gets platforms by ID.
2. I am using controller based .NET Core REST API.
3. Whenever a platform is created in platforms services, the created platforms are pushed to commands service synchronously using HTTP and it will also asynchronusly pushed to   
