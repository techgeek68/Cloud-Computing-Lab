# Deploying Web Services on AWS Using Elastic Beanstalk and API Based Architecture

---

**Objectives**
  - Deploy a RESTful web service on AWS Elastic Beanstalk
  - Understand web service concepts, including REST, APIs, and endpoints
  - Configure environment variables and application settings
  - Implement monitoring and logging for web applications
  - Explore AWS App Runner as an alternative deployment platform

---

**Theory**

Web services are software systems designed to support interoperable, machine-to-machine communication over a network. In cloud computing, they form the backbone of SaaS applications and microservices architectures.

**REST (Representational State Transfer)** is an architectural style that uses standard HTTP methods — GET, POST, PUT, and DELETE — to interact with resources identified by URLs. RESTful web services are stateless, cacheable, and horizontally scalable by design.


**AWS Deployment Platforms**

**Elastic Beanstalk** is a Platform-as-a-Service (PaaS) offering that supports deploying web applications written in Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker. It handles infrastructure provisioning automatically.

**AWS App Runner** is a fully managed service for deploying containerized web apps and APIs with minimal configuration.

**Amazon ECS and EKS** provide container orchestration for microservices at scale.

**AWS Amplify** is designed for full-stack web and mobile application hosting.

These platforms are AWS equivalents to Google App Engine, Microsoft Azure App Service, and Aneka, all of which represent the PaaS layer of cloud computing.

---

**Procedure**

**Step 1: Create the Flask Application Locally**

1. On your local machine, create a project folder and add the following two files.

`application.py`:

```python name=application.py
from flask import Flask, jsonify, request
import os

application = Flask(__name__)

@application.route('/')
def home():
    return jsonify({
        "service": "Cloud Computing Lab - Web Service",
        "platform": "AWS Elastic Beanstalk",
        "environment": os.environ.get('ENVIRONMENT', 'development'),
        "endpoints": ["/", "/api/students", "/api/courses", "/health"]
    })

@application.route('/api/students', methods=['GET'])
def get_students():
    students = [
        {"id": 1, "name": "Alice", "course": "Cloud Computing"},
        {"id": 2, "name": "Bob", "course": "Cloud Computing"},
        {"id": 3, "name": "Charlie", "course": "Cloud Computing"}
    ]
    return jsonify({"students": students, "count": len(students)})

@application.route('/api/courses', methods=['GET'])
def get_courses():
    courses = [
        {"id": 1, "name": "Cloud Computing", "units": 6},
        {"id": 2, "name": "Cloud Security", "units": 5},
        {"id": 3, "name": "Virtualization", "units": 10}
    ]
    return jsonify({"courses": courses})

@application.route('/health')
def health():
    return jsonify({"status": "healthy"}), 200

if __name__ == '__main__':
    application.run(debug=True, host='0.0.0.0', port=5000)
```

`requirements.txt`:

```text name=requirements.txt
flask==3.0.0
```

2. Take a screenshot of the project folder showing both files.

**Step 2: Create the Deployment Package**

1. Open a terminal in your project folder and run the following command to create a ZIP archive:

```bash name=package.sh
zip -r cloud-web-service.zip application.py requirements.txt
```

2. Confirm that `cloud-web-service.zip` was created in the same directory.

**Step 3: Create an Elastic Beanstalk Application**

1. Open the AWS Management Console and navigate to Elastic Beanstalk.
2. Click **Create application**.
3. Set the application name to `cloud-web-service`.
4. For the platform, select **Python** and choose the **Python 3.9+** branch.
5. Under application code, select **Upload your code** and upload the `cloud-web-service.zip` file you just created.
6. Take a screenshot of the application creation page with the ZIP file uploaded.

**Step 4: Configure the Environment**

1. On the environment configuration page, set the capacity to **Single instance** (appropriate for a lab environment).
2. Under environment properties, add the variable `ENVIRONMENT` with the value `production`.
3. Enable **Enhanced health reporting** under the monitoring section.
4. Take a screenshot of the environment configuration page.

**Step 5: Deploy the Application**

1. Click **Create application** and wait for Elastic Beanstalk to provision the environment and deploy the code. This typically takes 5 to 10 minutes.
2. Once deployment is complete, confirm that the environment health status shows **Ok** in green.
3. Take a screenshot of the Elastic Beanstalk dashboard showing the green health status.

**Step 6: Test All API Endpoints**

1. On the Elastic Beanstalk environment page, copy the environment URL.
2. Open a browser and navigate to the root URL: `http://<env-name>.<region>.elasticbeanstalk.com/`
3. Test the following endpoints by appending each path to the base URL:
   - `/api/students` — should return a JSON list of students
   - `/api/courses` — should return a JSON list of courses
   - `/health` — should return `{"status": "healthy"}` with a 200 status code
4. Take a screenshot of the browser showing the JSON response from `/api/students`.

**Step 7: View Application Logs**

1. In the Elastic Beanstalk console, open your environment and go to the **Logs** section.
2. Click **Request logs** and select **Last 100 lines**.
3. Review the log output to confirm that your API requests were handled and recorded.
4. Take a screenshot of the log output.

**Step 8: Explore AWS App Runner**

1. Navigate to the App Runner console and click **Create service**.
2. Review the available source options, which include a source code repository or a container image.
3. Browse the configuration options for auto-scaling, health checks, and custom domains to understand how App Runner differs from Elastic Beanstalk in its level of management and abstraction.
4. Take a screenshot of the App Runner service creation page for comparison.

**Step 9: Set Up a CloudWatch Alarm**

1. Navigate to the CloudWatch console and go to **Alarms**.
2. Click **Create alarm** and select a metric.
3. Choose the Elastic Beanstalk metric for `ApplicationRequests5xx`, set the threshold to greater than 5 over a 5-minute period.
4. Configure an action to send a notification via an SNS topic (create a new topic if needed and enter your email address to receive alerts).
5. Name the alarm and complete the setup.
6. Take a screenshot of the alarm configuration.

**Step 10: Update the Application**

1. Open `application.py` locally and add a new endpoint. For example:

```python name=application.py
@application.route('/api/info', methods=['GET'])
def get_info():
    return jsonify({"lab": "Lab 10", "topic": "Web Services"})
```

2. Repackage the application:

```bash name=repackage.sh
zip -r cloud-web-service-v2.zip application.py requirements.txt
```

3. In the Elastic Beanstalk console, go to your environment and click **Upload and deploy**.
4. Upload the new ZIP file and confirm the deployment.
5. Watch the deployment events to observe the rolling update process.
6. Take a screenshot of the deployment events showing a successful update.

---

**Results**

- The RESTful web service was deployed successfully on Elastic Beanstalk and all endpoints returned correct JSON responses.
- Environment variables and enhanced monitoring were configured through the Elastic Beanstalk console.
- Application updates were deployed using the upload and deploy workflow, with Elastic Beanstalk handling the rolling update automatically.
- A CloudWatch alarm was configured to detect elevated 5xx error rates and send notifications.
- AWS App Runner was reviewed as a fully managed alternative to Elastic Beanstalk.

---

**Discussion and Conclusion**

This lab demonstrated how to deploy and manage a RESTful web service on AWS using Elastic Beanstalk. As a PaaS offering, Elastic Beanstalk abstracts the underlying infrastructure so developers can concentrate on application code. Provisioning, load balancing, scaling, and health monitoring are all handled by the platform automatically.

The RESTful API design used here follows the same web service standards found across all major cloud platforms. Google App Engine, Microsoft Azure App Service, and Aneka serve the same fundamental purpose as Elastic Beanstalk, each offering a managed environment where developers deploy code without managing servers directly.

Comparing Elastic Beanstalk with App Runner illustrates the range within the PaaS category. Elastic Beanstalk offers more configuration flexibility, while App Runner is more opinionated and fully managed, requiring even less setup. Both are suitable depending on how much control versus simplicity a team needs.

CloudWatch alarms add a proactive monitoring layer that is essential in any production deployment. Combined with application logs, they provide the visibility needed to detect and respond to issues quickly. These same patterns apply whether the service is powering a data processing API for scientific workloads or a consumer-facing web application.

---
