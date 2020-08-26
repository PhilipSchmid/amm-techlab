---
title: "4.4 Image Deployment"
linkTitle: "Image Deployment"
weight: 440
sectionnumber: 4.4
description: >
  Container Image Deployment.
---


## {{% param sectionnumber %}}.1 Lab


## TODO Lab

* Beschreiben: Private Registry wie und wo muss man das pull secret angeben?
* Auf die Estellung von Services und Routen wurde verzichtet da diese schon in den vorhergehenden Labs ausführlich gezeigt wurde.


{{% alert title="Note" color="primary" %}}
For the Image deployment strategy we don't need to specify a BuildConfig. Thus because we use prebuilt images.
{{% /alert %}}

In this section we cover how to deploy an existing Docker Image from an image registry. Besides we show you create a ImageStream to track changes on the deployed image and trigger an update on the deployment.

Let's start with the deployment configuration

```YAML
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: spring-boot-id
spec:
  replicas: 1
  selector:
    name: spring-boot-id
  template:
    metadata:
      labels:
        name: spring-boot-id
    spec:
      containers:
      - image: 'appuio-spring-boot:latest'
        imagePullPolicy: IfNotPresent
        name: spring-boot-id
        ports:
        - containerPort: 8080
          protocol: TCP
        - containerPort: 9000
          protocol: TCP
  triggers:
  - type: ConfigChange
  - imageChangeParams:
      automatic: true
      containerNames:
      - spring-boot-id
      from:
        kind: ImageStreamTag
        name: spring-boot-id:latest
    type: ImageChange  
  strategy:
    type: Rolling  
```

Next we create the ImageStream definition. The important part is under the `tags` section. There we define a reference to an external Docker registry and define which image to track. Another important field is the import policy. If you query an image from an external registry, you can set scheduled import to true.

```YAML
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  labels:
    app: spring-boot-id
  name: spring-boot-id
spec:
  lookupPolicy:
    local: false
  tags:
  - from:
      kind: DockerImage
      name: appuio/example-spring-boot
    name: latest
    importPolicy:
      scheduled: true
    referencePolicy:
      type: Source
```
