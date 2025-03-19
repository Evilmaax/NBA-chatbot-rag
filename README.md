# NBA records chatbot using RAG 🏀🏀⛹🏾‍♂️

Welcome to my chatbot project that uses RAG to inform you about the most impressive records on the league!

image usage

In a nutshell, this repository showcases a fully operational Retrieval-Augmented Generation pipeline using Firecrawl, Langchain, Open AI, Pinecone, Firebase and Redis tools. All of that orchestrated on a local Kubernetes cluster. 

Image:

Below the pieces used to make everything works: 

* `Firecrawl` – Once a week a Kubernetes CronJob triggers Firecrawl, extracting data from the sources.
* `OpenAI` – Each chunk of data is converted into a numerical representation (embedding) that captures semantic meaning throug an OpenAI embedding model.
* `Pinecone Vector Store` – The embeddings are sent to Pinecone for storage, allowing fast and accurate retrieval based on semantic search.
* `Langchain` – With all that set, Langchain is used to create the runnable chains, which can either directly return an answer from Vector Store ou Cache, based on similarity, or call the LLM when the question is never seen.
* `Redis Cache` – As pointed out, everytime a question arrives and user as a caching layer to quickly respond to repeated queries without recomputing results.
* `OpenAI` – Used to process the scrapped data and turn into valuable content. Also, it is used on the other  front of the solution taking the retrieved context from Pinecone and crafting the final answer to user queries.
* `Firebase`*

* `Kubernetes` – All components are deployed in a local Kubernetes cluster.

## How It Works in a high level

Once in a week a cronjob is deployed and go to the sources to scrap the updated content about the records using [`Firecrawl`](https://github.com/mendableai/firecrawl). The result are them separated into chunks that keep the meaning of the records: `holder`, `record`, `date` and so on. 

Being ready the chunks, its time to embedding: convert them into numerical representation that captures semantic meaning. That part is done using an OpenAI embedding model. These vectors are them stored in Pinecone Vector Store, which allows fast and accurate retrieval based on similarity search.

This way, when a user send a question the path is: embed the question into a numerical vector, perform the a vector search based on similarity. If the top result is above a threshold, return it directly, otherwise, go to the next step.

The next step is perform a search on a cache layer that uses Redis. If found it based on a similarity threshold, return it immediatly, otherwise call the LLM. 

ok, so we cannot spare the tokens consumption.. it happens, but glad we have a cool `Langchain` runnable path that is either able to return a similar embedding or a previous answered question from cache and call a LLM model passing the context retrieved from vector store. 

Using that, the model generates a well-informed answer and send it back, not forgeting to update the cache future queries.

## Usage (Windows Example)

To run the project it is pretty simple
powershell
Copy
Edit
# deploy-k8s.ps1

# 1. Build Docker images
docker build -t myorg/firecrawl:latest ./firecrawl
docker build -t myorg/chatbot:latest ./chatbot

# 2. Start local Kubernetes (Docker Desktop)
# Make sure Kubernetes is enabled in Docker Desktop settings

# 3. Create namespaces
kubectl create namespace rag-system

# 4. Deploy Redis
kubectl apply -f redis-deployment.yaml -n rag-system

# 5. Deploy Pinecone or your local vector DB alternative
kubectl apply -f pinecone-deployment.yaml -n rag-system

# 6. Deploy Firecrawl CronJob
kubectl apply -f firecrawl-cronjob.yaml -n rag-system

# 7. Deploy Chatbot and other services
kubectl apply -f chatbot-deployment.yaml -n rag-system

# 8. Check if everything is running
kubectl get pods -n rag-system
Breakdown of the Script
docker build -t myorg/firecrawl:latest ./firecrawl

Builds the Docker image for the Firecrawl service from the Dockerfile located in the firecrawl directory and tags it as myorg/firecrawl:latest.
docker build -t myorg/chatbot:latest ./chatbot

Builds the Docker image for the chatbot service from the Dockerfile in the chatbot directory, tagging it as myorg/chatbot:latest.
(Comment) "Make sure Kubernetes is enabled in Docker Desktop settings"_

Reminds you to enable Kubernetes in Docker Desktop, ensuring you have a local Kubernetes cluster ready.
kubectl create namespace rag-system

Creates a dedicated namespace called rag-system to keep everything tidy and avoid naming collisions with other deployments.
kubectl apply -f redis-deployment.yaml -n rag-system

Deploys Redis using the configuration file redis-deployment.yaml.
kubectl apply -f pinecone-deployment.yaml -n rag-system

Deploys Pinecone (or your vector database alternative) with the specified configuration.
kubectl apply -f firecrawl-cronjob.yaml -n rag-system

Schedules Firecrawl to run once a week for data scraping and ingestion into the vector store.
kubectl apply -f chatbot-deployment.yaml -n rag-system

Deploys your main chatbot service, which will handle user queries, perform retrieval from Pinecone, and cache with Redis.
kubectl get pods -n rag-system

Verifies that all pods are running correctly in the rag-system namespace.
Run Locally (macOS Alternative)
If you’re on macOS, simply open the Terminal app instead of PowerShell. The commands are virtually the same:

bash
Copy
Edit
# Build Docker images
docker build -t myorg/firecrawl:latest ./firecrawl
docker build -t myorg/chatbot:latest ./chatbot

# Enable Kubernetes in Docker Desktop and then:
kubectl create namespace rag-system

# Deploy everything
kubectl apply -f redis-deployment.yaml -n rag-system
kubectl apply -f pinecone-deployment.yaml -n rag-system
kubectl apply -f firecrawl-cronjob.yaml -n rag-system
kubectl apply -f chatbot-deployment.yaml -n rag-system

# Check pods
kubectl get pods -n rag-system
Everything else remains the same. Just ensure you have Docker Desktop with Kubernetes support enabled.

Contributing and Future Plans
Scalability: As your data grows, you can scale up Pinecone and Redis deployments right in Kubernetes.
Customization: Swap out Firecrawl with another scraping tool or even schedule more frequent CronJobs.
Extensions: Integrate with other LLMs or refine the entire pipeline for domain-specific contexts.
We hope this readme helps you set up and understand the core idea behind a modern RAG pipeline. Feel free to open issues or pull requests for any improvements!







