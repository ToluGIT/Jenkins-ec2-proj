# Jenkins Project - Custom Pipeline

This repository was forked from the original [KodeKloud Jenkins Project](https://github.com/kodekloudhub/course-jenkins-project). Below is a documentation of the changes made to the Jenkinsfile to adapt it for deploying to an EC2 instance and working with Python environments.

## Changes Made to Jenkinsfile

### 1. **Setting up a Virtual Environment**
   - A `python3 -m venv venv` command was added to create a virtual environment within the deployment directory.
   - The environment is activated with `source venv/bin/activate` to ensure that the required Python packages are installed within the isolated environment.
   - The `requirements.txt` file is used to install dependencies using `pip install -r requirements.txt`.

### 2. **Packaging and Zipping Code for Deployment**
   - The code from the directory is zipped using the command:
     ```bash
     zip -r myapp.zip ./* -x '*.git*'
     ```
   - This step ensures that the relevant project files are bundled for deployment to the EC2 instance.

### 3. **Deploying to EC2 with Secure Credentials**
   - The `Deploy to Prod` stage securely transfers the application package and script to the EC2 instance:
     ```bash
     scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/
     scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no script.sh ${username}@${SERVER_IP}:/home/ec2-user/
     ```

   - It then executes the deployment script remotely:
     ```bash
     ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
         chmod +x /home/ec2-user/script.sh
         /home/ec2-user/script.sh
     EOF
     ```

### 4. **Automated Deployment Script Execution**
   - The deployment script automates key tasks:
     - Creates the application directory if it doesn’t exist.
     - Configures the Flask service.
     - Manages the virtual environment.
     - Installs dependencies from `requirements.txt`.
     - Restarts the Flask service to apply the changes.

   - This removes the need to SSH into the EC2 instance for manual setup, ensuring a streamlined deployment process.

### 5. **Restarting the Flask Service**
   - The Flask service is restarted as part of the deployment script to apply the latest code:
     ```bash
     sudo systemctl restart flask.service
     ```

## Conclusion
These changes enhance the pipeline by automating deployment tasks, leveraging Jenkins’ credentials management, and streamlining the setup on the EC2 instance. With the deployment script handling the full configuration, manual SSH access is no longer needed, ensuring secure and reliable continuous deployment.
