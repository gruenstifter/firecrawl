# Debugging Plan: Firecrawl Timeout Issues

The goal of this plan is to identify and resolve the network connectivity issues causing timeouts when scraping with Firecrawl managed by Coolify.

**Phase 1: Information Gathering and Initial Checks**

1.  **Review Timeout Details:** The user has provided details of a 408 Request Timeout when calling `http://host.docker.internal:3002/v1/scrape` from an n8n Docker container. This indicates a problem with the n8n container reaching the Firecrawl API.
2.  **Examine `docker-compose.coolify.yaml`:** (Completed) Reviewed the `docker-compose.coolify.yaml` file to understand the Firecrawl service setup and networking. The `firecrawl-api` is on the `coolify` network and exposed on port 3002.
3.  **Verify Firecrawl API Status:**
    *   **Action:** Check if the `firecrawl-api` container is running and healthy within Coolify.
    *   **Action:** From the host machine, attempt to access the Firecrawl API directly using `curl http://localhost:3002/v1/health`. This verifies if the API is accessible from the host.

**Phase 2: Debugging Network Connectivity from within the n8n Container**

1.  **Access the n8n Container:**
    *   **Action:** Get the container ID or name of the running n8n container.
    *   **Action:** Execute a shell within the n8n container (e.g., `docker exec -it <n8n_container_id> /bin/bash`).
2.  **Test Connectivity to Firecrawl API:**
    *   **Action:** From within the n8n container, use `ping host.docker.internal` to see if the host machine is reachable.
    *   **Action:** From within the n8n container, use `curl http://host.docker.internal:3002/v1/health` to test if the Firecrawl API is reachable and responding.
    *   **Action:** If `host.docker.internal` is not resolving or reachable, try accessing the Firecrawl API using the host machine's actual IP address on the Docker network (if known) or the IP address assigned to the `firecrawl-api` container on the `coolify` network (if the n8n container can be connected to that network).

**Phase 3: Inspecting Docker and Host Network Configuration**

1.  **Inspect Docker Networks:**
    *   **Action:** On the host machine, use `docker network ls` to list all Docker networks. Identify the `coolify` network and the network the n8n container is attached to.
    *   **Action:** Use `docker network inspect <network_name>` for both networks to examine their configurations, including subnets, gateways, and connected containers.
2.  **Inspect n8n Container Network Configuration:**
    *   **Action:** On the host machine, use `docker inspect <n8n_container_id>` to view the detailed configuration of the n8n container, focusing on the "NetworkSettings" section to see which network(s) it's connected to and its IP address(es).
3.  **Inspect Host Network Configuration:**
    *   **Action:** On the host machine, examine firewall rules (e.g., `iptables -L` or `ufw status`) to ensure traffic on port 3002 is not being blocked.
    *   **Action:** Check the host machine's network interfaces and routing table (e.g., `ip addr show`, `ip route show`) to understand how traffic is routed.

**Phase 4: Analyzing Logs**

1.  **Examine Firecrawl API Logs:**
    *   **Action:** View the logs of the `firecrawl-api` container for any errors or indications of incoming connection attempts from the n8n container's IP address.
2.  **Examine Coolify Logs:**
    *   **Action:** Check Coolify's logs for any relevant error messages related to the Firecrawl service or networking.

**Phase 5: Potential Solutions**

Based on the findings from the previous phases, potential solutions could include:

*   Connecting the n8n container to the same `coolify` network as the Firecrawl API.
*   Adjusting firewall rules on the host machine.
*   Configuring n8n to use the correct network address or service name to reach the Firecrawl API.
*   Investigating potential issues with `host.docker.internal` resolution in the specific Docker environment.

**Diagram:**

```mermaid
graph TD
    A[n8n Docker Container] -->|Attempts to connect via host.docker.internal:3002| B(Host Machine)
    B -->|Routes traffic?| C[Firecrawl API Docker Container]
    C -->|Responds?| B
    B -->|Routes response?| A

    A -->|Direct connection if on same network?| C

    subgraph Docker Network
        A
        C
    end