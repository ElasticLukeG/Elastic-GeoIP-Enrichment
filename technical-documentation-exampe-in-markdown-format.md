# Index Lifecycle Management

- [Index Lifecycle Management](#index-lifecycle-management)
  - [Overview](#overview)
  - [Phases](#phases)
  - [Beats](#beats)
    - [Lifecycle Policy](#lifecycle-policy)
    - [Component Template](#component-template)
    - [Index Template](#index-template)
  - [Agent](#agent)
    - [Naming Scheme](#naming-scheme)
    - [Lifecycle Policy](#lifecycle-policy-1)
    - [Component Template](#component-template-1)
    - [Index Template](#index-template-1)

## Overview

Index Lifecycle Management(ILM) policies define the overall retention of data in a cluster and also the different actions that can be performed in different phases as your data moves through different data tiers.

There are 5 phases in ILM of hot, warm, cold, frozen, and delete. Read more about these phases [here](#phases).

ILM will move the indices through the lifecycle based on its age and if all actions in the current phase have been completed. Read more about these actions [here](https://www.elastic.co/guide/en/elasticsearch/reference/master/ilm-actions.html).

>Each phase will only do certain actions. No phase can do all actions.

We will only be focusing on rollover and the delete actions for demonstration purposes.

## Phases

Hot: The index is actively being updated and queried.

Warm: The index is no longer being updated but is still being queried.

Cold: The index is no longer being updated and is queried infrequently. The information still needs to be searchable, but it’s okay if those queries are slower.

Frozen: The index is no longer being updated and is queried rarely. The information still needs to be searchable, but it’s okay if those queries are extremely slow.

Delete: The index is no longer needed and can safely be removed.

## Beats

In this example we will update the `filebeat` index template to include a custom component template that has our new lifecycle policy

### Lifecycle Policy

Our lifecycle policy will rollover after a `max_age` of 15 days and delete with a `min_age` of 0.

> `min_age` is the amount of time since the rollover. We set this to 0 becuase we want it to delete immediately after the rollover

All steps will be completed in Dev Tools which is located in Kibana

To navigate to Dev Tools go to **Management** **&rarr;** **Dev Tools**

Copy and paste this policy which is named `delete_policy` and submit the request to Elasticsearch using the green triangle button.

```json
PUT _ilm/policy/delete_policy
{
  "policy": {
    "phases": {
      "hot": {                                
        "actions": {
          "rollover": {
            "max_age": "15d"
          }
        }
      },
      "delete": {
        "min_age": "0",                     
        "actions": {
          "delete": {}                        
        }
      }
    }
  }
}
```

### Component Template

Now we need to create a component template that will use our new policy

Copy and paste this component template named `delete_comp_template` in Dev Tools and submit the request to Elasticsearch using the green triangle button.

```json
PUT _component_template/delete_comp_template
{
  "template": {
    "settings": {
      "lifecycle": {
        "name": "delete_policy"
      }
    }
  }
}
```

### Index Template

Now we need to copy the existing filebeat index template to update it

Copy and paste this `GET` request in Dev Tools and submit the request to Elasticsearch using the green triangle button.

```text
GET _index_template/filebeat-*?flat_settings=true
```

The response is the index template enclosed in `"index_template": {}`. Only copy the contents inside the `index_template` field

What you copy will look similar to this:

>This template is very large so this example removes the contents in fields `mappings` and `index.query.default_field`

```json
{
  "index_patterns": [
    "filebeat-*"
  ],
  "template": {
    "settings": {
      "index.lifecycle.name": "filebeat",
      "index.mapping.total_fields.limit": "10000",
      "index.max_docvalue_fields_search": "200",
      "index.number_of_shards": "1",
      "index.query.default_field": [],
      "index.refresh_interval": "5s"
    },
    "mappings": {}
  },
  "composed_of": [],
  "priority": 150,
  "data_stream": {
    "hidden": false,
    "allow_custom_routing": false
  }
}
```

Now that we have our index template that we want to update we need to add our component template.

The field `composed_of` is an array of component templates. This is where we will add our component template `delete_comp_template`

Take note of the value of the `priority` field

Here we have a value of `150` and the new index template we add needs to take precedence over it so we must have a higher priority number.

We also need to remove the existing field `index.lifecycle.name`

Copy and paste our new index template called `filebeat-new` in Dev Tools and submit the request to Elasticsearch using the green triangle button.

```json
PUT _index_template/filebeat-new
{
  "index_patterns": [
    "filebeat-*"
  ],
  "template": {
    "settings": {
      "index.mapping.total_fields.limit": "10000",
      "index.max_docvalue_fields_search": "200",
      "index.number_of_shards": "1",
      "index.query.default_field": [],
      "index.refresh_interval": "5s"
    },
    "mappings": {}
  },
  "composed_of": ["delete_comp_template"],
  "priority": 200,
  "data_stream": {
    "hidden": false,
    "allow_custom_routing": false
  }
}
```

## Agent

In this example we will configure the `metrics-system.network` data stream in the default space to include a custom component template that has our new lifecycle policy

### Naming Scheme

Elasticsearch has 4 built-in index templates with a priority of 100-200

- logs-\*-\*
- metrics-\*-\*
- synthetics-\*-\*
- profiling-*

We will need to set our priority higher than 200 to avoid overriding those templates

Our index template name will have to follow the datastream naming scheme which is `<type>-<dataset>-<namespace>`

The `type` is one of the 4

- logs
- metrics
- traces
- synthetics

The `data` is the integration the agent is using so in our example its `system.network`

The `namespace` is the namespace where the template will be enforced and in this example we will be using the `default` space

Finally all custom component templates must end with `@custom`

So our component template will be named `metrics-system.network-default@custom` and index template `metrics-system.network-default`

### Lifecycle Policy

Our lifecycle policy will rollover after a `max_age` of 15 days and delete with a `min_age` of 0.

> `min_age` is the amount of time since the rollover. We set this to 0 becuase we want it to delete immediately after the rollover

All steps will be completed in Dev Tools which is located in Kibana

To navigate to Dev Tools go to **Management** **&rarr;** **Dev Tools**

Copy and paste this policy which is named `delete_policy` and submit the request to Elasticsearch using the green triangle button.

```json
PUT _ilm/policy/delete_policy
{
  "policy": {
    "phases": {
      "hot": {                                
        "actions": {
          "rollover": {
            "max_age": "15d"
          }
        }
      },
      "delete": {
        "min_age": "0",                     
        "actions": {
          "delete": {}                        
        }
      }
    }
  }
}
```

### Component Template

Now we need to create a component template that will use our new policy

Copy and paste this component template named `metrics-system.network-default@custom` in Dev Tools and submit the request to Elasticsearch using the green triangle button.

```json
PUT _component_template/metrics-system.network-default@custom
{
  "template": {
    "settings": {
      "lifecycle": {
        "name": "delete_policy"
      }
    }
  }
}
```

### Index Template

Now we need to copy the existing `metrics-system.network` index template to update it

Copy and paste this `GET` request in Dev Tools and submit the request to Elasticsearch using the green triangle button.

```text
GET _index_template/metrics-system.network?flat_settings=true
```

The response is the index template enclosed in `"index_template": {}`. Only copy the contents inside the `index_template` field

What you copy will look similar to this:

>This template is very large so this example removes the contents in fields `mappings` and `index.query.default_field`

```json
{
  "index_patterns": ["metrics-system.network-*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "metrics",
      "index.mapping.total_fields.limit": "10000",
      "index.max_docvalue_fields_search": "200",
      "index.number_of_shards": "1",
      "index.query.default_field": [],
      "index.refresh_interval": "5s"
    },
    "mappings": {}
  },
  "composed_of": [],
  "priority": 200,
  "data_stream": {
    "hidden": false,
    "allow_custom_routing": false
  }
}
```

Now that we have our index template that we want to update we need to add our component template.

The field `composed_of` is an array of component templates. This is where we will add our component template `metrics-system.network-default@custom`

Take note of the value of the `priority` field

Here we have a value of `250` and the new index template we add needs to take precedence over it so we must have a higher priority number.

We also need to remove the existing field `index.lifecycle.name`

Copy and paste our new index template called `metrics-system.network-default` in Dev Tools and submit the request to Elasticsearch using the green triangle button.

```json
PUT _index_template/metrics-system.network-default
{
  "index_patterns": [
    "metrics-system.network-default-*"
  ],
  "template": {
    "settings": {
      "index.mapping.total_fields.limit": "10000",
      "index.max_docvalue_fields_search": "200",
      "index.number_of_shards": "1",
      "index.query.default_field": [],
      "index.refresh_interval": "5s"
    },
    "mappings": {}
  },
  "composed_of": ["metrics-system.network-default@custom"],
  "priority": 250,
  "data_stream": {
    "hidden": false,
    "allow_custom_routing": false
  }
}
```
