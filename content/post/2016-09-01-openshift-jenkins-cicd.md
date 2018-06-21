---
author: Kent Hua
comments: true
date: "2016-09-01T11:00:00Z"
tags: [ocp, openshift, jenkins, cicd, redhat, java, pipeline, source]
title: OpenShift Container Platform 3.2 CI/CD with Jenkins
---

I recently jumped into Jenkins pipelines.  Though OpenShift Container Platform 3.2 currently only includes Jenkins 1.x and not Jenkins 2.x, we are still able to leverage the new pipeline plugin.  The new pipeline plugin leverages a Jenkinsfile based on groovy to define the pipeline steps, called stages.  The previous workflow pipeline which connected multiple jobs together.  

You can can define a Jenkinsfile locally within the Jenkins job itself or you can include it in your git project so that the job is dynamic and specific only to that particular project.

I [forked](https://github.com/kenthua/summit2016-ose-cicd) a project from Andrew Block at Red Hat and used it do a deploy a pre-provisioned demo on Red Hat's demo system, for use my partners and associates.  This particular project leverages Jenkinsfile pipeline to deploy a Wildfly Swarm application across various environments within OpenShift.  Fully automated, based on changes in the git repo source.  A visual [demonstration](https://vimeo.com/178679773) of this project is available.

Leveraging the underlying components of the previous demo, Jenkins, gogs, and nexus.  I built two other [demos](https://github.com/kenthua/openshift/tree/master/demo/cicd) which demonstrate a pipeline using OpenShift's source builder and OpenShift's binary builder.  Some of the [concepts](https://github.com/OpenShiftDemos/openshift-cd-demo) of this demo came from another individual from Red hat Siamak.  These demos do require that you already have the previous project environment already in place.  Running this setup script will then call ansible playbooks to prepare the environment for the two new Jenkins pipelines.

The goal was to be able to demonstrate that one can build code outside of OpenShift and be able to run it, while also being able to have OpenShift build the source for you.  Each has it's own advantages and disadvantages, it just depends on your environment.  

