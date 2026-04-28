# Building the First Java SDK for FEAGI

*By Benjamin Bravo, Asliddin Nurboev, and Weifeng Song, University of Pittsburgh, CS 1980 Capstone, Spring 2026*

If you're a Java developer building robotics applications, you've probably run into a familiar wall: the tools you want to use only support Python. That was exactly the case with FEAGI, until now.

Over the course of a semester, our team built the first-ever Java SDK for FEAGI, bringing its brain-inspired AI capabilities to the JVM ecosystem. This post walks through what we built, how we built it, and what we learned along the way.

## What is FEAGI?

[FEAGI](https://feagi.org) (Framework for Evolutionary Artificial General Intelligence) is a brain-inspired AI platform developed by [Neuraville Inc.](https://neuraville.com) that takes a fundamentally different approach from traditional machine learning frameworks like PyTorch or TensorFlow.

Instead of training a model, deploying it, and retraining when conditions change, FEAGI uses an architecture inspired by the stages of human brain development. It learns continuously in real time with no retraining required. This makes it particularly well-suited for robotics, where environments are unpredictable and agents need to adapt on the fly.

At its core, FEAGI is a Rust library. Sensory data, like a camera feed, flows into the neural engine, the engine processes it, and motor commands flow back out to control a robot or simulator. The existing Python SDK made this workflow accessible to Python developers, but Java developers, a huge community in the robotics world, had no way in.

## The Problem

FEAGI's Python SDK is mature. It provides an agent framework, configuration tooling, CLI utilities, and a full PNS (Peripheral Nervous System) communication layer for sending sensory data and receiving motor commands. Python developers can `pip install feagi` and start building agents immediately.

Java developers? Nothing. No SDK, no bindings, no way to interface with FEAGI from the JVM. Our capstone project was to change that.

## Architecture

The Java SDK follows a layered architecture that bridges the gap between Java application code and the Rust-powered FEAGI core:

1. A **Java application** creates an agent using the SDK's public API (`sdk-core`)
2. The SDK passes calls through a **JNI bridge** (`sdk-native`) that handles native library loading and ABI version verification
3. The JNI bridge calls into `feagi-java-ffi`, a **Rust library with a stable C ABI** that exposes opaque handles for client lifecycle, configuration, and data transfer
4. The FFI layer communicates with the **Rust FEAGI core**, which runs the neural engine

Only bytes and JSON cross the FFI boundary. No Rust types leak into Java. Motor commands flow back through the same pipeline in reverse.

## What We Built

Over five two-week sprints, we built the complete SDK from the ground up. Here's what's in the box:

### CLI and Engine Control

The SDK includes a command-line tool that lets you manage FEAGI directly from your terminal. You can start and stop the FEAGI engine, check its status, view configuration, and restart it, all from Java. The engine control layer automatically discovers FEAGI executables from bundled binaries, dev builds, or your system PATH, so it works across different environments without manual configuration.

### Native Rust Integration

This was the hardest part of the project. The SDK loads the platform-specific Rust native library at startup and performs an ABI version handshake to ensure compatibility. If the versions don't match, it fails fast with a clear error instead of silently corrupting data. The JNI bridge handles all the memory management and type marshalling between Java and the C ABI, which is something Python gets almost for free through PyO3 but that required a lot more manual work in Java.

### Configuration Management

FEAGI uses TOML configuration files for agent and engine settings. The SDK provides full configuration support: loading from TOML files with validation, generating default configs to platform-agnostic paths, and cross-platform path utilities that work on Linux, macOS, and Windows.

### Agent Framework

The `BaseAgent` class provides a lifecycle pattern familiar to anyone who's worked with game engines or robotics frameworks:

```java
public class MyRobotAgent extends BaseAgent {
    @Override
    public void initializeHardware() {
        // Connect to your robot or simulator
    }

    @Override
    public SensorData mapSensors(HardwareData data) {
        // Convert robot sensors to FEAGI format
        return new SensorData(cameraBytes);
    }

    @Override
    public MotorCommands mapMotors(FeagiOutput output) {
        // Convert FEAGI commands to robot format
        return new MotorCommands(output);
    }
}
```

We also built a `VideoStreamAgent` template specifically for camera and vision-based agents, since webcam input is one of the most common FEAGI use cases.

### PNS Communication

The Peripheral Nervous System layer is the communication backbone. It handles sensory data transmission by sending XYZP-formatted data to FEAGI non-blocking, motor data reception by receiving and decoding motor commands with cortical ID mapping, transport flexibility through both ZMQ and WebSocket with config-driven endpoints, registration and heartbeat through automatic agent registration with retry logic, capability management for declaring vision units, motor units, and custom capabilities with JSON import/export, and a full set of input/output types including Camera, NumericStream, Infrared, TextStream, ServoMotor, and RotaryMotor.

### Observability

Structured logging via SLF4J with optional metrics gives you visibility into what the SDK is doing at runtime, which is useful for debugging agent behavior and diagnosing connection issues.

## Demo

To demonstrate the SDK end-to-end, we built a demo agent that captures webcam input, sends it to FEAGI as sensory data, and displays the motor output received back from the neural engine. The video below shows the full pipeline working, from the CLI launching the engine, to the brain visualizer showing neural activity in real time, to motor commands flowing back to the Java application.

![video](feagi-java-sdk-demo.mp4)

## Technical Challenges

### Java FFI is Harder Than Python FFI

This was our biggest lesson. Python has PyO3, which provides near-seamless Rust bindings. You write Rust functions and they appear in Python almost magically. Java has JNI, which requires you to manually define native method signatures, write C shim code to bridge between JNI types and the C ABI, handle memory allocation and deallocation across the boundary, and deal with thread safety issues that Python's GIL abstracts away. Every feature that takes a few lines in the Python SDK required significantly more work in Java.

### Understanding FEAGI's Architecture

FEAGI is not a typical software system. It models biological neural architecture, which means concepts like cortical areas, sensory neurons, motor neurons, and connectomes are core abstractions. As computer science students, we had to learn enough neuroscience to understand why the API is shaped the way it is. The Python SDK was our reference implementation, but understanding *why* it worked the way it did required understanding FEAGI's brain-inspired design philosophy.

### Working Within an Existing Ecosystem

We weren't building from scratch. We were building a port. Every design decision had to maintain compatibility with the existing Rust FFI layer, match the Python SDK's behavior, and follow the FEAGI ecosystem's conventions like Apache 2.0 licensing, TOML configuration, and specific transport protocols. This meant constantly cross-referencing three different codebases in three different languages.

## What We Learned

**AI-assisted code review works, but has limits.** We used Claude to review every PR, which caught consistency issues and bugs early. But the usage limits meant we had to be strategic about which PRs got full AI review versus manual team review.

**Issue-per-branch workflow is worth the overhead.** We started with long-lived personal branches and switched to one branch per GitHub issue early in the project. The reduction in merge conflicts and the clarity in the project board made it an easy win.

**Write tests as you build.** Every module shipped with tests. This caught bugs in configuration loading, CLI parsing, and FFI type marshalling before they became integration problems.

**Two-week sprints with a clear backlog keep you honest.** Our sponsor prioritized the backlog, we tracked everything in GitHub Projects with size labels, and sprint reports forced us to confront delays early.

## Getting Started

The FEAGI Java SDK is published on [Maven Central](https://central.sonatype.com/namespace/org.feagi). Add the dependency to your project:

```kotlin
// build.gradle.kts
dependencies {
    implementation("org.feagi:sdk-core:1.0.0")
    implementation("org.feagi:sdk-native:1.0.0")
}
```

From there, create a configuration file, write an agent class extending `BaseAgent`, and run it. The CLI can help you initialize a project and start the FEAGI engine.

The full source code is open source and available on GitHub at [github.com/feagi/feagi-java-sdk](https://github.com/feagi/feagi-java-sdk). Contributions, issues, and feedback are welcome.

## Acknowledgments

This project was completed as part of CS 1980 (Capstone) at the University of Pittsburgh during Spring 2026. We want to thank our sponsor Mohammad at Neuraville Inc. for his guidance, clear prioritization, and responsiveness throughout the semester.

## Links

- **Java SDK on Maven Central:** [central.sonatype.com/namespace/org.feagi](https://central.sonatype.com/namespace/org.feagi)
- **Java SDK Source Code:** [github.com/feagi/feagi-java-sdk](https://github.com/feagi/feagi-java-sdk)
- **FEAGI Core:** [github.com/feagi/feagi](https://github.com/feagi/feagi)
- **FEAGI Website:** [feagi.org](https://feagi.org)
- **Neuraville:** [neuraville.com](https://neuraville.com)
- **Neurorobotics Studio:** [brainsforrobots.com](https://brainsforrobots.com)
