# Image Layer Caching 

## What is Caching?
- Caching is the process of storing copies of data in a temporary storage location so that they can be accessed more quickly.
	- this is done to improve the performance of systems/applications by reducing the need to access the original data source every time.

## What is Layer?
- A layer is a physical or logical grouping of related components that perform a specific function.
	- layers are used to simplify the design and implementation of complex systems by dividing them into smaller manageable components.

## What is an Image?
- An image can be defined as a visual representation of a digital object.
	- but in this case we are talking about Docker images.
Before defining a Docker image, let it be known that a Docker image contains an OS filesystem, an application and all the application's dependencies. (big words huh!)
Therefore, a Docker image can be said to be a representation of a small server running an application. It contains all the things needed to run an application. 
```text
Imagine you are running an application on your local machine;
you have an OS that it runs on,
an application code and the dependencies needed to run the code.
Now start that application, and pause it.
mentally take that specific process off the machine
and essentially you have a representation of the machine+application+dependencies,
and that is an image.
```
In other words, a `Docker image` is a lightweight, standalone and executable package of software that includes everything needed to run an application; contains the code, runtime (OS) system tools, libraries (dependencies) and settings (tweaks to make the application run).

### So what is Image Layer Caching?
- Combining the three definitions above; we can define Image Layer Caching to be:
	- the storage of data copies of groupings of related components that perform a specific function, 
	- storing the desired outcome of those functions in a lightweight executable package of an application 
	- so that they can be accessed quickly when the application is run without having to perform the functions all over again.
- In Docker, the image layer caching is a feature that saves the individual layers of the Docker images that have been built so that they can be reused when the image is run again.
- To build a Docker image, a Dockerfile is used;
	- the Dockerfile contains step-by-step instructions/commands on how to build the image. 
	- Each instruction/command then becomes a Layer.
	the outcomes of the instruction is copied and stored so that the next time the image is run with the same Dockerfile, every instruction that has been run before is not run again but its outcome passed to the next instruction (or essentially the final command).

### Benefits on Image Layer Caching
With the layers cached when building an image, there is one significant benefit of this and that is:
1. *Faster builds & build times* - this is because the layer instructions do not have to be run again, just copied over, saving time and decreasing the time taken to finish building an image.
2. *Reduced bandwidth usage & reduced latency* - layer instructions sometimes need the internet to run and perform certain actions. With the functions already run and stored locally,  the building process does not need to access the internet to get the results, it just copies them over.
3. *Improved reliability* - with caching, the build layers are always used preventing the possibility of an error occurring during a layer execution process that exposes the image build to errors. Also,  when building images and there is a network outage, the cached layers are used to ensure that the build process is not interrupted. All these improve the reliability of the build.

All these wouldn't be nice without some drawbacks on the benefits:
1. *Increased disk usage* - the more cache is being stored locally, the more disk space is used and an IO operations on the disk.
2. *Complexity* - builds will need the user to configure the caching mechanism and manage the cached layers, this can prove challenging if the user is not familiar with the caching mechanism (or plugin, if it provides more advanced features).

I use two PAAS tools, to package and run applications in a standardized way. Docker and Podman. 
For these tools, there are a number of ways each to perform image layer caching; these are:

| Docker                  | Podman                 |
| ----------------------- | ---------------------- |
| Built-in images caching | Built-in image caching |
| BuildKit                | Buildah                |
| Caching plugins         | Caching plugins                       |

