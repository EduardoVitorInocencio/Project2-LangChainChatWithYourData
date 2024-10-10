# Section 04: Ice Breaker Real World Generative AI Agent Application (1)

Lesson 15

ETAPAS:

Obter o token no Proxyc Url

Gerar a consulta via python utilizando requests

Copiar o json

Abrir o gist e colocar o Json lá

Criar o código abaixo:

```powershell
import os
import requests

from dotenv import load_dotenv

load_dotenv()

def scrape_linkedin_profile(linkedin_profile_url: str, mock: bool = True):
    """
    Scrape information from LinkedIn profiles,
    Mannualy scrape the information from LinkedIn profile.
    
    """
    
    if mock:
        linkedin_profile_url = "https://gist.githubusercontent.com/EduardoVitorInocencio/7d64f82b4a5b48c1040fe65ac2b420b2/raw/22a6184f266f5ddbcef78992bb9fa41d9422301b/eduardo-inocencio.json"
        response = requests.get(
            linkedin_profile_url,
            timeout=20,
        )
        
    else:
        api_endpoint = 'https://nubela.co/proxycurl/api/v2/linkedin'
        headers_dic = {'Authorization': f'Bearer {os.environ.get("PROXYCURL_API_KEY")}'}
        response = requests.get(
            api_endpoint,
            params={"url": linkedin_profile_url},
            headers=headers_dic,
            timeout=20,
        )
    
    data = response.json()
    
    # REMOVER OS CONTEÚDOS/TOKENS QUE NÃO AGREGAM VALOR AO CONJUNTO DE DADOS
    # AQUI REMOVEMOS TODAS AS COLEÇÕES VAZIAS E OS TÓPICOS ["people_also_viewed","certifications"]
    # ONDE: K CORRESPONDE À CHAVE E V CORRESPONDE AO VALOR
    data = {
        k:v
        for k, v in data.items()
        if v not in ([],"","",None)
        and k not in ["people_also_viewed","certifications"]
    }
    
    return data

if __name__ == '__main__':
    print(
        scrape_linkedin_profile(
            linkedin_profile_url='https://www.linkedin.com/in/eduardoinocencio/'
        )
    )

```

Alterar o código no icebreaker.py

```python
# CORE PACKAGES
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain_core.output_parsers import StrOutputParser
from third_parties.linkedin import scrape_linkedin_profile

# ADDITIONAL PACKAGES
from dotenv import load_dotenv
import os

load_dotenv()
openai_api_key = os.getenv('OPENAI_KEY')

if __name__ == '__main__':
    print(os.getenv('OPENAI_KEY'))
    
    summary_template = """
    
        given the LinkedIn information {information} about a person from I want you to create:
        1. a short summary
        2. two interesting fact about them
    """
      
    summary_prompt_template = PromptTemplate(input_variables =["information"], template = summary_template)
    
    llm = ChatOpenAI(temperature=0, model_name='gpt-3.5-turbo',openai_api_key=openai_api_key)
    # llm = ChatOllama(model='mistral')
    
    # Criar a cadeia LLM (PromptTemplate + ChatOpenAI)
    chain = prompt=summary_prompt_template | llm |StrOutputParser()
    
    linkedin_data = scrape_linkedin_profile(linkedin_profile_url="https://www.linkedin.com/in/eduardoinocencio/")
    
    # Executar a cadeia de prompts
    res = chain.invoke(input={"information": linkedin_data})
    
    print(res)
```

# **Lesson 16 - LinkedIn data processing - Part2 - Agents Theory**

## Agents

https://python.langchain.com/v0.1/docs/modules/agents/

The core idea of agents is to use a language model to choose a sequence of actions to take. In chains, a sequence of actions is hardcoded (in code). In agents, a language model is used as a reasoning engine to determine which actions to take and in which order.

# Lesson 17 - **LinkedIn data processing - Part3: Tools, Agent Executor, create_react_agent**

## Prompt template: hwchase17/react

https://smith.langchain.com/hub/hwchase17/react

Answer the following questions as best you can. You have access to the following tools:

Connecting the LLM to internet

https://app.tavily.com/home