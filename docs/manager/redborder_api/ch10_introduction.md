
# Introduction to RedBorder's API

The full *API* uses a *HTTP REST* style via *HTTPS* web interface. The query must be expressed in *JSON* format to specify which the desired values. For example:

!!! example "Query to the *API*"

        {
          "dataSource": "rb_flow",
          "granularity": "all",
          "namespace_uuids": ["2584761509592299795"],
          "intervals":["2013-11-21T19:07/2013-11-21T19:30"],
          "queryType": "groupBy",
          "aggregations": [
            {"type": "longSum", "name": "events", "fieldName": "events"}
          ],
          "dimensions": [ "http_host", "src", "http_user_agent" ],
          "filter": {
            "type": "and",
            "fields": [
              { "type": "selector", "dimension": "client_mac", "value": "8c:58:77:0b:56:fc" },
              { "type": "regex", "dimension": "src", "pattern": "^10\\.13\\." }
            ]
          }
        }

The query will return something similar to this (the output has been truncated):

!!! example "*API*'s output"

        [
          {
            "version":"v1",
            "timestamp":"2013-11-21T19:07:00.000Z",
            "event":{
              "count":198,
              "http_user_agent":null,
              "http_host":null,
              "src":"10.13.209.147"
            }
          },
          {
            "version":"v1",
            "timestamp":"2013-11-21T19:07:00.000Z",
            "event":{
              "count":1,
              "http_user_agent":"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_3 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Mobile/11B511 [FBAN/FBIOS;FBAV/6.7;FBBV/566055;FBDV/iPhone3,3;FBMD/iPhone;FBSN/iPhone OS;FBSV/7.0.3;FBSS/2; FBCR/Verizon;FBID/phone;FBLC/en_US;FBOP/5]",
              "http_host":"ajax.googleapis.com",
              "src":"10.13.209.147"
            }
          },
          {
            "version":"v1",
            "timestamp":"2013-11-21T19:07:00.000Z",
            "event": {
              "count":1,
              "http_user_agent":"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_3 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Mobile/11B511 [FBAN/FBIOS;FBAV/6.7;FBBV/566055;FBDV/iPhone3,3;FBMD/iPhone;FBSN/iPhone OS;FBSV/7.0.3;FBSS/2; FBCR/Verizon;FBID/phone;FBLC/en_US;FBOP/5]",
              "http_host":"busymockingbird.files.wordpress.com",
              "src":"10.13.209.147"
            }
          },
          {
            "version":"v1",
            "timestamp":"2013-11-21T19:07:00.000Z",
            "event":{
              "count":1,
              "http_user_agent":"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_3 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Mobile/11B511 [FBAN/FBIOS;FBAV/6.7;FBBV/566055;FBDV/iPhone3,3;FBMD/iPhone;FBSN/iPhone OS;FBSV/7.0.3;FBSS/2; FBCR/Verizon;FBID/phone;FBLC/en_US;FBOP/5]",
              "http_host":"connect.facebook.net",
              "src":"10.13.209.147"
            }
          }
        ]

## Authentication

The *Rest API* uses an **Access Key** generated with the **Web Graphical Interface**. To obtain this key, you need to click on your username in the **Configuration Area**, located on the right side of the **Menu Bar**. Here, you will see the space indicating the access key.

![User Settings: *API* Access Key](images/ch10_img001.png)

User Settings: *API* Access Key

This option should be included in the URL of the *API* request.

    https://<hostname>/api/data?auth_token=<API_key>

Each *API* request must contain this access key, and the request will have the permissions associated with the user who owns the access key.

The remaining parameters should be provided in the payload data of the *POST* request. The payload format should be *JSON*, as we saw in the previous example. For instance, if we save the last query in a file named `data.body`, we can make a simple request like this:

    curl --insecure -X POST 'https://<hostname>/api/data?auth_token=<API_key>' -H 'content-type: application/json' -d @data.body

## *JSON* parameters

| Parameter          | Description                            |
| ------------------ | -------------------------------------- |
| dataSource         | Database being accessed.               |
| namespace_uuids    | (Optional) Array of one or more values. Should contain the *UUIDs* of the *Namespace* where the data will be searched. If not specified, the default *dataSource* will be used. |
| granularity        | Specifies the size of the *bucket* for values. |
| intervals          | Specifies the time range(s) of the request. The interval must be specified in ISO format and may include the timezone if desired. |
| queryType          | Specifies which type of query operator to use. |
| aggregations       | Array of one or more values. These fields apply to the columns specified by *fieldName*, and the output of the aggregations will be named according to the value in the *name* field. |
| metric             | (Only for requests with *queryType* of *topN*) Must be the same value as specified in *aggregations*. Used for sorting. |
| dimensions         | (Only for requests with *queryType* of *groupBy* or *timeseries*) Array of zero or more values. These are used to restrict grouping. If empty, one value per time granularity bucket is requested in *groupBy*. |
| dimension          | (Only for requests with *queryType* of *topN*) Single value. All available dimensions can be found later on this page. |
| filter             | Specifies the filter for the request. Available filters can be found later on this page. |
| threshold          | (Only for requests with *queryType* of *topN*) Single value. Limits the number of values returned in *topN* requests. |
| orderBy            | (Only for requests with *queryType* of *groupBy*). |

To see the different values that the fields can take, expand the corresponding tabs:

??? note "`dataSource`"

    - `rb_flow`: *Netflow/sFlow* database with traffic data.
    - `rb_state`: health status of wireless stations obtained via *NMSP*.
    - `rb_event`: *IDS/IPS* database with intrusion data.
    - `rb_monitor`: *SNMP* monitoring database.

??? note "`granularity`"

    It can be a predefined interval such as `second`, `minute`, `fifteen_minute`, `thirty_minute`, `hour`, or `day`.

    It can also be an expression like `{"type": "period", "period":"PT6m"}` which means "6-minute *buckets*".

    In the example, it is set to the special value `all`, meaning all data points are grouped into the same time *bucket*.

??? note "`queryType`"

    - `groupBy`: this is the most flexible type of query but can also have poorer performance.
    - `topN`: similar to `groupBy` queries but groups and sorts by a single dimension. This is much faster than `groupBy`.
    - `timeseries`: used for generating time series using specified aggregators.

??? note "`aggregations`"

    There are different aggregators depending on the `dataSource` used:

    - `rb_flow`

        1. `{type: "longSum", name: "events", fieldName: "events"}` -> flow count / occurrences
        * `{type: "longSum", name: "pkts", fieldName: "sum_pkts"}` -> sum of packets
        * `{type: "longSum", name: "bytes", fieldName: "sum_bytes"}` -> sum of bytes

    - `rb_event`

        1. `{type: "longSum", name: "events", fieldName: "events"}` -> IDS/IPS event count / occurrences

    - `rb_monitor`

        1. `{type: "longSum", name: "events", fieldName: "events"}` -> result of monitored item within specified interval
        * `{type: "min", name: "min", fieldName: "min_value"}` -> minimum value for monitored parameter
        * `{type: "max", name: "max", fieldName: "max_value"}` -> maximum value for monitored parameter

??? note "`orderBy`"

    This parameter has the following fields:

    `{"type":"default","limit":25,"columns":[{"dimension":"bytes","direction":"DESCENDING"}]}`

### Dimensions for `rb_flow`

Within `rb_flow` we have *Netflow* and *sFlow* data, so we will find the information typically found in the **Traffic** module through the web interface.

To see a list of available dimensions, expand the following tab:

??? info "Dimensions"

    |                      |                    |                       |                     |                  |
    |----------------------|--------------------|-----------------------|---------------------|------------------|
    | application_id_name  | service_provider   | dot11_status          | input_vrf           | tos              |
    | biflow_direction     | organization_uuid  | dst                   | ip_protocol_version | type             |
    | client_id            | organization       | dst_as_name           | dst_vlan            | wireless_id      |
    | client_latlong       | namespace_uuid     | dst_country_code      | l4_proto            | wireless_station |
    | client_mac           | namespace          | dst_map               | output_snmp         |                  |
    | client_mac_vendor    | deployment_uuid    | dst_net_name          | output_vrf          |                  |
    | client_rssi          | deployment         | dst_port              | rat                 |                  |
    | client_rssi_num      | market_uuid        | dst_vlan              | scatterplot         |                  |
    | client_snr           | market             | duration              | sensor_uuid         |                  |
    | client_snr_num       | building_uuid      | engine_id_name        | sensor_name         |                  |
    | conversation         | building           | hnbgeolocation        | src                 |                  |
    | coordinates_map      | campus_uuid        | hnblocation           | src_as_name         |                  |
    | darklist_category    | campus             | http_host             | src_country_code    |                  |
    | darklist_direction   | floor_uuid         | http_referer_l1       | src_map             |                  |
    | darklist_protocol    | floor              | http_social_media     | src_net_name        |                  |
    | darklist_score       | zone               | http_social_user      | src_port            |                  |
    | darklist_score_name  | direction          | http_user_agent_os    | src_vlan            |                  |
    | service_provider_uuid| dot11_protocol     | input_snmp            | srv_port            |                  |

### Dimensions for `rb_event`

Within `rb_event`, we have *IDS/IPS* data, so we will find the information typically found in the **Intrusion** module through the web interface.

To see a list of available dimensions, expand the following tab:

??? info "Dimensions"

    |                      |                    |                       |                     |                  |
    |----------------------|--------------------|-----------------------|---------------------|------------------|
    | action               | client_latlong     | ethsrc                | src                 |                  |
    | classification       | zone               | ethsrc_vendor         | src_as_name         |                  |
    | service_provider_uuid| conversation       | file_hostname         | src_country_code    |                  |
    | service_provider     | darklist_category  | file_size             | src_map             |                  |
    | organization_uuid    | darklist_direction | file_uri              | src_net_name        |                  |
    | organization         | darklist_protocol  | group_name            | src_port            |                  |
    | namespace_uuid       | darklist_score     | icmptype              | tos                 |                  |
    | namespace            | darklist_score_name| iplen_range           | ttl                 |                  |
    | deployment_uuid      | domain_name        | l4_proto              | vlan                |                  |
    | deployment           | dst                | msg                   | wireless_station    |                  |
    | market_uuid          | dst_as_name        | priority              |                     |                  |
    | market               | dst_country_code   | rev                   |                     |                  |
    | building_uuid        | dst_map            | scatterplot           |                     |                  |
    | building             | dst_net_name       | sensor_uuid           |                     |                  |
    | campus_uuid          | dst_port           | sensor_name           |                     |                  |
    | campus               | ethdst             | sha256                |                     |                  |
    | floor_uuid           | ethdst_vendor      | sig_generator         |                     |                  |
    | floor                | ethlength_range    | sig_id                |                     |                  |

### Dimensions for `rb_monitor`

Within `rb_monitor`, we have *SNMP* monitoring data, so we will find the information typically found in the **Monitor** module through the web interface.

To see a list of available dimensions, expand the following tab:

??? info "Dimensions"

    |                      |                    |                  |                  |
    |----------------------|--------------------|------------------|------------------|
    | service_provider_uuid| monitor            |                  |                  |
    | service_provider     | sensor_uuid        |                  |                  |
    | organization_uuid    | unit               |                  |                  |
    | organization         | value              |                  |                  |
    | namespace_uuid       |                    |                  |                  |
    | namespace            |                    |                  |                  |
    | deployment_uuid      |                    |                  |                  |
    | deployment           |                    |                  |                  |
    | market_uuid          |                    |                  |                  |
    | market               |                    |                  |                  |
    | building_uuid        |                    |                  |                  |
    | building             |                    |                  |                  |
    | campus_uuid          |                    |                  |                  |
    | campus               |                    |                  |                  |
    | floor_uuid           |                    |                  |                  |
    | floor                |                    |                  |                  |
    | group_name           |                    |                  |                  |
    | instance             |                    |                  |                  |

### Dimensions for `rb_state`

Within `rb_state`, we have data on the health status of wireless stations obtained via SNMP.

To see a list of available dimensions, expand the following tab:

??? info "Dimensions"

    |                      |                    |                  |                  |
    |----------------------|--------------------|------------------|------------------|
    | service_provider_uuid| type               |                  |                  |
    | service_provider     |wireless_admin_state|                  |                  |
    | organization_uuid    | wireless_channel   |                  |                  |
    | organization         | wireless_mode      |                  |                  |
    | namespace_uuid       | wireless_op_state  |                  |                  |
    | namespace            | wireless_slot      |                  |                  |
    | deployment_uuid      |wireless_station_name|                  |                  |
    | deployment           | wireless_station   |                  |                  |
    | market_uuid          | wireless_tx_power  |                  |                  |
    | market               | status             |                  |                  |
    | building_uuid        |                    |                  |                  |
    | building             |                    |                  |                  |
    | campus_uuid          |                    |                  |                  |
    | campus               |                    |                  |                  |
    | floor_uuid           |                    |                  |                  |
    | floor                |                    |                  |                  |
    | sensor_uuid          |                    |                  |                  |
    | sensor_name          |                    |                  |                  |

## Filters

A filter is a *JSON* object that specifies which data rows should be included in the computation of a request. It is essentially equivalent to the WHERE clause in SQL, allowing filters to be used similarly to how you would use such a clause.

### `selector`

The simplest filter is a `selector` filter. This filter matches a specific dimension with a specific value. Selector filters can be used as base filters for more complex boolean expressions of filters.

!!! example "Syntax for `selector` filter"

        {
            "type": "selector",
            "dimension": "client_mac",
            "value": "8c:58:77:0b:56:fc"
        }

### `regex`

This filter is similar to the `selector` filter but uses regular expressions. It matches the specified dimension with the given pattern. The pattern can be any standard Java regular expression.

!!! example "Syntax for `regex` filter"

        {
            "type": "regex",
            "dimension": "http_host",
            "pattern": "youtube\.com"            
        }

### Logical Expression Filters

It is possible to use logical expressions AND, OR, and NOT to combine filters or create more complex filters using the previously mentioned types.

!!! example "Syntax for `and` filter"

        {
            "type": "and",
            "fields": [
                {
                    "type": "selector",
                    "dimension": "client_mac",
                    "value": "8c:58:77:0b:56:fc"
                },
                {
                    "type": "regex",
                    "dimension": "src",
                    "pattern": "^10\\.13\\."
                }
            ]
        }

!!! example "Syntax for `or` filter"

        {
            "type": "or",
            "fields": [
                {
                    "type": "selector",
                    "dimension": "client_mac",
                    "value": "8c:58:77:0b:56:fc"
                },
                {
                    "type": "selector",
                    "dimension": "client_mac",
                    "value": "83:53:44:4b:56:fd"
                }
            ]
        }

!!! example "Syntax for `not` filter"

        {
            "type": "not", 
            "field": {
                "type": "selector",
                "dimension": "client_mac",
                "value": "8c:58:77:0b:56:fc"
            }
        }