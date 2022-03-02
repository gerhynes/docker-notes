# Docker Notes
Docker uses containers to run applications in isolated environments on a computer.

Without something like Docker, running multiple applications on a machine can require a considerable amount of setup and configuration, with different apps requiring different versions of the same language or dependencies.

A container packs up everything your app needs to run, from source code to dependencies to runtime environment. The container can run this app in isolation, away from any other processes on the machine, providing a predictable and consistent environment.

A virtual machine has its own full operating system and is typically slower, while containers share the host machine's operating system and are typically quicker.

## Images and Containers
Images are analogous blueprints for containers. They store:
- the runtime environment
- application code
- any dependencies
- extra configuration, such as environment variables
- commands

Images also have their own independent filesystem.

Images are read-only. To change them you need to create a brand new image.

Containers are runnable instances of images. When you run an image, it creates a container, a process that can run your app exactly as outlined in the image.

Containers are isolated processes, they run independently of any other process on the machine.

By sharing an image, multiple containers can be created and run on multiple machines.
