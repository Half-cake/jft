<div style="background-color: #FFFFFF;">

```mermaid
  %%{init: { 'theme':'default' } }%%

  sequenceDiagram

    actor Bob

    participant DWH as DWH
    participant AA as AdminAPI
    participant S as Sidekiq
    participant V as Vault
    participant P as Provider

    Bob->>AA: Add source

    AA->>+S: Create refresh token job

    loop Based on expired_at periodical job
        S->>P: refresh access token
        activate P
        P-->>S: refreshed access token
        deactivate P
        activate V
        S->>V: put new version of access token
        deactivate V
    end

    loop Gathering data Job
        DWH->>V: Get access_token
        V-->>DWH: access_key
        DWH->>P: Gathering data
        break when access_token expired
            activate AA
            DWH->>+AA: /refresh?provider
            AA-->>+P: refreshed access token
            activate P
            P-->>AA: refreshed access token
            deactivate P
            AA-->>+DWH: HTTP 201
            activate V
            AA->>V: add new version of access token
            V->>AA: HTTP 201
            deactivate V
            deactivate AA
        end
    end
```

</div>
