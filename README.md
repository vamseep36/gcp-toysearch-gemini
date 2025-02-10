# Toy Store Search App

## 1. Set up AlloyDB Database
Follow steps in step 4 in this codelab: https://codelabs.developers.google.com/smart-shop-agent-alloydb#3
Remember to name the cluster and instance as follows:
cluster: vector-cluster
instances: vector-instance
change the user, password, db_name in your Cloud Run Function application code (in the "get-toys-alloydb" Cloud Run Function) according to what you set in this setup step.

### 1. CREATE Script
CREATE TABLE toys ( id VARCHAR(25), name VARCHAR(25), description VARCHAR(20000), quantity INT, price FLOAT, image_url VARCHAR(200), text_embeddings vector(768)) ;

### 2. INSERT Script
INSERT SCRIPTS in the file data.sql in this repo

### 3. Enable Extensions
CREATE EXTENSION vector;
CREATE EXTENSION google_ml_integration;

### 4. Grant Permission
GRANT EXECUTE ON FUNCTION embedding TO postgres;

### 5. Grant Vertex AI User ROLE to the AlloyDB service account

PROJECT_ID=$(gcloud config get-value project)

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:service-$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")@gcp-sa-alloydb.iam.gserviceaccount.com" \
--role="roles/aiplatform.user"

### 6. Update Text Embeddings
UPDATE toys set text_embeddings = embedding( 'text-embedding-005', description);

### 7. Vector Search (RAG)

At this point you are ready to test your Nearest Neighbor query results using the embeddings just created from AlloyDB Studio.
For detailed steps on how to create ScaNN index, refer to this blog: 

    select * from toys
    ORDER BY text_embeddings <=> CAST(embedding('text-embedding-005', 'white plush teddy bear toy with floral pattern') as vector(768))
    LIMIT 5;

### 7. ScaNN Index

The ScaNN index is a tree-based quantization index for approximate nearest neighbor search. In Tree-quantization techniques, indexes learn a search tree together with a quantization (or hashing) function. When you run a query, the search tree is used to prune the search space while quantization is used to compress the index size. This pruning speeds up the scoring of the similarity (i.e., distance) between the query vector (user search text vector) and the database vectors. This results in increased efficiency of the nearest neighbor search. Read more about it [here]([url](https://cloud.google.com/alloydb/docs/ai/tune-indexes)).

   Refer to this blog for creating ScaNN index and executing Vector Search on indexed data: https://medium.com/google-cloud/upgrade-your-vector-search-efficiency-and-recall-with-scann-index-9bc8b2018377

## Gemini 2.0 for image based Vector Search
For this step refer to the callGemini(String base64ImgWithPrefix) of the GeminiCall.java class

## Imagen 3 Implementation
For this step, refer to the generateImage(String projectId, String location, String prompt) method of the generateToy class

## LangChain4j Integration
Integration as part of Gemini 2.0 invocation

## Toolbox Integration

Toolbox helps you build Gen AI tools that let your agents access data in your database. Toolbox provides:

Simplified development: Integrate tools to your agent in less than 10 lines of code, reuse tools between multiple agents or frameworks, and deploy new versions of tools more easily.
Better performance: Best practices such as connection pooling, authentication, and more.
Enhanced security: Integrated auth for more secure access to your data
End-to-end observability: Out of the box metrics and tracing with built-in support for OpenTelemetry.

Toolbox sits between your application's orchestration framework and your database, providing a control plane that is used to modify, distribute, or invoke tools. It simplifies the management of your tools by providing you with a centralized location to store and update tools, allowing you to share tools between agents and applications and update those tools without necessarily redeploying your application.

### Toolbox installation and details:

https://github.com/googleapis/genai-toolbox
[Toolbo](https://pypi.org/project/toolbox-langchain/0.1.0/)
https://github.com/googleapis/genai-toolbox-langchain-python


---
## Serverless Deployment
gcloud run deploy --source .

