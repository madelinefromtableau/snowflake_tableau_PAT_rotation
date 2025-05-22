# Update Snowflake Auth to Programmatic Access Token in Tableau Data Sources via REST API

> **Note:** This is a basic set of instructions. It is meant to be shared as a starting resource, not official help docs.

## Prerequisites
- API version 3.2+ (for connection updates)
- New Snowflake generated PAT per user with network policy enabled on user account

## Step 1: Authenticate
**Reference:** https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api_concepts_auth.htm

```bash
POST /api/{version}/auth/signin
Content-Type: application/json

{
  "credentials": {
    "name": "username",
    "password": "password",
    "site": {"contentUrl": "site-name"}
  }
}
```

Save the returned `token` and `site-id`.

## Step 2: Get All Data Sources and Connection IDs
**References:**
- https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api_ref.htm#query_data_sources
- https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api_ref.htm#query_data_source_connections

```bash
GET /api/{version}/sites/{site-id}/datasources?filter=connectiontype:eq:snowflake&authenticationType:eq:Username+Password
X-Tableau-Auth: {token}
```

Filter results for Snowflake connections by checking auth type is username/password and `connectionType="snowflake"`.

For each data source, get the connection ID used to authenticate:

```bash
GET /api/{version}/sites/{site-id}/datasources/{datasource-id}/connections?filter=authenticationType:eq:Username+Password
```

Store the data source IDs and Connection IDs.

## Step 3: Update Each Data Source Connection
**Reference:** https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api_ref.htm#update_data_source_connection

For each Snowflake data source:

```bash
PUT /api/{version}/sites/{site-id}/datasources/{datasource-id}/connections/{connection-id}
X-Tableau-Auth: {token}
Content-Type: application/json

{
  "connection": {
    "serverAddress": "xyz.snowflakecomputing.com",
    "serverPort": "443",
    "userName": "your_username",
    "password": "your_pat_string_here",
    "embedPassword": "true",
    "queryTaggingEnabled": "true"
  }
}
```

## Step 4: Batch Script Example

```python
import requests

# After authentication...

for ds in snowflake_datasources:
    for conn in ds['connections']:
        if conn['type'] == 'snowflake':
            url = f"/api/{version}/sites/{site_id}/datasources/{ds['id']}/connections/{conn['id']}"
            data = f'<tsRequest><connection password="{new_password}" /></tsRequest>'
            response = requests.put(base_url + url, data=data, headers=headers)
```

## Key Points
- Use the **Update Data Source Connection** method
- Connection ID is different from data source ID and are not the IDs in the URLs, they are LUIDs from workgroup
- Test with one data source before batch updating
- Server name, port, username, password are required. The Snowflake port is 443. Embed flag and query tag are not required.
- For embedded datasources, use **Update Workbook Connection** method
