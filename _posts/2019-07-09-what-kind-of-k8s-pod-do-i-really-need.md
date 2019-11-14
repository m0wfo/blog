---
id: 896
title: What kind of K8s pod do I really need?
date: 2019-07-09T16:50:20+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=896
permalink: /2019/07/09/what-kind-of-k8s-pod-do-i-really-need/
categories:
  - Uncategorized
---
I&#8217;m sure you&#8217;ve been there; by some twist of fate you&#8217;ve ended up working in a place where Kubernetes has appeared on the radar and now you&#8217;ve been dropped into a brave new world where pushing a code change involves something called a pod. More nomenclature. Shit just got real. While there&#8217;s no shortage of detailed and nuanced explorations of the plethora of resource types in both the <a href="https://kubernetes.io/docs/tutorials/kubernetes-basics/" target="_blank">official Kubernetes documentation</a> and elsewhere on the internet, it&#8217;s hard (in words at least) to give a cohesive top-down view of how all these resources fit together, and why you&#8217;d choose one over the other.

With that in mind, I went and did a thing: (hope it helps somebody)

[<img class="alignnone size-full wp-image-897" src="https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2019/07/k8s-decision-tree.png" alt="k8s-decision-tree" width="823" height="913" />](https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2019/07/k8s-decision-tree.png)