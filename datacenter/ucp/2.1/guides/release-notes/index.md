---
title: UCP 2.1 release notes
description: Release notes for Docker Universal Control Plane. Learn more about the
  changes introduced in the latest versions.
keywords: Docker, UCP, release notes
redirect_from:
- /datacenter/ucp/2.1/guides/admin/upgrade/release-notes/
---

Here you can learn about new features, bug fixes, breaking changes and
known issues for the latest UCP version.
You can then use [the upgrade instructions](../admin/upgrade.md), to
upgrade your installation to the latest release.


## Version 2.1.1

(14 Mar 2017)

**Known issues**

If you are currently running UCP 2.1.0 and previously customized the sessions
lifetime parameter in the Authentication settings UI, upgrading to UCP 2.1.1 may
cause users to not be able to log into UCP and DTR. This is caused by a faulty
default value which sets maximum concurrent user sessions to zero.

You can either wait for UCP 2.1.2 to be released so that the problem is
automatically fixed, or upgrade to 2.1.1, and use the following steps to fix
the problem.

Start by getting the current configuration for user sessions by running:

```bash
curl -u admin "https://$UCP_HOST/enzi/v0/config/sessions"
```

The command will prompt for the `admin` user's password and then return
the current sessions config which should look something like:

```json
{
  "lifetimeHours": 72,
  "renewalThresholdHours": 24,
  "perUserLimit": 0
}
```

If `perUserLimit` is set to `0`, you need to set it to a value between 1 and 100.
The recommended value is 5. You should also customize the command below with
the `lifetimeHours` and `perUserLimit` values returned by the first command.

```bash
curl -u admin "https://$UCP_HOST/enzi/v0/config/sessions" \
  -X PUT \
  -H 'Content-Type: application/json' \
  -d '{"lifetimeHours": 72, "renewalThresholdHours": 24, "perUserLimit": 5}'
```

You'll now be able to log into UCP and DTR.

**New features**

* Core
  * Administrators can now configure the frequency with which UCP polls metrics.
  Use `docker service update --env-add METRICS_SCRAPE_INTERVAL=10m ucp-agent`,
  and the frequency can be in s/m/h/d.
  * Administrators can now configure the frequency with which UCP gathers disk usage data.
  Use `docker service update --env-add METRICS_DISK_USAGE_INTERVAL=12h ucp-agent`,
  and the frequency can be in s/m/h/d.
  * Support for syncing users and teams from multiple LDAP servers/domains
  (e.g. a separate server to use for `dc=domain2,dc=example,dc=com`)
  * Support for limiting the number of maximum concurrent login sessions any
  user may have

**Bug fixes**

* Core
  * Fixed an issue in which UCP manager would panic and be unable to return
  the right system status after the cluster became unhealthy
  * `ucp-hrm` container now provides debug logs through `stdout`
  * HTTP Routing Mesh now checks to ensure an ingress port is not already
  in use by UCP or DTR before becoming active
  * Fixed an issue in which UCP did not use swarm-mode node IDs, preventing
  usage of node constraints and other features when using cloned VMs as UCP nodes
  * Fixed an issue in which certain Docker API 1.26 commands were not correctly supported
  * Disk usage metrics no longer display 0% when using devicemapper filesystem
  * Disk usage metrics are now collected every 2 hours by default, and can be tunned
  * Fixed an issue causing Content Trust enforcement to ignore an optional `tag` for
  `/images/create`, causing some signed content to not run correctly
  * LDAP sync logs now take up less disk space on manager nodes
  * UCP support dumps are now correctly compressed to take up less disk space,
  and provide information on HTTP Routing Mesh and metrics
* docker/ucp image
    * UCP install now correctly fails and presents an error when trying to
    specify `host-address` to an existing swarm-mode cluster
    * Clarified upgrade message to make it clear that the upgrade command now
    works at once for the entire cluster rather than needing to be run on every
    node
* UI/UX
    * UI now displays a warning if there is significant latency or network issues
    in communications between UCP manager nodes
    * UI no longer incorrectly displays 'No Services' while still loading the
    Services tab
    * UI no longer displays errors when global tasks are removed due to node
    constraints
    * UI now displays a warning when underlying engines in the swarm-mode
    cluster are running different versions
    * UI now displays an error when 'Load Image' command fails
    * 'KV Store Timeout' option now displays correct units (milliseconds)
    * Dashboard now correctly displays errors when metrics are unavailable
    * The DTR deployment page now validates if a DTR replica ID is valid or not

## Version 2.1.0

(9 Feb 2017)

This version of UCP extends the functionality provided by CS Docker Engine
1.13. Before installing or upgrading this version, you need to install CS
Docker Engine 1.13 in the nodes that you plan to manage with UCP.

**New features**

* Core
  * Support for managing secrets (e.g. sensitive information such as passwords
  or private keys) and using them when deploying services. You can store secrets
  securely on the cluster and configure who has access to them, all without having
  to give users access to the sensitive information directly
  * Support for Compose yml 3.1 to deploy stacks of services, networks, volumes,
  and secrets.
  * HTTP Routing Mesh now generally available. It now supports HTTPS passthrough
  where the TLS termination is performed by your services, Service Name  Indication
  (SNI) extension of TLS, multiple networks for app isolation, and Sticky Sessions
  * Granular label-based access control for secrets and volumes
  (NOTE: unlike other resources controlled via label-based access control, a
  volume without a label is accessible by all UCP users with Restricted Control
  or higher default permissions)

* UI/UX
  * You can now view and manage application stacks directly from the UI
  * You can now view cluster and node level resource usage metrics
  * When updating a service, the UI now shows more information about the service status
  * Rolling update for services now have `failure-action` which you can use to
  * Several improvements to service lifecycle management
  specify rollback, pausing, or continuing if the update fails for a task
  * LDAP synching has more configuration options for extra flexibility
  * UCP now warns when the cluster has nodes with different Docker Engine versions
  * The HTTP routing mesh settings page now lists all services using the
  routing mesh, with details on parameters and health status
  * Admins can now view team membership in a user's details screen
  * You can now customize session timeouts in the authentication settings page
  * Can now mount `tmpfs` or existing local volumes to a service when deploying
  services from the UI
  * Added more tooltips to guide users on the above features

**Bug fixes**

* Core
    * HTTP routing mesh can now be enabled or reconfigured when UCP is configured
    to only run images signed by specific teams
    * Fixed an error in which `_ping` calls were causing multiple TCP connections
    to open up on the cluster
    * Fixed an issue in which UCP install occasionally failed with the error
    "failed to change temp password"
    * Fixed an issue where multiple rapid updates of HTTP Routing Mesh configuration
    would not register correctly
    * Demoting a manager while in HA configuration no longer causes the `ucp-auth-api`
     container to provide errors

* UI/UX
    * When creating a user, pressing enter on keyboard no longer causes problems
    * Fixed assorted icon and text visibility glitches
    * Installing DTR no longer fails when "Enable scheduling on UCP controllers and
    DTR nodes" is unchecked.
    * Publishing a port to both TCP and UDP in a service via UI now works correctly

**Known issues**


The `docker stats` command is sometimes wrongly reporting high CPU usage.
Use the `top` command to confirm the real CPU usage of your node.
[Learn more](https://github.com/docker/docker/issues/28941).


**Version compatibility**

UCP 2.1 requires minimum versions of the following Docker components:

* Docker Engine 1.13.0
* Docker Remote API 1.25
* Compose 1.9
