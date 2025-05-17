# MCP Server for Crossplane Service Management

## Overview

This project is an MCP (Model Context Protocol) server designed to manage Crossplane services. The server provides a programmatic interface for an AI agent to interact with a Kubernetes cluster. It primarily targets Crossplane Claims, defaulting to the `devopstoolkit.live` API group for operations like creation, observation, and deletion, but allows for this API group to be overridden by the agent/user.

The main goal is to abstract the complexities of direct Kubernetes and Crossplane interactions, enabling a conversational AI to guide users through service management tasks seamlessly.

## Project Goals

*   To provide a stable and reliable API for managing Crossplane Claims (defaulting to `devopstoolkit.live`, but configurable).
*   To simplify the process of service creation, observation, and deletion for end-users via an AI agent.
*   To centralize the logic for interacting with Kubernetes and Crossplane for these specific tasks.
*   To enable an AI agent to effectively follow instructions for service management (e.g., based on `commands/create-service.md`, `commands/observe-service.md`, `commands/delete-service.md`).
*   Deliver an MVP (Minimum Viable Product) focusing initially on a single core service (a simplified "Observe Service" functionality - listing Claim names and namespaces for the configured API group) with the smallest possible scope, followed by iterative improvements and expansion to other services.

## Current Status

*   **Language:** Go
*   **Initial MVP Focus:** Basic "Observe Service" functionality (listing Claim names and namespaces).
    *   API Endpoint: `ListClaimsBasic(apiGroup?: string)`

*(Further details can be found in the PRD: `mcp_server_prd.md`)* 