
# Langgraph Learning Journey

## Day 1
### Duration : 3 hours

### Learnings

* All code available in Google Colab notebook (langchain learning journey)

* State : State is the current value

* Nodes : These are functions that contain logic of the agents

* Edges : Edges specify how the bot should transition between these functions(i.e. nodes)

* Edge vs Conditional Edge : A conditional edge may not always be traversed, but only when a condition is satisfied. A normal edge is always traversed if the associated node is reached

* Environment variable : A variable whose value is set outside of a program and can affect how that program runs. Below are a few ways to set environment variables (like LLM API keys)

```
## Approach 1 (common in Jupyter notebook)
import os
import getpass

os.environ['OPENAI_API_KEY'] = getpass.getpass("openai_api_key ")

## Approach 2 (using dotenv)



```

* Below is a simple example of a basic chatbot using Langgraph (that using gpt-3.5-turbo model in the backend)

```
!pip install -U langgraph langchain_openai

from typing import Annotated
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI


class State(TypedDict):
    # Messages have the type "list". The `add_messages` function
    # in the annotation defines how this state key should be updated
    # (in this case, it appends messages to the list, rather than overwriting them)
    messages: Annotated[list, add_messages]


graph_builder = StateGraph(State)

# https://platform.openai.com/docs/pricing
llm = ChatOpenAI(model="gpt-3.5-turbo")

def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}


# The first argument is the unique node name
# The second argument is the function or object that will be called whenever
# the node is used.
graph_builder.add_node("chatbot", chatbot)

# Add connections bw nodes
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)


graph = graph_builder.compile()

final_state = graph.invoke(
    {"messages": [{"role": "user", "content": "what is the capital of India?"}]},
)

print(final_state["messages"][-1].content)

```

* Below is the response generated on invoking the graph in the above code

```

{'messages': 
    [
        HumanMessage(content='what is the capital of India?', additional_kwargs={}, response_metadata={}, id='3f834877-da63-4c64-bf26-d22c080aa18d'),
        
        AIMessage(content='The capital of India is New Delhi.', additional_kwargs={},response_metadata={'token_usage': {'completion_tokens': 9, 'prompt_tokens': 14, 'total_tokens': 23, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-3.5-turbo-0125'}, id='run-46bc7501-87b0-4dc8-a5d9-a8d9a9721129-0')
        
    ]
}


```

* Superstep : Each sequential node is a separate super step, wheras parallel nodes share the same super step

* Checkpoint : a snapshot of the graph (saved at each super step). It is represented by `StateSnapshot` and can be seen using `graph.get_state(config)`

* Thread : When invoking graph with a checkpointer, **you must specify a thread_id as part of the configurable portion of the config** A thread is a unique ID or thread identifier assigned to each checkpoint saved by a checkpointer.

* Every thread saves a separate histroy. So if we have a series of messages on thread 1, and then change config to thread 2, then we won't have access to messages of thread 1, unless and until we change the congif back to thread 1

* Persistent checkpointing : When you compile graph with a checkpointer, the checkpointer saves a checkpoint of the graph state. 

* We can add memory to the chatbot using MemorySaver as shown below

```
from typing import Annotated
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import MemorySaver


class State(TypedDict):
    # Messages have the type "list". The `add_messages` function
    # in the annotation defines how this state key should be updated
    # (in this case, it appends messages to the list, rather than overwriting them)
    messages: Annotated[list, add_messages]


graph_builder = StateGraph(State)

# https://platform.openai.com/docs/pricing
llm = ChatOpenAI(model="gpt-3.5-turbo")

memory = MemorySaver()


def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}


# The first argument is the unique node name
# The second argument is the function or object that will be called whenever
# the node is used.
graph_builder.add_node("chatbot", chatbot)

# Add connections bw nodes
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)


graph = graph_builder.compile(checkpointer=memory)

# thread is a must while using checkpoint
config = {"configurable": {"thread_id": "1"}}


final_state = graph.invoke(
    {"messages": [{"role": "user", "content": "what is the capital of India?"}]},
)

print(final_state["messages"][-1].content)
# The capital of India is New Delhi.

final_state = graph.invoke(
    {"messages": [{"role": "user", "content": "How about United Kingdom?"}]},
)

print(final_state["messages"][-1].content) 
# The capital of the United Kingdom is London.

print(graph.get_state(config))
```

* Below is the checkpoint we get for the above code

```
StateSnapshot(
        values={
        'messages': 
            [
                HumanMessage(content='what is the capital of India?', additional_kwargs={}, response_metadata={}, id='54101102-8dcc-40d8-8a00-0b8ba26b7d2c'), 
                
                AIMessage(content='The capital of India is New Delhi.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 9, 'prompt_tokens': 14, 'total_tokens': 23, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-fd763bf8-7a55-4aaf-ab78-8d24fe7e250b-0', usage_metadata={'input_tokens': 14, 'output_tokens': 9, 'total_tokens': 23, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}}), 
                
                HumanMessage(content='How about United Kingdom?', additional_kwargs={}, response_metadata={}, id='9c204d68-ac10-4c30-a45f-6ee0f059b031'), AIMessage(content='The capital of the United Kingdom is London.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 35, 'total_tokens': 45, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-07e9b093-014c-4c56-9307-d50a1d793f09-0', usage_metadata={'input_tokens': 35, 'output_tokens': 10, 'total_tokens': 45, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})
            ]
        }, 
        next=(), 
        
        config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1eff9802-0889-611a-8004-5251499b707d'}}, 
        
        metadata={'source': 'loop', 'writes': {'chatbot': {'messages': [AIMessage(content='The capital of the United Kingdom is London.', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 35, 'total_tokens': 45, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-07e9b093-014c-4c56-9307-d50a1d793f09-0', usage_metadata={'input_tokens': 35, 'output_tokens': 10, 'total_tokens': 45, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 0}})]}}, 'thread_id': '1', 'step': 4, 'parents': {}}, 
        
        created_at='2025-03-05T05:10:16.872536+00:00', 
        
        parent_config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1eff9802-036a-60c5-8003-53e22cb475e0'}}, 
        
        tasks=()
    )

```


### Doubts
1. How to explain state concept to a 5 year old?

### References
1. https://langchain-ai.github.io/langgraph/concepts/low_level/
2. https://langchain-ai.github.io/langgraph/tutorials/introduction
3. https://langchain-ai.github.io/langgraph/concepts/persistence
4. https://dev.to/jamesli/advanced-langgraph-implementing-conditional-edges-and-tool-calling-agents-3pdn


## Day 2
### Duration : 2 hours

### Learnings
* Conditional edges usually contain "if" statements to route to different nodes depending on the current graph state. The if condition is generally within a router function. This router function returns a string, based on which the next node is decided


```
from typing import Annotated
from typing_extensions import TypedDict

from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import MemorySaver
from langchain_core.prompts import ChatPromptTemplate


class State(TypedDict):
    # Messages have the type "list". The `add_messages` function
    # in the annotation defines how this state key should be updated
    # (in this case, it appends messages to the list, rather than overwriting them)
    messages: Annotated[list, add_messages]


graph_builder = StateGraph(State)

# https://platform.openai.com/docs/pricing
llm = ChatOpenAI(model="gpt-3.5-turbo")

memory = MemorySaver()


tagging_prompt = ChatPromptTemplate.from_template(
"""
You are an expert classifier. Based on the sentence below, respond with the word "Tableau" if the sentence is related to Tableau, respond with the word "Power BI" if the sentence is related to Power BI.

Sentence : 
{sentence}

"""
)

def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}


def tableau_chatbot(state: State):
  return {"messages": "Welcome to Tableau support"}

def powerbi_chatbot(state: State):
  return {"messages": "Welcome to Power BI support"}


def router(state: State):
  user_input = state["messages"][-1].content
  prompt = tagging_prompt.invoke({"sentence": user_input})
  response = llm.invoke(prompt)

  if response.content.lower() == "tableau":
    return "Tableau"
  if response.content.lower() == "power bi":
    return "Power BI"

  return "END"

# The first argument is the unique node name
# The second argument is the function or object that will be called whenever
# the node is used.
graph_builder.add_node("chatbot", chatbot)
graph_builder.add_node("tableau_chatbot", tableau_chatbot)
graph_builder.add_node("powerbi_chatbot", powerbi_chatbot)

# Add connections bw nodes
graph_builder.add_edge(START, "chatbot")
graph_builder.add_conditional_edges(
    "chatbot",
    router,
    # The following dictionary lets you tell the graph to interpret the condition's outputs as a specific node
    # If the output/return value of the router function is "Tableau", it goes to tableau_chatbot node
    # If the output/return value of the router function is "Power BI", it goes to powerbi_chatbot node
    {"Tableau": "tableau_chatbot", "Power BI": "powerbi_chatbot", END: END},
)
graph_builder.add_edge("tableau_chatbot", END)
graph_builder.add_edge("powerbi_chatbot", END)



graph = graph_builder.compile(checkpointer=memory)

print(graph.get_graph().draw_mermaid())

# from IPython.display import Image, display
# from langchain_core.runnables.graph import CurveStyle, MermaidDrawMethod, NodeStyles

# display(
#     Image(
#         graph.get_graph().draw_mermaid_png(
#             draw_method=MermaidDrawMethod.API,
#         )
#     )
# )


# thread is a must while using checkpoint
config = {"configurable": {"thread_id": "1"}}


final_state = graph.invoke(
    {"messages": [{"role": "user", "content": "I have a query related to DAX"}]},
    config
)

print(final_state)


```

* While we have used LLM to classify the text, in production, it would be cheaper to use LLM to generate training data and then train a BERT on top of that. Refer some of the links below: 
    1. https://www.reddit.com/r/MachineLearning/comments/1ayx6xf/p_text_classification_using_llms/
    2. https://github.com/BianchiGiulia/Portfolio/tree/main/Document_Classification
    3. https://www.reddit.com/r/MachineLearning/comments/1eaxghq/r_zero_shot_llm_classification/
    4. 

### Doubts

1. What exactly should the router function in add_conditional_edges return? And how is that return balue mapped to the next node?
2. Once we go to a node, how do we stay at the node, until a given condition? Can we create an edge to same node?
3. What happens if in the tableau_chatbot function we return message directly without wrapping it in a dictionary?

### References

1. https://langchain-ai.github.io/langgraph/tutorials/introduction/#part-2-enhancing-the-chatbot-with-tools
2. https://stackoverflow.com/questions/26000198/what-does-colon-equal-in-python-mean
3. https://langchain-ai.github.io/langgraph/how-tos/visualization/



#### TO DO
1. Answer this question : https://stackoverflow.com/questions/79433194/how-can-i-implement-conditional-edges-in-langgraph-for-agent-decision
2. Check out : https://www.reddit.com/r/LangChain/comments/1e4rcha/graph_with_a_for_loop/?rdt=47877
https://langchain-ai.github.io/langgraph/tutorials/customer-support/customer-support
https://www.reddit.com/r/LangChain/comments/1dhy51m/how_to_add_an_agent_in_a_langgraph_as_a_node/
https://medium.com/@ashpaklmulani/multi-agent-with-langgraph-23c26e9bf076
