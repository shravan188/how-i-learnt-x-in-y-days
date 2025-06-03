
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

* When the LLM decides to do a function call it does not actually call the functions, instead it just returns the name of the function. We have to then parse it and then call the function ourselves

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
