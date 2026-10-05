# MarketMind AI – Financial Multi-Agent Orchestration

An AI-powered **financial multi-agent system** built using **LangGraph, MCP, Groq, Angel One SmartAPI, BharatStock API, Tavily, PostgreSQL, and Streamlit**.

MarketMind AI provides a unified financial assistant for **stock analysis, company comparison, current market prices, fundamental analysis, technical analysis, financial research, latest financial news, risk analysis, and virtual paper trading**.

The system uses a **LangGraph Supervisor** to understand the user's request and route it to the appropriate specialized agent.

For trading, **Angel One is used only for read-only stock/instrument resolution and market-price retrieval**. BUY and SELL operations are executed only inside the application's **virtual paper-trading engine** and stored in PostgreSQL.

**No real-money order is placed by this project.**

---

## Key Features

* Multi-Agent financial AI architecture
* LangGraph Supervisor-based orchestration
* MCP-based tool integration
* Guardrail for validating user requests
* Intent detection and deterministic routing
* Stock market-price retrieval
* Fundamental analysis
* Technical analysis
* Company comparison
* Risk analysis
* Latest financial news
* Financial and company research
* Virtual BUY/SELL paper trading
* Human approval before virtual trade execution
* PostgreSQL virtual portfolio
* Virtual cash management
* Trade history
* LangGraph state persistence
* Groq-powered financial reasoning
* Streamlit interactive interface
* Angel One TOTP authentication
* BharatStock financial-data integration
* Tavily web research
* Modular and extensible architecture

---

# Architecture Concepts

## 1. Multi-Agent System

MarketMind AI uses a **Multi-Agent Architecture** instead of one single AI agent.

Each specialized agent has a specific responsibility.

```text
                    LANGGRAPH SUPERVISOR
                            │
       ┌────────────┬───────┼────────┬─────────────┐
       ▼            ▼       ▼        ▼             ▼
   Market Agent  Fundamental  Technical  News    Research
                    Agent      Agent     Agent     Agent
       │              │          │        │          │
       └──────────────┴──────────┴────────┴──────────┘
                              │
                         Final Response
```

### Agents used

| Agent                      | Responsibility                             |
| -------------------------- | ------------------------------------------ |
| `market_agent`             | Current stock price and market information |
| `fundamental_agent`        | Financial fundamentals and ratios          |
| `technical_agent`          | Technical/market analysis                  |
| `news_agent`               | Latest financial and company news          |
| `financial_research_agent` | Detailed financial/company research        |
| `comparison_agent`         | Compare two companies/stocks               |
| `risk_agent`               | Identify financial and business risks      |
| `trade_agent`              | BUY/SELL virtual paper trading             |

---

# 2. LangGraph

**LangGraph** is used to build the workflow and coordinate the different agents.

Instead of allowing the LLM to perform everything in one step, LangGraph controls the execution flow.

```text
User Query
    ↓
Guardrail
    ↓
Intent Detection
    ↓
Supervisor
    ↓
Select Agent(s)
    ↓
Execute Tools
    ↓
Combine Results
    ↓
Groq Reasoning
    ↓
Final Response
```

LangGraph also maintains the application state between different nodes.

---

# 3. Supervisor Agent

The **Supervisor** acts as the central router.

For example:

```text
"What is the current price of TCS?"
              ↓
        Market Agent
```

```text
"Give me latest news about TCS"
              ↓
         News Agent
```

```text
"Compare TCS and Infosys"
              ↓
     Comparison Agent
```

```text
"Buy 10 INFY"
              ↓
        Trade Agent
```

For complex requests, the supervisor can select multiple agents.

Example:

```text
"Compare TCS and Infosys with latest news and fundamentals"
```

The supervisor can route the request to:

```text
Comparison Agent
       +
Fundamental Agent
       +
News Agent
```

---

# 4. Guardrails

MarketMind AI contains a **Guardrail layer** before agent execution.

The guardrail checks whether the user's request is appropriate for the capabilities of the financial assistant.

```text
User Query
    ↓
Guardrail
    │
    ├── Allowed → Continue
    │
    └── Rejected → Safe Response
```

The LangGraph state maintains:

```python
guardrail_allowed
guardrail_reason
```

This prevents unrelated or unsupported requests from being blindly passed through the financial agents.

---

# 5. MCP – Model Context Protocol

MarketMind AI uses **MCP (Model Context Protocol)** to expose external tools to the application.

The MCP server provides access to financial-data tools while keeping the application architecture modular.

```text
                MarketMind AI
                     │
                     ▼
                 MCP Client
                     │
                     ▼
            Custom Market MCP Server
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      Angel One             BharatStock
```

Tavily is also accessed through an MCP server.

This makes external tools easier to integrate without tightly coupling every API directly into the agent logic.

---

# 6. Human-in-the-Loop

BUY and SELL operations require **human approval** before the virtual trade is executed.

Example:

```text
User:
Buy 10 INFY
       ↓
Trade Agent
       ↓
Angel One
       ↓
Current LTP
       ↓
Virtual Trade Preview
       ↓
Human Approval
       ↓
Virtual Execution
```

This prevents an accidental command from immediately changing the virtual portfolio.

---

# 7. Paper Trading

The project uses **paper trading only**.

When the user says:

```text
Buy 10 INFY
```

the system:

1. Identifies INFY.
2. Retrieves the market price.
3. Calculates the trade value.
4. Shows a virtual trade preview.
5. Requests approval.
6. Deducts virtual cash.
7. Adds INFY to the virtual portfolio.
8. Stores the transaction in PostgreSQL.

Example:

```text
Virtual Cash
₹50,00,000
      ↓
BUY 10 INFY @ ₹1,500
      ↓
Investment = ₹15,000
      ↓
Virtual Cash
₹49,85,000
```

No real shares are purchased.

---

# 8. Angel One

Angel One SmartAPI is used specifically for the **paper-trading market-data workflow**.

It is used for:

* Angel One authentication
* TOTP authentication
* NSE instrument search
* Trading-symbol resolution
* Symbol token resolution
* Current LTP retrieval

The application **does not call**:

```text
placeOrder()
modifyOrder()
cancelOrder()
```

Therefore:

```text
Angel One
   ↓
Market Data
   ↓
Virtual Trading Engine
   ↓
PostgreSQL
```

and not:

```text
Angel One
   ↓
Real Broker Order
```

---

# 9. BharatStock

BharatStock is used for financial and market-data operations such as:

* Current market data
* Fundamental information
* Financial ratios
* Historical information
* Technical/market information
* Stock research data

Example:

```text
"What are the fundamentals of TCS?"
```

```text
User
 ↓
Supervisor
 ↓
Fundamental Agent
 ↓
BharatStock
 ↓
Groq
 ↓
Financial Analysis
```

---

# 10. Tavily

Tavily is used for **web-based financial research and latest news**.

### Latest News

Example:

```text
Give me the latest news about INFY
```

Flow:

```text
User
 ↓
Supervisor
 ↓
News Agent
 ↓
Tavily
 ↓
Latest Web Results
 ↓
Groq
 ↓
News Summary
```

### Financial Research

Example:

```text
Do financial research on TCS
```

Tavily can retrieve information about:

* Business model
* Company developments
* Earnings
* Revenue trends
* Industry
* Competitors
* Management developments
* Contracts
* Regulatory developments
* Business risks

The results are then processed by Groq.

---

# 11. Groq

Groq provides the **Large Language Model** used for reasoning and response generation.

It is used for:

* Natural-language understanding
* Intent interpretation
* Financial reasoning
* Company analysis
* Comparison
* Research summarization
* Final response generation

Example:

```text
Tavily
   ↓
Raw Web Results
   ↓
Groq
   ↓
Structured Financial Research
```

---

# 12. PostgreSQL

PostgreSQL is the persistent database for the application.

It stores:

```text
Virtual Cash
Virtual Portfolio
Trade History
LangGraph State
```

### Virtual Portfolio

Example:

```text
Symbol: INFY
Quantity: 10
Average Price: ₹1,500
```

### Trade History

Example:

```text
Action: BUY
Symbol: INFY
Quantity: 10
Price: ₹1,500
Total: ₹15,000
```

The database allows the virtual portfolio to remain available after restarting Streamlit.

---

# Complete Architecture

```text
                              USER
                                │
                                ▼
                         STREAMLIT UI
                                │
                                ▼
                         LANGGRAPH GRAPH
                                │
                                ▼
                            GUARDRAIL
                                │
                                ▼
                           SUPERVISOR
                                │
       ┌──────────┬─────────────┼────────────┬─────────────┐
       │          │             │            │             │
       ▼          ▼             ▼            ▼             ▼
   MARKET     FUNDAMENTAL   TECHNICAL      NEWS       RESEARCH
   AGENT        AGENT         AGENT        AGENT        AGENT
       │          │             │            │             │
       ▼          ▼             ▼            ▼             ▼
   Angel One   BharatStock   BharatStock   Tavily       Tavily
       │          │             │            │             │
       └──────────┴─────────────┴────────────┴─────────────┘
                                │
                                ▼
                         GROQ LLM
                                │
                                ▼
                       FINAL RESPONSE


                         TRADING PATH
                              │
                              ▼
                         TRADE AGENT
                              │
                              ▼
                         ANGEL ONE
                        Read-Only LTP
                              │
                              ▼
                    VIRTUAL TRADE PREVIEW
                              │
                              ▼
                       HUMAN APPROVAL
                              │
                              ▼
                    PAPER TRADE EXECUTION
                              │
                              ▼
                         POSTGRESQL
                     ┌────────┼────────┐
                     ▼        ▼        ▼
                 Cash     Portfolio   Trades
```

---

# Provider Responsibility Matrix

| Component                 | Provider                    | Purpose                                |
| ------------------------- | --------------------------- | -------------------------------------- |
| AI reasoning              | Groq                        | LLM reasoning and response generation  |
| Agent orchestration       | LangGraph                   | Multi-agent workflow                   |
| Guardrails                | LangGraph/Application Logic | Validate and control requests          |
| Tool integration          | MCP                         | Connect external tools                 |
| Paper-trading market data | Angel One                   | Instrument search + LTP                |
| Market/fundamental data   | BharatStock                 | Price, fundamentals and financial data |
| Latest news               | Tavily                      | Recent web/news search                 |
| Financial research        | Tavily                      | Company and industry research          |
| Persistence               | PostgreSQL                  | Portfolio, trades and state            |
| User interface            | Streamlit                   | Interactive financial assistant        |

---

# Project File Structure

This is the **actual structure of the current project**:

```text
MarketMind_AI/
│
├── apps.py
│   └── Streamlit user interface
│
├── backent.py
│   └── LangGraph workflow
│       ├── State definition
│       ├── Guardrail
│       ├── Supervisor
│       ├── Multi-agent routing
│       ├── Market agent
│       ├── Fundamental agent
│       ├── Technical agent
│       ├── News agent
│       ├── Financial Research agent
│       ├── Comparison agent
│       ├── Risk agent
│       ├── Trade agent
│       ├── Human approval
│       ├── Virtual trade execution
│       └── PostgreSQL operations
│
├── custom_market_mcp_server.py
│   └── Custom MCP financial tools
│       ├── Angel One integration
│       ├── Angel One instrument search
│       ├── Angel One LTP
│       └── BharatStock tools
│
├── mcp_client.py
│   └── MCP client configuration
│       ├── Groq LLM
│       ├── Market MCP server
│       └── Tavily MCP server
│
├── requirements.txt
│   └── Python dependencies
│
├── .env.example
│   └── Environment-variable template
│
└── README.md
    └── Project documentation
```

---

# Detailed File Responsibilities

## `apps.py`

Streamlit frontend.

Responsible for:

* Chat interface
* User input
* Trade approval
* Portfolio display
* Trade results
* Session/thread handling

---

## `backent.py`

Main application logic and LangGraph workflow.

Contains:

```text
MarketMindState
Guardrail
Supervisor
Agents
Trading Logic
Human Approval
PostgreSQL
LangGraph Graph
```

This is the central orchestration file.

---

## `custom_market_mcp_server.py`

Custom MCP server for financial APIs.

Connects the application with:

```text
Angel One
BharatStock
```

It exposes market-data tools to the MCP client.

---

## `mcp_client.py`

Creates and manages MCP connections.

It connects:

```text
Groq
   +
Market MCP Server
   +
Tavily MCP Server
```

and exposes the tools to the application.

---

## `requirements.txt`

Contains required Python packages:

```text
Streamlit
LangGraph
LangChain
Groq
MCP
PostgreSQL
Psycopg
Angel One SmartAPI
PyOTP
Pandas
Requests
Python-dotenv
```

---

## `.env.example`

Template for API credentials and configuration.

```env
GROQ_API_KEY=YOUR_GROQ_API_KEY
GROQ_MODEL=llama-3.3-70b-versatile

ANGEL_API_KEY=YOUR_ANGEL_API_KEY
ANGEL_CLIENT_ID=YOUR_ANGEL_CLIENT_ID
ANGEL_PASSWORD=YOUR_ANGEL_PASSWORD
ANGEL_TOTP_KEY=YOUR_ANGEL_TOTP_KEY

BHARATSTOCK_API_KEY=YOUR_BHARATSTOCK_API_KEY
BHARATSTOCK_BASE_URL=YOUR_BHARATSTOCK_BASE_URL

TAVILY_API_KEY=YOUR_TAVILY_API_KEY

DATABASE_URL=YOUR_POSTGRESQL_DATABASE_URL

VIRTUAL_INITIAL_CASH=5000000
```

---

# Example Queries

### Market Price

```text
What is the current share price of TCS?
```

### Fundamental Analysis

```text
What are the fundamentals of Infosys?
```

### Technical Analysis

```text
Give me technical analysis of RELIANCE.
```

### Latest News

```text
Give me the latest news about HDFC Bank.
```

### Financial Research

```text
Do financial research on TCS.
```

### Company Comparison

```text
Compare TCS and Infosys.
```

### Risk Analysis

```text
What are the major risks of Reliance?
```

### Paper Trading

```text
Buy 10 INFY
```

```text
Sell 5 INFY
```

### Portfolio

```text
Show my virtual portfolio.
```

```text
Show my trade history.
```

---

# End-to-End BUY Example

```text
User
 │
 │ "Buy 10 INFY"
 ▼
Guardrail
 │
 ▼
Intent Detection
 │
 ▼
Trade Agent
 │
 ▼
Angel One Authentication
 │
 ▼
Search INFY
 │
 ▼
Get INFY LTP
 │
 ▼
Virtual Trade Preview
 │
 ▼
Human Approval
 │
 ▼
PostgreSQL Transaction
 │
 ├── Deduct Virtual Cash
 ├── Add INFY Holdings
 └── Save Trade History
 │
 ▼
Streamlit
 │
 ▼
"Virtual BUY successful"
```

---

# End-to-End Company Research Example

```text
User
 │
 │ "Do financial research on TCS"
 ▼
Guardrail
 │
 ▼
Supervisor
 │
 ▼
Financial Research Agent
 │
 ▼
Tavily MCP
 │
 ▼
Web Research
 │
 ▼
Groq
 │
 ▼
Financial Research Response
```

---

# End-to-End Comparison Example

```text
User
 │
 │ "Compare TCS and Infosys"
 ▼
Supervisor
 │
 ├───────────────┐
 ▼               ▼
BharatStock     Tavily
 │               │
 ▼               ▼
Fundamentals    Latest News
Price           Research
Ratios
 │               │
 └───────┬───────┘
         ▼
       Groq
         │
         ▼
 Company Comparison
```

---

# How to Run

## 1. Clone Repository

```bash
git clone https://github.com/priyanshukushwa0/MarketMind-AI-Financial-Multi-Agent-Orchestration.git
cd MarketMind_AI
```

## 2. Create Virtual Environment

```bash
python -m venv venv
```

### Windows

```bash
venv\Scripts\activate
```

### Linux/macOS

```bash
source venv/bin/activate
```

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

## 4. Configure `.env`

Copy:

```text
.env.example
```

to:

```text
.env
```

Then add your API credentials.

## 5. Create PostgreSQL Database

```sql
CREATE DATABASE marketmind;
```

## 6. Start Application

```bash
python -m streamlit run apps.py
```

---

# Security

Never commit the following to GitHub:

```text
.env
API keys
Database passwords
Angel One credentials
Tavily API key
Groq API key
```

Add `.env` to `.gitignore`:

```text
.env
venv/
__pycache__/
*.pyc
```

---

# Paper Trading Safety

The trading system is designed as a **virtual/paper-trading system**.

The project uses Angel One for:

```text
Authentication
Instrument Search
LTP Retrieval
```

It does **not** use:

```text
placeOrder()
modifyOrder()
cancelOrder()
```

The actual portfolio transaction happens inside PostgreSQL.

Therefore:

```text
BUY
 ↓
Virtual Portfolio
```

and:

```text
SELL
 ↓
Virtual Portfolio
```

No real-money transaction is performed.

---

# Technology Summary

```text
                    MarketMind AI

                         │
              ┌──────────┴──────────┐
              │                     │
         Multi-Agent            MCP Tools
         Architecture               │
              │              ┌──────┼──────┐
              │              ▼      ▼      ▼
          LangGraph       Angel  Bharat  Tavily
              │            One   Stock
              │
              ▼
            Groq
              │
              ▼
         PostgreSQL
              │
              ▼
          Streamlit
```


---

# Project Overview

This project is developed for **research, and paper-trading purposes**.

The virtual BUY/SELL system does not place real-money orders. Market data and AI-generated analysis may contain errors, delays, or incomplete information. Users should independently verify financial information before making real investment decisions.

---
