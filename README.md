# AI Red Teaming Agent (Preview)

> **⚠️ DISCLAIMER**
>
> The author of this repository **did not create, author, or curate any of the red teaming prompts** used to test or evaluate the filtering capabilities of AI models or agents. All adversarial prompts, attack objectives, and risk category examples are sourced from Microsoft's open-source [PyRIT](https://github.com/microsoft/pyrit) (Python Risk Identification Tool) and/or the Azure AI Evaluation SDK. These prompts are provided solely for research, evaluation, and safety testing purposes. Use them responsibly and in accordance with all applicable laws and ethical guidelines.

---

## Overview

The **AI Red Teaming Agent** is a tool designed to help organizations proactively identify safety risks in generative AI systems during design and development. By integrating Microsoft's open-source PyRIT red teaming capabilities directly into Azure AI Foundry, teams can automatically scan their model and application endpoints for risks, simulate adversarial probing, and generate detailed reports.

---

## Features

- **Automated Adversarial Probing:** Simulate attacks on your AI system using curated seed prompts and advanced attack strategies.
- **Integration with Azure AI Foundry:** Seamlessly connect to your Azure AI projects and model deployments.
- **Flexible Targeting:** Scan base models, application callbacks, or PyRIT prompt targets.
- **Customizable Risk Categories:** Focus on specific risk areas such as Violence, Hate/Unfairness, Sexual, and Self-Harm.
- **Detailed Reporting:** Generate comprehensive reports on identified risks.

---

## How AI Red Teaming Works

The agent automates adversarial probing of your target AI system. It uses a curated dataset of seed prompts or attack objectives per supported risk category. Attack strategies from PyRIT can help bypass or subvert the AI system’s safety alignments.

For example, a direct question about illegal activity may be refused by the model, but an attack strategy (like character flipping) may trick the model into responding.

---

## Supported Azure Regions

Currently, the AI Red Teaming Agent is only available in the following Azure regions:

- East US2
- Sweden Central
- France Central
- Switzerland West

Ensure your Azure AI Project is located in one of these regions.

---

## Installation

Install the required packages:

```bash
pip install azure-identity azure-ai-agents
pip install azure-ai-evaluation
pip install "azure-ai-evaluation[redteam]"
```

---

## Usage

### 1. Set Up Environment Variables

Set the following environment variables (e.g., in a `.env` file):

- `PROJECT_ENDPOINT`: Your Azure AI Foundry project endpoint
- `MODEL_ENDPOINT`: Your Azure OpenAI model endpoint
- `MODEL_API_KEY`: API key for your model (if not using Entra ID)
- `MODEL_DEPLOYMENT_NAME`: Name of your model deployment

### 2. Instantiate the Red Teaming Agent

```python
from azure.identity import DefaultAzureCredential
from azure.ai.evaluation.red_team import RedTeam

azure_ai_project = os.getenv("PROJECT_ENDPOINT")
red_team_agent = RedTeam(
    azure_ai_project=azure_ai_project,
    credential=DefaultAzureCredential()
)
```

### 3. Supported Targets

You can scan various targets:

#### a. Model Configuration

```python
azure_openai_config = {
    "azure_endpoint": os.getenv("MODEL_ENDPOINT"),
    "api_key": os.getenv("MODEL_API_KEY"),
    "azure_deployment": os.getenv("MODEL_DEPLOYMENT_NAME"),
}
red_team_result = await red_team_agent.scan(target=azure_openai_config)
```

#### b. Simple Callback

```python
def simple_callback(query: str) -> str:
    return "I'm an AI assistant that follows ethical guidelines. I cannot provide harmful content."

red_team_result = await red_team_agent.scan(target=simple_callback)
```

#### c. Complex Callback

```python
async def advanced_callback(messages, stream=False, session_state=None, context=None):
    response = "I'm an AI assistant that follows safety guidelines. I cannot provide harmful content."
    formatted_response = {
        "content": response,
        "role": "assistant"
    }
    return {"messages": [formatted_response]}

red_team_result = await red_team_agent.scan(target=advanced_callback)
```

#### d. PyRIT Prompt Target

```python
from pyrit.prompt_target import OpenAIChatTarget

chat_target = OpenAIChatTarget(
    model_name=os.environ.get("MODEL_DEPLOYMENT_NAME"),
    endpoint=os.environ.get("MODEL_ENDPOINT"),
    api_key=os.environ.get("MODEL_API_KEY")
)
red_team_result = await red_team_agent.scan(target=chat_target)
```

---

## Customization

You can specify risk categories and the number of attack objectives:

```python
from azure.ai.evaluation.red_team import RiskCategory

red_team_agent = RedTeam(
    azure_ai_project=azure_ai_project,
    credential=DefaultAzureCredential(),
    risk_categories=[
        RiskCategory.Violence,
        RiskCategory.HateUnfairness,
        RiskCategory.Sexual,
        RiskCategory.SelfHarm
    ],
    num_objectives=5
)
```

---

## References

- [Microsoft Learn: Run scans with AI Red Teaming Agent](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/run-scans-ai-red-teaming-agent)
- [PyRIT GitHub Repository](https://github.com/microsoft/pyrit)

---

## License

This project is licensed under the MIT