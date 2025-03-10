<div style="background-color: #FFFFFF;">


```mermaid 
%%{init: { 'theme':'default' } }%%
sequenceDiagram
actor Bob

box "GCP" #LightBlue
    participant AF as Airflow
    participant CF as Cloud Functions
    participant SM as Secret Manager
    participant CS as Cloud Storage
    participant BQ as BigQuery
    participant LS as Looker Studio
end

participant WS as AdminUI

note over AF: Contains Loading DAG and Processing DAG

AF ->> CF: Pass data from Loading DAG, incl. Secret Manager link
activate CF
CF ->> SM: Retrieve token
activate SM
SM -->> CF: Token
deactivate SM
CF ->> API: Request data using token
activate API
API -->> CF: Data
deactivate API
CF ->> CS: Retrieve .avsc schema
activate CS
CS -->> CF: .avro schema
CF ->> CS: Save data in .avro format using .avsc schema
deactivate CS

CF ->> BQ: Create external table from avro in Cloud Storage
deactivate CF

note over AF: Loading DAG triggers Processing DAG
AF ->> BQ: SQL queries from Processing DAG to transform data
BQ -->> BQ: Create native tables

BQ ->> LS: Data for Looker Studio
LS ->> WS: Looker Studio preview
WS -->> Bob: Interact with data preview
```

</div>