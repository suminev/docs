---
editable: false
---

# yc compute snapshot

Manage snapshots

#### Command Usage

Syntax: 

`yc compute snapshot <command>`

Aliases: 

- `snapshots`

#### Command Tree

- [yc compute snapshot get](get.md) — Show information about the specified snapshot
- [yc compute snapshot list](list.md) — List snapshots
- [yc compute snapshot create](create.md) — Create a snapshot of the specified disk
- [yc compute snapshot update](update.md) — Update the specified snapshot
- [yc compute snapshot add-labels](add-labels.md) — Add labels to specified snapshot
- [yc compute snapshot remove-labels](remove-labels.md) — Remove labels from specified snapshot
- [yc compute snapshot delete](delete.md) — Delete the specified snapshot
- [yc compute snapshot list-operations](list-operations.md) — List operations for the specified snapshot
- [yc compute snapshot list-access-bindings](list-access-bindings.md) — List access bindings for the specified snapshot
- [yc compute snapshot set-access-bindings](set-access-bindings.md) — Set access bindings for the specified snapshot and delete all existing access bindings if there were any
- [yc compute snapshot add-access-binding](add-access-binding.md) — Add access binding for the specified snapshot
- [yc compute snapshot remove-access-binding](remove-access-binding.md) — Remove access binding for the specified snapshot

#### Global Flags

| Flag | Description |
|----|----|
|`--profile`|<b>`string`</b><br/>Set the custom configuration file.|
|`--debug`|Debug logging.|
|`--debug-grpc`|Debug gRPC logging. Very verbose, used for debugging connection problems.|
|`--no-user-output`|Disable printing user intended output to stderr.|
|`--retry`|<b>`int`</b><br/>Enable gRPC retries. By default, retries are enabled with maximum 5 attempts.<br/>Pass 0 to disable retries. Pass any negative value for infinite retries.<br/>Even infinite retries are capped with 2 minutes timeout.|
|`--cloud-id`|<b>`string`</b><br/>Set the ID of the cloud to use.|
|`--folder-id`|<b>`string`</b><br/>Set the ID of the folder to use.|
|`--folder-name`|<b>`string`</b><br/>Set the name of the folder to use (will be resolved to id).|
|`--endpoint`|<b>`string`</b><br/>Set the Cloud API endpoint (host:port).|
|`--token`|<b>`string`</b><br/>Set the OAuth token to use.|
|`--impersonate-service-account-id`|<b>`string`</b><br/>Set the ID of the service account to impersonate.|
|`--no-browser`|Disable opening browser for authentication.|
|`--format`|<b>`string`</b><br/>Set the output format: text (default), yaml, json, json-rest.|
|`-h`,`--help`|Display help for the command.|
