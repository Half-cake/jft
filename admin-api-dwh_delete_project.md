<div style="background-color: #FFFFFF;">

```mermaid
  %%{init: { 'theme':'default' } }%%

    sequenceDiagram

    actor Bob

    box Reporting services (AWS)
        participant UI as AdminUI
        participant AA as AdminAPI
        participant S as Sidekiq
        participant DG as Dag Web Service
    end

    box GCP
        participant SEC as Secret Manager
        participant IAM as IAM API
        participant BQ as BQ API
        participant AF as Airflow API
        participant CS as Cloud Storage
    end


    Bob ->> UI: Delete project
    UI ->> AA: HTTP DELETE /projects

    activate AA

    AA ->> S: Run delete project job
    loop Delete project job 
      S ->> DG: HTTP DELETE /delete-service-accounts
      activate DG
      DG ->> IAM: Delete service accounts
      activate IAM
      IAM -->> DG: 
      deactivate IAM
      DG -->> S: 
      deactivate DG
    end

    AA ->> S: Run delete datasets job
    loop Run delete datasets job
      S ->> DG: HTTP DELETE /delete-datasets
      activate DG
      DG ->> SEC: Ungrant access
      activate SEC
      SEC -->> DG: 
      deactivate SEC 
      DG ->> BQ: Delete datasets
      activate BQ
      BQ -->> DG: 
      deactivate BQ
      DG -->> S: 
      deactivate DG
    end

    AA ->> S: Run delete dags job
    loop Run delete dags job
      S ->> DG: HTTP DELETE /delete-dags
      DG -->> S: 
      activate DG
      DG ->> AF: Stop dag
      activate AF
      AF -->> DG: 
      deactivate AF
      DG ->> CS: Delete dag
      activate CS
      CS -->> DG: 
      deactivate CS 
      deactivate DG
    end

        AA ->> S: Run delete buckets job
    loop Run delete buckets job
        S ->>+ DG: HTTP DELETE /delete-bucket
        DG ->>+ CS: Delete bucket
        CS -->>- DG: 
    end

    AA ->> S: Run delete secrets job
    loop Run delete secrets job
        S ->>+ DG: HTTP DELETE /delete_secrets
        DG ->>+ SEC: Delete secrets
        SEC -->>- DG: 
    end

    deactivate AA

    
    



    
    


    

    

```

</div>
