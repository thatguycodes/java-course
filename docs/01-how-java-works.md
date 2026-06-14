# How Java Runs Everywhere

[← Back to README](../README.md)

---

Java's promise is **Write Once, Run Anywhere (WORA)**. You write and compile Java code once, and it runs on any machine — Windows, macOS, Linux, mobile — without recompilation.

## How it works

When you compile a `.java` file, the Java compiler (`javac`) does not produce native machine code. Instead it produces **bytecode** — a platform-neutral instruction set stored in `.class` files. Bytecode is not tied to any CPU or operating system.

At runtime, the **Java Virtual Machine (JVM)** reads that bytecode and translates it into native instructions for whatever machine it is running on. Each platform has its own JVM implementation, but all JVMs understand the same bytecode — that's what makes portability possible.

```mermaid
flowchart TD
    src["📄 Source Code\n(.java)"]
    compiler["☕ Java Compiler\njavac"]
    bytecode["📦 Bytecode\n(.class)"]

    src --> compiler --> bytecode

    bytecode --> jvm_win
    bytecode --> jvm_mac
    bytecode --> jvm_linux

    subgraph win["🪟 Windows Machine"]
        jvm_win["JVM"]
        jre_win["JRE"]
        os_win["Windows OS"]
        hw_win["x86 Hardware"]
        jvm_win --> jre_win --> os_win --> hw_win
    end

    subgraph mac["🍎 macOS Machine"]
        jvm_mac["JVM"]
        jre_mac["JRE"]
        os_mac["macOS"]
        hw_mac["ARM / x86 Hardware"]
        jvm_mac --> jre_mac --> os_mac --> hw_mac
    end

    subgraph linux["🐧 Linux Machine"]
        jvm_linux["JVM"]
        jre_linux["JRE"]
        os_linux["Linux OS"]
        hw_linux["Any Hardware"]
        jvm_linux --> jre_linux --> os_linux --> hw_linux
    end
```

## The Java Platform Stack

Each machine that runs Java has these layers:

```mermaid
block-beta
    columns 1
    jdk["☕ JDK — Java Development Kit\n(compiler + tools + JRE)"]
    jre["📚 JRE — Java Runtime Environment\n(standard libraries + JVM)"]
    jvm["⚙️ JVM — Java Virtual Machine\n(executes bytecode)"]
    os["🖥️ Operating System\n(Windows / macOS / Linux)"]
    hw["🔩 Hardware\n(CPU + Memory)"]

    jdk --> jre --> jvm --> os --> hw
```

| Layer | Full name | Role |
|-------|-----------|------|
| **JDK** | Java Development Kit | Everything needed to write and compile Java. Includes `javac`, debugger, profiler, and the JRE. Install this to develop. |
| **JRE** | Java Runtime Environment | Everything needed to *run* Java programs. Includes the standard library (`java.lang`, `java.util`, etc.) and the JVM. End users need this. |
| **JVM** | Java Virtual Machine | Loads `.class` files, verifies bytecode, and executes it by translating to native machine instructions (via JIT compilation). |
| **OS** | Operating System | Manages hardware resources. The JVM talks to the OS, not directly to hardware. |
| **Hardware** | CPU + Memory | The physical machine. Java code never targets this directly. |

## JIT Compilation

The JVM doesn't just interpret bytecode line by line — it uses a **Just-In-Time (JIT) compiler** to detect frequently executed code ("hot paths") and compile them to optimised native machine code at runtime. This is why long-running Java programs often get *faster* over time.

```mermaid
flowchart LR
    bc["Bytecode"] --> interp["Interpreter\n(starts immediately)"]
    interp --> detect["JIT detects\nhot paths"]
    detect --> jit["JIT Compiler\n(compiles to native)"]
    jit --> native["Native Machine Code\n(runs at full speed)"]
```

---

[← Back to README](../README.md)
