# Azure API Management's AI Gateway for AOAI use-case scenarios

XXXXXXXXXXXXXX

## Table of contents:
- [Scenario 1: Enforcing custom token limit]()
- [Scenario 2: Usage analysis by specific customer]()
- [Scenario 3: Load-balancing between several AOAI endpoints]()

## Scenario 1: Enforcing custom token limit
1. In API-M's Azure portal settings choose _APIs -> Add API_ and then click "**Azure OpenAI Service**" tile under "**Create from Azure resource**" category.
![APIM - Adding APIs for AOAI](/images/apim_api_add.png)
2. Select your existing AOAI resource, and enter values for **Display Name** and **Name** fields. Optionally, you can tick _SDK Compatibility_ field, to enable OpenAI-compatible consumption of exposed APIs from popular Generative AI frameworks and libraries:
![APIM - Defining AOAI endpoint](/images/apim_aoai_ep.png)
3. After clicking **Next**, enable "_Manage token consumption_" API-M policy and set desired Tokens-per-Minute (TPM) limit. Optionaly you can add "consumed tokens" and "remaining tokens" headers to API-M endpoint's responses.
![APIM - Enabling TPM policy](/images/apim_tpm_config.png)
> _Note_: provided [Jupyter notebook](AOAI_APIM_TPM_Limit.ipynb) assumes that you have both headers enabled and was tested against API-M endpoint with **100** TPM limit.
5. 

## Scenario 2: Usage analysis by specific customer
1. Repeat Steps # 1 and 2 from Scenario 1 above.

## Scenario 3: Load-balancing between several AOAI endpoints
