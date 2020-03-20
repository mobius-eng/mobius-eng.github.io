---
layout: post
title: Setting up VSCode for Fortran
date:  2020-03-20 21:30:00 +0200
categories: fortran
---

### Fortran IDEs

Fortran is a domain specific language (numerical computation). As a result, it cannot compete in popularity with such general purpose programming languages as C, C++, C#, Java, etc. This, unfortunately, brings a problem: the tooling support for Fortran is not always the best.

Intel Fortran for Windows comes with a decent support for Fortran in MS Visual Studio. Although it lacks autocompletion, its support for debugging makes up for it. It is, however, not free.

[CodeBlocks](http://www.codeblocks.org/) and especially [CBFortran](http://cbfortran.sourceforge.net/) flavour of it is a dependable open-source IDE for Fortran projects. It has a good support for autocompletion and access to documentation (via Doxygen). Debugging is decent, however GDB sometimes might not be able to access certain variables. The interface is a bit outdated and the support for things like font ligatures in e.g. [Fira Code](https://github.com/tonsky/FiraCode) is not there.

[Visual Studio Code](https://code.visualstudio.com/) is a popular extendible editor, one may even say it is a modern age Emacs or Vim. Even though Fortran is not its focus, it is surprisingly easy to set up a productive environment in it. The rest of this post is about how to do it.

<!--more-->

### Pre-requisites

The following needs to be available:
- `gfortran` compiler (preferably in the system path).
- `make` system.
- `gdb` debugger.

On Windows, the recommended way of getting all this is to use [MSYS2 with MINGW64](https://www.msys2.org/).

VSCode itself can be found here. The following plugins need to be installed:
- **Native Debug** for GDB debugging support.
- **Modern Fortran** provides the majority of the Fortran features.
- **Fortran Breakpoint Support**.

For **Modern Fortran** just need to make sure that the executable specifies `gfortran` (or the full path if `gfortran` is not in the system path).

### Starting a project

Create a directory for your project and open it in VSCode (`Ctrl-K, Ctrl-O`). To keep things tidy, I prefer to keep my source files in `src/` directory. We will set up a build process using `Makefile` to put object files into `obj/` and an executable file (assuming it is a program) into `bin/`.

To demonstrate how the build system can cope with a complex project, I have three files: program, module and its sub-module. They look as follows.

Main program:

```fortran
program makeanddebug

    use mymodule

    implicit none

    print *, 'Hello World!'
    call foo
    ! Call a procedure from a module
    call bar(10)

    contains
        subroutine foo
            print *, 'From foo'
        end
end program
```

Module (interface):

```fortran
module mymodule

    implicit none

    interface
        module subroutine bar(x)
            integer, intent(in) :: x
        end subroutine
    end interface

end module
```

And, finally, a sub-module:

```fortran
submodule (mymodule) mymodulea

contains

    module subroutine bar(x)
        integer, intent(in) :: x
        print *, 'bar', x
    end

end submodule
```

All these files are placed in `src/`.

For a very big project with complex dependencies, it is better to set a build system with CMake. For relatively small projects it does look like an overkill. So, for time being I opted for Make. This is my `Makefile`:

```make
# Directories
SRCDIR=src
OBJDIR=obj
BINDIR=bin

# Files
PROGFILE=makeanddebug.f90
SRCFILES= mymodule.f90 mymodulea.f90

OBJFILES=$(SRCFILES:.f90=.o)
PROGOBJ=$(PROGFILE:.f90=.o)
MODFILES=$(SRCFILES:.f90=.mod)
EXE=$(PROGFILE:.f90=.exe)

# Compilers
FC=gfortran
LD=gfortran

# Fortran and linker flags and libraries
FCFLAGS= -g -Wall -J$(OBJDIR) -cpp
LDFLAGS=
LIBS=

all: $(BINDIR)/$(EXE)

$(BINDIR)/$(EXE): $(addprefix $(OBJDIR)/, $(OBJFILES)) $(OBJDIR)/$(PROGOBJ) | $(BINDIR)
	$(LD)  $^ -o $@ $(LDFLAGS) $(LIBS)

$(OBJDIR)/%.o: $(SRCDIR)/%.f90 | $(OBJDIR)
	$(FC) -c -o $@ $< $(FCFLAGS)

$(OBJDIR)/%.mod: $(SRCDIR)/%.f90 | $(OBJDIR)
	$(FC) -c -o $@ $< $(FCFLAGS)

# Make directories as necessary
$(BINDIR):
	mkdir $(BINDIR)

$(OBJDIR):
	mkdir $(OBJDIR)


clean:
	rm $(OBJDIR)/*.*
	rm $(BINDIR)/*.*

run: $(BINDIR)/$(EXE)
	$(BINDIR)/$(EXE)

.PHONY: all clean run

```

The skeleton of this `Makefile` can stay the same across different projects. The only things that need to change are
- The name of the program file (`PROGFILE`),
- List of module files (`SRCFILES`),
- And compiler and linker flags (`FCFLAGS` and `LDFLAGS`) and linked libraries (`LIBS`).

An important note: the order of the files in linker call is important as any modules that are used from the program must appear before the program object file. This is, however, reverse with the static libraries: they need to specified in the order "user-provider", otherwise provided functions are marked as unused and excluded from linkage.

At this point calling `make all` and `make clean` should work. Next step, to tell VSCode to use our build system.

### VSCode project settings

VSCode keeps project-specific settings in the sub-directory `.vscode`. The settings are specified in JSON format. We will need 3 setting files: `settings.json`, `tasks.json`, and `launch.json`. The first is used for general settings. The tasks file contains the build settings. And the launch file is used to set up a debugger.

The linter that comes with the **Modern Fortran** plugin by default is searching for `.mod` files in the current directory of the source file. With the foregoing setup it will put a squiggly line under `mymodule` in the program file. The `settings.json` needs to contain the following setting:

```json
"fortran.includePaths": ["../obj"]
```

The task file is formed by calling the command "Tasks: Configure Task" (`Ctrl-Shift-p` and type `Tasks`). In the list of options choose "Other" and edit `.vscode/tasks.json` to look as follows:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "label": "make",
            "type": "shell",
            "command": "C:\\msys64\\usr\\bin\\make",
            "args": ["all"],
            "problemMatcher": [
                "$gcc"
            ]
        },
        {
            "label": "run",
            "type": "shell",
            "command": "C:\\msys64\\usr\\bin\\make",
            "args": ["run"],
            "problemMatcher" : []           
        }
    ]
}
```

Change the path to `make` to wherever your `make` lives.

Finally, the skeleton for `.vscode/lunch.json` is created by clicking on Run/Debug button on the panel and choosing "create a launch.json file". Edit this file to look as follows:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "gdb",
            "request": "launch",
            "target": "./bin/makeanddebug.exe",
            "cwd": "${workspaceRoot}",
            "valuesFormatting": "parseText"
        }
    ]
}
```

I get a warning with this setup ("Matches multiple schemas..."), but it works fine.

Although the setup took a bit of effort, it was not too difficult. On a positive side, during debugging hovering over variables shows their values. A quick experiment showed the debugger can handle allocatable arrays.

The full code can be found on [Github](https://github.com/mobius-eng/FortranVSCode).