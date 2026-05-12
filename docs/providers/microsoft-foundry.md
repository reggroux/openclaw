---
summary: "Set up the Microsoft Foundry model provider plugin in OpenClaw"
read_when:
  - You want to use Microsoft Foundry models with OpenClaw
  - You want a GPT or OpenAI-family deployment in Microsoft Foundry
  - You want an OpenAI-compatible open model deployment such as Kimi
title: "Microsoft Foundry"
---

Microsoft Foundry can host model deployments that expose OpenAI-compatible
`/openai/v1` Responses or chat completions APIs. OpenClaw uses those
deployments through the bundled `microsoft-foundry` model provider plugin.

Use this model ref format:

```text
microsoft-foundry/<deployment-name>
```

`<deployment-name>` is the deployment name you created in Microsoft Foundry. It
is not always the catalog model name.

## Two Microsoft Foundry use cases

<Tabs>
  <Tab title="GPT / OpenAI-family deployment">
    Use this for GPT, `o1`, `o3`, `o4`, or `computer-use-preview`
    deployments. The plugin uses the Responses API for these model families.

    <Steps>
      <Step title="Deploy the model">
        In Microsoft Foundry, deploy the OpenAI-family model you want to use.

        Example deployment name:

        ```text
        gpt-5-prod
        ```
      </Step>
      <Step title="Sign in or set the Microsoft Foundry API key">
        For Entra ID:

        ```bash
        az login
        ```

        For Microsoft Foundry API-key auth:

        ```bash
        export AZURE_OPENAI_API_KEY="<microsoft-foundry-api-key>"
        export AZURE_OPENAI_ENDPOINT="https://project-name-resource.services.ai.azure.com/api/projects/openclaw/openai/v1/responses"
        ```
      </Step>
      <Step title="Run Microsoft Foundry onboarding">
        For Entra ID:

        ```bash
        openclaw onboard --auth-choice microsoft-foundry-entra
        ```

        For API-key auth:

        ```bash
        openclaw onboard --auth-choice microsoft-foundry-apikey
        ```
      </Step>
      <Step title="Pick the deployment">
        Select or enter the endpoint and deployment name.

        Target URL from the Microsoft Foundry UI:

        ```text
        https://project-name-resource.services.ai.azure.com/api/projects/openclaw/openai/v1/responses
        ```

        For the example above, the OpenClaw model ref is:

        ```text
        microsoft-foundry/gpt-5-prod
        ```
      </Step>
      <Step title="Verify the provider">
        ```bash
        openclaw models list --provider microsoft-foundry
        ```
      </Step>
    </Steps>

    Or configure it directly. Drop the final `/responses` from the UI target;
    the provider `baseUrl` should end at `/openai/v1`.

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "microsoft-foundry/gpt-5-prod",
          },
          models: {
            "microsoft-foundry/gpt-5-prod": {},
          },
        },
      },
      models: {
        mode: "merge",
        providers: {
          "microsoft-foundry": {
            api: "openai-responses",
            authHeader: false,
            baseUrl: "https://project-name-resource.services.ai.azure.com/api/projects/openclaw/openai/v1",
            headers: {
              "api-key": "<YOUR_API_KEY_HERE>",
            },
            models: [
              {
                id: "gpt-5-prod",
                name: "gpt-5",
                api: "openai-responses",
                reasoning: false,
                input: ["text", "image"],
                contextWindow: 128000,
                maxTokens: 16384,
                cost: {
                  input: 0,
                  output: 0,
                  cacheRead: 0,
                  cacheWrite: 0,
                },
                compat: {
                  supportsStore: false,
                  maxTokensField: "max_completion_tokens",
                },
              },
            ],
          },
        },
      },
    }
    ```

  </Tab>

  <Tab title="Open model deployment, such as Kimi">
    Use this for a model deployment that accepts OpenAI-compatible chat
    completions, such as a Kimi deployment exposed through Microsoft Foundry.
    The plugin uses the chat completions API for non-GPT model families unless
    you choose otherwise.

    <Steps>
      <Step title="Deploy the model">
        Deploy the open model in Microsoft Foundry.

        Example deployment name:

        ```text
        TAMU-Kimi-K2.6
        ```
      </Step>
      <Step title="Set the Microsoft Foundry API key">
        ```bash
        export AZURE_OPENAI_API_KEY="<microsoft-foundry-api-key>"
        export AZURE_OPENAI_ENDPOINT="https://project-name-resource.services.ai.azure.com"
        ```
      </Step>
      <Step title="Run onboarding">
        ```bash
        openclaw onboard --auth-choice microsoft-foundry-apikey
        ```
      </Step>
      <Step title="Choose chat completions">
        When onboarding asks for the request API, choose chat completions.

        Target URL from the Microsoft Foundry UI:

        ```text
        https://project-name-resource.services.ai.azure.com
        ```

        The OpenClaw model ref is:

        ```text
        microsoft-foundry/TAMU-Kimi-K2.6
        ```
      </Step>
      <Step title="Verify the provider">
        ```bash
        openclaw models list --provider microsoft-foundry
        ```
      </Step>
    </Steps>

    Or configure it directly. This follows the known working Kimi shape: the UI
    target is the resource root, and the provider `baseUrl` adds `/openai/v1`.

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "microsoft-foundry/TAMU-Kimi-K2.6",
          },
          models: {
            "microsoft-foundry/TAMU-Kimi-K2.6": {},
          },
        },
      },
      models: {
        mode: "merge",
        providers: {
          "microsoft-foundry": {
            api: "openai-completions",
            authHeader: false,
            baseUrl: "https://project-name-resource.services.ai.azure.com/openai/v1",
            headers: {
              "api-key": "<YOUR_API_KEY_HERE>",
            },
            models: [
              {
                id: "TAMU-Kimi-K2.6",
                name: "Kimi-K2.6",
                api: "openai-completions",
                reasoning: false,
                input: ["text"],
                contextWindow: 128000,
                maxTokens: 16384,
                cost: {
                  input: 0,
                  output: 0,
                  cacheRead: 0,
                  cacheWrite: 0,
                },
                compat: {
                  supportsStore: false,
                  maxTokensField: "max_completion_tokens",
                },
              },
            ],
          },
        },
      },
    }
    ```

  </Tab>
</Tabs>

## Auth choices

| Method                     | Use when                                                            |
| -------------------------- | ------------------------------------------------------------------- |
| `microsoft-foundry-entra`  | You want Entra ID through Azure CLI (`az login`)                    |
| `microsoft-foundry-apikey` | You want Microsoft Foundry API-key auth with `AZURE_OPENAI_API_KEY` |

## API selection

OpenClaw chooses the request API from the deployment metadata when it can.

| Deployment family                                    | API                  |
| ---------------------------------------------------- | -------------------- |
| `gpt-*`, `o1*`, `o3*`, `o4*`, `computer-use-preview` | `openai-responses`   |
| Other OpenAI-compatible chat models                  | `openai-completions` |

If your deployment name is an alias like `prod-primary`, set the saved model
name to the real model family or choose the API explicitly during onboarding.

<AccordionGroup>
  <Accordion title="Deployment names">
    The model ref suffix is the Microsoft Foundry deployment name. If you deploy
    `gpt-5` as `gpt-5-prod`, use `microsoft-foundry/gpt-5-prod`.
  </Accordion>

  <Accordion title="Unsupported APIs">
    This plugin does not support Anthropic Claude deployments that require
    `/anthropic/v1/messages`, Azure AI Model Inference `/models/*` endpoints,
    or deployments that do not support OpenAI-compatible Responses or chat
    completions.
  </Accordion>

</AccordionGroup>

## Related

<CardGroup cols={2}>
  <Card title="Model selection" href="/concepts/model-providers" icon="layers">
    Choosing providers, model refs, and failover behavior.
  </Card>
  <Card title="Plugin reference" href="/plugins/reference/microsoft-foundry" icon="plug">
    Minimal plugin reference for `microsoft-foundry`.
  </Card>
</CardGroup>
