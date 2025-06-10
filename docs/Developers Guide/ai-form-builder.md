# AI form builder

## Intro

This is a closed-source feature that allows you to build forms based on the Questionnaire resource using AI tools.

If you are interested in trying this tool, please contact us in one of the available [ways](../contact-us.md)


## Set up guide

- Obtain a deployment token to pull the Docker image from the private registry.

- Set up environment variables for the container:
    - OPENAI_API_KEY: required   
    - OPENAI_GPT_MODEL. OpenAI GPT model, `gpt-4o` by default   
    - TOKEN_VALIDATION_URL: required. Endpoint for checking auth token. Usually userinfo endpoint of your identity server (e.g "identity-server-url/auth/userinfo")   
    - APP_PORT: the port which the application is listening to. Optional. Default is 3002

- Update EMR configuration file:
    - Set the `aiQuestionnaireBuilderUrl` variable in the `contrib/emr-config/config.production.js` file to the public URL where the AI builder can be accessed.
