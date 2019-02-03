# Auditd in Action



## Features

1. See the output of `sudo aureport` and the underlying events with `sudo ausearch --raw` or filter them with `sudo ausearch --success no`. Optionally point to the rules in */etc/audit/audit.rules*.
1. Show the dashboard *[Filebeat Auditd] Audit Events* and show additional Filebeat modules:
  * *[Filebeat System] New users and groups*
  * *[Filebeat System] Sudo commands*
1. Show the Auditbeat configuration and the raw data in the Discover tab (also point out the `host` and `meta.cloud` data).
1. Show the *[Auditbeat Auditd] Overview* dashboard.
1. `ssh elastic-user@xeraa.wtf` with a bad password and show the failed login on the *[Filebeat System] SSH login attempts* dashboard.
1. SSH with the same user and make it work this time.
1. Run `cat /etc/passwd` and find the event in the Discover tab and filter for `tags is passwd-access` and filter down on `user.name_map.uid` or simply search for `cat` in the searchbar.
1. For a more fine grained filter run `cat /etc/pam.conf` and find the event with `tags is developers-pam-read`.
1. Run `service nginx restart` and pick the `elastic-admin` user to run the command. Show the execution on the *[Auditbeat Auditd] Executions* dashboard by filtering down to the `elastic-user` user.
1. Detect when an admin may be abusing power by looking in a user's home directory. Let the `ssh elastic-admin@xeraa.wtf` check the directory */home/elastic-user* and read the file */home/elastic-user/secret.txt* (will require sudo). Search for the tag `power-abuse` to see the violation.
1. Show */etc/auditbeat/auditbeat.yml* that requires sudo privileges and find the call in `tags is elevated-privs`.
1. Open a socket with `netcat -l 1025` and start a chat with `telnet <hostname> 1025`. Find it by doing a full-text search for `1025`, which will find the Auditd `process-execution` tag as well as the system module `event.action` for `socket_opened`, `socket_closed`, `executed`, `process_started`, `process_stopped`.
1. Filter to `event.module is system` (dashboards coming soon) and show information for users, processes, connections, and the system. This is based on the non-Auditd configuration syntax.
1. Change the content of the website in `/var/www/html/.index.html`. See the change in the *[Auditbeat File Integrity] Overview* dashboard. Depending on the editor the actions might be slightly different; *nano* will generate an `updated` event wheras *vi* does a `moved` and `deleted`.



## Setup

Make sure you have run this before the demo.

1. Have your AWS account set up, access key created, and added as environment variables in `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. Protip: Use [https://github.com/sorah/envchain](https://github.com/sorah/envchain) to keep your environment variables safe.
1. Create the Elastic Cloud instance with the same version as specified in *variables.yml*'s `elastic_version`, enable Kibana as well as the GeoIP & user agent plugins, and set the environment variables with the values for `ELASTICSEARCH_HOST`, `ELASTICSEARCH_USER`, `ELASTICSEARCH_PASSWORD`, as well as `KIBANA_HOST`, `KIBANA_ID`.
1. Change the settings to a domain you have registered under Route53 in *inventory*, *variables.tf*, and *variables.yml*. Set the Hosted Zone for that domain and export the Zone ID under the environment variable `TF_VAR_zone_id`. If you haven't created the Hosted Zone yet, you should set it up in the AWS Console first and then set the environment variable.
1. If you haven't installed the AWS plugin for Terraform, get it with `terraform init` first. Then create the keypair, DNS settings, and instances with `terraform apply`.
1. Open HTTPS and TCP/1025 on the network configuration (waiting for this [Terraform issue](https://github.com/terraform-providers/terraform-provider-aws/issues/700)).
1. Apply the configuration to the instance with `ansible-playbook configure.yml`.

When you are done, remove the instances, DNS settings, and key with `terraform destroy`.
