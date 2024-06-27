# Azure API Management's AI Gateway for AOAI use-case scenarios

XXXXXXXXXXXXXX

## Table of contents:
- [Scenario 1: Enforcing custom token limit](https://github.com/LazaUK/AOAI-APIM-AIGateway?tab=readme-ov-file#scenario-1-enforcing-custom-token-limit)
- [Scenario 2: Usage analysis by specific customer](https://github.com/LazaUK/AOAI-APIM-AIGateway?tab=readme-ov-file#scenario-2-usage-analysis-by-specific-customer)
- [Scenario 3: Load-balancing between several AOAI endpoints](https://github.com/LazaUK/AOAI-APIM-AIGateway?tab=readme-ov-file#scenario-3-load-balancing-between-several-aoai-endpoints)

## Scenario 1: Enforcing custom token limit
1. In API-M's Azure portal settings choose _APIs -> Add API_ and then click "**Azure OpenAI Service**" tile under "**Create from Azure resource**" category.
![APIM - Adding APIs for AOAI](/images/apim_api_add.png)
2. Select your existing AOAI resource, and enter values for **Display Name** and **Name** fields. Optionally, you can tick _SDK Compatibility_ field, to enable OpenAI-compatible consumption of exposed APIs from popular Generative AI frameworks and libraries:
![APIM - Defining AOAI endpoint](/images/apim_aoai_ep.png)
3. After clicking **Next**, enable "_Manage token consumption_" API-M policy and set desired Tokens-per-Minute (TPM) limit. Optionaly you can add "consumed tokens" and "remaining tokens" headers to API-M endpoint's responses.
![APIM - Enabling TPM policy](/images/apim_tpm_config.png)
> _Note_: provided [Jupyter notebook](AOAI_APIM_TPM_Limit.ipynb) assumes that you have both headers enabled and was tested against API-M endpoint with **100** TPM limit.
4. Once you click the Create button, a new set of APIs will be provisioned to support interactions with various AOAI models. API-M will also add token consumption policy to all newly provisioned API operations. Technical aspects of this policy can be found in [this reference document](https://learn.microsoft.com/en-gb/azure/api-management/azure-openai-token-limit-policy):
![APIM - Detailing TPM policy](/images/apim_tpm_policy.png)
5. If you want to test your TPM limit, ensure that you set the following 4 environment variables prior to running the notebook:
| Environment Variable | Description |
| --- | --- |
| _APIM_TPM_URL_ | URL of the provisioned API-M's endpoint. |
| _APIM_TPM_SUB_KEY_ | Subscription key, with its scope covering target APIs. |
| _APIM_TPM_API_VERSION_ | API version of AOAI endpoint. |
| _APIM_TPM_AOAI_DEPLOY_ | Name of AOAI depoyment. |
![APIM - Setting environment variables](/images/apim_env_var.png)

## Scenario 2: Usage analysis by specific customer
1. Repeat Steps # 1 and 2 from Scenario 1 above.

## Scenario 3: Load-balancing between several AOAI endpoints
