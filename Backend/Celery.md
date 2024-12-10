Attention
Absolutely no Agencies or sub-contractors. Only individual highly qualified developers will be considered. Misrepresentation will be considered as a ground for terminating the contract at any stage without a payment.

Summary
We need to implement a task scheduler for a Python backend with FastAPI using Celery with Redis as the broker, Celery Beat for recurring tasks, and PostgreSQL for persisting tasks, while hosting all components on an AWS Kubernetes cluster.

A. Architecture Overview
The system consists of several key components:
Celery: Distributed task queue for handling asynchronous tasks.
Redis: Message broker to pass messages between Celery workers and task queues.
Celery Beat: Scheduler for periodic tasks.
PostgreSQL: Persistent database for storing task results.
NOTE: PostgreSQL database running on AWS RDS with tables and CRUD API to create, read, update, or delete both simple and recurring reminders already exists and does not need to be set up. Adjustments to make it work with the rest of the services may be required.
Kubernetes: Orchestrating the entire infrastructure on AWS.
TLS/SSL: Encryption in transit for communication between services.
AWS Services Used
AWS EKS: Kubernetes orchestration.
AWS RDS (PostgreSQL): For task persistence with KMS encryption.
AWS ACM: SSL certificates for securing communication.
AWS KMS & S3: Encryption at rest for data using AES 256.
AWS IAM: Role-based access control.
NOTE: Encryption at rest should be enabled as the last step, after debugging and an initial testing, but before the final load testing.
B. Components

1. Celery: Task Queue and Workers
   Celery is a distributed task queue system that allows for asynchronous task execution. In this design, Celery will:
   Handle the dispatching, scheduling, and execution of tasks in the backend.
   Interface with Redis as the message broker and PostgreSQL for persisting task states.
   Use Celery Beat for scheduling repeating tasks.
   Failed tasks should be re-tried until a success or until a max number of tries is reached.
   Key Libraries
   Celery: Core task queue system.
   celery[redis]: Redis backend support.
   celery[sqlalchemy]: PostgreSQL backend support for persisting task metadata.
   celery[beat]: Celery Beat for task scheduling.
   cryptography: For encrypting task payloads.

2. Redis: Broker
   Redis will act as the message broker for Celery. It will:
   Store task payload in a queue.
   Facilitate communication between Celery clients (which send tasks) and Celery workers (which consume and process tasks).
3. PostgreSQL: Task state persistence
   PostgreSQL will be used for persisting the payload and metadata of tasks. This allows querying task history, inspecting task results, and handling persistent task state in a durable manner.
   Key Libraries
   SQLAlchemy: Object-relational mapping (ORM) library used by Celery to interface with PostgreSQL.
4. Celery Beat: Scheduler for Recurring Tasks
   Celery Beat is used to schedule periodic tasks. It stores task schedules in a database, and workers execute them at the specified intervals. It interacts with Celery and Redis to dispatch these tasks automatically without user input.

5. AWS EKS (Elastic Kubernetes Service): Hosting Platform
   All services will be hosted on AWS's managed Kubernetes service (EKS). EKS allows to deploy and manage Kubernetes clusters, providing orchestration for Docker containers.
   Kubernetes (K8s): Container orchestration platform for deploying, managing, and scaling containerized applications.
6. Docker: Containerization
   Each service (Celery, Redis, Celery Beat) will be containerized using Docker. Docker allows to package the application and its dependencies in containers that are portable and consistent across environments.

C. Workflow and Component Architecture

1. Task Flow
   Producer (Celery Client):
   A user or application requests a task to be performed (e.g., offloading data processing, setting up a background job, or scheduling a reminder). Custom tasks are defined on the Python backend
   Tasks can be requested for an immediate execution (offloading of CPU-bound processing), as a one-time scheduled task, or as a recurring task.
   Each task is represented by a JSON. Task data are encapsulated in ‘payload’ field.
   Tasks get persisted in the PostgreSQL database.
   At the scheduled time, the Celery client sends the task to the Redis broker. Unscheduled tasks are sent to Redis immediately.
   Broker (Redis):
   Redis stores the task in a queue and waits for a worker to pick it up.
   Upon a restart/failure, it should be able to restore its state from PostgreSQL - get all tasks with status ‘queued’.
   Consumer (Celery Worker):
   The Celery worker retrieves the task from the Redis queue.
   The worker processes the task and performs the required work.
   After a one-time task is completed, or if the last occurrence of the periodic task is completed, the worker should schedule a delayed removal of the task from PostgreSQL. Removal delay will be part of the task metadata. For instance, removal of reminder tasks requires to be delayed by 24 h to allow a user to read a reminder.
   Workers are deployed as Kubernetes pods on AWS EKS.
   PostgreSQL:
   Stores task payload and metadata (status, timestamp, removal delay, removal time, etc.).
   Persists task states in between Redis restarts by managing task status, i.e. switching from "waiting” to ‘queued’ to “complete”.
   Allows querying for the task history, statuses, and results.
2. Recurring Task Flow (via Celery Beat)
   Celery Beat:
   Stores a schedule of tasks in PostgreSQL.
   Periodically dispatches tasks to Redis according to the schedule (e.g., run every 15 minutes).
   Task Dispatch:
   Celery Beat pushes scheduled tasks to Redis.
   Workers pick up these tasks and execute them similarly to one-time tasks.
3. Kubernetes Deployment Architecture
   AWS EKS Cluster will contain:
   Redis: Running as a single pod or replicated for high availability.
   Celery Workers: Deployed as a Kubernetes Deployment with multiple replicas for horizontal scaling.
   Celery Beat: Deployed as a separate (single) pod, managing the schedule of periodic tasks.
   Kubernetes should have secure access to PostgreSQL database to ensure data persistence. PostgreSQL runs separately on AWS RDS.

4. Other requirements
   The solution must detect scheduling failures and have an automatic retry mechanism.
   The solution needs to be fully integrated with the current application backend (Python + FastAPI).
   If required, adjustments to the existing app’s infrastructure will need to be made to achieve the 10K/sec peak load performance.
   D. Kubernetes and AWS Integration
5. EKS Cluster Setup
   Create an EKS cluster: Use AWS EKS to provision a Kubernetes cluster.
   Install Kubernetes Metrics Server: To monitor the health and resource consumption of the nodes and pods in the cluster.
   Use AWS Certificate Manager (ACM) to manage TLS certificates for encrypting traffic between services.
6. Service Setup Using Kubernetes YAML Files
   Redis Deployment:
   Use Kubernetes YAML to define a Redis pod deployment and service for communication.
   Provision persistent storage for Redis (to persist data across restarts - PostgreSQL).
   Ensure Redis is deployed securely, using encryption (via TLS) for communication between services.
   PostgreSQL:
   PostgreSQL (RDS): Runs as an AWS RDS instance outside of Kubernetes, secured with encryption.
   A Kubernetes Service object will allow workers and clients to communicate with PostgreSQL.
   Enable SSL/TLS for data in transit between Celery workers, API, and the database.
   Celery Worker Deployment:
   Define a Celery worker deployment with replicas based on the desired level of parallelism (need to handle bursts of 10K concurrent tasks per second with a sustained level of 300 tasks/sec).
   Scale workers using Horizontal Pod Autoscaler (HPA) based on task load (e.g., CPU or memory metrics - TBD).
   Celery Beat Deployment:
   Deploy Celery Beat as a separate pod - a single one, to avoid racing conditions while scheduling recurring tasks.
   Attach PostgreSQL as the backend for the scheduler, where it stores periodic task data.
   Existing Application Backend:
   Python backend is already dockerized and deployed to AWS EC2 instance. It needs to be migrated and deployed to the same EKS.
   Existing Web Frontend:
   React Web frontend is already dockerized and deployed to AWS EC2 instance. It needs to be migrated and deployed to the same EKS.
7. Security Workflow
   Encryption in Transit:
   All communications between Celery workers, Redis, PostgreSQL, and the API will use TLS/SSL encryption.
   Redis and PostgreSQL will be configured with SSL certificates for secure communication.
   Encryption at Rest:
   PostgreSQL: Enable AWS KMS for encrypting data at rest in RDS.
   Redis: Configure Elasticache for encryption at rest.
   AWS EBS Volumes: Enable encryption for any volumes attached to the Celery workers.
   All at rest encryptions are done using AES 256.
   VPC Configuration:
   Set up Redis, PostgreSQL, and Celery workers in a private subnet within a VPC to prevent exposure to the public internet.
   Security Groups:
   Configure AWS security groups to restrict access to the relevant services and enforce network-level security.

8. Automation
   Deployment process needs to be automated:
   Terraform script to plan, apply, and destroy AWS EKS cluster and all relevant infrastructure, including all relevant settings and an integration with PostgreSQL.
   Terraform script should include settings for encryption in transit and at rest, and automate all manual infrastructure setup tasks.
   Github Actions CI/CD pipeline for deployment of Celery to EKS, including provisions for auto-scaling Celery workers.
   Github Actions CI/CD pipeline for deployment of Celery Beat to EKS.
   Github Actions CI/CD pipeline for deployment of Redis to EKS.
   Github Actions CI/CD pipeline for deployment of existing Backend to EKS.
   Github Actions CI/CD pipeline for deployment of existing Frontend to EKS.
   E. Testing
   The solution needs to be load tested for scheduling/execution of 300 tasks/sec of sustained load for 1 min.
   The solution needs to be load tested for scheduling/execution of 10K tasks/sec of burst load for 1 sec.
   Any performance bottlenecks should be addressed until the above performance level is reached.
   A combination of custom scripts, JMeter or Locust can be used.
   F. Documentation
   The entire solution needs to be well documented - both in the code, and in a README.md file, or a set of files.
