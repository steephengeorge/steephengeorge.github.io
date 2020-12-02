---
layout: inner
position: center
title: An Introduction to GraalVM
date: 2020-06-14 14:15:00
categories: GraalVM
tags: Software Java GraalVM
lead_text: 'GraalVm is a polyglot Virtual Machine(VM) engine built using Java. It provides an interoperability runtime environment with JVM languages;
guest languages like WebAssembly, Javascript, Ruby, Python, and R; LLVM supported languages like C, C++, and Rust. And on top of interoperability,
it provides a high-performance platform for execution.
GraalVM has several patented Just In Time performance optimization capabilities...'
project_link: 'http://steephengeorge.github.io/graalvm/2020/06/14/introduction_graalvm.html'
---

# An Introduction to GraalVM

GraalVm is a polyglot 'VM' engine built using Java. It provides an interoperability runtime environment with JVM languages; guest languages like WebAssembly, Javascript, Ruby, Python, and R; LLVM supported languages like C, C++, and Rust. And on top of interoperability, it provides a high-performance platform for execution. GraalVM has several patented Just In Time performance optimization capabilities.

## Runtime Environments

GraalVM provides multiple runtime options:

- Java Hotspot VM with the GraalVM compiler enabled
- js runtime environment
- LLVM runtime environment

## GraalVM Architecture

![GraalVM Architecture](/img/posts/graalvm-architecture.PNG)

GraalVM provides a Language implementation framework known as Tuffle, and it is an open-source library. It provides a mechanism to create language interpreters for GraalVM by generating a self-modifying Abstract Syntax Tree(AST) for guest languages. Hot paths in ASTs are optimized by GraalVM during the compilation process using JIT.

In the case of LLVM supported languages, they are usually statically compiled. GraalVM provides a tool `lli` that first interprets the program and later compiles it with GraalVM to optimize the hot path.

Let's make our hands dirty now.

GraalVm documentation provides detailed steps to set up with IntelliJ IDE. If you prefer to use IntelliJ use this [guide](https://www.graalvm.org/guides/#run-java-applications-on-graalvm-from-an-ide).

## Set up GralVM with Eclipse

- Download the GraalVM community edition package from the [Github](https://github.com/graalvm/graalvm-ce-builds/releases/tag/vm-20.3.0)repository depends on your operating system.
- Unzip and untar to your local directory. Ex: C:/Tools/graalVm
- Set environment variable JAVA\_HOME and append the JAVA\_HOME to the PATH variable. Ex: JAVA\_HOME=C:/Tools/graalVm/bin and PATH= $PATH:$JAVA\_HOME
- Create a new java project in Eclipse
- Right-click on the project and go to properties
- From the left pan, double click 'Build path'
- From the right pan, select the tab 'Libraries'
- Now you will see the 'JRE System Library' listed below. Select it.
- On the right pan, press the button 'Edit'. It will pop up next window.

![graalvm_eclipse_1](/img/posts/graalvm_eclipse_1.png)

- Press the button 'Installed JREs'. It will pop up next window
- Press the button 'Add' and from the pop-up window select the 'Standard VM'

![graalvm_eclipse_2](/img/posts/graalvm_eclipse_2.PNG)

![graalvm_eclipse_3](/img/posts/graalvm_eclipse_3.PNG)

- In the next pop-up window, press the button 'Directory' to browse and choose the GraalVM home directory. Ex: C:\Tools\graalvm

![graalvm_eclipse_4](/img/posts/graalvm_eclipse_4.PNG)

- Press button Finish
- From the 'Installed JREs' window, choose the newly added GraalVM as our JRE for this project and press the button 'Apply and Close'.
- In the next pending opened window press the buttons Finish
- Hope now you are back to properties windows for your project. Select Java compiler from the left pane.
- On the right pan, against 'the compiler compliance level' chooses the matching Java version from the drop-down box.

![graalvm_eclipse_5](/img/posts/graalvm_eclipse_5.PNG)

- Now you are all set, press the button 'Apply and Close'

Now we can write our first java program using GraalVM. Happy coding!!
