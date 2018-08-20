---
title: App AutoScaler API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://github.com/cloudfoundry-incubator/app-autoscaler'>Cloud Foundry App Auto-Scaler</a>

search: true
---

# App AutoScaler

This topic explains how to use the App-AutoScaler with its RESTful API. 

The App Autoscaler API allows you to manage `App Autoscaler` with `curl` from command line simply or in other programming approach. 

If you access `App Autoscaler API` through `shell`, we assume you have installed [Cloud Foundry Command Line][a] which could be used to get Authentication token and Application guid related information. 

# Authentication

> To get the authorization token, use this code:

```shell
token=`cf oauth-token`
curl "api_endpoint_here" \
  -H "Authorization:$token`  
```

`App Autoscaler API` uses the [Cloud Foundry User Token][uaa] to identify the user's information and only responses to the requests raised by authorized Space developers. 


# Policy

## Get policy 

> To get the AutoScaling policy of your application, please use the code below:

```shell
token=`cf oauth-token`
appId=`cf app <Your-App> --guid`
curl "http://example.com/v1/apps/$appId/policy" \
  -X GET \
  -H "Authorization:$token` 
```
> Make sure to replace `Your-App` with your application name.

> The above command returns JSON structured like this, which is built per [App AutoScaler policy specification][policy]

```json
{
  "instance_min_count": 1,
  "instance_max_count": 4,
  "scaling_rules": [
    {
      "metric_type": "memoryutil",
      "breach_duration_secs": 600,
      "threshold": 30,
      "operator": "<",
      "cool_down_secs": 300,
      "adjustment": "-1"
    },
    {
      "metric_type": "memoryutil",
      "breach_duration_secs": 600,
      "threshold": 90,
      "operator": ">=",
      "cool_down_secs": 300,
      "adjustment": "+1"
    }
  ],
  "schedules": {
    "timezone": "Asia/Shanghai",
    "recurring_schedule": [
      {
        "start_time": "10:00",
        "end_time": "18:00",
        "days_of_week": [
          1,
          2,
          3
        ],
        "instance_min_count": 1,
        "instance_max_count": 10,
        "initial_min_instance_count": 5
      },
      {
        "start_date": "2099-06-27",
        "end_date": "2099-07-23",
        "start_time": "11:00",
        "end_time": "19:30",
        "days_of_month": [
          5,
          15,
          25
        ],
        "instance_min_count": 3,
        "instance_max_count": 10,
        "initial_min_instance_count": 5
      }
    ],
    "specific_date": [
      {
        "start_date_time": "2099-06-02T10:00",
        "end_date_time": "2099-06-15T13:59",
        "instance_min_count": 1,
        "instance_max_count": 4,
        "initial_min_instance_count": 2
      },
      {
        "start_date_time": "2099-01-04T20:00",
        "end_date_time": "2099-02-19T23:15",
        "instance_min_count": 2,
        "instance_max_count": 5,
        "initial_min_instance_count": 3
      }
    ]
  }
}


```

This endpoint retrieves the Auto-Scaling policy of your application. 

### HTTP Request

`GET /v1/apps/:guid/policy`

where `:guid` is the application's GUID. 

## Create/Update policy

> To create an AutoScaling policy to your application, please prepare a policy JSON according to [App AutoScaler policy specification][policy] first, then attach the policy with the code below: 

```shell
token=`cf oauth-token`
appId=`cf app Your-App --guid`
curl "http://example.com/v1/apps/$appId/policy" \
  -X PUT \
  -H "Authorization:$token` \
  -H "Content-Type: application/json" \
  -d @policy.json 
```
> Make sure to replace `Your-App` with your application name, also replace `policy.json` with your policy file. 

> If the policy content is written in valid schema, the example response is :

```
HTTP/1.1 200 OK
<file contents>
```

> If the policy content is invalid, the example response is :

```
HTTP/1.1 400 Bad Request
<Error messages>
```

This endpoint create/update the Auto-Scaling policy to your application. 

### HTTP Request

`PUT /v1/apps/:guid/policy`

where `:guid` is the application's GUID. 

<aside class="notice">The request body should be a valid <a href='/app-autoscaler/policy.html'>App AutoScaler policy</a> </aside>


## Delete policy

> To remove `App AutoScaler` from your application, you can delete the policy with the code below:

```shell
token=`cf oauth-token`
appId=`cf app <Your-App> --guid`
curl "http://example.com/v1/apps/$appId/policy" \
  -X DELETE \
  -H "Authorization:$token` 
```
> Make sure to replace `Your-App` with your application name.

>Example response is :

```
HTTP/1.1 200 OK
```

This endpoint delete the Auto-Scaling policy from your application. 

### HTTP Request

`DELETE /v1/apps/:guid/policy`

where `:guid` is the application's GUID. 

# Metrics

## Aggregated Metrics

> To get historical aggreagated metrics of your application, please use the code below:

```shell
token=`cf oauth-token`
appId=`cf app <Your-App> --guid`
curl "http://example.com/v1/apps/$appId/aggregated_metric_histories/<Metric-Type>" \
  -X GET \
  -H "Authorization:$token` 
```
> Make sure to replace `Your-App` with your application name, and replace the `Metric-Type` to one of the supported metric types. 

> The above command returns JSON Array like below:

```json
[
  "total_results": 2,
  "total_pages": 1,
  "page": 1,
  "prev_url": null,
  "next_url": null,
  "resources": [
    {
      "app_guid": "appId",
      "timestamp": 1494989539138350433,
      "metric_type": "memoryused",
      "value": "400",
      "unit": "megabytes"
    },
    {
      "app_guid": "appId",
      "timestamp": 1494989539138350433,
      "metric_type": "memoryused",
      "value": "400",
      "unit": "megabytes"
    }
  ]
]
```
This endpoint retrieves the aggregated metrics of your application. 

`App AutoScaler` has more interests in the application’s aggregated average metric than individual instance metrics and use the aggregated metrics in evaluation stage.  So, querying the aggrgated metrics will help you to understand why a scaling action happens at that time.  

### HTTP Request

`GET /v1/apps/:guid/aggregated_metric_histories/:<Metric-Type>`, in which 

* `:guid` is the application's GUID
* `:<Metric-Type>` is one of the supported metric type value, including 
   * memoryused 
   * memoryutil
   * responsetime
   * throughput

### Query Parameters

Parameter | Description                 | Values           | Required
--------- | ----------------------------|----------------- | --------
start-time| The start time              | int, the number of nanoseconds elapsed since January 1, 1970 UTC  | false, default 0
end-time  | The end time                | int, the number of nanoseconds elapsed since January 1, 1970 UTC  | false, default `now`
order | The order of display             | string,”asc” or “desc” | false, default `desc`
page | The required page number          | int | false, default 1
results-per-page | Number of results which displayed in the same page       | int | false, default 10


## Instances Metrics

> To get historical instance's metrics of your application, please use the code below:

```shell
token=`cf oauth-token`
appId=`cf app <Your-App> --guid`
curl "http://example.com/v1/apps/$appId/metric_histories/<Metric-Type>" \
  -X GET \
  -H "Authorization:$token` 
```
> Make sure to replace `Your-App` with your application name, and replace the `Metric-Type` to one of the supported metric types. 

> The above command returns JSON Array like below:

```json
[
  "total_results": 2,
  "total_pages": 1,
  "page": 1,
  "resources": [
    {
      "app_guid": "appId",
      "instanceIndex": 0,
      "timestamp": 1494989539138350433,
      "collected_at": 1494989539138350000,
      "metric_type": "memoryused",
      "value": "400",
      "unit": "megabytes"
    },
    {
      "app_guid": "appId",
      "instance_index": 1,
      "timestamp": 1494989539138350433,
      "collected_at": 1494989539138350000,
      "metric_type": "memoryused",
      "value": "400",
      "unit": "megabytes"
    }
  ]
]
```
This endpoint retrieves the instance's metrics of your application. 

### HTTP Request

`GET /v1/apps/:guid/metric_histories/:<Metric-Type>`, in which 

* `:guid` is the application's GUID
* `:<Metric-Type>` is one of the supported metric type value, including 
   * memoryused 
   * memoryutil
   * responsetime
   * throughput

### Query Parameters

Parameter | Description                 | Values           | Required
--------- | ----------------------------|----------------- | --------
start-time| The start time              | int, the number of nanoseconds elapsed since January 1, 1970 UTC  | false, default 0
end-time  | The end time                | int, the number of nanoseconds elapsed since January 1, 1970 UTC  | false, default `now`
order | The order of display             | string,”asc” or “desc” | false, default `desc`
page | The required page number          | int | false, default 1
results-per-page | Number of results which displayed in the same page       | int | false, default 10


# Histories

> To get the autoscaling histories of your application, please use the code below:

```shell
token=`cf oauth-token`
appId=`cf app <Your-App> --guid`
curl "http://example.com/v1/apps/$appId/scaling_histories" \
  -X GET \
  -H "Authorization:$token` 
```
> Make sure to replace `Your-App` with your application name.

> The above command returns JSON Array like below:

```json
{
  "total_results": 2,
  "total_pages": 1,
  "page": 1,
  "resources": [
    {
      "app_guid": "appId",
      "timestamp": 1494989539138350433,
      "scaling_type": 1,
      "status": 0,
      "old_instances": 1,
      "new_instances": 2,
      "reason": "",
      "message": "",
      "error": ""
    },
    {
      "app_guid": "appId",
      "timestamp": 1494989539138350435,
      "scaling_type": 1,
      "status": 0,
      "old_instances": 1,
      "new_instances": 2,
      "reason": "",
      "message": "",
      "error": ""
    }
  ]
}
```
This endpoint retrieves the autoscaling history events of your application for audit purpose.

### HTTP Request

`GET /v1/apps/:guid/scaling_histories`

where `:guid` is the application's GUID


### Query Parameters

Parameter | Description                 | Values           | Required
--------- | ----------------------------|----------------- | --------
start-time| The start time              | int, the number of nanoseconds elapsed since January 1, 1970 UTC  | false, default 0
end-time  | The end time                | int, the number of nanoseconds elapsed since January 1, 1970 UTC  | false, default `now`
order | The order of display             | string,”asc” or “desc” | false, default `desc`
page | The required page number          | int | false, default 1
results-per-page | Number of results which displayed in the same page       | int | false, default 10



[uaa]:https://github.com/cloudfoundry/uaa/blob/develop/docs/UAA-Tokens.md
[policy]:/app-autoscaler/policy.html