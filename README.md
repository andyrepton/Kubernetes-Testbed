# Kubernetes Testbed - An experiment to test different security options in K8S

Disclaimer: this is an experiment for myself to build a new system from scratch to keep my skills up to date, I make no promises as to how secure this is! Please review before copying code.

There are various folders here that start as a basic application and gradually add more complicated Kubernetes functionality. You can start at any folder; the changes are additive.

I have done my best to use Kustomize to add features to each section, so you could use this as a base for dev/acc/prod etc if desired.

## Architecture

### A basic website with segregated namespaces

#### Frontend

**Note**: I am using the domain `k8s.personal.andyrepton.com` for this, you should substitute your own domain name if wanting to use certificates from LetsEncrypt

- An Ingress with Nginx, set to best practises with TLS 1.3, a valid SSL certificate etc
- Access allowed to Admin, SysAdmin
- Port 443 and port 80 allowed inbound

#### The App layer

- A basic wordpress website
- Access allowed to Admin, SysAdmin
- Port 443 allowed from Frontend

#### The Database layer

- A basic MySQL deployment
- Access allowed to Admin, SysAdmin
- Port 3389 allowed from App

#### The monitoring system

- A prometheus and blackbox system
- Access allowed to Admin, SysAdmin, read only access allowed to auditor
- Port 9090 allowed from Observability

#### The logging system

- A fluentd system that aggregates logs

#### The observability system

- Grafana, with Loki, as a frontend to Prometheus and FluentD
- Access allowed to Admin, SysAdmin, read only access allowed to auditor
- Port 3100 allowed inbound from Frontend and logging

## The (eventual) ground rules:

- Network policies should isolate the systems from each other
- Pod Security Policies should be best practise
- There should be three roles: Admin (access to everything), SysAdmin (Access to the application, but no K8S namespaces), and auditor (Access to view logs, and monitoring/logging namespaces, but no edit rights)

## General notes

- I'm not using Persistent Volumes to save money. The focus here is on configuration and security, not a 'working' Wordpress site that persists across reboots. I may make a version including these in the future.
- You need to bring your own domain name
- I'm assuming you have a Kubernetes cluster already

# 1. Deploying a wordpress app with a database and ingress

- `cd 1.Basic-App`

## Overview:

In this section, we're deploying a standard deployment pod (wordpress), with a MySQL backend deployment in a different namespace, and an Ingress controller in front. The system uses kustomize to generate a mysql Password (You should change the password in here) and deploy the system. The namespaces are created individually first.

## Deploying:

1. `kubectl apply -f namespaces.yml`
2. Check things look valid with `kubectl diff -k ./`
3. Apply with `kubectl apply -k ./`
3.1 If you get the error `Error from server (InternalError): error when creating "./": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1beta1/ingresses?timeout=10s: no endpoints available for service "ingress-nginx-controller-admission"`, wait a couple of minutes and try again.
4. Get the IP address of your load balancer by using `kubectl -n frontend get svc`.
5. Set the A record of your subdomain name to point to this IP address. I have set *.k8s.personal.andyrepton.com to the IP address here.

### Alternative option: Moving the nameservers for the domain to GCP

As I want to in the future have Kubernetes automatically set up my global DNS for me, I've moved the nameservers of personal.andyrepton.com to my GCP project by setting the following in Cloudflare:

And here is what it looks like in Cloudflare:
![DNS Info](./images/dns-config-cloudflare.png)

And then setting my A record in GCP directly:

![DNS Info](./images/dns-config-gcp.png)

## 1.a) Upgrade: adding an SSL certificate using LetsEncrypt staging

- `cd 1.a.Basic-App-with-TLS-staging`

## Overview:

Let's Encrypt has a staging service that you can use to test your configuration before you proceed to prod and potentially lock yourself out of the LE API. First up you need to make an 'Issuer' for Lets Encrypt, identifying yourself. You can either make an Issuer (locked to a namespace) or a ClusterIssuer (for anything in the cluster) that can be used to request a certificate.

## Deploying:

1. **Important** edit the your-info.yml and set your email address correctly
2. Apply with `kubectl apply -k ./`
3. Browse to your website after a minute or so and you should see you now have a LetsEncrypt Staging certificate

## 1.b) Upgrade: adding an SSL certificate using LetsEncrypt production

- cd `1.b.Basic-App-with-TLS-prod`

## Overview:

We're not going to replace this with a valid SSL certificate using the production letsencrypt provider. We'll make a new issuer with the production API of LE. You can take a look at the differences by looking at the frontend/cert-manager-prod.yml file. We're using kustomize to overwrite the email once again via the your-info.yml file.

## Deploying:

1. **Important** edit the your-info.yml and set your email address correctly
2. Apply with `kubectl apply -k ./`
3. Browse to your website after a minute or so and you should see you now have a valid LE certificate

