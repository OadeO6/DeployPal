# DeployPal

## Infrastructure Diagram
```mermaid
   graph TB
            %% Physical Servers
            subgraph "Server A"
                rootAPI[Root API]
                database[(Main Database)]
                rootAPI --> database
            end

            subgraph "Server B"
                jenkinsCtrl[Jenkins Controller]
            end

            subgraph "Server C (Agent AA)"
                agentAA[Jenkins Agent Daemon]
                k8sAA[Kubernetes Cluster]
                otherJobAA[Non-K8s Job]
                
                subgraph "AA Deployments"
                    deployAA1["Deployment: project1-aa
                        - Service: project1-aa-svc:80
                        - Containers:
                          • Web: localhost:3000
                          • Proxy: localhost:3000 ➔ 0.0.0.0:80"]
                    
                    deployAA2["Deployment: project2-aa
                        - Service: project2-aa-svc:80
                        - Containers:
                          • Web: localhost:4000
                          • Proxy: localhost:4000 ➔ 0.0.0.0:80"]
                end
                
                traefikAA[Traefik Ingress<br/>Routing Rules:
                    - project1.AA.aiz.com.ng ➔ project1-aa-svc:80
                    - project2.AA.aiz.com.ng ➔ project2-aa-svc:80]
                tunnelAA[Cloudflare Tunnel<br/>*.AA.aiz.com.ng ➔ traefik:80]
            end

            subgraph "Server D (Agent BB)"
                agentBB[Jenkins Agent Daemon]
                k8sBB[Kubernetes Cluster]
                otherJobBB[Non-K8s Job]
                
                subgraph "BB Deployments"
                    deployBB1["Deployment: project1-bb
                        - Service: project1-bb-svc:80
                        - Containers:
                          • Web: localhost:3000
                          • Proxy: localhost:3000 ➔ 0.0.0.0:80"]
                    
                    deployBB2["Deployment: project2-bb
                        - Service: project2-bb-svc:80
                        - Containers:
                          • Web: localhost:4000
                          • Proxy: localhost:4000 ➔ 0.0.0.0:80"]
                end
                
                traefikBB[Traefik Ingress<br/>Routing Rules:
                    - project1.BB.aiz.com.ng ➔ project1-bb-svc:80
                    - project2.BB.aiz.com.ng ➔ project2-bb-svc:80]
                tunnelBB[Cloudflare Tunnel<br/>*.BB.aiz.com.ng ➔ traefik:80]
            end

            %% Connections
            rootAPI -->|Trigger Builds| jenkinsCtrl
            
            jenkinsCtrl -->|Job Commands| agentAA
            jenkinsCtrl -->|Job Commands| agentBB
            
            agentAA -->|K8s Job| k8sAA
            agentAA -->|Non-K8s Job| otherJobAA
            agentBB -->|K8s Job| k8sBB
            agentBB -->|Non-K8s Job| otherJobBB
            
            k8sAA -->|Manages| deployAA1
            k8sAA -->|Manages| deployAA2
            k8sBB -->|Manages| deployBB1
            k8sBB -->|Manages| deployBB2
            
            tunnelAA -->|Wildcard Traffic| traefikAA
            traefikAA -->|project1.AA.aiz.com.ng| deployAA1
            traefikAA -->|project2.AA.aiz.com.ng| deployAA2
            
            tunnelBB -->|Wildcard Traffic| traefikBB
            traefikBB -->|project1.BB.aiz.com.ng| deployBB1
            traefikBB -->|project2.BB.aiz.com.ng| deployBB2
