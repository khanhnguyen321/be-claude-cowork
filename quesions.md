## 9. Potential "Deep Dives" Questions

- **On Vector DB:** "Why Pinecone over pgvector?" (Answer: Managed service simplicity for a startup vs. operational overhead of RDS).
- **On Cold Starts:** "How do we handle Lambda cold starts for a chat app?" (Answer: Provisioned Concurrency for the core API or keeping functions 'warm' via EventBridge pings).
- **On Privacy:** "Can we use Claude without our data training their model?" (Answer: Yes, by using AWS Bedrock, data is NOT used to train the base models).