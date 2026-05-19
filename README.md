# PART 1 
  # VPC Creation
  The VPCs are designed using custom-mode networking with Global dynamic routing to support secure segmentation, future scalability, and enterprise-grade hybrid networking patterns. Standard best path selection was chosen for modern BGP-compliant route handling. IPv6 and DNS Armor were intentionally disabled to reduce unnecessary operational complexity for the current scope while maintaining a secure and production-aligned fintech architecture.
  
  app-subnet : 10.10.1.0/24
  
  shared-subnet: 10.100.1.0/24
  
  ## Create app-vpc and app-subnet
  
    gcloud compute networks create app-vpc --project=quiet-rigging-490521-d6 --subnet-mode=custom --bgp-routing-mode=global --bgp-best-path-selection-mode=standard --bgp-bps-inter-region-cost=default
    gcloud compute networks subnets create app-subnet --project=quiet-rigging-490521-d6 --range=10.10.1.0/24 --stack-type=IPV4_ONLY --network=app-vpc --region=us-central1 --enable-private-ip-google-access

  <img width="928" height="489" alt="image" src="https://github.com/user-attachments/assets/81e0978e-7ed2-4c1a-bd95-1e7787323891" />


  
  ## Create shared-vpc and shared-subnet
  
    gcloud compute networks create shared-vpc --project=quiet-rigging-490521-d6 --subnet-mode=custom --bgp-routing-mode=global --bgp-best-path-selection-mode=standard --bgp-bps-inter-region-cost=default
    gcloud compute networks subnets create shared-subnet --project=quiet-rigging-490521-d6 --range=10.100.1.0/24 --stack-type=IPV4_ONLY --network=shared-vpc --region=us-central1 --enable-private-ip-google-access

  <img width="986" height="488" alt="image" src="https://github.com/user-attachments/assets/96a6a8ae-26bf-4720-a570-9afd1cdfbf85" />

  
  ## VPC Peering
  VPC peering was configured using IPv4 single-stack networking with custom route exchange enabled to support scalable private communication between isolated application and shared services networks. Independent update strategy was selected for operational simplicity and flexible network administration
  ### app-vpc to shared-vpc
  <img width="2208" height="1436" alt="image" src="https://github.com/user-attachments/assets/a4c02b4d-0117-42db-9be8-cb863975f5cb" />

  
  ### shared-vpc to app-vpc
  <img width="2208" height="1472" alt="image" src="https://github.com/user-attachments/assets/74a10729-b68a-4fc4-9822-fe8f9abca617" />

  <img width="1234" height="316" alt="image" src="https://github.com/user-attachments/assets/fc0b0e6b-8267-4500-ac69-7d453db6a523" />

  
  ## Firewall Rules
  Allowed TCP,UDP and ICMP app to shared and shared to app VPC. HTTPS allowed only to app-vpc along with GCP IP ranges for health checks
  ### Allow shared-vpc to app-vpc
      gcloud compute --project=quiet-rigging-490521-d6 firewall-rules create allow-shared-to-app --description=allow-shared-to-app --direction=INGRESS --priority=1000 --network=app-vpc --action=ALLOW --rules=tcp,udp,icmp --source-ranges=10.100.0.0/16
  ### Allow app-vpc to shared-vpc
      gcloud compute --project=quiet-rigging-490521-d6 firewall-rules create allow-app-to-shared --description=allow-app-to-shared --direction=INGRESS --priority=1000 --network=shared-vpc --action=ALLOW --rules=tcp,udp,icmp --source-ranges=10.10.0.0/16
  ### allow-https to app-vpc
      gcloud compute --project=quiet-rigging-490521-d6 firewall-rules create allow-https --direction=INGRESS --priority=1000 --network=app-vpc --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0
  ### GCP Health Checks
      gcloud compute --project=quiet-rigging-490521-d6 firewall-rules create allow-gcp-health-checks --direction=INGRESS --priority=1000 --network=app-vpc --action=ALLOW --rules=tcp --source-ranges=35.191.0.0/16,130.211.0.0/22

<img width="1234" height="316" alt="image" src="https://github.com/user-attachments/assets/0715020b-cb39-4a2c-a80e-37112b02d6e9" />
  
  ### Create Cloud Router and Cloud NAT for accessing app VM in app-subnet
     gcloud compute routers create us-central1-nat-router --network app-vpc --region us-central1
     gcloud compute routers nats create us-central1-cloud-nat --router=us-central1-nat-router --region=us-central1 --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges 
     
# PART 2: GKE Setup
    Deployed a secure regional GKE cluster within the application VPC.
    Configured Kubernetes networking and cluster access.
    Prepared the platform for highly available application deployment.
    Ensured integration with Google Cloud native networking and load balancing services.
  
  <img width="1315" height="736" alt="image" src="https://github.com/user-attachments/assets/f12ea6d6-da87-4c63-a949-af3052e97ff8" />

  <img width="1315" height="736" alt="image" src="https://github.com/user-attachments/assets/cb27cd78-56fc-4a28-a48e-9d20b885629a" />

  <img width="1315" height="736" alt="image" src="https://github.com/user-attachments/assets/40501746-7302-4263-906a-a5ac4a631aee" />

  <img width="1315" height="736" alt="image" src="https://github.com/user-attachments/assets/4c9bd601-45f5-4580-8331-6762e7031f18" />

  <img width="1315" height="736" alt="image" src="https://github.com/user-attachments/assets/03a51536-47da-4985-8fab-78ab2543d557" />

  <img width="1315" height="736" alt="image" src="https://github.com/user-attachments/assets/5630164f-6a6b-499f-96c5-ede2870d1c97" />

  <img width="1162" height="615" alt="image" src="https://github.com/user-attachments/assets/4c1569a2-7eb3-4dcb-9e1f-85877ea0a4b2" />

  <img width="1237" height="451" alt="image" src="https://github.com/user-attachments/assets/efe1be03-8acc-4dc4-913f-9bf62b0dae7f" />


# Part 3: Application Deployment & Cloud Armor Configuration
    Below Steps and validation performed:
    Security Controls Applied for Cloud Armor:
    IP-based access blocking for malicious traffic
    SQL Injection (SQLi) protection using preconfigured WAF rules
    Cross-Site Scripting (XSS) protection
    Backend security policy integration with Kubernetes ingress
    Deployed a highly available NGINX application with multiple replicas as below:
    Created Kubernetes resources:
    Namespace
    Deployment
    Service
    BackendConfig
    Ingress
    Managed SSL Certificate
    Global external HTTP(S) Load Balancer
    Static public IP
    DNS mapping
    
  ## Create Cloud Armour Policy:
      gcloud compute security-policies create fintech-armor-policy --description "Cloud Armor policy for fintech application"
  ## Add IP Blocking rule
        gcloud compute security-policies rules create 1000 \
          --security-policy fintech-armor-policy \
          --src-ip-ranges 122.183.40.17/32 \
          --action deny-403 \
          --description "Block malicious IP range"
  ## Add WAF for SQLi Protection
        gcloud compute security-policies rules create 2000 \
          --security-policy fintech-armor-policy \
          --expression "evaluatePreconfiguredWaf('sqli-stable')" \
          --action deny-403 \
          --description "Block SQL injection attacks"
  ## XSS Protection 
        gcloud compute security-policies rules create 3000 \
          --security-policy fintech-armor-policy \
          --expression "evaluatePreconfiguredWaf('xss-stable')" \
          --action deny-403 \
          --description "Block XSS attacks"
   ## Rate limit
         gcloud compute security-policies rules create 4000 \
            --security-policy=fintech-armor-policy \
            --action=throttle \
            --src-ip-ranges="*" \
            --rate-limit-threshold-count=1000 \
            --rate-limit-threshold-interval-sec=60 \
            --conform-action=allow \
            --exceed-action=deny-429 \
            --enforce-on-key=IP
   ## L7 ddos Defence
         gcloud compute security-policies update fintech-armor-policy \
            --enable-layer7-ddos-defense

  <img width="1162" height="615" alt="image" src="https://github.com/user-attachments/assets/814bbd07-b13b-4c79-afa0-4bdcc994063a" />


  ## Reserve Static IP for ELB:
        gcloud compute addresses create fintech-ingress-ip --global
        gcloud compute addresses describe fintech-ingress-ip  --global   -> Get above static reserved IP
  <img width="909" height="316" alt="image" src="https://github.com/user-attachments/assets/17164c8f-4305-4f26-999c-898f3ce608ed" />

  ## Connect to cluster:
    gcloud container clusters get-credentials gke-dev --region=us-central1

  ## Create namespace:
    kubectl create namespace fintech-app
  ## Create deployment.yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: nginx-app
        namespace: fintech-app
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: nginx-app
        template:
          metadata:
            labels:
              app: nginx-app
          spec:
            containers:
            - name: nginx
              image: nginx:stable
              ports:
              - containerPort: 80

  ## Create service.yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: nginx-service
        namespace: fintech-app   
        annotations:
          cloud.google.com/backend-config: '{"default": "fintech-backendconfig"}' 
      spec:
        type: ClusterIP 
        selector:
          app: nginx-app
        ports:
        - port: 80
          targetPort: 80

  ## Create backendconfig.yaml
      apiVersion: cloud.google.com/v1
      kind: BackendConfig
      metadata:
        name: fintech-backendconfig
        namespace: fintech-app
      spec:
        securityPolicy:
          name: fintech-armor-policy

  ## Create ingress.yaml
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: fintech-ingress
          namespace: fintech-app
          annotations:
            kubernetes.io/ingress.class: "gce"
            networking.gke.io/managed-certificates: "fintech-managed-cert"
            kubernetes.io/ingress.global-static-ip-name: "fintech-ingress-ip"  
        spec:
          defaultBackend:
            service:
              name: nginx-service
              port:
                number: 80

   ## Create managed-cert.yaml
        apiVersion: networking.gke.io/v1
        kind: ManagedCertificate
        metadata:
          name: fintech-managed-cert
          namespace: fintech-app
        spec:
          domains:
            - app.wealthbridgezone.com
  ## Apply All Kubernetes Resources
        kubectl apply -f deployment.yaml
        kubectl apply -f backendconfig.yaml
        kubectl apply -f service.yaml
        kubectl apply -f managed-cert.yaml
        kubectl apply -f ingress.yaml
        
  ## ELB
  <img width="1393" height="823" alt="image" src="https://github.com/user-attachments/assets/41871572-37d6-49d0-89f9-2f5efdd0f9a5" />

  ## DNS mapping 
  <img width="909" height="316" alt="image" src="https://github.com/user-attachments/assets/29d7dcf2-1cad-42b7-ba5b-292b60c8d6b9" />

  ## Outputs 
  <img width="997" height="426" alt="image" src="https://github.com/user-attachments/assets/cc0a45aa-a7aa-4c10-b6f2-5f9a1149344a" />

  ### Forbidden: from IP: 122.183.40.17 (WIFI)
  <img width="1417" height="871" alt="image" src="https://github.com/user-attachments/assets/c9f44552-cc7f-4c1d-9f73-fa7f405073d9" />


  ### Allowed from other IPs(MOBILE HOTSPOT) - 106.192.202.12:
  <img width="1417" height="871" alt="image" src="https://github.com/user-attachments/assets/0a54864a-f7b7-4667-96cd-41afa7930f5a" />

  ### Similar output from Command line:
  <img width="978" height="842" alt="image" src="https://github.com/user-attachments/assets/5b64f296-8013-4c0c-919a-23e105873ab0" />





  

      
