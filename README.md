# Spring AI – Tool Calling with LLMs

A hands-on Spring Boot project demonstrating **Tool Calling with Large Language Models (LLMs)** using Spring AI.

The goal of this project is to understand how an LLM can decide when it needs external information or functionality, request a tool call, and use the result returned by the application to generate the final response.

## 🤖 What is Tool Calling?

An LLM by itself can generate text based on the information available in its context. It cannot directly access your application's database, APIs, Java methods, or other systems.

**Tool Calling** allows an application to expose specific functions/tools to an LLM.

The LLM can then:

1. Understand the user's request
2. Decide whether a tool is required
3. Select the appropriate tool
4. Generate the required arguments
5. Ask the application to execute the tool
6. Receive the tool result
7. Use that result to generate the final response

### The basic flow

```text
                    User
                      |
                      v
                Spring Boot
                      |
                      v
                 Spring AI
                      |
                      v
                     LLM
                      |
             "I need to call a tool"
                      |
                      v
              Tool Call Request
                      |
                      v
             Spring AI / Application
                      |
                      v
                Java Tool
                      |
                      v
                Tool Result
                      |
                      v
                     LLM
                      |
                      v
              Final Response
                      |
                      v
                    User
```

The important point is that **the LLM requests the tool call; the application actually executes the tool**. The model does not get direct access to the underlying API or Java method.

---

## 🛠️ Technologies

* Java
* Spring Boot
* Spring AI
* Maven
* LLM / Chat Model
* Spring AI `ChatClient`
* Spring AI `@Tool`

---

## 📚 Concepts Covered

### 1. Defining a Tool

Spring AI allows Java methods to be exposed as tools using the `@Tool` annotation.

For example:

```java
@Tool(description = "Get the current weather for a city")
public String getWeather(String city) {
    return weatherService.getWeather(city);
}
```

The description is important because the LLM uses the tool's name, description, and input schema to determine when the tool is appropriate.

---

### 2. Providing Tools to the LLM

The tool can be made available to the `ChatClient`.

```java
chatClient
    .prompt()
    .user("What is the weather in Toronto?")
    .tools(new WeatherTools())
    .call()
    .content();
```

The application provides the tool definition to the model. The model then decides whether it needs to invoke it.

---

### 3. The Tool Calling Loop

The interesting part happens behind the scenes.

```text
1. User
   |
   | "What is the weather in Toronto?"
   v
2. LLM
   |
   | Decides that weather information is required
   v
3. Tool Call
   |
   | getWeather("Toronto")
   v
4. Spring AI
   |
   | Executes Java method
   v
5. Tool Result
   |
   | "18°C, Cloudy"
   v
6. LLM
   |
   | Uses tool result
   v
7. Final Answer
   |
   | "The weather in Toronto is 18°C and cloudy."
   v
8. User
```

Spring AI manages this tool-calling lifecycle, including executing the requested tool and sending the result back to the model. In Spring AI 2.x, this loop is implemented through the `ToolCallingAdvisor` and `ToolCallingManager`.

---

## 🔑 Important Distinction

One of the main things I learned from this project is that **Tool Calling is not the same as the LLM directly calling an API**.

Instead:

```text
LLM
 |
 | "Call getWeather with city=Toronto"
 v
Spring AI
 |
 | Executes Java method
 v
Your Application
 |
 | Calls API / Database / Service
 v
External System
```

The application remains responsible for executing the actual operation.

This is important from both an architecture and security perspective because the LLM only requests the tool call; it does not receive direct access to the underlying system.

---

## 💡 Why Tool Calling is Useful

Tools allow an LLM application to interact with information and systems outside of the model's built-in knowledge.

### Information retrieval

Tools can retrieve:

* Database records
* Current weather
* Customer information
* Product information
* External API data
* File system information
* Search results

### Taking actions

Tools can also perform actions such as:

* Creating a record
* Sending an email
* Updating information
* Booking something
* Triggering a workflow
* Calling another service

Spring AI categorizes these broadly as information retrieval and taking action.

---

## 🏗️ Architecture

```text
┌───────────────────────┐
│         User          │
└───────────┬───────────┘
            │
            v
┌───────────────────────┐
│     Spring Boot       │
│                       │
│      ChatClient       │
└───────────┬───────────┘
            │
            v
┌───────────────────────┐
│       Spring AI       │
│                       │
│    Tool Calling       │
└───────────┬───────────┘
            │
            v
┌───────────────────────┐
│          LLM          │
│                       │
│ Decides which tool    │
│ should be called      │
└───────────┬───────────┘
            │
            │ Tool Call
            v
┌───────────────────────┐
│      Java Tool        │
│                       │
│       @Tool           │
└───────────┬───────────┘
            │
            v
┌───────────────────────┐
│ External API / DB /   │
│ Other Application     │
└───────────────────────┘
```

---

## ▶️ Running the Project

### Prerequisites

* Java 17+
* Maven
* IntelliJ IDEA or another Java IDE
* An LLM provider that supports tool calling
* Required API credentials, if applicable

### Clone the repository

```bash
git clone https://github.com/avneet297/<repository-name>.git
```

### Build the project

```bash
mvn clean install
```

### Run the application

```bash
mvn spring-boot:run
```

---

## 🔐 API Keys

If the application uses a cloud-based LLM provider, configure the API key using an environment variable.

For example:

```properties
spring.ai.openai.api-key=${OPENAI_API_KEY}
```

Never commit API keys or other secrets to GitHub.

---

## 🧠 Key Takeaways

Through this hands-on implementation, I learned:

* What LLM tool calling is
* How an LLM decides which tool to use
* How Java methods can be exposed as tools
* How `@Tool` works in Spring AI
* How tool descriptions help the LLM select the appropriate tool
* How tool arguments are generated by the model
* How Spring AI executes the requested tool
* How the tool result is returned to the LLM
* How the LLM uses the tool result to produce the final response
* Why the application, rather than the LLM, remains responsible for executing tools

---

## 🚀 What's Next?

Tool calling is an important building block for more advanced AI applications.

Some areas I'm exploring next:

* Spring AI Advisors
* Conversation Memory
* RAG
* Embeddings
* Vector Databases
* MCP (Model Context Protocol)
* AI Agents
* Docker
* Kubernetes

---

## 📖 Resources

* [Spring AI Documentation](https://docs.spring.io/spring-ai/reference/)
* [Spring AI Tool Calling](https://docs.spring.io/spring-ai/reference/api/tools.html)
* [Spring AI Project](https://spring.io/projects/spring-ai)

---

## 📌 About This Repository

This is a **hands-on learning project** created while learning Spring AI and understanding how LLMs can interact with application-defined tools using Java and Spring Boot.
