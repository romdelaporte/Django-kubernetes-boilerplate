# Django Boilerplate for DigitalOcean's Kubernetes Cluster

### Tech Stack:
Stack           | Version
--------------- | -------------
Django          | 4.2
Postgresql      | Database Cluster (Postgres v14) from DigitalOcean
Static Storage  | DigitalOcean Spaces
Ingress NGINX   | DigitalOcean panel installation with cert
Kubernetes      | DigitalOcean cluster with 3 pods



### Instructions:
1. Configuration: Create an env file with the corresponding key value pairs
2. Building Docker images in local machine
    - `docker build -t <tagname>:<version> .`
    - `docker run -dp 80:8000 --env-file <env_file>`
    - Remember to allow your database cluster incoming connections from your local machine
    - `docker run --env-file <env_file> <image> sh -c "python manage.py makemigrations && python manage.py migrate"`
    - `docker run -it --env-file <env_file> <image> sh`
        - `python manage.py createsuperuser`
        - `docker run --env-file <env_file> <image> sh -c "python manage.py collectstatic --noinput"`
    - `docker run --env-file <env_file> -p 80:8000 <image>`
3. Pushing the Docker image to Hub
    - `docker login`
    - `docker tag polls:latest <your_dockerhub_username>/<your_dockerhub_repo_name>:latest`
    - `docker push <your_dockerhub_username>/<tagged_image>:latest`
4. Setting up Kubernetes ConfigMap
    - Add corresponding values to `yaml/backend-configmap.yaml`
    - `kubectl apply -f yaml/backend-configmap.yaml`
5. Setting up secrets
    - Add corresponding values to `yaml/backend-secret`
    - `kubectl create secret generic backend-secret --from-env-file=yaml/backend-secret`
    - `kubectl describe secret backend-secret`
6. Django deployment
    - Edit `yaml/backend-deployment.yaml` and add the corresponding values to point to your docker hub image
    - `kubectl apply -f yaml/backend-deployment.yaml`
    - `kubectl get deploy backend-app`
    - `kubectl describe deploy`
    - `kubectl get pod`
7. Allowing external access using a service
    - `kubectl apply -f yaml/backend-svc.yaml`
    - `kubectl get svc backend`
    - `kubectl get node -o wide`
    - Access `http://<your_ip>:<service_port>/admin` to check everything is running properly
8. Configuring HTTPs using NGINX ingress and cert-manager
    - Configure your domain by adding an A record pointing to your load balancer's IP
    - Edit `yaml/backend-ingress.yaml` and add your domain
    - `kubectl apply -f yaml/backend-ingress.yaml`
    - `kubectl describe ingress backend-ingress`
    - `kubectl describe certificate backend-tls`
    - `kubectl describe ingress backend-ingress`
    - Modify `yaml/backend-svc.yaml` and in spect.type, change `NodePort` to `ClusterIP`
    - `kubectl apply -f yaml/backend-svc.yaml --force`
    - `kubectl get svc backend`
    - Now you can access from your modern browser using secure connection