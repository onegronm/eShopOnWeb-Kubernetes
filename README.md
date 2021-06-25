# Microsoft eShopOnWeb ASP.NET Core Reference Application on Kubernetes

Sample ASP.NET Core reference application, powered by Microsoft, demonstrating a single-process (monolithic) application architecture and deployment model. 

## Running the sample using Docker

You can run the Web sample by running these commands from the root folder (where the .sln file is located):

```
docker-compose build
docker-compose up
```

You should be able to make requests to localhost:5106 for the Web project, and localhost:5200 for the Public API project once these commands complete. If you have any problems, especially with login, try from a new guest or incognito browser instance.

You can also run the applications by using the instructions located in their `Dockerfile` file in the root of each project. Again, run these commands from the root of the solution (where the .sln file is located).
