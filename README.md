# ADK Basics Training Guide

This repository contains beginner-friendly examples for learning Google's Agent Development Kit (ADK). Use this README during the training as the main copy-paste guide.

By the end, the team should be able to:

- Set up a Python environment for ADK.
- Create a simple ADK agent.
- Add Python function tools to an agent.
- Run and test agents locally with the ADK web UI.
- Explore the larger examples in this repository.

## 1. Create `requirements.txt`

Create or replace `requirements.txt` in the project root:

```txt
google-adk
google-cloud-aiplatform[adk,agent_engines]
google-genai
google-api-python-client
google-cloud-storage
a2a-sdk
python-dotenv
```

## 2. Create and Activate a Virtual Environment

Use Python 3.10 or newer.

Run these commands from the project root.

macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Windows CMD:

```bat
python -m venv .venv
.venv\Scripts\activate.bat
pip install -r requirements.txt
```

Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## 3. Configure Environment Variables

Create a `.env` file in the project root.

For Google AI Studio API key usage:

```bash
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=key
```

For Vertex AI usage:

```bash
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=your-gcp-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

Use one option only during the basics training. The examples below load `.env` automatically.

## 4. ADK Project Shape

An ADK agent is usually a Python package folder with this shape:

```txt
simple_agent/
  __init__.py
  agent.py
```

The important convention is that `agent.py` exposes a variable named `root_agent`.

## 5. First Agent: No Tools

Create the folder:

```bash
mkdir -p simple_agent
```

Create `simple_agent/__init__.py`:

```python
"""Simple ADK training agent."""
```

Create `simple_agent/agent.py`:

```python
from __future__ import annotations

import dotenv
from google.adk.agents import Agent

dotenv.load_dotenv()


root_agent = Agent(
    name="simple_agent",
    model="gemini-2.5-flash",
    description="A basic ADK training assistant.",
    instruction="""
You are a helpful assistant for an ADK basics training session.

Keep answers clear, practical, and beginner-friendly.
When explaining code, use short examples.
""",
)
```

Run it:

```bash
adk web
```

Open the local URL shown in the terminal, then choose `simple_agent`.

Try these prompts:

```txt
What is ADK in simple terms?
```

```txt
Explain what root_agent means.
```

```txt
Give me three ideas for a useful company agent.
```

## 6. Second Agent: Add Function Tools

Create a new folder:

```bash
mkdir -p product_agent
```

Create `product_agent/__init__.py`:

```python
"""Product information ADK training agent."""
```

Create `product_agent/agent.py`:

```python
from __future__ import annotations

from typing import Literal

import dotenv
from google.adk.agents import Agent

dotenv.load_dotenv()


PRODUCTS = [
    {
        "product_id": "LAPTOP-PRO-14",
        "name": "Laptop Pro 14",
        "category": "laptop",
        "price_lkr": 425000,
        "stock": 8,
        "description": "14-inch laptop for engineering and design teams.",
    },
    {
        "product_id": "MONITOR-27-4K",
        "name": "27-inch 4K Monitor",
        "category": "monitor",
        "price_lkr": 145000,
        "stock": 15,
        "description": "High-resolution monitor for productivity workstations.",
    },
    {
        "product_id": "HEADSET-USB-C",
        "name": "USB-C Headset",
        "category": "accessory",
        "price_lkr": 18500,
        "stock": 32,
        "description": "Noise-cancelling headset for calls and meetings.",
    },
]


def list_products(
    category: Literal["laptop", "monitor", "accessory"] | None = None,
) -> dict:
    """List products, optionally filtered by category.

    Args:
        category: Optional product category to filter by.

    Returns:
        Matching products and a count.
    """
    matches = []
    for product in PRODUCTS:
        if category and product["category"] != category:
            continue
        matches.append(product.copy())

    return {"status": "success", "count": len(matches), "products": matches}


def get_product_details(product_id: str) -> dict:
    """Get details for one product.

    Args:
        product_id: Product identifier, for example LAPTOP-PRO-14.

    Returns:
        Product details if found.
    """
    for product in PRODUCTS:
        if product["product_id"] == product_id:
            return {"status": "success", "product": product.copy()}

    return {"status": "error", "message": f"Unknown product_id: {product_id}"}


def check_stock(product_id: str, quantity: int = 1) -> dict:
    """Check whether the requested product quantity is available.

    Args:
        product_id: Product identifier.
        quantity: Requested quantity.

    Returns:
        Availability result.
    """
    for product in PRODUCTS:
        if product["product_id"] == product_id:
            available = product["stock"] >= quantity
            return {
                "status": "success",
                "product_id": product_id,
                "requested_quantity": quantity,
                "available_stock": product["stock"],
                "is_available": available,
            }

    return {"status": "error", "message": f"Unknown product_id: {product_id}"}


root_agent = Agent(
    name="product_agent",
    model="gemini-2.5-flash",
    description="Product catalogue assistant with simple function tools.",
    instruction="""
You are a product catalogue assistant.

Use the provided tools as the source of truth for product names, prices, stock, and descriptions.
Do not invent product details.
When users ask what is available, list products with product id, name, price, and stock.
When users ask about availability, call the stock tool before answering.
""",
    tools=[
        list_products,
        get_product_details,
        check_stock,
    ],
)
```

Run it:

```bash
adk web
```

Choose `product_agent` in the ADK web UI.

Try these prompts:

```txt
Show me all products.
```

```txt
Do we have 10 monitors available?
```

```txt
Give me details for LAPTOP-PRO-14.
```

```txt
Which accessories are in stock?
```

## 7. What Makes a Python Function an ADK Tool?

ADK can use normal Python functions as tools when you pass them to the agent's `tools` list.

Good tool functions should have:

- A clear function name.
- Typed parameters.
- A docstring with `Args` and `Returns`.
- A structured return value, usually a `dict`.
- No hidden user interaction like `input()`.

Example:

```python
def calculate_total(unit_price: float, quantity: int) -> dict:
    """Calculate a total price.

    Args:
        unit_price: Price for one item.
        quantity: Number of items.

    Returns:
        Total price calculation.
    """
    return {
        "status": "success",
        "unit_price": unit_price,
        "quantity": quantity,
        "total": unit_price * quantity,
    }
```

## 8. Common ADK Commands

Run the web UI:

```bash
adk web
```

Run from a specific parent directory:

```bash
adk web .
```

Install dependencies again after pulling changes:

```bash
pip install -r requirements.txt
```

Deactivate the virtual environment:

```bash
deactivate
```

## 9. Debugging Checklist

If the agent does not show up in `adk web`, check:

- The agent folder has `__init__.py`.
- The agent folder has `agent.py`.
- `agent.py` has `root_agent = Agent(...)`.
- You started `adk web` from the project root.
- Your virtual environment is activated.
- Dependencies installed without errors.

If model calls fail, check:

- `.env` exists in the project root.
- You set either `GOOGLE_API_KEY` or Vertex AI variables.
- `GOOGLE_GENAI_USE_VERTEXAI` matches the option you are using.
- Your API key or Google Cloud credentials are valid.

## 10. Existing Agents in This Repository

Use these after the basics are clear.

`iit_info_agent`
- Information assistant with local data and function tools.

`finetech_crm_agent`
- BigQuery-backed CRM agent.

`finetech_crm_sql_agent`
- SQL-style variant of the CRM BigQuery agent.

`hotel_inventory_a2a_agent`
- Hotel inventory and reservation service exposed over the A2A protocol.

`hotel_booking_a2a_agent`
- Customer-facing hotel booking concierge that calls the inventory agent through A2A.

`nsb_atm_agent`
- Internal NSB ATM operations assistant.
- Includes fleet status, rollout planning, incidents, location tools, and an ADK skill example.

`nsb_loans_agent`
- NSB bank loans information assistant.
- Includes product listing, details, comparison, search, and document tools.

## 11. Google Drive Video Agent Setup

If using the Google Drive video agent, set these environment variables before running or deploying it:

```bash
export GOOGLE_CLOUD_PROJECT="your-project"
export GOOGLE_CLOUD_LOCATION="us-central1"
export GOOGLE_CLOUD_STAGING_BUCKET="gs://your-staging-bucket"
export GOOGLE_DRIVE_VIDEO_BUCKET="your-video-bucket"
export GOOGLE_GENAI_USE_VERTEXAI=TRUE
export VIDEO_SUMMARY_MODEL="gemini-2.5-flash"
```

The runtime identity must have:

- Vertex AI User permissions.
- Storage Object Admin or equivalent access to the target GCS bucket.
- Google Drive read access to the target files.

## 12. A2A Hotel Booking Demo

See [hotel_booking_a2a_demo/README.md](/Users/thusharajayasekara/finetech/ai-agents/hotel_booking_a2a_demo/README.md) for the two-agent A2A demo:

- `hotel_inventory_a2a_agent`
- `hotel_booking_a2a_agent`
