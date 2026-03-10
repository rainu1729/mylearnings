
# Dead Letter Queue (DLQ)

A design pattern for isolating records that fail processing or validation so the pipeline can continue running. Failed records are routed to a separate store for later inspection, debugging, or reprocessing.

- Purpose
    - Capture "poison" or malformed records without crashing the job.
    - Preserve original payload and error context for debugging/replay.

- When to use
    - Validation errors (missing required fields, type mismatches)
    - Deserialization or schema evolution failures
    - Repeated processing errors

- Dataflow(Apache Beam) implementations
    - In Google Dataflow, a DLQ is executed using a ParDo(Parallel Do) transform 
    - In ParDo custom function known as a "Do function" processes each individual record
    - The ParDo function code must be fully serializable, idempotent, and thread-safe.
- ApacheSpark serverless 
    - Templates available to apply DLQ pattern.

# Facade View Pattern 

This architectural pattern isolates data consumers from backend changes, ensuring a seamless transition with no downtime

- Configure and deploy an updated version of your pipeline to write the transformed data to a new, separate destination table
- create or update a single VIEW that sits in front of both the old and new tables

# Schema-on-Write vs. Schema-on-Read

## Schema-on-Write
- The schema is strictly enforced when the pipeline tries to write the data.
- Pipelines clean, structure, and validate data before loading it into a data warehouse.
- 
## Schema-on-Read
- Data is stored in the raw format.The structure is flexible schema is applied when the pipeline reads the data.  
- Raw data is loaded directly with minimal transformations.

  
