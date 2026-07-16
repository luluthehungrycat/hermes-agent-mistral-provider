# Hermes Agent — Mistral AI Provider

A model provider plugin for [Hermes Agent](https://github.com/NousResearch/hermes-agent) that adds native Mistral AI API support.

## Installation

Copy into your Hermes plugins directory:



Or symlink for automatic updates:



## Setup

Set your Mistral API key in ~/.hermes/.env:



Then run `hermes model` and select **Mistral AI** from the provider list.

## Supported Models

- mistral-large-latest — flagship, tool calling + vision
- mistral-medium-latest — balanced, tool calling + vision
- mistral-small-latest — fast, tool calling + vision + reasoning
- codestral-latest — code generation specialist
- pixtral-12b-latest — vision-focused, cheap

## Features

- Standard OpenAI-compatible chat completions
- Tool/function calling
- Vision (base64 images)
- reasoning_effort pass-through for thinking models
- Auto-registers with Hermes provider discovery

## License

Same as Hermes Agent.
