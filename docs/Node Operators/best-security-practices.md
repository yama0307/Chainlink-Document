---
layout: nodes.liquid
date: Last Modified
title: "Best Security and Operating Practices"
permalink: "docs/best-security-practices/"
whatsnext: {"FluxAggregator Uses":"/docs/fluxaggregator-uses/"}
hidden: false
---
The following information provides a set of best security and operating practices that node operators need to utilize at a minimum to enhance the security and reliability of their infrastructure.
[block:api-header]
{
  "title": "Restricting Access"
}
[/block]
To run a Chainlink node, the Operator UI port does not need to be open on the internet for it to correctly function. Due to this, we recommend restricting access to all of the services required over the internet.

**Minimum Requirements:**
- SSH (port 22 or changed from the default) is open, and access to the node is granted via SSH tunnelling. This is done typically by adding `-L 6688:localhost:6688` to your SSH command.
- Access to the Ethereum client that the Chainlink node uses is restricted to solely the Chainlink node. This includes ports 8545 and 8546, but excludes 30303 for P2P traffic.
 
**Recommended:**
- The use of a VPN restricts access to only those who are signed into the VPN in order to access internal resources. For example, this can be achieved by using something like <a target="_blank" href="https://openvpn.net/vpn-server/">OpenVPN Access Server</a>.
- With the use of the VPN, all traffic between Chainlink nodes and Ethereum clients is routed internally rather than over the internet. For example, all servers are placed in an internal subnet range such as `10.0.0.0/16` and use these IP addresses for communicating.
- Chainlink nodes have the potential to send arbitrary HTTP GET and POST requests, exposing internal network resources. We recommend deploying with a DMZ which has strong outbound network restrictions.
[block:api-header]
{
  "title": "Failover Capabilities"
}
[/block]
To ensure there is very minimal downtime, failover capabilities are required on both the Chainlink and Ethereum clients so that if any one server fails, the service is still online.

**Minimum Requirements:**
- Chainlink nodes are using a PostgreSQL database that are not on the same servers as the Chainlink nodes.
- At least two Chainlink nodes are running at any one time, with both of them pointing to the same database to ensure failover if one fails. 

**Ethereum-specific:**
- Ethereum client websocket connectivity is fronted by a load balancer, used by the Chainlink nodes. <a href="https://docs.aws.amazon.com/elasticloadbalancing/latest/application/tutorial-target-ecs-containers.html" target="_blank">Here is an example on how to set up a load balancer.</a>
    - If a VPN and internal routing is configured, SSL is not needed but still recommended, as all traffic is purely internal.
    - If both Ethereum and Chainlink nodes are public facing without a VPN, SSL is required to ensure that no communication between both can be intercepted.
[block:api-header]
{
  "title": "Disaster Recovery"
}
[/block]
Problems occur and when they do, the right processes need to be in-place to ensure that as little downtime as possible occurs. The main impediment to incurring large amounts of downtime in the context of Chainlink node operators is a fully corrupted Ethereum node that requires a re-sync.

Due to the challenge of recovering an Ethereum client, we recommend:
- Daily snapshots of the Ethereum chain on a separate server than what the Chainlink node is connected to.
- An Ethereum client start-up process that pulls down the latest template of the chain and syncs it to the latest height.

With this process in-place, the elapsed time of full disaster is kept to a minimum.
[block:api-header]
{
  "title": "Active Monitoring"
}
[/block]
To be proactive in detecting any issues before or when they occur, active monitoring needs to be in place. The areas where we recommend to monitor are:
- (Minimum Required) ETH balance of the wallet address assigned to the node.
- Errored job runs.
- Operator UI port to be open and responsive. (Usually: 6688)
- Ethereum http and websocket ports to be open and responsive. (Usually: 8545 & 8546)
- Ethereum client disk, RAM and CPU usage.

Monitoring can be set up from the Docker container's output and fed into most major logging providers. For example, you can use Docker's docs to set up the logging driver for <a href="https://docs.docker.com/config/containers/logging/awslogs/" target="_blank">Amazon CloudWatch</a> and <a href="https://docs.docker.com/config/containers/logging/gcplogs/" target="_blank">Google Cloud Logging</a>. You will want to set the [
JSON_CONSOLE](../configuration-variables/#json_console) configuration variable to `true` so that the output of the container is JSON-formatted for logging.
[block:api-header]
{
  "title": "Frequent Updates"
}
[/block]
Due to the early nature of the software, it may be required to perform frequent updates to your Chainlink node. 

On performing system maintenance to update the Chainlink node, follow [this](/docs/performing-system-maintenance/#failover-node-example) guide.
[block:api-header]
{
  "title": "Jobs and Config"
}
[/block]
The following are suggestions for job specifications and configuration settings for the node.

[Job Specifications](../job-specifications/):
- Include the address of your oracle contract address for all RunLog initiated jobs, as shown in the [Fulfilling Requests](../fulfilling-requests/#add-jobs-to-the-node) guide.
- Override the global `MIN_INCOMING_CONFIRMATIONS` config by setting a `confirmations` field in jobs which perform off-chain payments to allow for greater security by making the node ensure the transaction is still valid after X blocks.

[Configuration Variables](../configuration-variables/):
- [MINIMUM_CONTRACT_PAYMENT](../configuration-variables/#minimum-contract-payment): ensure your required payment amount is high enough to meet the costs of responding on-chain.
- [MIN_INCOMING_CONFIRMATIONS](../configuration-variables/#min-incoming-confirmations): this can be set to 0 for common data request jobs. See the bullet above on setting individual `confirmations` for specific jobs.
- [LOG_TO_DISK](../configuration-variables/#log-to-disk): Set to `false` if you're using external log drivers which parse the output from Docker containers. This will save you disk space.
- [JSON_CONSOLE](../configuration-variables/#json-console): Set to `true` if you're using external log drivers to parse the output of Docker containers. This will make it easier to parse individual fields of the log and set up alerts.
[block:api-header]
{
  "title": "Addresses"
}
[/block]
- Chainlink node address: this is the address used by the Chainlink node to sign and send responses on-chain. This address should not be used by any other service since the Chainlink node keeps track of its nonce locally (instead of polling the network each transaction). This address only needs to be funded with ETH to be able to write back to the blockchain.
- Oracle contract address: this is the address that users will send Chainlink requests to, and which your Chainlink node will fulfill them through this contract. For best practice, it should be owned by a secure address in your control, like a hardware or cold wallet, since it has access to your earned LINK funds. You will not need to fund this contract with anything (LINK or ETH). Instead, users will fund the contract with LINK as they send requests to it, and those funds will become available for withdrawal as your node fulfills the requests.
[block:api-header]
{
  "title": "Infrastructure as Code (IaC)"
}
[/block]
Running a Chainlink node works well if you template out your infrastructure using tools like Kubernetes or Terraform. The following repositories can assist you with doing that:
- <a href="https://github.com/Pega88/chainlink-gcp" target="_blank">Pega88's Kubernetes & Terraform setup</a>
- <a href="https://github.com/securedatalinks/ChainlinkKubernetes" target="_blank">SDL Chainlink Kubernetes Deployment</a>
- <a href="https://github.com/linkpoolio/terraform-provider-chainlink" target="_blank">LinkPool's Terraform Provider</a>
- <a href="https://github.com/WilsonBillkia/bane" target="_blank">Ansible hardened Chainlink</a>