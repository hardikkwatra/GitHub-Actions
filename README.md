#### Task 1

```markdown
# Testing the PodState Script

Follow these steps to test the `PodState.py` script:

## 1. Install Docker

Make sure Docker is installed on your system. Follow these steps to install Docker:

### For Ubuntu:

1. **Update the apt package index and install packages to allow apt to use a repository over HTTPS:**
   ```sh
   sudo apt-get update
   sudo apt-get install \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
   ```

2. **Add Docker's official GPG key:**
   ```sh
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

3. **Set up the stable repository:**
   ```sh
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

4. **Install Docker Engine:**
   ```sh
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

5. **Verify that Docker Engine is installed correctly by running the hello-world image:**
   ```sh
   sudo docker run hello-world
   ```

## 2. Install Required Libraries

Make sure you have the necessary Python libraries installed:

```sh
pip install kubernetes
```

## 3. Create and Edit the Python Script

Create a new file and paste the script into it. Let's call the file `PodState.py`:

```sh
nano PodState.py
```
Paste your script into the file and save it.

## 4. Setup and Start Minikube (if not already running)

If you're using Minikube, start it:

```sh
minikube start
```

## 5. Ensure Kubeconfig is Correctly Set

Verify that your kubeconfig is correctly configured to point to your cluster:

```sh
kubectl config view
```

## 6. Check the Namespace

Ensure the namespace you are using (default is `default`) exists and has some pods. You can list all namespaces with:

```sh
kubectl get namespaces
```

## 7. List Pods in the Namespace

Check if there are any pods in the specified namespace:

```sh
kubectl get pods -n default
```

## 8. Scale Up/Down Your Deployment (Optional)

If you need to scale your deployment to generate pods:

```sh
kubectl scale deployment flask-app --replicas=1 -n default
```

or

```sh
kubectl scale deployment flask-app --replicas=0 -n default
```

## 9. Run the Script

Now you can run your Python script to check the pod statuses and fetch logs if necessary:

```sh
python3 PodState.py
```

## 10. Check the Log File

After running the script, you can check the `all-pods-logs.txt` file to verify the output:

```sh
cat all-pods-logs.txt
```


#### Task 2

### Setting Up GitHub Actions Workflow

1. **Create the Workflow File**: Ensure you have created a workflow file in your repository at `.github/workflows/deploy.yml`.
2. **Configure Your Docker Hub Credentials**: 
   - Go to your repository settings on GitHub.
   - Navigate to `Settings` > `Secrets` and add `DOCKER_USERNAME` and `DOCKER_PASSWORD` with your Docker Hub credentials.
3. **Push Changes**: Push your code to the `main` branch. This will trigger the GitHub Actions workflow you have set up.

### Running the Linux Shell Script

1. **Create the Shell Script**: Ensure you have created a shell script named `setup.sh` in your project directory.
2. **Make the Script Executable**: 
   ```bash
   chmod +x setup.sh
   ```
3. **Execute the Script**:
   ```bash
   ./setup.sh
   ```


