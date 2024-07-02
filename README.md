# Azure API Management's AI Gateway for AOAI use-case scenarios

**_Azure API Management (API-M)_** helps you publish and securely manage custom Application Programming Interfaces (APIs), acting as a gateway between clients and backend APIs.

**_Azure OpenAI (AOAI)_** lets you deploy and use OpenAI's powerful large language models (LLMs) like **GPT-4o** on Azure to process and generate multimodal content and easily integrate with other solutions of your choice.

In this repo, I'll demonstrate how to combine the functionalities of API-M and AOAI to enable the following use-case scenarios:
- ```Enforce custom token limit```, so that calling apps can co-share AOAI backends without causing "noisy neighbour" situations;
- ```Get detailed token usage breakdown```, to understand consumption and accurately re-charge cost to customers or business functions;
- ```Enable load-balancing between target AOAI deployments``` to ensure data residency, performance and reliability of your AI solutions.

## Table of contents:
- [Scenario 1: Enforcing custom token limit](https://github.com/LazaUK/AOAI-APIM-AIGateway?tab=readme-ov-file#scenario-1-enforcing-custom-token-limit)
- [Scenario 2: Usage analysis by specific customer](https://github.com/LazaUK/AOAI-APIM-AIGateway?tab=readme-ov-file#scenario-2-usage-analysis-by-specific-customer)
- [Scenario 3: Load-balancing between several AOAI endpoints](https://github.com/LazaUK/AOAI-APIM-AIGateway?tab=readme-ov-file#scenario-3-load-balancing-between-several-aoai-endpoints)

## Scenario 1: Enforcing custom token limit
This section describes setting up API-M and then end-to-end testing of the token limit enforcement scenario.
1. In the Azure portal, navigate to your API Management settings. Under _APIs_, click _Add API_ and then select the "**Azure OpenAI Service**" tile under the "**Create from Azure resource**" category.
![APIM - Adding APIs for AOAI](/images/apim_api_add.png)
2. Select your existing AOAI resource, and enter values for the **Display Name** and **Name** fields. Optionally, you can tick the _SDK Compatibility_ field, to enable OpenAI-compatible consumption of exposed APIs from popular Generative AI frameworks and libraries:
![APIM - Defining AOAI endpoint](/images/apim_aoai_ep.png)
3. After clicking **Next**, enable the "_Manage token consumption_" API-M policy and set your desired Tokens-per-Minute (TPM) limit. You can optionally add "consumed tokens" and "remaining tokens" headers to API-M endpoint's responses.
![APIM - Enabling TPM policy](/images/apim_tpm_config.png)
> _Note_: The provided [Jupyter notebook](AOAI_APIM_TPM_Limit.ipynb) assumes you have both headers enabled and it was tested against an API-M endpoint with a **100** TPM limit.
4. Once you click the **Create** button, a new set of APIs will be provisioned to support interactions with various AOAI models. API-M will also add the token limit policy to all new API operations. Technical aspects of this policy can be found in [this reference document](https://learn.microsoft.com/en-gb/azure/api-management/azure-openai-token-limit-policy):
``` XML
<policies>
    <inbound>
        <set-backend-service id="apim-generated-policy" backend-id="aoai-tpm-limit-openai-endpoint" />
        <azure-openai-token-limit tokens-per-minute="100" counter-key="@(context.Subscription.Id)" estimate-prompt-tokens="false" tokens-consumed-header-name="consumed-tokens" remaining-tokens-header-name="remaining-tokens" />
        <authentication-managed-identity resource="https://cognitiveservices.azure.com/" />
        <base />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```
5. To test your TPM limit, ensure that you set the following 4 environment variables before running the notebook:
![APIM - Setting TPM environment variables](/images/apim_tpm_env_var.png)

| Environment Variable | Description |
| --- | --- |
| _APIM_TPM_AOAI_DEPLOY_ | Name of AOAI deployment |
| _APIM_TPM_API_VERSION_ | API version of AOAI endpoint |
| _APIM_TPM_SUB_KEY_ | Subscription key, with the scope of target API-M APIs |
| _APIM_TPM_URL_ | URL of provisioned API-M's API for AOAI endpoint |

6. We can now use a Helper function to interact with the AOAI backend through the API-M endpoint:
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
7. If you set your TPM value to 100 and the average consumption of tokens in your request is about 50, then after a few API calls, you should reach the token limit, with API-M enforcing the new policy as shown in the testing results below:
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
8. If you enabled SDK compatibility in Step 2 above, you could use the OpenAI Python SDK to interact with your AOAI models through the API-M endpoint. Here's how to instantiate the AzureOpenAI class with your API-M's subscription key:
``` Python
client = AzureOpenAI(
    azure_endpoint = APIM_TPM_URL,
    api_key = APIM_TPM_SUB_KEY,
    api_version = AOAI_API_VERSION
)
```
9. This enables an OpenAI-compatible interface, with an example Helper function shown below:
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
10. When using the SDK interface, instead of receiving 429 errors, you might observe throttling enforced by API-M, because of the TPM limit policy:
``` JSON
Run # 0 completed in 45.82 seconds
Consumed tokens: 55
Remaining tokens: 0
Pausing for 15 seconds...
-----------------------------
Run # 1 completed in 0.65 seconds
Consumed tokens: 55
Remaining tokens: 0
Pausing for 15 seconds...
-----------------------------
Run # 2 completed in 30.87 seconds
Consumed tokens: 55
Remaining tokens: 0
Pausing for 15 seconds...
-----------------------------
Run # 3 completed in 1.20 seconds
Consumed tokens: 63
Remaining tokens: 0
Pausing for 15 seconds...
-----------------------------
Run # 4 completed in 29.88 seconds
Consumed tokens: 56
Remaining tokens: 0
-----------------------------
```

## Scenario 2: Usage analysis by specific customer
This section describes setting up API-M and then performing end-to-end testing of the token usage collection and visualisation process.
1. Repeat Steps # 1 and 2 from Scenario 1 above.
2. After clicking **Next**, enable "_Track token usage_" API-M policy. Select an existing Application Insights instance to log token metrics into and add dimensions that you want the metrics to be grouped by:
![APIM - Enabling Usage policy](/images/apim_usage_config.png)
> _Note_: The provided [Jupyter notebook](AOAI_APIM_Usage_Analysis.ipynb) assumes that you have added **Subscription ID** as one of the logging dimensions.
3. Once you click the **Create** button, a new set of APIs will be provisioned to support interactions with various AOAI models. API-M will also add the token usage metrics policy to all new API operations. Technical aspects of this policy can be found in [this reference document](https://learn.microsoft.com/en-gb/azure/api-management/azure-openai-emit-token-metric-policy):
``` XML
<policies>
    <inbound>
        <set-backend-service id="apim-generated-policy" backend-id="aoai-usage-by-cx-openai-endpoint" />
        <azure-openai-emit-token-metric namespace="AzureOpenAI">
            <dimension name="Subscription ID" value="@(context.Subscription.Id)" />
        </azure-openai-emit-token-metric>
        <authentication-managed-identity resource="https://cognitiveservices.azure.com/" />
        <base />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```
4. If you want to log and visualise tokens usage, ensure that you set the following 5 environment variables before running the notebook:
![APIM - Setting Usage environment variables](/images/apim_usage_env_var.png)

| Environment Variable | Description |
| --- | --- |
| _APIM_USAGE_AOAI_DEPLOY_ | Name of AOAI deployment |
| _APIM_USAGE_API_VERSION_ | API version of AOAI endpoint |
| _APIM_USAGE_KEY_CONTOSO_ | Subscription key, created for Contoso client |
| _APIM_USAGE_KEY_NORTHWIND_ | Subscription key, created for Northwind client |
| _APIM_USAGE_URL_ | URL of provisioned API-M's API for AOAI endpoint |

5. You can now generate workload with a degree of randomness for both Contoso and Northwind clients, connected to the same Azure OpenAI deployment:
``` Python
for key in SUBSCRIPTION_KEYS:
    randomness = random.randint(0, 5)
    for i in range(NUMBER_OF_RUNS - randomness):    
        start_time = time.time()
        response = get_rest_completion(subscription_key=key, system_prompt=SYSTEM_PROMPT, user_prompt=USER_PROMPT)
        end_time = time.time()
        print(f"Run # {i} completed in {end_time - start_time:.2f} seconds with response code {response.status_code}")
    
        if i < NUMBER_OF_RUNS - 1:
            print(f"Pausing for {SLEEP_TIME} seconds...")
            time.sleep(SLEEP_TIME)
    print("-----------------------------")
```
6. Collected token usage logs can be visualised in Application Insights charts, e.g. the total tokens split by Subscription IDs as shown below:
![APIM - Visualising usage stats](/images/apim_usage_chart.png)

## Scenario 3: Load-balancing between several AOAI endpoints
This section describes setting up API-M and then performing end-to-end testing of an AOAI load-balancing scenario.
1. For each backend AOAI endpoint, you can configure _circuit breaker_ logic using API-M's REST API. Such logic determines when to temporarily stop sending requests to an unhealthy endpoint. The provided **LoadBalancer_CircuitBreaker.json** can be re-used as a jump-start template, where you trip the circuit breaker for **30 seconds** if the AOAI endpoint returns _429_ (Too Many Requests) or _5xx_ (server errors) within any **2-second** interval.
``` JSON
{
    "properties": {
        "description": "<DESCRIPTION>",
        "title": "<TITLE>",
        "type": "Single",
        "protocol": "http",
        "url": "<URL>",
        "circuitBreaker": {
            "rules": [
                {
                    "failureCondition": {
                        "count": 1,
                        "interval": "PT2S",
                        "statusCodeRanges": [
                            {
                                "min": 429,
                                "max": 429
                            },
                            {
                                "min": 500,
                                "max": 599
                            }
                        ]
                    },
                    "name": "<NAME>",
                    "tripDuration": "PT30S",
                    "acceptRetryAfter": true
                }
            ]
        }
    }
}
```
> Note: At the time of writing, configuring circuit breakers directly within the Azure portal UI for API-M was not supported.
2. You can combine your backend AOAI endpoints into a load-balancing pool, using either **round-robin**, **weight-based** or **priority-based** logic. The provided ```LoadBalancer_Pool.json``` can be re-used as a jump-start template to configure such a pool.
``` JSON
{
    "properties": {
        "description": "<DESCRIPTION>",
        "title": "<TITLE>",
        "type": "Pool",
        "pool": {
            "services": [
                {
                    "id": "<BACKEND_1>",
                    "priority": 1
                },
                {
                    "id": "<BACKEND_2>",
                    "priority": 2
                }
            ]
        }
    }
}
```
> Note: At the time of writing, configuring load-balancing pools directly within the Azure portal UI for API-M was not supported.
3. If you want to test load-balancing between your defined AOAI endpoints, ensure that you set the following 4 environment variables before running the provided [Jupyter notebook](AOAI_APIM_Load_Balance.ipynb):
![APIM - Setting Usage environment variables](/images/apim_usage_env_var.png)

| Environment Variable | Description |
| --- | --- |
| _APIM_LB_AOAI_DEPLOY_ | Name of AOAI deployment |
| _APIM_LB_API_VERSION_ | API version of AOAI endpoint |
| _APIM_LB_SUB_KEY_ | Subscription key, created for load-balancing API-M endpoint |
| _APIM_LB_URL_ | URL of load-balancing API-M's API for AOAI endpoint |

4. Consider a use-case where you configure an AOAI deployment of GPT-4o in Sweden Central with an ultra-low Token-per-Minute (TPM) quota of 1K. You then load-balance it with a higher TPM quota GPT-4 deployment in France Central. Your test results might be similar to what is shown below, with successful routing to the France Central endpoint when the circuit breaker trips for the Sweden Central endpoint:
``` JSON
Run # 0: Sweden Central, Duration: 0.84, Response Code: 200
Pausing for 2 seconds...
Run # 1: None, Duration: 1.23, Response Code: 503
Pausing for 2 seconds...
Run # 2: France Central, Duration: 2.57, Response Code: 200
Pausing for 2 seconds...
Run # 3: France Central, Duration: 1.94, Response Code: 200
Pausing for 2 seconds...
Run # 4: France Central, Duration: 1.97, Response Code: 200
Pausing for 2 seconds...
Run # 5: France Central, Duration: 2.18, Response Code: 200
Pausing for 2 seconds...
Run # 6: France Central, Duration: 1.72, Response Code: 200
Pausing for 2 seconds...
Run # 7: France Central, Duration: 2.17, Response Code: 200
Pausing for 2 seconds...
Run # 8: France Central, Duration: 1.99, Response Code: 200
Pausing for 2 seconds...
Run # 9: Sweden Central, Duration: 0.88, Response Code: 200
``` 
