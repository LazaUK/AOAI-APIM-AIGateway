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
![APIM - Setting environment variables](/images/apim_env_var.png)

| Environment Variable | Description |
| --- | --- |
| _APIM_TPM_AOAI_DEPLOY_ | Name of AOAI depoyment |
| _APIM_TPM_API_VERSION_ | API version of AOAI endpoint |
| _APIM_TPM_SUB_KEY_ | Subscription key, with the scope covering target API-M APIs |
| _APIM_TPM_URL_ | URL of provisioned API-M's endpoint for AOAI endpoint |

6. We can use now a Helper function to interact with AOAI backend through API-M endpoint:
``` Python
def get_rest_completion(system_prompt, user_prompt):
    response = requests.post(
        url = f"{APIM_TPM_URL}openai/deployments/{AOAI_DEPLOYMENT}/chat/completions",
        headers = {
            "Content-Type": "application/json",
            "api-key": APIM_TPM_SUB_KEY
        },
        params={'api-version': AOAI_API_VERSION},
        json = {
            "messages": [
                {
                   "role": "system",
                    "content": system_prompt
                },
                {
                    "role": "user",
                    "content": user_prompt
                }
            ]
        }
    )
    return response
```
7. If you set your TPM value to 100 and the average consumption of tokens in your request is about 50, then after few API calls you will should reach the token limit, with API-M enforcing new policy as shown in the testing results below. 
``` JSON
Run # 0 completed in 1.93 seconds
Consumed tokens: 59
Remaining tokens: 41
Pausing for 15 seconds...
-----------------------------
Run # 1 completed in 0.78 seconds
Consumed tokens: 55
Remaining tokens: 0
Pausing for 15 seconds...
-----------------------------
Run # 2 completed in 0.40 seconds
Response code: 429
Response message: Token limit is exceeded. Try again in 29 seconds.
Pausing for 15 seconds...
-----------------------------
Run # 3 completed in 0.35 seconds
Response code: 429
Response message: Token limit is exceeded. Try again in 14 seconds.
Pausing for 15 seconds...
-----------------------------
Run # 4 completed in 0.91 seconds
Consumed tokens: 55
Remaining tokens: 0
-----------------------------
```
8. If you enabled SDK compatibility in Step 2 above, then you can use OpenAI Python SDK to instantiate _AzureOpenAI_ class with API-M URL and credentials:
``` Python
client = AzureOpenAI(
    azure_endpoint = APIM_TPM_URL,
    api_key = APIM_TPM_SUB_KEY,
    api_version = AOAI_API_VERSION
)
```
9. This would allow then OpenAI-compatible interfacing, with example Helper function shown below.
``` Python
def get_sdk_completion(system_prompt, prompt):
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ]

    response = client.chat.completions.create(
        model = AOAI_DEPLOYMENT,
        messages = messages
    )
    return response
```
10. When using SDK interface, instead of getting 429 errors you may notice throttling being enforced by API-M, because of our TPM limit policy.
``` JSON
Run # 0 completed in 14.09 seconds
Tokens consumed: 51
First few words of completion: Sure, here...
Pausing for 15 seconds...
-----------------------------
Run # 1 completed in 31.48 seconds
Tokens consumed: 55
First few words of completion: Sure, here...
Pausing for 15 seconds...
-----------------------------
Run # 2 completed in 0.90 seconds
Tokens consumed: 58
First few words of completion: Sure, here...
Pausing for 15 seconds...
-----------------------------
Run # 3 completed in 30.22 seconds
Tokens consumed: 55
First few words of completion: Sure! Here...
Pausing for 15 seconds...
-----------------------------
Run # 4 completed in 1.25 seconds
Tokens consumed: 55
First few words of completion: Sure, here...
-----------------------------
```

## Scenario 2: Usage analysis by specific customer
1. Repeat Steps # 1 and 2 from Scenario 1 above.
![APIM - Visualising usage stats](/images/apim_usage_chart.png)

## Scenario 3: Load-balancing between several AOAI endpoints
