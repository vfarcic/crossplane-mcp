# PRD: MCP Server for Crossplane Service Management

## 1. Introduction
This document outlines the requirements for an MCP (Model Context Protocol) server designed to manage Crossplane services. The server will provide a programmatic interface for an AI agent to interact with a Kubernetes cluster. It will primarily target Crossplane Claims, defaulting to the `devopstoolkit.live` API group for operations like creation, observation, and deletion, but will allow for this API group to be overridden by the agent/user. This server aims to abstract the complexities of direct Kubernetes and Crossplane interactions, enabling a conversational AI to guide users through service management tasks.

## 2. Goals
*   To provide a stable and reliable API for managing Crossplane Claims (`devopstoolkit.live`).
*   To simplify the process of service creation, observation, and deletion for end-users via an AI agent.
*   To centralize the logic for interacting with Kubernetes and Crossplane for these specific tasks.
*   To enable an AI agent to effectively follow instructions outlined in `commands/create-service.md`, `commands/observe-service.md`, and `commands/delete-service.md`.
*   Deliver an MVP (Minimum Viable Product) focusing initially on a single core service (a simplified "Observe Service" functionality) with the smallest possible scope, followed by iterative improvements and expansion to other services.

## 3. Target Audience
*   **Primary Consumer:** AI Agent (e.g., a conversational AI assistant).
*   **Indirect Users:** Platform engineers, developers, or SREs who will interact with the AI agent to manage services.

## 4. Scope & Features
**Note:** The initial MVP will be extremely focused, targeting only the "Observe Service" functionality in its most basic form (e.g., listing available service names). "Create Service" and "Delete Service" functionalities, as well as advanced features within "Observe Service" (like detailed managed resource views and full status aggregation), are planned for subsequent iterations.

### 4.1 Core Functionalities
The MCP server MUST implement the underlying capabilities to support the workflows described in the `commands` directory. This includes:

*   **Service Creation (`create-service.md`) - Post-MVP for initial release:**
    *   Discover available Crossplane CRDs (filtered for Claims, using the configured API group which defaults to `devopstoolkit.live`).
    *   Provide CRD schema information (metadata, spec) to the calling agent for a CRD within the configured API group.
    *   List available Compositions for a given CRD.
    *   List existing Namespaces (filtered for `team` if specified).
    *   Receive parameters from the agent and generate a valid Kubernetes YAML manifest for the Claim.
    *   Select appropriate Compositions using `spec.crossplane.compositionSelector.matchLabels`.
    *   Optionally create associated Kubernetes Secrets (e.g., for SQL passwords), except for specific providers like UpCloud.
    *   Return the generated manifest(s) to the agent.

*   **Service Observation (`observe-service.md`):**
    *   **MVP Scope:** 
        *   Discover and list existing Crossplane Claims (names and namespaces only) associated with the configured API group (defaults to `devopstoolkit.live`).
    *   **Post-MVP for initial release (full functionality):**
        *   For a user-selected Claim (identified from the MVP list or by name/namespace provided by the agent, within the configured API group):
            *   Retrieve and list all directly and indirectly managed Kubernetes resources.
            *   Provide detailed status information for the selected Claim and each of its managed resources.
            *   Return a structured overview of the service and its components to the agent.
            *   Allow the agent to request more detailed information on specific managed resources.

*   **Service Deletion (`delete-service.md`) - Post-MVP for initial release:**
    *   Discover existing Crossplane Claims (within the configured API group, defaulting to `devopstoolkit.live`) that can be targeted for deletion.
    *   Allow the agent to specify one or more Claims to delete (from the configured API group).
    *   For each selected Claim, identify all associated Kubernetes resources that will be removed as a consequence of its deletion.
    *   Provide this list of impacted resources to the agent for user confirmation.
    *   Upon confirmation, support deletion of the Claim resource (e.g., by returning a `kubectl delete` command or by directly performing the deletion if the server evolves to support this and has appropriate permissions).
    *   Consider options for deleting associated local project manifests versus direct cluster resource deletion, as per the original `delete-service.md` prompt.

### 4.2 API Design (High-Level):
*   The server should expose a clear, versioned API (e.g., gRPC or REST/HTTP).
*   Requests and responses should use structured data formats (e.g., JSON, Protobuf).
*   API endpoints will correspond to logical operations:
    *   **MVP Endpoint:**
        *   `ListClaimsBasic(apiGroup?: string)`: Retrieves a list of Claim names and their namespaces. Defaults to `devopstoolkit.live` if `apiGroup` is not provided.
    *   **Post-MVP Endpoints (for initial release):**
        *   `GetClaimDetails(claimName: string, namespace: string, apiGroup?: string)`: Retrieves detailed information. Defaults `apiGroup` if not provided.
        *   `ListCRDs(apiGroup?: string)`: Discovers and lists available Crossplane CRDs (filtered for Claims). Defaults `apiGroup`.
        *   `GetCRDSchema(crdName: string, apiGroup?: string)`: Provides the schema for a specified CRD. Defaults `apiGroup`.
        *   `ListCompositions(crdName: string, apiGroup?: string)`: Lists available Compositions. Defaults `apiGroup`.
        *   `ListNamespaces`: Lists existing Kubernetes Namespaces (with optional filtering, e.g., for `team`).
        *   `GenerateClaimManifest(parameters: object, apiGroup?: string)`: Generates a Claim manifest. Defaults `apiGroup`.
        *   `GenerateSecretManifest(parameters: object)`: Generates a Secret manifest.
        *   `GetDeletionImpact(claimName: string, namespace: string, apiGroup?: string)`: Returns impact of deletion. Defaults `apiGroup`.
        *   (Potentially, if direct cluster modification is added later: `CreateResource`, `DeleteResource`)

### 4.3 Kubernetes/Crossplane Interaction:
*   The server will use appropriate Kubernetes client libraries (e.g., `client-go` if implemented in Go, or `kubernetes-client/python` if in Python).
*   It must have necessary RBAC permissions for read/write operations on relevant Kubernetes resources.

## 5. Non-Functional Requirements
*   **Chosen Language:** Go
*   **Key SDK/Library:** The project will leverage the `mark3labs/mcp-go` SDK (https://github.com/mark3labs/mcp-go) for implementing the Model Context Protocol server functionalities. This SDK is expected to simplify the handling of MCP communication, allowing focus on the core service logic.
*   **Reliability:** Robust error handling for cluster interactions.
*   **Testing:** Comprehensive testing (unit, integration) is required.
*   **Security:** Secure communication with Kubernetes API. Consider authN/authZ for the MCP server's API.
*   **Statelessness:** Prefer stateless design where possible.

## 6. Success Metrics
*   AI agent can successfully execute `create-service`, `observe-service`, and `delete-service` workflows via the MCP server.
*   Reduced complexity for the AI agent in these operations.
*   Positive feedback from indirect users on ease and reliability.

## 7. Future Considerations
*   Direct application of manifests to the cluster by the MCP server.
*   Integration with Git workflows (e.g., creating Pull Requests).
*   Advanced resource discovery and filtering.
*   Support for additional Crossplane resources or other API groups.
*   Caching mechanisms for cluster information. 