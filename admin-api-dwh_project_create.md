<div style="background-color: #FFFFFF;">

```mermaid
  %%{init: { 'theme':'default' } }%%
  sequenceDiagram

    actor Bob

    box Reporting services (AWS)
      participant UI as AdminUI
      participant AA as AdminAPI
      participant S as Sidekiq
    end

    box GCP
      participant IAM as IAM API
      participant BQ as BigQuery
      participant SEC as Secret Manager
    end

    Bob ->> UI: Create project
    UI ->> AA: HTTP POST /projects/create
    activate AA 
    AA ->> IAM: SA create
    activate IAM
    IAM -->> AA: Respond with credentials
    AA ->> IAM: SA permission grant
    IAM -->> AA: 
    AA ->> IAM: Creator's email BQ role grant
    IAM -->> AA: 
    deactivate IAM
    AA ->> SEC: Store credentials (private_key_data)
    activate SEC
    SEC -->> AA: 
    deactivate SEC
    AA ->> BQ: Create raw dataset (raw_<project_id>_general)
    activate BQ
    BQ -->> AA: 
    AA ->> BQ: Creat processing dataset (prod_<project_id>_general)
    BQ -->> AA: 
    deactivate BQ
    AA ->> S: Run ReplicateDataset job
    deactivate AA
    loop Running ReplicateDataset job
        S ->> BQ: Copy structure from service_dataset to processing dataset
        activate BQ
        deactivate BQ
    end
```
</div>