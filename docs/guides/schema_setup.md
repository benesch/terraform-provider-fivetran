---
page_title: "Connector Schema Setup"
subcategory: "Getting Started"
---

# How to set up Fivetran connector schema config using Terraform

In this guide we will set up simple pipeline with one connector and schema using Fivetran Terraform Provider. 

## Create a connector resource

Create `fivetran_connector` resource:

```hcl
resource "fivetran_connector" "connector" {
   ...
   run_setup_tests = "true" # it is necessary to authorise connector
}
```

Connector will be in paused state, but ready to sync.

-> Connector should be **authorized** to be able to fetch schema from source. Set `run_setup_tests = "true"`.

## Set up connector schema config

Let's define what exactly we want to sync using `fivetran_connector_schema_config` resource:

```hcl
resource "fivetran_connector_schema_config" "connector_schema" {
  connector_id = fivetran_connector.connector.id
  schema_change_handling = "BLOCK_ALL"
  schema {
    name = "my_fivetran_log_connector"
    table {
      name = "log"
      column {
        name = "event"
        enabled = "true"
      }
      column {
        name = "message_data"
        enabled = "true"
      }
      column {
        name = "message_event"
        enabled = "true"
      }
      column {
        name = "sync_id"
        enabled = "true"
      }
    }
  }
}
```

## Set up connector schedule configuration

-> Schedule should depend on schema resource to enable connector **after** schema changes apply.

```hcl
resource "fivetran_connector_schedule" "my_connector_schedule" {
    connector_id = fivetran_connector_schema_config.connector_schema.id

    sync_frequency     = "5"

    paused             = false
    pause_after_trial  = true

    schedule_type      = "auto"
}
```

## Apply configuration

```bash
terraform apply
```

## Example configuration

Example .tf file with configuration could be found [here](https://github.com/fivetran/terraform-provider-fivetran/tree/main/config-examples/connector_schema_setup.tf).