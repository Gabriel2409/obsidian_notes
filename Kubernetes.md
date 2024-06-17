 #k8s



 - [[Kubernetes - Installation]]
 - [[Kubernetes - Creating local clusters]]
 - [[Kubernetes - Production clusters]]
 - [[Kubernetes - kubectl]]
 - [[Kubernetes - Web interface]]
 - [[Kubernetes - under the hood]]

 - [[Kubernetes - Pods]]
 - [[Kubernetes - Services]]
 - [[Kubernetes - Deployments]]
 - [[Kubernetes - Namespaces]]
 - [[Kubernetes - ConfigMaps]]
 - [[Kubernetes - Secrets]]
 - [[Kubernetes - initContainers]]
 - [[Kubernetes - Security]]
 - [[Kubernetes - DaemonSets]]
 - [[Kubernetes - Jobs and CronJobs]]
 - [[Kubernetes - Networking]]
 - [[Kubernetes - Ingress]]
 - [[Kubernetes - Stateful Workload]]
 - [[Kubernetes - Cluster maintenance]]


 - [[Kubernetes - Helm]]
 - [[Kubernetes - Azure specifics]]





# Summary of useful concepts

- Container:

  - is a process
  - visible from host machine
  - with limited vision of the system
  - with limited accessible resources
  - combines two linux primitives: namespace and control groups (cgroups)

- Docker:

  - simplifies use of containers
  - brings concept of images = packaging of an app and its dependencies instantiated in a container that can be deployed in several environments
  - see http://hub.docker.com

- Microservices architecture:

  - application separated in several services
  - each service can be developed / updated independently
  - well defined interface is needed for communication between services
  - containers are particularly adapted for Microservices
  - complexity is moved to the orchestration of full application

- Cloud native app:

  - Microservices oriented application
  - packaged in containers
  - Dynamic orchestration
  - a lot of project supported by Cloud Native Computing Foundation: https://www.cncf.io (kubernetes for ex)

- Devops

  - goal: minimise time to deliver a feature
  - Frequent deployments
  - With tested code
  - With automated processes
  - Short improvement loop
  - Example:
    - infrastructure (aws, digitalOcean, azure)
    - provisionning (terraform, CloudFormation)
    - build (github, gitlab, docker)
    - test (selenium, jenkins, circleci)
    - deploy (Ansible, CHEF, registry docker)
    - run (kubernetes, Swarm)
    - mesure (prometheus, elastic)

- Kubernetes role:
  - deal with applications running in containers (deployment, scaling, self-healing)
  - ensures that application is running as intended (reconciliation loop)
  - can deal with stateless and stateful apps
  - can handle secrets and configs
  - can run long-running processes or batch jobs
  - RBAC (role based access control)
