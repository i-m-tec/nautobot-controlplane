# nautobot-controlplane
Deploy Nautobot, a Netbox alternative, to ControlPlane.com

## Nautobot
[Nautobot](https://networktocode.com/nautobot/) is a modernized fork of Netbox useful as a "source of truth" for network and infrastructure management with API integration.

## ControlPlane
[ControlPlane](https://controlplane.com/) is a multi-cloud developer platform.

## YouTube Demo
[![Nautobot in ControlPlane.com Demo](https://img.youtube.com/vi/rbdstWflEZY/0.jpg)](https://www.youtube.com/watch?v=rbdstWflEZY)


# Install cpln cli
```
npm install -g @controlplane/cli
```
## Configure cpln
```
cpln login
cpln profile update default --org <YOUR_ORG> --gvc <YOUR_GVC>
cpln profile get
```

# Deploy in ControlPlane
Deploy the postgres and redis workloads first, then deploy the nautobot workload when postgres and redis are ready.

## Postgres and Redis
```
for i in postgres redis; do cpln apply -f $i.yaml --gvc <YOUR_GVC>; done
```
## Nautobot
There are 3 instances of the nautobot images used for specific tasks: web, scheduler, and worker.  For production these should be deployed in separate workloads so they can scale independently.  A multi-container workload is used for this demo to streamline deployment.
## Nautobot combo workload
Generate a random string for NAUTOBOT_SECRET_KEY in nautobot-combo.yaml.  Normally this would be passed in as a cpln secret.
```
pwgen -s 64 1
```
Apply the manifest for your GVC.
```
cpln apply -f nautobot-combo.yaml --gvc <YOUR_GVC>
```
It can take 10-15 minutes for the nautobot workload to complete migrations and become ready.  Check deploymenent status and logs for the nautobot workload in the ControlPlane portal until the workload becomes ready, then the nautobot portal will be available at the endpoint provided in the workload details.
## Initialize Nautobot
Log into the nautobot web container and run these commands:
```
nautobot-server migrate
nautobot-server createsuperuser
```
You can now log into the portal of your nautobot instance as the superuser just created.
# Life Cycle
## Suspend
Temporarily disable workloads from consuming resources. Data in volumesets will be preserved.
```
for i in nautobot nautobot-db nautobot-redis; do cpln workload stop $i --gvc <YOUR_GVC>; done
```
## Resume
Resume workloads from suspended state.
```
for i in nautobot nautobot-db redis; do cpln workload start $i --gvc <YOUR_GVC>; done
```
## Delete
🚨This will permanently destroy resources and data for these workloads!🚨
```
for i in nautobot-combo postgres redis; do cpln delete -f $i.yaml --gvc <YOUR_GVC>; done
```
# Pricing
Cost estimate from 8/25/2025 not including bandwidth. Pricing subject to change.  
Check current pricing: https://controlplane.com/pricing  
| workload-container | cpu(m) | %cpu | ram(Mi) | %ram | disk (GB) | cost/mo   | notes                                   |
| ------------------ | ------ | ---- | ------- | ---- | --------- | --------- | --------------------------------------- |
| nautobot-web       | 400    | 40   | 1600    | 72   | -         | $36.12    |                                         |
| nautobot-worker    | 250    | 30   | 800     | 48   | -         | $21.16    |                                         |
| nautobot-scheduler | 250    | 22   | 800     | 48   | -         | $21.16    |                                         |
| nautobot-postgres  | 150    | 3    | 256     | 22   | 10        | $13.24    | dynamic storage, upto 100GB             |
| nautobot-redis     | 150    | 0.8  | 200     | 2    | -         | $10.72    |                                         |
|                    |        |      |         |      |           | $102.40   | Total cost/mo., not including bandwidth |
|                    |        |      |         |      |           | $1,228.80 | Yearly cost                             |
