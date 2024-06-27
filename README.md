# Azure API Management's AI Gateway for AOAI use-case scenarios

XXXXXXXXXXXXXX

## Table of contents:
- [Scenario 1: Enforcing custom token limit]()
- [Scenario 2: Usage analysis by specific customer]()
- [Scenario 3: Load-balancing between several AOAI endpoints]()

## Scenario 1: Enforcing custom token limit
1. In API-M's Azure portal settings choose _APIs -> Add API_ and then click "**Azure OpenAI Service**" tile under "**Create from Azure resource**" category.
![APIM - Adding APIs for AOAI](/images/apim_api_add.png)
2. Select your existing AOAI resource, and enter values for Display Name and Name fields. Optionally, you can tick SDK Compatibility field, to enable OpenAI-compatible consumption of exposed APIs from popular Generative AI frameworks and libraries:
![APIM - Defining AOAI endpoint](/images/apim_aoai_ep.png)
3. 

## Scenario 2: Usage analysis by specific customer
1. Repeat Steps # 1 and 2 from Scenario 1 above.

## Scenario 3: Load-balancing between several AOAI endpoints
