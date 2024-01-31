---
title: 'Directus as a Kubernetes puzzle'
description: '120-160 characters'
author:
  name: ''
  avatar_file_name: 'add-to-directory'
---

## Introduction

Originated in Googles project Borg, which started in 2003, Kubernetes is today the most popular open-source container orchestration system, and it automates deployments, scaling and management of applications. And Kubernetes is in many ways, a complicated beast to handle, like a puzzle with many pieces. If you never used Kubernetes before, it is maybe not the easiest way forward for your self-hosted Directus project. But there a lot of gains with Kubernetes running an application like Directus.

If you never used Kubernetes, you may be surprised that you very often already have it on your development computer. Kubernetes, in a simple flavour is for instance shipped with Docker Desktop for Mac. So the place to test out Kubernetes itself it is very often nearby.

I set up my own Kubernetes cluster on Amazon some year ago - before they had their offering EKS (Elastic Kubernetes Service). I learned a lot, but I would not recommend my worst enemy to do it. Today there are a lot of ready Kubernetes solutions for you to choose from - like EKS, GKE (Google Kubernetes Engine), Microsoft Azure Kubernetes Service (AKS) etc. And if you want more control you can setup Kubernetes with [Rancher](https://www.rancher.com/) (which we mostly do these days).

I will do a walk through of some of the basic pieces of the Kubernetes puzzle, and what they mean in a Directus context.

## Pods

A pod is what is running a docker container, a pod could be Directus, MySQL or Redis. A pod is normaly created by a deployment or a statefulset, but a pod could also be created without them - but if it crashes, or is deleted, it doesn't come back - it's like when you start a docker container with `docker run myimage`.

## Deployments

A deployment is a way to describe the pod(s) you want to run.

## Statefulsets

## Replicasets

## Volumes

## Jobs and CronJobs

## Configmaps

## Secrets

## Services

## Ingresses

## HPA - Horizontal Pod Autoscaler

## Helm charts

Helm charts is a way to package all the needed pieces above, and there a lot of ready ones out there. As an employee at [Digitalist Open Tech](), I setup one for Directus, [that you can try out](https://artifacthub.io/packages/helm/directus/directus).

## Summary
