# Markturn project report

The purpose of this report is to highlight the tools used and explain their uses and their features. The intended audience of this report is an intermediate level computer science student or higher, meaning simple concepts such as coding practices will not be explained but more advanced tools will be. Each tool will be explained only to the point of its use or potential use in each activity, meaning only features useful to this project will be covered. This report will be separated into 5 main activities. The reason for dividing the report into these categories is to list and explain each tool used for each category. 

## Coding

The Coding process was optimized and automated by two main tools as described below.

- [Git](#git)
- [GitHub](#github)

In this project Git and GitHub were used together to store the codebase as well as any changes that were made in order to update the project as it was being worked on by both sides. For example, in this project one person was working on the CMakeLists.txt and the other was working on the code base. When a code change was made the developer made a change to the code in the form of a pull request. The Developer working on the CMakeLists.txt could then accept these pull requests and it would make changes to their code and they could see all of the changes made. This allowed for simultaneous development of the project.
## Debugging

The debugging process was optimized and automated by five main tools as described below.

- [Make and Ninja](#make-and-ninja)
- [Cmake](#cmake)
- [CTests](#ctests)
- [Bash aliases](#bash-aliases)
- [Bash functions](#bash-functions)

In this project CMake is used to generate all build files and the build files can be either make or ninja depending on the generator specified. When debugging, it is important to have quality build tools to give comprehensive error messages and warnings. Ninja utilizes g++ which creates compiler errors and warnings. In CMake, CTests is utilized and is used to create executables for multiple test programs. Ninja is then used to compile and run the tests. 

### example commands used

run tests
``` bash
CMake /Source -G Ninja
Ninja tests
```
When attempting to find packages on multiple operating systems, long commands were often repeated line for line. This made aliases and function extremely useful. An Example of this is
``` bash
function runDockerContainer 
{
    docker run --workdir /markturn/build --mount type=bind,source=$(pwd)/markturn-csutyak,target=/markturn -it $1
}
```
This command created a docker container based on the image given by the parameter of the function. It was extremely useful in testing multiple Images, without the need to create a docker compose file. Creating a docker compose file would be helpful for testing but in this case, single use case functions and aliases are more efficient to create.

## Building

The building process was optimized and automated by the three main tools that are described as follows.

- [Make and Ninja](#make-and-ninja)
- [Cmake](#cmake)
- [Docker](#docker)

In this project, multiple docker files are used to simulate building on multiple Linux based operating systems such as Ubuntu, and Fedora. In each docker container the same source code is implemented. This is done using the command 

``` docker
docker run --workdir /markturn/build --mount type=bind,source=$(pwd)/markturn-csutyak,target=/markturn -td markturn
```

This command launches the docker container (in this case it’s a generic ubuntu installation named “markturn”)in the background and binds the source file folder($(pwd)/markturn-csutyak) into the container. With this setup different commands can be executed into the container by getting the container name via the following commands:

``` docker
docker ps
```

And then executing the command via

``` docker
docker exec CONTAINTER_NAME COMMAND
```

In each container the tools CMake and Ninja are utilized to build the source code and the static and shared libraries. CMake is used to generate ninja build files and Ninja is used to generate executables. The commands used to generate these build files are:

``` bash
CMake .. -G Ninja
Ninja
```

## Testing

The testing process was optimized and automated using these two tools:

- [CTest](#ctests)
- [Docker Compose](#docker-compose)

The main tool used in testing for this project was CTest. CTest allows for multiple test executables to be automatically generated by ninja. After generating the ninja files through normal CMake and compiling, the following command can be used to run each test. 

``` bash
ninja test
```

Docker compose was also used in this project to test multiple different images at once. The following docker compose file compiles, tests, and installs markturn on all 4 images. 

``` docker
version: '3.8'

x-setup: &MKTURNTEST
    volumes:
        - type: bind
          source: /markturn-csutyak
          target: /Source
    working_dir: /Source/build
    command: bash -c
        "cmake .. -G Ninja
        && ninja
        && ninja test
        && ninja install "

services:

    markturnubuntu2010:
        image: markturnubuntu2010
        <<: *MKTURNTEST

    markturnfedora32:
        image: markturnfedora32
        <<: *MKTURNTEST
    
    markturnubuntu2004:
        image: markturnubuntu2004
        <<: *MKTURNTEST

    markturnfedora34:
        image: markturnfedora34
        <<: *MKTURNTEST
```      

## installation

The installation process was optimized and automated using this tool

- [CPack](#cpack)
  
with potential future automations with this tool

- [Systemd](#systemd)

The main process of installation is done through CPack. CPack is very simple and installs the executables and the libraries through the code in the CMakeLists.txt file
``` CMake 
install(TARGETS markturn RUNTIME)
install(TARGETS markturn_shared LIBRARY)
install(TARGETS markturn_static LIBRARY)
```
The runtime install, installs the markturn executable and the library install, installs the shared and static libraries. To install these the commands executed are:
``` bash
Cmake .. -G Ninja
Ninja install
```
A notable potential future tool is systemd or another init tool. Markturn currently does not run on a loop. As it stands markturn runs, produces an output, and then the program completes. If the program did run on a loop, needed to run as the system started, or in the background, systemd is a good tool to use. In this instance, systemd would allow the program to run in parallel. Systemd uses service files to generate specific services.  An example of a service file in this instance would be:
```
[Unit]
Description=Markturn service
After=network.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=1
User=root
ExecStart=/Markturn

[Install]
WantedBy=multi-user.target
```
This service file only runs if the markturn executable is in the initial directory. 
After this service file is implemented markturn can then be ran in the background via the command
``` bash 
systemctl start markturn
```
And ended via the command
``` bash
systemctl stopmarkturn
```
---

# Tool Explanation

## Bash aliases

Using bash it is possible to create something called an alias. An alias in bash is a short hand for a command that is set by the user. An example of this would be 
``` bash
alias echoWorld=”echo ‘hello world’”
```
After running the previous command it would then be possible to run
``` bash
echoWorld
```
Which would give the output 
``` bash
Hello world
```

## Bash functions
Bash functions are similar to bash aliases in that they allow for a shorthand of a command. The difference is that bash functions allow for dynamic changes through parameters. With bash functions it is possible to pass the function a parameter when calling it to dynamically change the command that is ran. An example of this would be where each parameter is shown by a $
``` bash
function testFunction
{
    echo $1 
}  
```
After running the previous command it would then be possible to run
``` bash
testFunction Hello
```
Which would then output
``` bash
Hello
```

## Git
Git and GitHub are both tools used to optimize version control. Version control is a software engineering concept to manage changes to files. To optimize version control git has multiple key features used in this project. The main feature used is the feature to commit changes in stages through multiple commits and assign each commit a message and description. The commit concept is very important to version control as it allows one to mark each stage of development and if drastic changes are made to the code base it allows for a rollback to a previous stage.  Git also allows the user to push the code base, as well as a log of all commits and changes to GitHub.

### example Commands

Commit all changes with message

``` bash
git commit -am "add change to document"
```

push changes to github repository
``` bash
git push
```

## GitHub 
GitHub is a platform used to store projects that are version controlled by git. GitHub facilitates its version control changes via pushes and pulls. When someone makes a commit to a project, they are able to push those changes to GitHub. GitHub then stores their commit and changes it in the code. Someone else who has access to the project can then pull the changes from GitHub and get the latest version of the code. Git also checks your local version of the project and makes sure your project is up to date every time you pull so if it is not, one can pull all changes made by others. 

## Make and Ninja
Make and Ninja are two build automation tools that create executable programs and libraries from source code. Make is the predecessor of Ninja and was written as a standalone tool. Make generates a build from a makefile created by the user. When creating a makefile, it is possible to create dependencies to each file and have a specific command run every time the dependencies are changed. This is very important because Make optimizes the build process of source code and makes implementing new changes to the code almost instant. It is possible to create aliases that will run commands. These aliases can have dependencies on specific parts of the source code. For example, creating a “run” command can make a shorthand to run a simple version of the executable and it will depend on the files that are used to create it. This means that when running the command, the file will recompile if any of the dependencies have changed. Ninja is a newer version of Make that is much faster at building. Ninja files, unlike make files, are not meant to be built by hand. Instead, they are meant to be built by meta build generators such as CMake

## CMake
CMake is a meta build generator that is used to generate build files for source code. Unlike Make and Ninja, CMake is not operating system dependent and is made to simplify the process of compiling and generating proper executables, installations, and libraries. CMake uses simple configuration files placed in each source directory called “CMakeLists.txt”.  

## CTests
CMake can be used to generate tests using the command
``` CMake
enable_testing()
```
These tests are created through a tool in CMake called CTest. CTest is an executable generator that creates build trees for each project that will run multiple different test executables declared by the user. 

## CPack
CPack is a package generator that generates binary and source installers in multiple formats using the CPack program within CMake. 

## Docker
Docker is a software platform for building applications based on containers. Containers are execution environments run on a shared use of an operating system kernel. Aside from sharing the kernel each container runs in isolation from each other. This makes it possible to run commands in each container without affecting the other containers. The purpose of a container is to emulate specific OS installations. Docker is a very powerful tool because it allows the user to create a specific OS installation and run commands on it to test software on multiple different OS environments. In essence, one computer could be the host to multiple different operating systems at once via 1 operating system kernel. 

## Docker Compose
Docker compose is a tool for running multi-container docker applications. Docker compose is written in the YAML format. The main purpose is to streamline multi-container testing using a single command. The command can run through multiple containers, do work in them, and then return any errors that can occur.

## systemd
systemd is an init system used to bootstrap user space and manage user processes. Systemd handles starting programs and event logging of said programs.