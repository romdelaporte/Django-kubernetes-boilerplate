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
    i. `docker build -t <tagname>:<version> .`
    ii. `docker run -dp 80:8000 --env-file <env_file>`
    iii. Remember to allow your database cluster incoming connections from your local machine
    iv. `docker run --env-file <env_file> <image> sh -c "python manage.py makemigrations && python manage.py migrate"`
    v. `docker run -it --env-file <env_file> <image> sh`
        1. `python manage.py createsuperuser`
        2. `docker run --env-file <env_file> <image> sh -c "python manage.py collectstatic --noinput"`
    vi. `docker run --env-file <env_file> -p 80:8000 <image>`
3. Pushing the Docker image to Hub
    i. `docker login`
    ii. `docker tag polls:latest <your_dockerhub_username>/<your_dockerhub_repo_name>:latest`
    iii. `docker push <your_dockerhub_username>/<tagged_image>:latest`
4. Setting up Kubernetes ConfigMap
    i. Add corresponding values to `yaml/backend-configmap.yaml`
    ii. `kubectl apply -f yaml/backend-configmap.yaml`
5. Setting up secrets
    i. Add corresponding values to `yaml/backend-secret`
    ii. `kubectl create secret generic backend-secret --from-env-file=yaml/backend-secret`
    iii. `kubectl describe secret backend-secret`
6. Django deployment
    i. Edit `yaml/backend-deployment.yaml` and add the corresponding values to point to your docker hub image
    ii. `kubectl apply -f yaml/backend-deployment.yaml`
    iii. `kubectl get deploy backend-app`
    iv. `kubectl describe deploy`
    v. `kubectl get pod`
7. Allowing external access using a service
    i. `kubectl apply -f yaml/backend-svc.yaml`
    ii. `kubectl get svc backend`
    iii. `kubectl get node -o wide`
    iv. Access `http://<your_ip>:<service_port>/admin` to check everything is running properly
8. Configuring HTTPs using NGINX ingress and cert-manager
    i. Configure your domain by adding an A record pointing to your load balancer's IP
    ii. Edit `yaml/backend-ingress.yaml` and add your domain
    iii. `kubectl apply -f yaml/backend-ingress.yaml`
    iv. `kubectl describe ingress backend-ingress`
    v. `kubectl describe certificate backend-tls`
    vi. `kubectl describe ingress backend-ingress`
    vii. Modify `yaml/backend-svc.yaml` and in spect.type, change `NodePort` to `ClusterIP`
    viii. `kubectl apply -f yaml/backend-svc.yaml --force`
    ix. `kubectl get svc backend`
    x. Now you can access from your modern browser using secure connection