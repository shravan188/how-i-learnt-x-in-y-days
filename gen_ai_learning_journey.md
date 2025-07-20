
## Day 1

* System message : This sets the behavior of the assistant and can be used to provide specific instructions for how it should behave throughout the conversation. It is optional

* Free APIs are available from Groq Cloud and Google. Anthropic and OpenAI apis are paid only. Even Huggingface has some limited free API usage (refer 4)

```
# pip install groq 
poetry add groq
poetry add google-genai
poetry add anthropic
poetry add openai

from openai import OpenAI
from groq import Groq
from anthropic import Anthropic
from google import genai
from google.genai import types

import tiktoken

user_message = "Give a 1 day trip plan for Shimla"

### API KEYS
OPENAI_API_KEY = ""
GROQ_API_KEY = ""
GEMINI_API_KEY =""


### Grok API call (Llama-3.3-70b-versatile)
client = Groq(api_key=GROQ_API_KEY)

messages = [
            ##{"role": "system", "content": "You are a helpful assistant"},
            {"role": "user", "content": user_message}
        ]

response = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=messages,
        temperature=0,
    )

content = response.choices[0].message.content
print("##### LLAMA-3.3-70B RESPONSE ############")
print(content)


### Google API Call (Gemini-2.0-Flash)

client = genai.Client(api_key=GEMINI_API_KEY)

response = client.models.generate_content(
    model="gemini-2.0-flash", 
    contents=user_message,
    config=types.GenerateContentConfig(
        max_output_tokens=1000,
        temperature=0
    )
)
print("##### GEMINI-2.0-FLASH RESPONSE ############")
print(response.text)

### OpenAI API call (GPT-3.5-turbo model)

client = OpenAI(api_key=OPENAI_API_KEY)

messages = [
            ##{"role": "system", "content": "You are a helpful assistant"},
            {"role": "user", "content": user_message}
        ]

response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=messages,
        temperature=0,
    )

content = response.choices[0].message.content
print("##### GPT-3.5-TURBO RESPONSE ############")
print(content)

# ### Anthropic API call (claude-3-7-sonnet-20250219)

# client = Anthropic(api_key=ANTHROPIC_API_KEY)

# messages = [
#             ##{"role": "system", "content": "You are a helpful assistant"},
#             {"role": "user", "content": user_message}
#         ]

# response = client.messages.create(
#     model="claude-3-7-sonnet-20250219",
#     messages=messages,
#     temperature=0,
#     max_tokens=1000
# )

# content = response.content

# print("##### CLAUDR-3.7-SONNET RESPONSE ############")
# print(content)



```



### References

1. https://ai.google.dev/gemini-api/docs/text-generation
2. https://console.groq.com/docs/quickstart
3. https://www.reddit.com/r/huggingface/comments/1f5json/any_free_llm_apis/
4. https://huggingface.co/docs/inference-providers/index

## Day 2

* AI agent : Can be conceptually understood as a while loop, where the loop continues until a defined objective is reached. Within this loop, we make Language Model (LLM) calls to process information, make decisions, and take actions

* Built simple multi turn LLM calling without memory
```

from groq import Groq

GROQ_API_KEY = ""
client = Groq(api_key=GROQ_API_KEY)

def get_llm_response(user_message):
    response = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=[
            {"role":"user", "content":user_message}
        ]
    )

    return response.choices[0].message.content

while True:
    user_input = input("Enter you question (enter quit to exit)")
    if user_input.lower() == 'quit':
        break
    else:
        response = get_llm_response(user_input)
        print(response)
        print()


```
* Asked the following questions
    * Describe Shimla in 1 sentence
    * Describe Manali in 1 sentence
    * Which of the two should I visit?
Answer to 3rd question : It seems like we just started our conversation, and I'm not sure what you're referring to. Could you provide more context or information about what you're considering doing together? I'd be happy to help you weigh the pros and cons.

* Requirement : Program should remember previous conversations and respond based on that (that is it should have memory)

* To solve this created a separate list that stores all the conversation history, and passed that to the LLM call (while adding to history also mentioned role as "user" or "assistant", based on whether the user asked the question or it was an LLM response)

```

from groq import Groq


GROQ_API_KEY = ""
client = Groq(api_key=GROQ_API_KEY)

conversation = []

def get_llm_response(user_message):
    global conversation
    conversation.append({"role":"user", "content":user_message})

    response = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=conversation
    )
    
    content = response.choices[0].message.content
    
    conversation.append({"role":"assistant", "content":content})
    
    return content

while True:
    user_input = input("Enter you question (enter quit to exit)")
    if user_input.lower() == 'quit':
        break
    else:
        response = get_llm_response(user_input)
        print(response)
        print(conversation)


```

### References
1. https://community.openai.com/t/how-to-pass-conversation-history-back-to-the-api/697083/14
2. https://github.com/mem0ai/mem0
3. https://community.openai.com/t/multi-turn-conversation-best-practice/282349
4. https://stackoverflow.com/questions/423379/how-can-i-use-a-global-variable-in-a-function
5. https://stackoverflow.com/questions/53528532/syntaxwarning-name-item-is-assigned-to-before-global-declaration-global-item
6. https://community.openai.com/t/multi-turn-conversation-best-practice/282349/5
7. https://github.com/langchain-ai/langchain/blob/master/libs/langchain/langchain/memory/chat_memory.py




## Day 3

* Function calling : An LLM basically generates text/images. Now suppose we want the LLM to perform some action which it cannot do by itself (like calling some api), we teach the model to write a bit of text based on the user input that can be easily parsed out with a program, to perform an action. A secondary bit of code parses that bit of text and performs the action (like calling the api)

* Function calling is a subset of tool calling

* Function calling is mainly useful in 2 scenarios
    * Fetching Data : Retrieve up-to-date information to incorporate into the model's response (RAG). Useful for searching knowledge bases and retrieving specific data from APIs (e.g. current weather data)
    * Taking action : Perform actions like submitting a form, calling APIs



```
import requests
import json
from openai import OpenAI


def get_weather(lat, lon):
    response = requests.get(f"https://api.open-meteo.com/v1/forecast?latitude={lat}&longitude={lon}&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m")
    data = response.json()
    return data['current']['temperature_2m']

OPENAI_API_KEY=""

client = OpenAI(api_key=OPENAI_API_KEY)

tools = [{
    "type":"function",
    "name":"get_weather",
    "description": "Get current temperature for a given location.",
    "parameters": {
        "type":"object",
        "properties":{
            "lat":{
                "type":"number",
                "description": "Latitude"
            },
            "lon":{
                "type":"number",
                "description": "Longitude"
            }

        },
        "required": ["lat", "lon"],
        "additionalProperties": False

    },
    "strict":True
    
}]

response = client.responses.create(
    model="gpt-3.5-turbo",
    input=[{"role": "user", "content": "What is the weather like in Paris today?"}],
    tools=tools
)

print(response.output)
## OUTPUT : [ResponseFunctionToolCall(arguments='{"lat":48.8566,"lon":2.3522}', call_id='call_Yd3b8MgUw5CNWaTwk8oQPG4g', name='get_weather', type='function_call', id='fc_6829f52f6e708191a3e662810a878f4a0a953ff7b5d1d869', status='completed')]

tool_call = response.output[0]
args = json.loads(tool_call.arguments)

result = get_weather(args["lat"], args["lon"])
print(result)
## OUTPUT : 21.0

```

* A function is defined using following fields
    * type (always function)
    * name
    * description
    * parameters

* Try replicating tool calling without using the tool calling abilities provided by OpenAI/Groq


* Errors faced:
    * openai.OpenAIError: The api_key client option must be set either by passing api_key to the client or by setting the OPENAI_API_KEY environment variable

### Doubts
1. What is difference between function calling and tool calling?
2. Whats the difference between Function Calling and just asking the LLM to reply with a specific string that I then do a substring lookup for?
3. What is difference bw `client.responses.create` and `client.chat.completions.create`?
4. How does the model decide when to go for function calling?
5. Is RAG essentially function calling? What are the differences?

### References
1. https://www.reddit.com/r/LocalLLM/comments/1e84286/intuitively_how_does_function_calling_work/
2. https://www.reddit.com/r/LocalLLaMA/comments/1c6jvoq/eli5_what_is_function_calling_in_llms/


## Day 4

* Requirement : Pass the output of the function call back to the LLM, and integrate that to generate a response

* Function calling steps (refer 1)
    1. User provides set of instructions (system prompt) along with list of functions available
    2. LLM processes user prompt and detemines if function call required and if so identifies right function (which it returns as a JSON response)
    3. Application parses the JSON response and invokes the identified function 
    4. After executing the function(s) and retrieving the required data, the output is fed back into the LLM. The LLM then integrates this information into its response,


* The above steps were implemented in the code below
```
import requests
import json
from openai import OpenAI


conversation = []

def get_weather(lat, lon):
    response = requests.get(f"https://api.open-meteo.com/v1/forecast?latitude={lat}&longitude={lon}&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m")
    data = response.json()
    return data['current']['temperature_2m']

OPENAI_API_KEY=""

client = OpenAI(api_key=OPENAI_API_KEY)

tools = [{
    "type":"function",
    "name":"get_weather",
    "description": "Get current temperature for a given location.",
    "parameters": {
        "type":"object",
        "properties":{
            "lat":{
                "type":"number",
                "description": "Latitude"
            },
            "lon":{
                "type":"number",
                "description": "Longitude"
            }

        },
        "required": ["lat", "lon"],
        "additionalProperties": False

    },
    "strict":True
    
}]

conversation.append({"role": "user", "content": "What is the weather like in Paris today?"})

response = client.responses.create(
    model="gpt-3.5-turbo",
    input=conversation,
    tools=tools
)

print(response.output)

tool_call = response.output[0]
args = json.loads(tool_call.arguments)

result = get_weather(args["lat"], args["lon"])
print(result)

## add LLM response and function response to the conversation history
conversation.append(tool_call)
conversation.append({
    "type":"function_call_output",
    "call_id":tool_call.call_id,
    "output":str(result)
})

print(conversation)

## call the llm with the function output (and also all of the previous conversation history)
response_2 = client.responses.create(
    model="gpt-3.5-turbo",
    input=conversation,
    tools=tools   
)

print(response_2.output_text)

```
* Under the hood, functions are injected into the system message in a syntax the model has been trained on. This means functions count against the model's context limit and are billed as input tokens.

### References

1. https://www.k2view.com/blog/llm-function-calling
2. https://platform.openai.com/docs/guides/function-calling?api-mode=responses

## Day 5

* Requirement : Use chat completions API instead of responses API, enhance the way conversation history is stored in function calling code

* Responses API is essentially the fusion of Chat Completions API (simplicity) with Assistants API (tool-use capabilities).

* Chat Completions API vs Responses API : 
    * Chat Completions API is stateless, Responses API is stateful(internally does more things)
    * Chat Completions API is an LLM endpoint, whereas Response API is not just an LLM endpoint — it's an integrated, end-to-end solution for building AI-powered systems (will cause vendor lockin)

* The amazing thing about function call is the LLM is not only able to determine which function to call (tool_call.function.name), but it is also able to determine what values to pass to the function as arguments(tool_call.function.arguments). In the example below, we ask about Paris weather, but the LLM passes the latitude and longitude of Paris as arguments to the function call.


* Note : Tool definition has different format for the 2 apis. Also each of the 2 apis allow only a certain set of roles (you cannot give whatever you like)

```

import requests
import json
from openai import OpenAI


conversation = []

def get_weather(lat, lon):
    response = requests.get(f"https://api.open-meteo.com/v1/forecast?latitude={lat}&longitude={lon}&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m")
    data = response.json()
    return data['current']['temperature_2m']

OPENAI_API_KEY=""

client = OpenAI(api_key=OPENAI_API_KEY)

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current temperature for a given location.",
            "parameters": {
                "type": "object",
                "properties": {
                    "lat": {
                        "type": "number",
                        "description": "Latitude"
                    },
                    "lon": {
                        "type": "number",
                        "description": "Longitude"
                    },
                },
                "required": ["lat", "lon"],
            }
        }
    }
]

conversation.append({"role": "user", "content": "What is the weather like in Paris today?"})

response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=conversation,
    temperature=0.1,
    tools=tools
)

print(response)

# Extract tool call

tool_call = response.choices[0].message.tool_calls[0]
args = json.loads(tool_call.function.arguments)
print(tool_call)

# Call the actual function
result = get_weather(args["lat"], args["lon"])


# Add tool call and function response to the messages


# conversation.append({
#     "role": "assistant",
#     "tool_calls": [tool_call]
# })

conversation.append({
    "role": "assistant",
    "tool_calls": [{
                "id": tool_call.id,
                "type": "function",
                "function": {
                    "name": tool_call.function.name,
                    "arguments": tool_call.function.arguments
                }
    }]
})

conversation.append({
    "role": "tool",
    "tool_call_id": tool_call.id,
    "content": str(result)
})

print("---------Conversation---------------")
print(conversation)

# Final LLM call including function response
response_2 = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=conversation,
    tools=tools   
)

print(response_2.choices[0].message.content)


```


* Errors
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Invalid type for 'input[1].content': expected one of an array of objects or string, but got null instead.", 'type': 'invalid_request_error', 'param': 'input[1].content', 'code': 'invalid_type'}} - Reason: Content is None for result of 1st llm call in conversation history
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Invalid value: 'function_call'. Supported values are: 'assistant', 'system', 'developer', and 'user'.", 'type': 'invalid_request_error', 'param': 'input[1]', 'code': 'invalid_value'}}
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Invalid value: 'tool'. Supported values are: 'code_interpreter_call', 'computer_call', 'computer_call_output', 'file_search_call', 'function_call', 'function_call_output', 'image_generation_call', 'item_reference', 'local_shell_call', 'local_shell_call_output', 'message', 'reasoning', and 'web_search_call'.", 'type': 'invalid_request_error', 'param': 'input[1]', 'code': 'invalid_value'}}
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Missing required parameter: 'input[1].call_id'.", 'type': 'invalid_request_error', 'param': 'input[1].call_id', 'code': 'missing_required_parameter'}}
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Missing required parameter: 'input[1].output'.", 'type': 'invalid_request_error', 'param': 'input[1].output', 'code': 'missing_required_parameter'}}
    * openai.BadRequestError: Error code: 400 - {'error': {'message': 'No tool call found for function call output with call_id call_CrtVWe7hI67Tdv0kslDddoYI.', 'type': 'invalid_request_error', 'param': 'input', 'code': None}}
    * TypeError: Missing required arguments; Expected either ('messages' and 'model') or ('messages', 'model' and 'stream') arguments to be given
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Missing required parameter: 'tools[0].function'.", 'type': 'invalid_request_error', 'param': 'tools[0].function', 'code': 'missing_required_parameter'}}
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Invalid value: 'tooool'. Supported values are: 'system', 'assistant', 'user', 'function', 'tool', and 'developer'.", 'type': 'invalid_request_error', 'param': 'messages[2].role', 'code': 'invalid_value'}}


### Doubts
1. What is the importance of the diffrent roles (assistant, system, developer, user) in OpenAI? How does the LLM treat these different roles?
2. What is the meaning of prefering chat over completions while tuning a LLM model (https://www.reddit.com/r/LocalLLaMA/comments/1awju86/what_are_actually_chat_and_completion/)? Also what is difference between different types of tuning like reflection tuning, instruction tuning etc?

### References
1. https://community.openai.com/t/content-is-required-property-error-400/486260/13
2. https://github.com/Sakshee5/Duke-Student-Advisor-Chatbot/blob/master/utils/function_calling.py
3. https://platform.openai.com/docs/guides/responses-vs-chat-completions
4. https://www.reddit.com/r/LocalLLaMA/comments/1jdytwm/thoughts_on_openais_new_responses_api
5. https://www.reddit.com/r/LangChain/comments/1j9hs58/everything_you_need_to_know_about_the_recent/


## Day 6

* Requirement: To create a simple LLM powered chat application using Streamlit which answers any question related to NITK ECE department


* Wrote the following code:
```
import os
import streamlit as st
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

try:
    openai_api_key = os.getenv("OPENAI_API_KEY")
except:
    openai_api_key = None


# Title and header
st.title("NITK ECE Advisor Bot")

## Sidebar
with st.sidebar:
    st.markdown("""
                Ask me about NITK ECE department. I can answer questions related to
                
                * Curriculum 
                * Events   
                * News
                
                """)
    st_openai_api_key = st.text_input("OpenAI API Key", type="password", help="Enter your OpenAI API key")

    if st.button("Clear chat"):
        st.session_state.messages = []
        st.success("Chat History cleared")

system_prompt = """
You are a helpful assistant who is an expert on everything related to NITK Electronics and Communication (ECE) department. You answer questions related to NITKs ECE program, curriculum, events and latest news

Guidelines:
1. You are **not** allowed to answer questions that are not directly related to NITK. This is very important. If the user query is **not relaated to NITK**, respond by saying its out of scope and that you're here to help with NITK-related questions
2. **If a user query is vague or underspecified**, do not assume. Instead, ask follow-up questions.

Be concise when appropriate, but offer long, elaborate answers when more detail would be helpful.
"""

# Initialize session state variables
if "messages" not in st.session_state:
    st.session_state.messages = [{"role":"system", "content":system_prompt}]

if openai_api_key or st_openai_api_key:
    st.session_state.api_key = openai_api_key or st_openai_api_key
    st.session_state.client = OpenAI(api_key=st.session_state.api_key)

else:
    st.warning("Please enter your OpenAI API key here or add it to the .env file to continue.")


# Display chat history (excluding system message)
# Without this only the latest user question will be shown and previous conversation will be erased
for msg in st.session_state.messages:
    if msg["role"] == "system":
        continue  # Don't show system message
    with st.chat_message(msg["role"]):
        st.markdown(msg["content"])

# Handle user input
if st.session_state.client:
    if user_input := st.chat_input("Enter your message here"):
        st.session_state.messages.append({"role": "user", "content": user_input})

        with st.chat_message("user"):
            st.markdown(user_input)


        with st.chat_message("assistant"):
            status_container = st.empty()
            response_container = st.empty()
            status_container.info("Initializing....")    

            response = st.session_state.client.chat.completions.create(
                temperature=0.1,
                messages=st.session_state.messages,
                model="gpt-3.5-turbo"
            )

            if response:
                status_container.empty()
                with response_container.container():
                    st.markdown(response.choices[0].message.content)
                
                st.session_state.messages.append({
                    "role":"assistant",
                    "content": response.choices[0].message.content
                })
                print(st.session_state.messages)
                 
        

```

* Ran the app using `streamlit run app.py`

* Commited code to Github using following commands
```
git init
git add .
git commit -m "add all files"  
git remote add origin https://github.com/shravankshenoy/ECE_Advisor_Bot.git
git pull origin main --allow-unrelated-histories 
git push origin master:main 
```



### References
1. https://docs.streamlit.io/develop/tutorials/chat-and-llm-apps/build-conversational-apps



## Day 7

* To convert folder to module you have to put `__init__.py`. Even then you have to run the code from the main folder

* The file `__init__.py` was required under Python 2.X but it is no longer required from Python 3.3 onwards, since Python uses namespace package instead of regular package

* Python Module : A Python file

* Python Package : Set of .py files (modules) organized in subdirectories (subpackages).

* Consider the folder structure below. Suppose in `curriculum_tool.py` I want to use the following import `from utils.openai_client import get_openai_client`. One approach to achive this is to import the curriculum_tool file into the main.py and use it there. But to run the curriculum_tool.py independently with that import, follow the steps in ref 3 (make your package equivalalent to any other package using pip install .)

```
(printed using tree /f /a)
 
|   main.py
|
+---tools
|   |   curriculum_tool.py
|   |   tools_schema.py
|   |   __init__.py
|   |
|
\---utils
    |   openai_client.py
    |   __init__.py

```


* During the import process, when Python finds the source of a module or package it is asked to import, it will read and execute all the `__init__.py` files of the package and subpackages of the module.

* Irrespective of whether we are reading a text file or pdf, ultimately we have to extract the text into a variable and pass it to the LLM. To extract text from pdf, we can use library like pypdf.

* Built a simple tool which can select relevant subjects based on user query, using the code below
```
import json
from openai import OpenAI

def load_courses(filename):
    with open(filename, "r") as file:
        courses = file.read()
    
    return courses


def get_courses(user_prompt):
    print("hello")
    # Create system prompt which takes user query, entire list of courses from text file and suggests the most relevant courses
    client = OpenAI(api_key='sk-proj')
    courses = load_courses(r"ece_curriculum_nitk.txt")

    system_prompt = f"""Your are a helpful assistant who selects the right courses based on requirements specified by the user. Given the user query select the matching courses from the list of available courses given below
                    
    Courses : {courses}
    
    Respond in json format as follows: 
    {{
        "matched_courses" : [<list of strings>] 
    }}  
    """

    messages = [
        {"role":"system", "content":system_prompt},
        {"role":"user", "content":user_prompt}
    ]
    
    response = client.chat.completions.create(
        messages=messages,
        temperature=0.1,
        model="gpt-3.5-turbo"
    )

    output = response.choices[0].message.content
    output = json.loads(output)
    print(output)
    return output['matched_courses']

if __name__ == '__main__':
    get_courses("Could you share courses related to VLSI?")


```
* In f-string, the double curly bracess are for when you want literal curly braces in the string, and single curly braces are for replacing with variables. So, this will work:

* f-string vs jinja : In a f-string, expressions are enclosed in single curly braces `{ }`, whereas in Jinja expressions are enclosed in double curly braces i.e. `{{ }}` . Double curly braces are used in f-string when we want literal curly braces in the string. 

* Since I wanted a literal curly brace in my prompt, I used double curly braces

* To convert text in json format to json, we use json.loads()

### Doubts
1. Can we pass a text file directly to a prompt?
2. What exactly is a namespace package?
3. How exactly is `__path__` useful?
4. How to indicate json output in prompt, because when we use curly braces, the f-string considers it as an expression to be evaluated?

### References
1. https://www.reddit.com/r/LocalLLaMA/comments/13zo3d8/what_prompts_can_be_used_for_text_classification/
2. https://www.reddit.com/r/LocalLLaMA/comments/1905s0f/how_can_i_provide_an_llm_a_text_file_to_read/
3. https://stackoverflow.com/questions/714063/importing-modules-from-parent-folder/50194143#50194143
4. https://dev.to/bastantoine/discovering-python-namespace-packages-4gi3
5. https://www.reddit.com/r/learnpython/comments/godq97/fstrings_with_double_curly_braces/


### Day 8 and 9

* When an imported module in Python uses a data file, the search for that file follows a specific order, similar to how Python searches for modules themselves. Here's the breakdown:
    1. Current Working Directory : Python first looks for the data file in the directory where the script that initiated the import is being executed.
    2. Module's Directory:If the data file is not found in the current working directory, Python will then search in the same directory where the imported module's .py file is located.To access this location programmatically within the module, you can use os.path.dirname(__file__).
    3. Directories in sys.path
    4. Relative Paths: If a relative path is used to specify the data file within the module, the path will be resolved relative to the module's directory, not the current working directory.

* When the code `open('data/ece_curriculum_nitk.txt', "r")` within curriculum_tool.py is imported in main.py, Python searches for the text file w.r.t main.py directory i.e. current working directory

* When the LLM decides to do a function call it does not actually call the functions, instead it just returns the name of the function. We have to then parse it and then call the function ourselves. Therefore in the app.py, **there is separate code to handle the scenario when LLM decides that tool call is required (vs when LLM decides to directly return an output)**

* The amazing thing about function call is the LLM is not only able to determine which function to call (tool_call.function.name), but it is also able to determine what values to pass to the function as arguments(tool_call.function.arguments)

* Made a mistake of by defining type of "user_prompt" as number instead of string in the function definition, leading to incorrect result


* Combined the curriculum tool with rest of the code. Entire code given below
```
#### app.py
import os
import json
import streamlit as st
from dotenv import load_dotenv
from openai import OpenAI
from tools.curriculum_tool import get_courses

load_dotenv()

try:
    openai_api_key = os.getenv("OPENAI_API_KEY")
except:
    openai_api_key = None


# Title and header
st.title("NITK ECE Advisor Bot")

## Sidebar
with st.sidebar:
    st.markdown("""
                Ask me about NITK ECE department. I can answer questions related to
                
                * Curriculum 
                * Events   
                * News
                
                """)
    st_openai_api_key = st.text_input("OpenAI API Key", type="password", help="Enter your OpenAI API key")

    if st.button("Clear chat"):
        st.session_state.messages = []
        st.success("Chat History cleared")

system_prompt = """
You are a helpful assistant who is an expert on everything related to NITK Electronics and Communication (ECE) department. You answer questions related to NITKs ECE program, curriculum, events and latest news

Guidelines:
1. You are **not** allowed to answer questions that are not directly related to NITK. This is very important. If the user query is **not relaated to NITK**, respond by saying its out of scope and that you're here to help with NITK-related questions
2. **If a user query is vague or underspecified**, do not assume. Instead, ask follow-up questions.
3. For any questions related to ECE courses or curriculum, use the curriculum_tool in your planning

Be concise when appropriate, but offer long, elaborate answers when more detail would be helpful.
"""

# Initialize session state variables
if "messages" not in st.session_state:
    st.session_state.messages = [{"role":"system", "content":system_prompt}]

if openai_api_key or st_openai_api_key:
    st.session_state.api_key = openai_api_key or st_openai_api_key
    st.session_state.client = OpenAI(api_key=st.session_state.api_key)

else:
    st.warning("Please enter your OpenAI API key here or add it to the .env file to continue.")


# Display chat history (excluding system message)
for msg in st.session_state.messages:
    if msg["role"] == "system":
        continue  # Don't show system message
    with st.chat_message(msg["role"]):
        st.markdown(msg["content"])


tools = [
    {
        "type":"function",
        "function": {
            "name":"get_courses",
            "description": "Get courses relevant to the user query",
            "parameters":{
                "type": "object",
                "properties": {
                    "user_prompt":{
                        "type": "string",
                        "description": "User query"
                    }
                },
                "required":["user_prompt"]
            }

        }
    }
]

# Handle user input
if st.session_state.client:
    if user_input := st.chat_input("Enter your message here"):
        st.session_state.messages.append({"role": "user", "content": user_input})

        with st.chat_message("user"):
            st.markdown(user_input)


        with st.chat_message("assistant"):
            status_container = st.empty()
            response_container = st.empty()
            status_container.info("Initializing....")    

            response = st.session_state.client.chat.completions.create(
                temperature=0.1,
                messages=st.session_state.messages,
                model="gpt-3.5-turbo",
                tools=tools
            )

            response = response.choices[0].message

            if response.content:
                status_container.empty()
                with response_container.container():
                    st.markdown(response.content)
                
                st.session_state.messages.append({
                    "role":"assistant",
                    "content": response.content
                })
                print(st.session_state.messages)
            
            if response.tool_calls:
                tool_call = response.tool_calls[0]
                function_name = tool_call.function.name
                function_args = json.loads(tool_call.function.arguments)

                print(function_name, function_args)
                if function_name == 'get_courses':
                    function_response = get_courses(**function_args)
                    print(function_response)

                st.session_state.messages.append({
                    "role":"assistant",
                    "content": None,
                    "tool_calls": [{
                        "id": tool_call.id,
                        "type": "function",
                        "function": {
                            "name": function_name,
                            "arguments": tool_call.function.arguments
                        }
                     }]
                })

                st.session_state.messages.append({
                    "role":"tool",
                    "tool_call_id": tool_call.id,
                    "content":str(function_response)
                }                
                )
                print(st.session_state.messages)
                response = st.session_state.client.chat.completions.create(
                temperature=0.1,
                messages=st.session_state.messages,
                model="gpt-3.5-turbo",
                )

                response = response.choices[0].message
                status_container.empty()
                with response_container.container():
                    st.markdown(response.content)

                st.session_state.messages.append(
                    {
                        "role": "assistant",
                        "content": response.content
                    }
                )

### Each subfolder has a __init__.py          
### tools/curriculum_tool.py
import json
from utils.openai_client import get_openai_client

def convert_txt_to_json():
    pass

def load_courses(filename: str):
    with open(filename, "r") as file:
        courses = file.read()
    
    return courses


def get_courses(user_prompt):
    
    client = get_openai_client()
    courses = load_courses(r"data/ece_curriculum_nitk.txt")

    system_prompt = f"""Your are a helpful assistant who selects the right courses based on requirements specified by the user. Given the user query select the matching courses from the list of available courses given below
                    
    Courses : {courses}
    
    Respond in json format as follows: 
    {{
        "matched_courses" : [<list of strings>] 
    }}  
    """

    messages = [
        {"role":"system", "content":system_prompt},
        {"role":"user", "content":user_prompt}
    ]
    
    response = client.chat.completions.create(
        messages=messages,
        temperature=0.1,
        model="gpt-3.5-turbo"
    )

    output = response.choices[0].message.content
    output = json.loads(output)

    return output['matched_courses']

if __name__ == '__main__':
    get_courses("Could you share courses related to VLSI?")


### utils/openai_client.py

import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

def get_openai_client():
    # Note: This function will not work if user enters API_KEY in streamlit app
    print(os.getenv("OPENAI_API_KEY"))
    api_key = os.getenv("OPENAI_API_KEY")
    client = OpenAI(api_key=api_key)
    return client

### pyproject.toml
[tool.poetry]
package-mode = false

[tool.poetry.dependencies]
python = ">=3.10,<4.0"
openai = "^1.82.0"
streamlit = "^1.45.1"
python-dotenv = "^1.1.0"

[tool.poetry.group.dev.dependencies]

[tool.poetry.group.test.dependencies]

[tool.poetry.extras]

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"

#### streamlit run app.py

```

* Error:
    * openai.BadRequestError: Error code: 400 - {'error': {'message': "Invalid type for 'messages[2].tool_calls[0].function.arguments': expected a string, but got an object instead.", 'type': 'invalid_request_error', 'param': 'messages[2].tool_calls[0].function.arguments', 'code': 'invalid_type'}}. Solution : Remove json.loads in the arguments
```
 st.session_state.messages.append({
                    "role":"assistant",
                    "content": None,
                    "tool_calls": [{
                        "id": tool_call.id,
                        "type": "function",
                        "function": {
                            "name": function_name,
                            "arguments": json.loads(tool_call.function.arguments)
                        }
                     }]
                })


```

### Doubts
1. If imported module use a data file in python, where does python search for data file?
2. How to handle tool calls, since the reponse of tool call is not directly output but a tool call we have to process?

## Day 10

* Built a simple summarization app using Streamlit

```
import streamlit as st
from openai import OpenAI
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("OPENAI_API_KEY")

client = OpenAI(api_key=api_key)

col1, col2 = st.columns(2)

col1.subheader("Input")
st_text = col1.text_area(
    label="Enter text",
    height=400,
    key="text_input"
)




system_prompt = f"""
You are an expert summarizer who has been given the following text

Text : {st_text}

Please create a concise summary. The summary should not exceed 30 words

"""

messages = [
    {"role":"system", "content":system_prompt},
    {"role":"user","content":st_text}
]

response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    temperature=0.1,
    messages=messages
)

output = response.choices[0].message.content

print(output)

col2.subheader("Output")
col2.text_area(
    label="Summarized version",
    value=output,
    height=400
)


```
* Used the following variations of system prompts based on 1 and 4

```

system_prompt_1 = f"""
As a professional summarizer, create a concise and comprehensive summary of the provided text, be it an article, post, conversation, or passage, while adhering to these guidelines:

1. Craft a summary that is detailed, thorough, in-depth, and complex, while maintaining clarity and conciseness.
2. Incorporate main ideas and essential information, eliminating extraneous language and focusing on critical aspects.
3. Rely strictly on the provided text, without including external information.
4. Format the summary in paragraph form for easy understanding. 

Text : {st_text}
"""

system_prompt_2 = f"""

Could you please provide a concise and comprehensive summary of the given text? The summary should capture the main points and key details of the text while conveying the author's intended meaning accurately. Please ensure that the summary is well-organized and easy to read, with clear headings and subheadings to guide the reader through each section. The length of the summary should be appropriate to capture the main points and key details of the text, without including unnecessary information or becoming overly long.

Text : {st_text}


"""


system_prompt_3 = f"""

Can you provide a comprehensive summary of the given text? The summary should cover all the key points and main ideas presented in the original text, while also condensing the information into a concise and easy-to-understand format. Please ensure that the summary includes relevant details and examples that support the main ideas, while avoiding any unnecessary information or repetition. The length of the summary should be appropriate for the length and complexity of the original text, providing a clear and accurate overview without omitting any important information.

Text : {st_text}


"""

system_prompt_4 = f"""
Could you please provide a summary of the given text, including all key points and supporting details? The summary should be comprehensive and accurately reflect the main message and arguments presented in the original text, while also being concise and easy to understand. To ensure accuracy, please read the text carefully and pay attention to any nuances or complexities in the language. Additionally, the summary should avoid any personal biases or interpretations and remain objective and factual throughout.


Text : {st_text}

"""

system_prompt_5 = f"""

Provide a comprehensive summary of the story so far. The summary should cover all the key points and main ideas presented in the original text, while also condensing the information into maximum concise form. Please ensure that the summary includes relevant details and examples that support the main ideas, while avoiding any unnecessary information or repetition. Make length less than 50 words, providing a clear and accurate overview without omitting any important information. While condensing absolutely as much as possible. Use the following rules. Remove articles (a, an, the) when they're not essential to the meaning of the sentence. Eliminate unnecessary prepositions (e.g., "in," "on," "at") when the context is clear without them. Avoid redundant or repetitive words and phrases that do not add new information. Use contractions (e.g., "don't" instead of "do not") to reduce word count. Simplify verb tenses when possible (e.g., use simple past instead of past perfect). Replace longer phrases with shorter synonyms or more concise expressions. Omit unnecessary modifiers, such as adverbs and adjectives, that do not significantly contribute to the meaning. Avoid conjunctions (e.g., "and," "but," "or") when the relationship between ideas is implied. Eliminate filler words and phrases (e.g., "basically," "in fact," "as a matter of fact") that do not add value to the sentence. Use sentence fragments, concise sentence structures, single words where meaning is inferable.

Text : {st_text}


"""

system_prompt_6 = f"""

Task: Summarize the following consumer complaint in 2-3 sentences.

Instructions:
1. Identify the main issue or dispute raised by the sender.
2. Note any legal references (e.g., Fair Credit Reporting Act).
3. Briefly list the specific accounts or items being challenged.
4. Mention the actions requested (e.g., investigation, correction, report update).
5. Use a neutral and professional tone. Do not include personally identifiable information (e.g., name, account numbers, addresses).

Consumer complaint : {st_text}


"""
```

* The idea is the model needs clear guidelines about what is expected out of a "summary" (although it seems like the guidelines are redundant). Hence it's best to define exactly what you want, by giving it a 
    * a role  (As a <ROLE> or You are an expert <ROLE>)
    * a task (if the task is complex, split it into subtasks i.e. instruction list)
    * a format
    * along with a purpose.
 Specifying how exactly you want the input to be generated, and why, cuts down significantly on the overall clutter, and generally increases the efficiency of its summaries.

* When you ask a model to summarize within a certain number of words, there is a chance it will cut out / skip over essential information or even include statements not in the original text

* Another trick to develop good prompts is : You can collaborate with ChatGPT to develop prompts.

* Another approach is MAp-Reduce approach : Divide entire content into chunks, summarize each chunk separately and then make a summary of all the individual chunk summaries



## Doubts
1. What are some ways to write effective prompts?


### References
1. https://www.reddit.com/r/ChatGPTPro/comments/13n55w7/highly_efficient_prompt_for_summarizing_gpt4/
2. https://www.promptingguide.ai/introduction/tips
3. https://github.com/microsoft/presidio/tree/main/docs/samples/python/streamlit
4. https://www.reddit.com/r/ChatGPT/comments/11twe7z/prompt_to_summarize/
5. https://www.reddit.com/r/SillyTavernAI/comments/1budovw/good_brief_accurate_summarization_prompt_for/
6. https://www.reddit.com/r/ChatGPTPro/comments/14c9vx2/what_is_the_best_way_to_get_start_writing_prompts/
7. https://www.reddit.com/r/ChatGPTPromptGenius/comments/1kl332l/how_do_you_find_the_best_prompts/
8. https://github.com/NirDiamant/Prompt_Engineering


## Day 11

* RAG : A framework/technique LLM uses to get relevant, up-to-date, and context-specific information by combining retrieval and generation capabilities.

* The core idea behind RAG is
    1. Given a bunch of text, an LLM can answer questions based on that text (generation)
    2. When we have a huge amount of text, it is more efficient to divide it into smaller chunks and feed only the relevant chunks to the LLM (retrieval)

* RAG will be relevant even if models can reason over super long context lengths. RAG is necessary because models can’t be trained on new information as quickly as the information can be indexed and subsequently retrieved. Furthermore, a model can't "know" real time data (weather, traffic, stocks, whatever) without retrieving it. 

* Another reason RAG will be relevant is using long context is very expensive

* We can view tool use as a form of RAG or vice versa i.e. RAG is essentially a type of tool/function call

* Stages in traditional RAG:
    * Query expansion 
    * Retrieval
    * Reranking (using tools like cross encoder, colbert)
    * Context enrichment (i.e. augmentation)
    * Generation

* Query expansion increases the number of results, so it increases recall (vs precision). In general, BM25 favors precision while embedding retrieval favors recall

* Recall : In NLP tasks, recall is used to measure how many of the positive instances were correctly identified by the model

* Precision : In NLP tasks, precision is used to measure how many of the instances identified as positive are actually positive.

* PREcision is to PREgnancy tests as reCALL is to CALL center (refer 3)

* Traditional RAG vs Agentic RAG : In traditional RAG, we retrieve the relevant documents once and feed it as context to the llm. In Agentic rag, the agent can do the retrieval process iteratively as many times as required until the relevant documents are retrieved, and then feed that as context (iterative context refinement) 

### Doubts
1. Why does BM25 favour precision, and embedding retrieval favour recall?
2. Are precision and recall always inversely proportional?
3. How exactly does an Agentic RAG improve its retrieval in successive iterations?

### References
1. https://www.reddit.com/r/LocalLLaMA/comments/1eg8v5b/rag_tools_and_agents_will_all_soon_be_irrelevant/
2. https://jaimin-ml2001.medium.com/evaluation-methods-in-natural-language-processing-nlp-part-1-ffd39c90c04f
3. https://datascience.stackexchange.com/questions/30881/when-is-precision-more-important-over-recall
4. https://haystack.deepset.ai/blog/query-expansion
5. https://langchain-ai.github.io/langgraph/tutorials/rag/langgraph_agentic_rag/
6. https://www.moveworks.com/us/en/resources/blog/agentic-rag
7. https://github.com/langchain-ai/langchain-community/tree/main/libs/community/langchain_community
8. https://langchain-ai.github.io/langgraph/tutorials/rag/langgraph_agentic_rag

## Day 12

* Goal : To create a pipeline which parses pdf, extracts relevant content as json, and put all of that json into excel

* Wrote code to 
    * Convert pdf to markdown using docling
    * Extract name and skills from markdown using Llama-3.3-70b (groq inferencing)
    * Convert the json to structured format using Pandas
```
# pip install docling groq

import json
import pandas as pd
from groq import Groq
from docling.document_converter import DocumentConverter

source = "/content/London-Resume-Template-Professional.pdf"

# Extract content from resume 
converter = DocumentConverter()
doc = converter.convert(source).document
print(doc.export_to_markdown())
resume = doc.export_to_markdown()

# Pass markdown to LLM and get relevant details as json

client = Groq(api_key="")
user_prompt = f"""
You are an expert in parsing relevant details from resumes. Given a resume, you extract the name and skills of the person 

Resume : {resume}

Respond in the json format shown below
{{
 "name" :  <string>
 "skills" : [<list of string>]
}}
"""

messages = [
{"role":"user", "content": user_prompt}
]


response = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=messages,
        temperature=0,
)


content = response.choices[0].message.content
json_content = content[content.find('{'):content.find('}') + 1]

person_details = json.loads(json_content)

person_details_df = pd.DataFrame.from_dict(person_details)

```


* Was getting error while parsing json because I had not parsed the closing bracket of json, added +1 so that it is resolved

### Doubts
1. What GPU (for PC) or config would be the minimum to run something like qwen 2.5 32B smoothly?

### References
1. https://www.reddit.com/r/LocalLLaMA/comments/1gp4g8a/hardware_requirements_to_run_qwen_25_32b/
2. https://www.resumeviking.com/templates/
3. https://www.youtube.com/watch?v=B5XD-qpL0FU (Venelin Valkov)
4. https://www.reddit.com/r/LocalLLaMA/comments/1dhyxq8/why_use_ollama/
5. https://www.reddit.com/r/ollama/comments/1giu0kz/post_your_model_what_do_you_have_and_use/
6. https://www.reddit.com/r/LLMDevs/comments/1gvymbp/what_is_the_best_open_source_llm_for_analyzing/
7. https://colab.research.google.com/#fileId=https%3A//huggingface.co/Qwen/Qwen2.5-VL-32B-Instruct.ipynb
8. https://www.reddit.com/r/LocalLLaMA/comments/1hd3mfa/best_vlm_in_the_market/

## Day 13

* Goal : To build a RAG pipeline for FAQ

* Built a RAG pipeline with following stages
    * Ingestion/Preprocessing
        * Chunked the text file using langchain RecursiveCharacterTextSplitter
        * Add chunks to ChromaDB collection with a unique UUID for each chunk
    * Retreival
        * Find chunks relevant to the user query 
        * Relevant chunks are joined together and passed as context to the LLM, which uses it to answer the user's query
   

* Chroma converts the text into the embeddings using all-MiniLM-L6-v2, a sentence transformer model

* Chroma db uses cosine similarity as the similarity metric

```
! pip install langchain chromadb chromadbx groq
import chromadb

from chromadbx import UUIDGenerator
from groq import Groq
from langchain.text_splitter import RecursiveCharacterTextSplitter 


with open("/content/data_engineer_zoomcamp_faq.txt","r") as file:
  doc = file.read()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=300, # Size of each chunk in characters
    chunk_overlap=100, # Overlap between consecutive chunks
    length_function=len, # Function to compute the length of the text
    add_start_index=True, # Flag to add start index to each chunk
  )

chunks = text_splitter.split_text(doc)

client = chromadb.PersistentClient()
collection = client.get_or_create_collection(name="test")

collection.add(documents=chunks, ids=UUIDGenerator(len(chunks)))

query = "Which Python version can we use?"

results = collection.query(
    query_texts=[query],
    n_results=2
).get('documents')[0]

content_text = "\n\n".join(results)

client = Groq(api_key="")

user_prompt = f"""
Answer the question using following context:
{content_text}
 - -
Answer the question based on the above context: {question}
"""

messages = [
{"role":"user", "content": user_prompt}
]

response = client.chat.completions.create(
        model="llama-3.3-70b-versatile",
        messages=messages,
        temperature=0,
)

print(response.choices[0].message.content)
```

* Used split_text instead of split_document

### Doubts

### References
1. https://stackoverflow.com/questions/78486966/langchaing-text-splitter-docs-saving-issue
2. https://research.trychroma.com/evaluating-chunking
3. https://cookbook.chromadb.dev/core/document-ids/
4. https://docs.trychroma.com/docs/overview/getting-started
5. https://docs.google.com/document/d/19bnYs80DwuUimHM65UV3sylsCn2j1vziPOwzBwQrebw/edit?tab=t.0
6. https://medium.com/@callumjmac/implementing-rag-in-langchain-with-chroma-a-step-by-step-guide-16fc21815339
7. https://www.datacamp.com/tutorial/chromadb-tutorial-step-by-step-guide



## Day 14 and 15

* Types of chunking strategies
    * Fixed length chunking
    * Recursive chunking
    * Document based chunking
    * Semantic chunking
    * Agentic chunking (using LLM to suggest chunks)


* Goal : To build a full RAG pipeline of a pdf document containing images

* Docling Force Full Page OCR : When you set `do_ocr=True`, OCR is not triggered unless there is an actual bitmap resource element. BY using Force Full Page OCR, we force a full page OCR even if not actual bitmap resource element is found. In a way, this is similar to converting the entire page into an image and doing an OCR on that (refer source code below)

```
# Soure code from https://github.com/docling-project/docling/pull/290/files#diff-90f1f740c73d9ee1f1b821ff4937a7016fa32b30faa91fedf034e6f3663b9c62
# We set bounding box on entire page

 if self.options.force_full_page_ocr or coverage > max(
            BITMAP_COVERAGE_TRESHOLD, self.options.bitmap_area_threshold
        ):
            return [
                BoundingBox(
                    l=0,
                    t=0,
                    r=page.size.width,
                    b=page.size.height,
                    coord_origin=CoordOrigin.TOPLEFT,
                )
            ]


```


* Docling PdfPipeline Images Scale : For operating with page images, we must keep them, otherwise the DocumentConverter will destroy them for cleaning up memory. This is done by setting PdfPipelineOptions.images_scale, which also defines the scale of images. scale=1 correspond to a standard 72 DPI image

* DPI (dots per inch) in image refers to the resolution of an image when it is printed. A higher DPI means more dots of ink per inch, resulting in a sharper and more detailed print. 72 DPI has been a standard screen resolution used in early computer monitors. For print-quality images, a resolution of 300 DPI or higher is considered high resolution.

* Data URI (Uniform Resource Identifier) : a scheme that allows data to be encoded into a string (and then embedded in html and css)
Syntax of data URIs is : data:content/type;base64, For a image stored in URI it would `data:image/png;base64,iVBORw0KGgoAAAANSUh....`. Basically the image pixels is stored in the form of base64, and it starts after the comma in the URI scheme
 

* Tried extracting text from pdf using docling
    * Approach 1: Use default pipeline
    * Approach 2: Enable ocr (no improvement - text on LHS not covered)
    * Approach 3: Enable ocr with force full page ocr using Tesseract ocr (significant improvment)


```
### Approach 1
from docling.document_converter import DocumentConverter
source = "/content/Fact-Sheet-X-Rays-June-2022.pdf"
converter = DocumentConverter()
doc = converter.convert(source).document
print(doc.export_to_markdown())

### Approach 2
from docling.datamodel.pipeline_options import PdfPipelineOptions, TesseractCliOcrOptions
from docling.datamodel.base_models import InputFormat
from docling.document_converter import DocumentConverter, PdfFormatOption

source = "/content/Fact-Sheet-X-Rays-June-2022.pdf"

pipeline_options = PdfPipelineOptions()
pipeline_options.do_ocr = True

doc_converter = DocumentConverter(format_options = {
  InputFormat.PDF : PdfFormatOption(pipeline_options=pipeline_options)   
}
)

doc = doc_converter.convert(source).document
print(doc.export_to_markdown())



### Approach 3
from docling.datamodel.pipeline_options import PdfPipelineOptions, TesseractCliOcrOptions
from docling.datamodel.base_models import InputFormat
from docling.document_converter import DocumentConverter, PdfFormatOption

source = "/content/Fact-Sheet-X-Rays-June-2022.pdf"

ocr_options = TesseractCliOcrOptions(force_full_page_ocr=True)

pipeline_options = PdfPipelineOptions()
pipeline_options.do_ocr = True
pipeline_options.ocr_options = ocr_options

doc_converter = DocumentConverter(format_options = {
  InputFormat.PDF : PdfFormatOption(pipeline_options=pipeline_options)   
}
)

doc = doc_converter.convert(source).document


```

* Images are typically binary files, but LLMs often require text-based input. Base64 encoding converts the binary image data into a string of ASCII characters. Hence we need to first encode our image to a base64 format string before passing it to model

* llama-3.3-70b-versatile is a text only model, cannot use it to summarize messages

* llama-4-scout-17b-16e-instruct can do both image understanding/visual reasoning and also assistant like chat

* Wrote code to extract text and images from pdf and summarize

```

from docling.datamodel.pipeline_options import PdfPipelineOptions, TesseractCliOcrOptions
from docling.datamodel.base_models import InputFormat
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling_core.types.doc import PictureItem
from pathlib import Path
IMAGE_RESOLUTION_SCALE = 2.0

source = "/content/Fact-Sheet-X-Rays-June-2022.pdf"

pipeline_options = PdfPipelineOptions()
pipeline_options.images_scale = IMAGE_RESOLUTION_SCALE
pipeline_options.generate_page_images = True
pipeline_options.generate_picture_images = True
pipeline_options.do_ocr = True
ocr_options = TesseractCliOcrOptions(force_full_page_ocr=True)
pipeline_options.ocr_options = ocr_options

doc_converter = DocumentConverter(format_options = {
  InputFormat.PDF : PdfFormatOption(pipeline_options=pipeline_options)
}
)

# Convert document
doc = doc_converter.convert(source)

doc_filename = doc.input.file.stem


for page_no, page in doc.document.pages.items():
  page_image_filename = Path(f"{doc_filename}-{page_no}.png")
  with page_image_filename.open("wb") as fp:
    page.image.pil_image.save(fp, format="PNG")

picture_counter = 0
for element, _level in doc.document.iterate_items():
  if isinstance(element, PictureItem):
    picture_path = Path(f"{doc_filename}-picture-{picture_counter}.png")
    with picture_path.open("wb") as fp:
      element.get_image(doc.document).save(fp, "PNG")

    picture_counter += 1

images = []
for picture in doc.document.pictures:
  ref = picture.get_ref().cref
  image = picture.image
  if image:
    images.append(str(image.uri))

print(images[0])

from groq import Groq

client = Groq(api_key="")


image_summarization_prompt = """
Describe the image in concisely in 1 to 2 sentences 
"""

image_summaries = []

image = images[0]

# messages = [
#     {"role":"user", "content":image},
#     {"role":"system", "content":image_summarization_prompt},
# ]

messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "Describe the image concisely in 1 to 2 sentences"
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": image
                    }
                }
            ]
        }
    ]

response = client.chat.completions.create(
    model="meta-llama/llama-4-scout-17b-16e-instruct",
    messages=messages,
    temperature=0.5
)

summary = response.choices[0].message.content
print(summary)

```


### Doubts
1. How to ask llm to summarize base64 images?
2. What does instruction tuning mean?
3. What does having an list of user prompts in the llm call mean?


### References
1. https://medium.com/@anuragmishra_27746/five-levels-of-chunking-strategies-in-rag-notes-from-gregs-video-7b735895694d
2. https://github.com/docling-project/docling/issues/185
3. https://www.nibib.nih.gov/sites/default/files/2022-09/Fact-Sheet-X-Rays-June-2022.pdf
4. https://stackoverflow.com/questions/19696418/what-does-it-means-dataimage-png-in-the-source-of-an-image
5. https://base64.guru/converter/decode/image
6. https://medium.com/@martin.crabtree/the-what-and-when-of-a-data-uri-599fe72f90d8
7. https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct
8. https://medium.com/@pritigupta.ds/docling-powered-rag-querying-over-complex-pdfs-d99f5f58bc33
9. https://console.groq.com/docs/vision

## To explore
1. Different types of RAG - https://www.linkedin.com/posts/pavan-belagatti_the-essential-rag-approaches-every-aiml-activity-7318683180134199296-fb_0

## Day 16

* "in" operator in Python cares only about presence, not the number of occurrences.
* replace function : all occurrences of the specified phrase will be replaced, if nothing else is specified.

```

from docling.datamodel.pipeline_options import PdfPipelineOptions, TesseractCliOcrOptions
from docling.datamodel.base_models import InputFormat
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling_core.types.doc import PictureItem
from pathlib import Path
IMAGE_RESOLUTION_SCALE = 2.0

source = "/content/Fact-Sheet-X-Rays-June-2022.pdf"

pipeline_options = PdfPipelineOptions()
pipeline_options.images_scale = IMAGE_RESOLUTION_SCALE
pipeline_options.generate_page_images = True
pipeline_options.generate_picture_images = True
pipeline_options.do_ocr = True
ocr_options = TesseractCliOcrOptions(force_full_page_ocr=True)
pipeline_options.ocr_options = ocr_options

doc_converter = DocumentConverter(format_options = {
  InputFormat.PDF : PdfFormatOption(pipeline_options=pipeline_options)
}
)

# Convert document
doc = doc_converter.convert(source)

doc_filename = doc.input.file.stem


for page_no, page in doc.document.pages.items():
  page_image_filename = Path(f"{doc_filename}-{page_no}.png")
  with page_image_filename.open("wb") as fp:
    page.image.pil_image.save(fp, format="PNG")

picture_counter = 0
for element, _level in doc.document.iterate_items():
  if isinstance(element, PictureItem):
    picture_path = Path(f"{doc_filename}-picture-{picture_counter}.png")
    with picture_path.open("wb") as fp:
      element.get_image(doc.document).save(fp, "PNG")

    picture_counter += 1

images = []
for picture in doc.document.pictures:
  ref = picture.get_ref().cref
  image = picture.image
  if image:
    images.append(str(image.uri))

print(images[0])

from groq import Groq

client = Groq(api_key="")


image_summarization_prompt = """
Describe the image concisely in 1 to 2 sentences
"""

image_summaries = []

image = images[0]

# messages = [
#     {"role":"user", "content":image},
#     {"role":"system", "content":image_summarization_prompt},
# ]

for image in images:
  messages=[
          {
              "role": "user",
              "content": [
                  {
                      "type": "text",
                      "text": "Describe the image concisely in 1 to 2 sentences"
                  },
                  {
                      "type": "image_url",
                      "image_url": {
                          "url": image
                      }
                  }
              ]
          }
      ]

  response = client.chat.completions.create(
      model="meta-llama/llama-4-scout-17b-16e-instruct",
      messages=messages,
      temperature=0.5
  )
  summary = response.choices[0].message.content

  image_summaries.append(summary)


def replace_occurences(text, replacements):
  result = text


  IMAGE_PLACEHOLDER = "<!-- image_placeholder -->"

  for replacement in replacements:
    if IMAGE_PLACEHOLDER in result:
      result = result.replace(IMAGE_PLACEHOLDER, replacement, 1)
    else:
      break

  return result

replace_occurences(doc.document, image_summaries)






```

*


1. When replacing images with corresponding text description, how to make sure that the text is placed in the right location?
2. Does the replace() function in Python replace first occurence or all occurences of matching string?


1. https://www.w3schools.com/python/ref_string_replace.asp

## Exercise
1. Connect to LLM
2. Build a multi turn LLM call
3. Add memory (use conversation history)
4. Prompt engineering (https://github.com/ai-boost/awesome-prompts)
5. https://langroid.github.io/langroid/blog/2023/09/19/language-models-completion-and-chat-completion/
6. https://www.reddit.com/r/MachineLearning/comments/1hxzij5/d_creating_proper_llm_summaries_is_surprisingly/
7. https://www.reddit.com/r/LocalLLaMA/comments/1ftjbz3/shockingly_good_superintelligent_summarization/
8. https://huggingface.co/DeepMostInnovations/sales-conversion-model-reinf-learning
9. https://www.reddit.com/r/SaaS/comments/1hrv2o5/elevenlabs_and_murfai_are_making_millions_with/


## Day 17 to 18

* Moved code from Google colab to local

* To measure time you use use the simple code snippet below
```
import time

start = time.time()
print("hello")
end = time.time()
print(end - start)

```

* Finished first version of RAG pipeline using docling (Created by own sample pdf for testing)
    * Chunking strategy is each section is broken into separate chunk

```

import pickle
import chromadb
import time

from pathlib import Path
from groq import Groq
from docling.datamodel.pipeline_options import PdfPipelineOptions, EasyOcrOptions, TesseractCliOcrOptions
from docling.datamodel.base_models import InputFormat
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling_core.types.doc import PictureItem
from chromadbx import UUIDGenerator


IMAGE_RESOLUTION_SCALE = 2.0

source = "Penguins.pdf"


def parse_docs():
    pipeline_options = PdfPipelineOptions()
    doc_converter = DocumentConverter(format_options = {
    InputFormat.PDF : PdfFormatOption(pipeline_options=pipeline_options)
    }
    )

    # Convert document
    doc = doc_converter.convert(source)
    doc_filename = doc.input.file.stem
    
    return doc.document



def summarize_images(images):

    client = Groq(api_key="")

    image_summarization_prompt = """
    Describe the image concisely in 1 to 2 sentences
    """

    image_summaries = []

    image = images[0]

    for image in images:
        messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "text",
                            "text": image_summarization_prompt
                        },
                        {
                            "type": "image_url",
                            "image_url": {
                                "url": image
                            }
                        }
                    ]
                }
            ]

        response = client.chat.completions.create(
            model="meta-llama/llama-4-scout-17b-16e-instruct",
            messages=messages,
            temperature=0.5
        )
        summary = response.choices[0].message.content

        image_summaries.append(summary)


def replace_occurences(text, replacements):
    result = text

    IMAGE_PLACEHOLDER = "<!-- image_placeholder -->"

    for replacement in replacements:
        if IMAGE_PLACEHOLDER in result:
            result = result.replace(IMAGE_PLACEHOLDER, replacement, 1)
        else:
            break

    return result



def chunk_document(text):
    SPLIT_PATTERN = "\n#"
    chunks = text.split(SPLIT_PATTERN)

    return chunks

start = time.time()
text = parse_docs()
text = text.export_to_markdown()
print(text)
end = time.time()
print("Time taken ", (end-start))
chunks = chunk_document(text)
print(len(chunks))
client = chromadb.PersistentClient()
collection = client.get_or_create_collection(name="test")
collection.add(documents=chunks, ids=UUIDGenerator(len(chunks)))
query = "What is the height of penguins"
result = collection.query(
    query_texts=[query],
    n_results=1,
).get('documents')[0]

print(result)
user_prompt = f"""
Answer the question using following context:
{result}
- -
Answer the question based on the above context: {query}
"""

messages = [
{"role":"user", "content": user_prompt}
]
client = Groq(api_key="")

response = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=messages,
    temperature=0,
)

print(response.choices[0].message.content)

```


* Errors:
    * Unknown metadata version: 2.4, Solution : Upgrade poetry. Easiest way to upgrade Poetry is to reinstall. Typed following in Windows Powershell `(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -`
    * RuntimeError: Tesseract is not available, aborting: [WinError 2] The system cannot find the file specified Install tesseract on your system and the tesseract binary is discoverable. The actual command for Tesseract can be specified in `pipeline_options.ocr_options.tesseract_cmd='tesseract'`. Alternatively, Docling has support for other OCR engines. See the documentation.
    * TypeError: Descriptors cannot be created directly. Protocol Buffer error. To solve this I set the environment variable for the virtual environment as PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python. In the end of venv\Scripts\activate.bat added `@set "PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python"` (refer 5)
    * chromadbx requires Python <3.13,>=3.9, so it will not be installable for Python >=3.13,<4.0, Solution: Updated pyproject.toml file accordingly

### Doubts
1. How to reinstall/upgrade Poetry in Windows?
2. Tesseract vs EasyOcr vs RapidOcr - what are the differences?
3. How to store a Python object so that it can be reused for the next program run?
4. How to set an environment variable in a virtual environment?

### References
1. https://github.com/python-poetry/poetry/issues/9885
2. https://pipx.pypa.io/stable/how-pipx-works/
3. https://stackoverflow.com/questions/62274252/how-can-i-make-a-python-program-so-that-even-after-termination-the-variable-li
4. https://stackoverflow.com/questions/50951955/pytesseract-tesseractnotfound-error-tesseract-is-not-installed-or-its-not-i
5. https://stackoverflow.com/questions/9554087/setting-an-environment-variable-in-virtualenv


## Day 19

* uuid (Universally Unique Identifier) :  A 128-bit number used to uniquely identify information in computer systems (such as database row, session state). Represented as a 36-character string with five groups of hexadecimal digits separated by hyphens eg. a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11 (8-4-4-4-12 hexadecimal i.e. standard format)

* uuid are globally unique, hence the name

* tempfile library creates a temporary folder in `C:\Users\dell\AppData\Local\Temp`

* Created the following Streamlit app with code below
```
import streamlit as st
import tempfile
import base64
import uuid
import gc
import time
import os

from rag import *

if "id" not in st.session_state:
    st.session_state.id = uuid.uuid4()
    st.session_state.file_cache = {}

session_id = st.session_state.id


def reset_chat():
    st.session_state.messages = []
    st.session_state.context = None
    gc.collect()

# Function to display the uploaded PDF in the app
def display_pdf(file):
    st.markdown("### 📄 PDF Preview")
    base64_pdf = base64.b64encode(file.read()).decode("utf-8")
    pdf_display = f"""<iframe src="data:application/pdf;base64,{base64_pdf}" width="500" height="100%" type="application/pdf"
                        style="height:100vh; width:100%"
                    >
                    </iframe>"""
    st.markdown(pdf_display, unsafe_allow_html=True)


# Sidebar: Upload Document
with st.sidebar:
    st.markdown("<h1 style='text-align: center;'>🤖  Multimodal RAG - Query your document</h1>", unsafe_allow_html=True)
    st.header("Upload your PDF")
    uploaded_file = st.file_uploader("", type="pdf")
    

    if uploaded_file:
        file_key = f"{session_id}-{uploaded_file.name}"
        if file_key not in st.session_state.file_cache:
            status_placeholder = st.empty()
            status_placeholder.info("📥 File uploaded successfully")
        
            time.sleep(2.5)  # Delay before switching message
        

            with tempfile.TemporaryDirectory() as temp_dir:
                # Save uploaded file to temp dir
                file_path = os.path.join(temp_dir, uploaded_file.name)
                print(f"Temporary file path: {file_path}")       
                with open(file_path, "wb") as f:
                    f.write(uploaded_file.getvalue())

                # Convert pdf to markdown
                status_placeholder.info("Identifying document layout...")
                progress_bar = st.progress(10)
                start = time.time()
                text = parse_docs(file_path)
                text = text.export_to_markdown()
                st.session_state.markdown_text = text
                end = time.time()
                print("Time taken ", (end-start))

                # Chunk document
                status_placeholder.info("Generating embeddings...")                    
                chunks = chunk_document(text)
                st.session_state.chunks = chunks                
                progress_bar.progress(50)

                # Load chunks into vectorstore
                status_placeholder.info("Indexing the document...")
                progress_bar.progress(80)
                collection = create_vectorstore(chunks, collection_name="penguins")
                st.session_state.collection = collection
                
                # Show pdf upload status as completed
                status_placeholder = st.empty()
                st.success("Ready to Chat...")
                progress_bar.progress(100)
                st.session_state.file_cache[file_key] = True

            
col1, col2 = st.columns([6, 1])

with col2:
    st.button("Clear ↺", on_click=reset_chat)

# Initialize chat history
if "messages" not in st.session_state:
    reset_chat()


# Show message history (preserved across reruns)
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# Accept user query
if prompt := st.chat_input("Ask a question..."):

    # Store and display user message
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    # Generate RAG-based response
    with st.chat_message("assistant"):
        message_placeholder = st.empty()
    
        with st.spinner("Thinking..."):
        
            collection = st.session_state.get("collection")
            relevant_chunks = retrieve_relevant_chunks(query=prompt, collection=collection)
            response_text = generate_response(prompt, relevant_chunks)
            message_placeholder.markdown(response_text)
            

    # Store assistant response
    st.session_state.messages.append({"role": "assistant", "content": response_text})


```


*Errors
    * `label` got an empty value. This is discouraged for accessibility reasons and may be disallowed in the future by raising an exception. Please provide a non-empty label and hide it with label_visibility if needed.
    * RuntimeError: Tried to instantiate class '__path__._path', but it does not exist! Ensure that it is registered via torch::class_
    * Not able to stop by just using Ctrl+C. Have to type Ctrl+C, then close streamlit, then open the url, only then app stops running

### Doubts


### References
1. https://www.cockroachlabs.com/blog/what-is-a-uuid/



## Day 20 and 21

* Learnt fundamentals of ElasticSearch

* Elasticsearch's full-text search requires minimal computational resources (can run on CPU) compared to GPU-intensive vector operations, hence cost effective

* Hybrid search : Full text search + Vector search

* Steps involved in full text seatch
    1. Text preprocessing (including stemming, lower case, stopword elimination)
    2. Inverted Index creation
    3. Query processing 
    4. Relevance scoring (using BM25 algorithm to find similar docs)

* Elasticsearch Analyzer : Wrapper around 3 functions
    * Character filter (add/modify/remove characters, for example remove html tags)
    * Tokenizer (split doc into tokens)
    * Token filter (processes the tokens like lowercase, removing stopword tokens)

* Query is processed using Query analyzer

* Encoder-decoder model : Type of neural network architecture used for sequential data processing and generation i.e. both input and output are sequences
    * Encoder : Converts entire input data into a single fixed size vector, called context vector
    * Decoder : Takes context vector and produces the output one step at a time

* Encoder-decoder model can be implemented using RNN, LSTM or Transformers. 

* Transformers are indirect descendants of the previous RNN models. The problem with RNN was since entire input was compressed into a single context vector, it was an information bottleneck as we are trying to squeeze it all through a single connection. The attention mechanism provided a solution to the bottleneck issue (check Vizuara LLM from scratch series for more depth)

* Cross Encoder : Produces very accurate results for similarity scores. A cross-encoder processes both inputs jointly, allowing full attention between all tokens in both sequences.

* Sparse Encoding Model : A class of neural retrieval models that represent text using sparse vector representations, as opposed to dense ones (like BERT embeddings). They aim to combine the interpretability and efficiency of traditional sparse methods (like BM25) with the semantic power of neural models.

* Sentence Transformer : Type of deep learning model designed to convert sentences into numerical vectors (embeddings) that capture the semantic meaning of the text.

* ELSER model used by ElasticSearch is not an open source model.

* Index template : a way to tell Elasticsearch how to configure an index when it the index created. Index templates define settings, mappings, and aliases that can be applied automatically to new indices.

* An index consists of
    1. Index name
    2. Index mapping
    3. Index setting

* Index mapping : Mapping defines how documents and their fields are indexed and stored. It's like a schema in a relational database. Specifies things like:
    1. Data type
    2. Analyzer i.e.how fields are analyzed (tokenized and filtered)

```
"mappings":{
        
        "properties":{
            "Question": {"type": "text"},
            "Answer": {"type": "text"},
            "Question Type":{"type": "keyword"},
            "id": {"type": "keyword"},
            "question_answer_vector": {
                "type": "dense_vector",
                "dims": 768,
                "index": True,
                "similarity": "cosine"
            }
        }
    }

```

* Types of query clauses
    1. Leaf query clauses : Leaf query clauses look for a particular value in a particular field, such as the match, term or range queries. These queries can be used by themselves
    2. Compound query clauses : Compound query clauses wrap other leaf or compound queries and are used to combine multiple queries in a logical fashion

* In Leaf Query clauses we have
    1. Full Text Query : For searching text. Includes simple_query_string, match, multi_match
    2. Vector Query 

* Below is the entire code to perform a keyword search in Elasticsearch. It involves the following steps
    1. Initialize Elasticsearch client
    2. Define index properties (using index template) and create index
    3. Load json documents into index 
    4. Define query properties (using query clause)
    5. Query the index

```
### keyword_search.py

import re
import json

from tqdm import tqdm
from elasticsearch import Elasticsearch

## Initialize Elasticsearch client
client = Elasticsearch('http://localhost:9200')

print(client.info())

## Index Properties
index_name = "medical-questions"

index_template = {
    "settings":{"number_of_shards":1},
    "mappings":{
        # Type of each field in the data : Question, Answer, etc
        "properties":{
            "Question": {"type": "text"},
            "Answer": {"type": "text"},
            "Question Type":{"type": "keyword"},
            "id": {"type": "keyword"},
            "question_answer_vector": {
                "type": "dense_vector",
                "dims": 768,
                "index": True,
                "similarity": "cosine"
            }
        }
    }
}

## Create index
if not client.indices.exists(index=index_name):
    client.indices.create(index=index_name, body=index_template)

all_indices = client.indices.get_alias(index='*')
for index_name in all_indices:
    print(index_name)

## Load documents into index
def index_documents():
    with open("Medical-QA-100.json","r") as f:
        document = json.load(f)

    for doc in tqdm(document):
        client.index(index=index_name, document=doc)


## Query clause for querying Elasticsearch    
keyword_query = {
    "query": {
        "simple_query_string":{
            "query": None,
            "fields":["Question", "Answer"],
        }
    }
}

## Execute search and get results
def keyword_search(query):
    keyword_query['query']['simple_query_string']['query'] = query
    print(keyword_query)
    keyword_results = client.search(index="medical-questions",
        body=keyword_query, size=5)['hits']['hits']
    
    print([res['_source']['Question'] for res in keyword_results])
    return(keyword_results)


keyword_search("parasites")



```
* Reciprocal rank fusion (RRF) is a method for combining multiple result sets with different relevance indicators into a single result set. Implementation is as follows:
    1. Assign Reciprocal Rank Scores: For each document in each ranked list, calculate its reciprocal rank (1 / rank).
    2. Combine Scores:Sum the reciprocal rank scores for each document across all the input lists.
    3. Re-rank: Sort the documents based on their combined scores, creating a new, unified ranked list. 


* Embedding model is stored in ` C:\Users\dell\.cache\huggingface\hub\models--sentence-transformers--multi-qa-distilbert-cos-v1`


* Errors
    * elasticsearch.BadRequestError: BadRequestError(400, 'media_type_header_exception', 'Invalid media-type value on headers [Content-Type, Accept]', Accept version must be either version 8 or 7, but found 9. Accept=application/vnd.elasticsearch+json; compatible-with=9) : Reason : elastic search version on docker was (8.15.0) but the Python client installed was 9.0.2. Hence did poetry add elasticsearch==8.15.0
```
poetry add elasticsearch
poetry remove elasticsearch
poetry add elasticsearch=8.15.0

```
    * ValueError: Invalid pattern: '**' can only be an entire path component . Reason : rror is likely due to a change in datasets package (somewhere between 2.1 to 2.14) is breaking fsspec. Hence pip install -u datasets
    * elasticsearch.BadRequestError: BadRequestError(400, 'parsing_exception', 'unknown query [query]') Reason : The query template was incorrect, with an additional layer of 'query'
    
```  
keyword_query = {
    "query": {
        "simple_query_string":{
            "query": None,
            "fields":["Question", "Answer"],
        }
    }
}

# Incorrect : Since we already have query in our query template, no need of putting again while passing to body as argument
keyword_results = client.search(index="medical-questions",
        body={
            'query':keyword_query,
            'size':5
        })['hits']['hits']


# Correct
keyword_results = client.search(index="medical-questions",
        body=keyword_query, size=5)['hits']['hits']

```
    * elasticsearch.BadRequestError: BadRequestError(400, 'parsing_exception', "[match] query doesn't support multiple fields, found [query] and [fields]")
    * DeprecationWarning: Received 'size' via a specific parameter in the presence of a 'body' parameter, which is deprecated and will be removed in a future version. Instead, use only 'body' or only specific parameters.
    * elasticsearch.BadRequestError: BadRequestError(400, 'parsing_exception', '[simple_query_string] malformed query, expected [END_OBJECT] but found [FIELD_NAME]')

```
## Wrong code
keyword_query['query']['simple_query_string']['query'] = query
keyword_query['query']['size'] = size

## Correct code
keyword_query['query']['simple_query_string']['query'] = query
keyword_query['size'] = size


```
    * UserWarning: `huggingface_hub` cache-system uses symlinks by default to efficiently store duplicated files but your machine does not support them. Caching files will still work but in a degraded version that might require more space on your disk. This warning can be disabled by setting the `HF_HUB_DISABLE_SYMLINKS_WARNING` environment variable.

### Doubts
1. What is query language and query dsl?


### References
1. https://www.elastic.co/docs/reference/elasticsearch/clients/python/connecting
2. https://www.elastic.co/docs/manage-data/data-store/text-analysis/anatomy-of-an-analyzer#_character_filters
3. https://stackoverflow.com/questions/51807333/what-is-analyzer-in-elasticsearch-for
4. https://stackoverflow.com/questions/77671277/valueerror-invalid-pattern-can-only-be-an-entire-path-component
5. https://www.pinecone.io/learn/series/nlp/sentence-embeddings/
6. https://www.reddit.com/r/LocalLLaMA/comments/1fdmoxl/open_source_alternatives_to_elastic_searchs_elser/
7. https://www.elastic.co/docs/manage-data/data-store/templates#create-index-templates
8. https://huggingface.co/datasets/keivalya/MedQuad-MedicalQnADataset
9. https://www.elastic.co/docs/api/doc/elasticsearch/authentication
10. https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/mapping-parameters
11. https://kshitijkutumbe.medium.com/blog-4-elasticsearch-query-deep-dive-text-keyword-and-vector-searches-with-python-e42586003abd
12. https://stackoverflow.com/questions/62324903/elasticsearch-parsing-exception-400
13. https://www.elastic.co/docs/reference/query-languages/query-dsl/full-text-queries
14. https://www.elastic.co/docs/explore-analyze/query-filter/languages/querydsl
15. https://www.youtube.com/watch?v=poERWnPrscc&list=PLGZAAioH7ZlMQGCt8GeAaJLvgehhq-gEK&index=1
16. https://github.com/elastic/elasticsearch-labs/blob/main/notebooks/search/02-hybrid-search.ipynb
17. https://www.elastic.co/docs/solutions/search/vector

## Day 22

* Requirement : Check number of records in json file from the Python interpreter
```
>>> f = open('Medical-QA.json')
>>> z = json.load(f)
>>> len(z)
1000
>>> f.close()
```
* Note : We need to open a file before we can read it using the json library

* MD5 is deterministic and hence used for thinngs like file verification.

* Mean Reciprocal Rank : Used to evaluate the performance of ranking systems (specially IR and Question Answering). Mean of the reciprocal ranks across all queries

* Examples for computing Mean Reciprocal Rank
```
## Input list of relevance flag
[[False, False, False, False, False], 
[False, False, False, False, False], 
[False, False, False, False, False], 
[False, False, False, False, False], 
[False, False, False, False, False], 
[True, True, True, True, False]]

MRR = (0 + 0 + 0 + 0 + 0 + (1/1)) / 6 = 0.1666

[[False, False, False, False, False], 
[False, False, False, False, False], 
[False, False, False, False, False], 
[False, False, False, False, False], 
[False, False, False, False, False], 
[True, True, True, True, False],
[False, False, False, False, True]
]

MRR = (0 + 0 + 0 + 0 + 0 + (1/1) + (1/5)) / 7 = 0.1714
```


* Errors:
    * ValueError: True is not in list. Reason : All values in list are False

```
## Old
rank = relevance_list.index(True)
return rank

## New
if True in relevance_list:
    rank = relevance_list.index(True)
    return rank

return 0

```
    * ZeroDivisionError: float division by zero
```
## Old
rank = relevance_list.index(True)
reciprocal_rank = 1.0 / (rank)

## New
rank = relevance_list.index(True)
reciprocal_rank = 1.0 / (rank + 1)
```

* TO DO : Add remaining code from repo to notes

### Doubts
1. Is MD5 algorithm deterministic i.e. eoes the MD5 algorithm always generate the same output for the same string?

### References

1. https://stackoverflow.com/questions/4354377/does-the-md5-algorithm-always-generate-the-same-output-for-the-same-string
2. https://www.geeksforgeeks.org/python/python-handling-no-element-found-in-index/
3. https://stackoverflow.com/questions/522372/finding-first-and-last-index-of-some-value-in-a-list-in-python



## Day 23 and 24 (To Complete)
* Requirement : Finetune OpenAI model on Customer support data and evaluate the fine tuning

* Summarize all learning and youtube video as well

* An important an initially challenging part for finetuning and evaluating data is preparing the data in the right format.

* Dataset preparation
    * For Finetuning : Use the chat completions format, and have it in jsonl
    ```
    {"messages": 
        [
            {"role": "system", "content": "You are a helpful assistant"}, 
            {"role": "user", "content": "My order hasn't arrived yet."}, 
            {"role": "assistant", "content": "We apologize for the inconvenience. Can you please provide your order number so we can investigate?"}
        ]
    }

    ```

    * For Evaluation : Input and output
    ```
    {"prompt":"WhatisthecapitalofFrance?", "completion":"Paris"}


    ```

* 4 core concepts in LLM evaluation
    * Dataset :
    * Evaluator :
    * Task : 
    * Interpretation : Understanding and analyzing evaluation outcomes

* To run evaluation from UI : https://platform.openai.com/evaluations

* If you want to dump the JSON into a file/socket or whatever, then you should go with dump(). If you only need it as a string (for printing, parsing or whatever) then use dumps() (dump string)

* An eval needs two key ingredients:
    * data_source_config: A schema for the test data you will use along with the eval.
    * testing_criteria: The graders that determine if the model output is correct.

*  BLEU looks at how many words and phrases (n-grams) in the machine translation match the human translations. The more matches and the closer the length, the higher the BLEU score.

* Errors
    * openai.OpenAIError: The api_key client option must be set either by passing api_key to the client or by setting the OPENAI_API_KEY environment variable.Reason : In .env it was OPEN_API_KEy instead of OPENAI_API_KEY
    * Training file has 3 examples, must atleast have 10

### Doubts
1. What is differece between `json.dump` and `json.dumps`?
2. What is difference between system and devloper role?

### References
1. https://stackoverflow.com/questions/36059194/what-is-the-difference-between-json-dump-and-json-dumps-in-python
2. https://platform.openai.com/docs/guides/graders#text-similarity-graders
3. https://www.youtube.com/watch?v=pgyhq-WagIg (Manny Bernabe)
4. https://community.openai.com/t/system-vs-developer-role-in-4o-model/1119179/2
5. https://platform.openai.com/docs/guides/supervised-fine-tuning
6. https://platform.openai.com/docs/guides/evals


## Day 25

* Requirement : Creating a Telegram bot using n8n hosted locally using Docker

* Ngrok : To serve local system to Internet, Ngrok provide a tunnel b/w Internet and local machine. Ngrok connects to the internet by establishing a secure, persistent tunnel between local machine and the ngrok cloud service. This tunnel allows you to expose local applications, like a web server, to the internet through a public URL provided by ngrok. When someone accesses that public URL, ngrok forwards the traffic through the secure tunnel to your local application. 

* We need Ngrok so that Telegram API can access our workflow. Else it can access our locally hosted workflow

* Steps to create a Telegram bot using N8N locally
    1. Create a Telegram Bot using BotFather in Telegram
    2. Create an account in Ngrok
    3. In Ngrok, in Setup and Installation, under Deploy your app online, select Static Domain to create a static url
    4. Create a .env file replacing `DOMAIN_NAME` and `SUBDOMAIN` with the domain and sub-domain obtained from Ngrok (in step 3)
    5. Go to a folder (n8n-project) where you want to store project files. Within that folder, create another folder called `local-files`
    6. In the main folder, create a compose.yml file with the configuration from 3rd link in reference
    7. Start Docker Compose using `docker compose up -d`
    8. Start Ngrok server using Docker with the command from Ngrok site `docker run --net=host -it -e NGROK_AUTHTOKEN=<AUTH-TOKEN> ngrok/ngrok:latest http --url=your-url.ngrok-free.app 5678`
    9. Open the Ngrok website in browser `your-url.ngrok-free.app`
    10. Start new workflow
    11. Add Telegram trigger. Use the credentials obtained from step 1
    12. Add OpenAI message a model step
    13. Add Telegram Send a Text Message step
    14. Make the workflow active

* N8N (node-based no-code) : Lowcode/nocode workflow automation tool. Similar to Power Automate

* Important n8n concepts
    * Nodes : Basic building block. Each node represents some operation/some action (like sending message to Telegram, calling OpenAI api, etc)
    * Trigger node : Special node responsible for starting the workflow in response to certain event (eg. Telegram trigger node)
    * Action node : Represent specific tasks within a workflow (eg. calling OpenAI api node)
    * Connection : Link between 2 nodes, passed data from one node to another
    * Execution : Single run of a workflow. 2 modes
        * Manual : We have to run flow manually. Used during dev and testing
        * Production : Flow runs automatically. Used in prod. For this we must set the workflow to Active
    

* Errors:
    * Telegram Trigger: Bad Request: bad webhook: An HTTPS URL must be provided for webhook. Solution : Use Ngrok

### Doubts
1. Why do we need to use Ngrok when we setup n8n locally, if we want to use the Telegram node (Send a message)?

### References
1. https://www.youtube.com/watch?v=azNX3bu0pwE
2. https://www.youtube.com/watch?v=_nMkQaguy3E
3. https://community.n8n.io/t/telegram-trigger-behind-swag-proxy-bad-request-bad-webhook-https-url-must-be-provided-for-webhook/7122/15
4. https://community.n8n.io/t/telegram-node-seperating-text-and-photos-in-different-workflows/70658/2
5. https://www.youtube.com/watch?v=HPEduQTSdT8

## To Do
1. Transformer from scratch : https://github.com/aladdinpersson/Machine-Learning-Collection/blob/master/ML/Pytorch/more_advanced/transformer_from_scratch/transformer_from_scratch.py (https://www.youtube.com/watch?v=U0s0f995w14)
2. Langraph Agent : https://github.com/schmitech/ai-driven-order-management
3. Langraph Crew AI : https://github.com/crewAIInc/crewAI-examples/tree/main/CrewAI-LangGraph

