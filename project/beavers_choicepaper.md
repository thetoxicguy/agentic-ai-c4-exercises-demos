# The Beaver's Choice Paper Company Sales Team - Project Overview

## The Beaver's Choice Paper Company Needs Your Help!

### Introduction

Welcome!

Imagine yourself as a trusted and seasoned consultant specialized in building smart, efficient, and powerful agent-based workflows. Businesses rely on you to solve their complex operational challenges with cutting-edge solutions.

Your latest client, **The Beaver's Choice Paper Company**, urgently requires your expertise to revolutionize their inventory management and quoting system.

Your role is to develop a **multi-agent system** that streamlines operations, enables quick quote generation, provides accurate inventory tracking, and ultimately drives increased sales.

---

## The Challenge

The Beaver's Choice Paper Company is struggling with:

- Managing paper supplies efficiently
- Responding promptly to customer inquiries
- Generating competitive quotes

Because of these inefficiencies, they are losing potential sales opportunities.

### Your Objective

Design and implement a **multi-agent solution** (maximum of **five agents**) capable of:

- Handling customer inquiries
- Checking inventory status
- Providing accurate quotations
- Completing transactions seamlessly

The solution must ensure:

- Responsiveness
- Accuracy
- Reliability

while managing requests and maintaining optimal stock levels.

---

## Core Components of the Multi-Agent System

To meet the company's needs, your implementation must:

- Use **at most five agents**
- Handle **text-based inputs and outputs only**
- Be validated using a provided set of sample requests

### Initial Planning

Begin by creating a workflow diagram that clearly illustrates your intended system design.

### Allowed Frameworks

You may choose one of the following Python agent frameworks:

- `smolagents`
- `pydantic-ai`
- `npcsh`

### Required Agent Capabilities

Your system should include agents that can create:

#### 1. Inventory Management

- Answer questions about current inventory
- Reorder supplies when necessary
- Use database information effectively
- Make purchasing decisions

#### 2. Quote Generation

- Provide accurate and intelligent customer quotes
- Consider historical quote data
- Apply appropriate pricing strategies

#### 3. Sales Processing

- Finalize sales transactions efficiently
- Verify inventory availability
- Consider delivery timelines

---

## Required Deliverables

Your submission must include:

### 1. Workflow Diagram

- Image file showing the multi-agent workflow

### 2. Source Code

- Exactly **one Python file**

### 3. Detailed Documentation

A comprehensive report describing:

- The system architecture
- Design decisions
- How the solution satisfies all requirements

---

## Technologies and Skills

You are expected to apply knowledge of:

- Multi-agent systems
- Python programming
- SQLite databases (`sqlite3`)
- Agent orchestration frameworks

---

## Project Summary

This is a **6-hour project** consisting of four major phases:

### 1. Diagramming & Planning

Create a detailed flow diagram outlining the multi-agent workflow.

### 2. Implementation

Develop the multi-agent system using:

- Python
- Your chosen agent framework

### 3. Testing & Debugging

Test the agents thoroughly using the provided sample inputs to ensure reliable performance.

### 4. Documentation

Write a comprehensive report that explains:

- The system design
- Design decisions
- Compliance with project requirements

---

## Goal

Build a multi-agent system that revolutionizes inventory management and quote generation for The Beaver's Choice Paper Company by improving operational efficiency, inventory accuracy, quotation quality, and sales execution.

---
---
# Project Instructions

## Step 1: Draft Your Agent Workflow

Begin by drafting a diagram illustrating the interactions and data flows between the agents in your multi-agent system.

Your diagram should demonstrate the sequence of operations for handling customer inquiries, inventory management, quote generation, and order fulfillment.

Some recommended agents for your multi-agent system are as follows:

- An orchestrator agent for handling customer inquiries and delegating tasks to different agents
- Answer inventory queries accurately, including deciding when to reorder supplies
- Generate quotes efficiently, applying bulk discounts strategically to encourage sales
- Finalize sales transactions, considering inventory levels and delivery timelines

These agents should also have access to tools that allow them to interact with the system database and the outside world.

Some recommended tools that might help you flesh out your system are:

- A tool that checks inventory for different paper types
- A tool that gets quote history related to a customer's request
- A tool that checks the timeline for delivery of an item from the supplier
- A tool that fulfills orders by updating the system database
- A tool that checks the company’s current cash balance
- A tool that generates global financial and inventory reports

Note that these recommendations for agents and tools are only recommendations. You may want to design the architecture of your multi-agent system differently.

To understand more about the context of this project, you may want to do some reading on inventory and sales management in companies selling physical products.

Count your agents carefully. The total number of agents (Orchestrator + all specialized Worker agents) can be maximum of five.

You can use diagramming tools like Diagrams.net (opens in a new tab) or Mermaid (opens in a new tab) to create your diagrams.

## Step 2: Review the Starter Code

Carefully examine the provided starter code (`project_starter.py`) in your workspace.

This code includes essential functionalities such as:

- Initializing and managing an SQLite database
- Managing inventory stock levels
- Generating and tracking financial transactions
- Utility functions to estimate supplier delivery dates and current cash balance

At the bottom of the `project_starter.py` file, you'll find a provided code stub designed to help you evaluate your agent implementation.

You can use this stub to test and refine your system effectively.

Spend at least 30 minutes reviewing the starter file.

Write brief descriptions for each function provided to ensure you thoroughly understand their purpose and usage within your system.

Your multi-agent system must utilize every single helper function provided in the starter code.

When you update your tool draft, ensure that `get_all_inventory`, `get_cash_balance`, and `generate_financial_report` are assigned to an agent, even if they are only used for internal reporting or health checks before fulfilling large orders.

Once you complete your review, revisit your agent flow draft from Step 1.

Update the tools you initially outlined, replacing hypothetical tools with tools defined using the helper functions provided in the starter code based on your newfound understanding.

## Step 3: Select Your Agent Framework

Decide on the agent orchestration framework you will use for this project.

Your options include:

- `smolagents`: Link to documentation (opens in a new tab)
- `pydantic-ai`: Link to documentation (opens in a new tab)
- `npcsh`: Link to documentation (opens in a new tab)

Make sure you are comfortable with your chosen framework and that it aligns well with the requirements and your intended agent interactions.

## Step 4: Implement Your System

Using the framework you have selected, implement the agents based on the updated flow you refined in Step 2.

Ensure your system follows the agent workflow diagram you drafted in step 1.

Feel free to update the diagram based on changes to your approach if any hiccups arise during the implementation.

You can get started by creating different agents which accomplish each of these tasks and an orchestration agent that orchestrates the flow of control and data between the different agents.

Use the helper functions provided in the starter file to aid the definition of tools for the different worker agents.

Additionally, refer to the project rubric to thoroughly understand the requirements and criteria for successful completion of your system.

## Step 5: Test and Evaluate Your Implementation

Test your multi-agent system thoroughly using the provided dataset (`quote_requests_sample.csv`).

Ensure:

- Your agents correctly handle various customer inquiries and orders
- Orders are accommodated effectively to optimize inventory use and profitability
- The quoting agent consistently provides competitive and attractive pricing
- Your agents can recognize impossible constraints

To pass this project, your system must successfully fulfill at least three orders, but it MUST also actively reject or leave unfulfilled at least one order (e.g., due to insufficient stock or impossible supplier delivery timelines) with a clear reason provided to the customer.

Refer to the project rubric at the end of this step to evaluate the results documented in the `test_results.csv` file.

## Step 6: Reflect and Document

After evaluating your system, prepare a clear and concise report detailing:

- A comprehensive explanation of your multi-agent system
- Evaluation results (`test_results.csv` generated by the evaluation code) highlighting strengths and areas for improvement
- At least two distinct suggestions for further improvements to the system

Your final submission should include:

- Your updated agent flow diagram
- Your completed implementation script
- Your reflective report with evaluations

Ensure that you check the project rubric to understand the requirements from the different parts of the project before submitting it.

You submission will be reviewed against the project rubric.

---
---
# Environment Setup

## Workspace Instructions

All the files have been provided in the VS Code workspace. Please install the agent orchestration framework of your choice.

All the files have been provided in the VS Code workspace on the Udacity platform.

Please install the agent orchestration framework of your choice.

### Start Workspace

Workspaces will shut down after 30 minutes of inactivity.

Any running processes will be stopped.

### Start Workspace

Workspaces may take up to 5 minutes to start.

### View Active Workspaces

## Local setup instructions

### Install dependencies

Make sure you have Python 3.8+ installed.

You can install all required packages using the provided requirements.txt file:

```bash
pip install -r requirements.txt
```

If you're using smolagents, install it separately:

```bash
pip install smolagents
```

For other options like pydantic-ai or npcsh[lite], refer to their documentation.

### .env File

The .env file is already created and includes
- OPENAI_API_KEY:
- OPENAI_BASE_URL proxy hosted at
  ```text
  https://openai.vocareum.com/v1
  ```
- TAVILY_API_KEY
---
---
# Multi-Agent Systems

# Rubric

Use this project rubric to understand and assess the project criteria.

## Agent Workflow Diagram

| Criteria | Submission Requirements |
| ----- | ----- |
| Illustrate the architecture of the multi-agent system, including agent responsibilities and orchestration. | - The workflow diagram includes all of the agents in the multi-agent system (maximum of five agents as per project constraints). <br> -Each agent has explicitly defined responsibilities that do not overlap with other systems. <br> - The orchestration logic and data flow between agents is clear. |
| Illustrate the interactions between agents and their tools, specifying the purpose of each tool. | - The workflow diagram depicts tools associated with specific agents. <br> - For each tool depicted, its purpose and the specific helper function(s) from the starter code it intends to use is specified in the diagram. <br> - The diagram shows interactions (e.g., data input/output) between agents and their respective tools. |

## Multi-Agent System Implementation

| Criteria | Submission Requirements |
| ----- | ----- |
| Implement the multi-agent system with distinct orchestrator and worker agent roles as per their diagram. | - The implemented multi-agent system architecture (agents, their primary roles) matches the submitted agent workflow diagram. <br> - The system includes an orchestrator agent that manages task delegation to other agents. <br> - The system implements distinct worker agents (or clearly separated functionalities within agents) for different tasks such as: <br> i - Inventory management (e.g., checking stock, assessing reorder needs). <br> ii - Quoting (e.g., generating prices, considering discounts). <br> iii - Sales finalization (e.g., processing orders, updating database). <br> - The student selects and utilizes one of the recommended agent orchestration frameworks (smolagents, pydantic-ai, or npcsh) for the implementation. |
| Implement tools for agents using the provided helper functions, ensuring all required functions are utilized. | - Tools for different agents are defined in the code according to the conventions of the selected agent orchestration framework. <br> - All of the following helper functions from the starter code are used in at least one tool definition within the implemented system: create_transaction, get_all_inventory, get_stock_level, get_supplier_delivery_date, get_cash_balance, generate_financial_report, search_quote_history. |

## Evaluation and Reflection

| Criteria | Submission Requirements |
| ----- | ----- |
| Evaluate the multi-agent system using the provided dataset and document the results. | - The multi-agent system is evaluated using the full set of requests provided in quote_requests_sample.csv and the results of the evaluation are submitted in test_results.csv. <br> - The test_results.csv file (or equivalent documented output) demonstrates that: <br> i - At least three requests result in a change to the cash balance. <br> ii - At least three quote requests are successfully fulfilled. <br> iii- Not all requests from quote_requests_sample.csv are fulfilled, with reasons provided or implied for unfulfilled requests (e.g., insufficient stock). |
| Reflect on the architecture, implementation, and performance evaluation of the multi-agent system. | The reflection report: <br> - contains an explanation of the agent workflow diagram, detailing the roles of the agents and the decision-making process that led to the chosen architecture. The student may refer to their diagram file, but the explanation must be in this text report. <br> - discusses the evaluation results from test_results.csv, identifying specific strengths of the implemented system. The student may refer to their test_results.csv file, but the discussion must be in this text report. <br> - includes at least two distinct suggestions for further improvements to the system, based on the identified areas of improvement or new potential features. |

# Industry Best Practices

| Criteria | Submission Requirements |
| ----- | ----- |
| Provide transparent and explainable outputs for customer-facing interactions. | - Outputs generated by the system (e.g., quotes, responses to inquiries) for the "customer" contain all the information directly relevant to the customer's request. <br> - Outputs provided to the "customer" include a rationale or justification for key decisions or outcomes, where appropriate (e.g., why a quote is priced a certain way if discounts are applied, why an order cannot be fulfilled). <br> - Customer-facing outputs do not reveal sensitive internal company information (e.g., exact profit margins, internal system error messages) or any personally identifiable information (PII) beyond what's essential for the transaction. |
| Write code that is readable, well-commented, and modular. | - Variable and function names in the Python code are descriptive and consistently follow a discernible naming convention (e.g., snake_case for functions and variables, PascalCase for classes, if applicable). <br> - The code consists of comments and docstrings at appropriate places. <br> - Logic within the code has been broken down sufficiently into individual modules.|

## Suggestions to Make the Project Stand Out

- Create a customer agent that uses the customer context from the sample requests to then can negotiate with the team multi-agent system here.
- Make a terminal animation that builds on top of outputs from various agents in the multi-agent system to show the customer how their request is being processed.
- Add a business advisor agent that analyses all the transactions being handled by the multi-agent system and proactively recommends changes to the business operations in order to improve its efficiency and revenue.