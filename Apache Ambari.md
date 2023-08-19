#hadoop

## First connection with hortonworks sandbox

- Connect on localhost:1080 with `maria_dev` / `maria_dev`
- To update admin password,
  - go to localhost:4200 and login with `root`/`hadoop` and change root password (for ex `N3wP4ss!`)
  - change admin password with `ambari-admin-password-reset`, for ex `Unlimit3dPwr!`
  - Then go to localhost:1080 to connect with either `maria_dev` profile or `admin` profile

## Overview

- Apache Ambari is an open-source graphical administration and management tool for Apache [[Hadoop]] clusters. It provides a web-based interface that simplifies the installation, configuration, monitoring, and management of Hadoop clusters.
- There is also a REST api

## Main functionalities

- Dashboard: Summary of the cluster
- Services: shows all hadoop services
- Hosts: lists all servers of the cluster
- Alerts: see logs and alerts
