---
layout: ../../layouts/BlogPostLayout.astro
title: 'k3d - Kubernetes auf Dockerbasis'
date: 2024-05-22
description: 'K3d ist ein Tool um einen k3s-Cluster innerhalb von Docker laufen zu lassen.'
author: 'Tony Nutzmann'
---

## Vorraussetzung
- Docker installiert
- kubectl installiert

## Instalation
### macOS
```shell
brew install k3d
```
### Linux
```shell
#wget:
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
#curl:
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```
## Run
### Simpel

```shell
k3d cluster create
```
Mit diesem einzigen Befehl wird ein K3s-Cluster mit zwei Containern erzeugt: Eine Kubernetes Control-Plane Node (Server) und ein Loadbalancer (Serverlb) davor. Beide werden in ein dediziertes Docker-Netzwerk eingebunden und die Kubernetes-API wird auf einem zufällig gewählten freien Port des Docker-Hosts bereitgestellt. Außerdem wird ein Docker-Volume für den Import von Container-Images erstellt.
### Anspruchsvoller

```shell
k3d cluster create mycluster --api-port 127.0.0.1:6445 --servers 3 --agents 2 --volume '/home/me/mycode:/code@agent[*]' --port '8080:80@loadbalancer'
```

Dieser Befehl erstellt einen k3s Cluster mit 6 Containern:

- 1 Load-Balancer
- 3 Server (Control-Plane Nodes)
- 2 Agents (früher Worker Nodes)

Mit dem _--api-port 127.0.0.1:6445_ wird k3s angewiesen, den Kubernetes API Port (_6443_ intern) auf _127.0.0.1/localhost_'s port _6445_ zu mappen. Das bedeutet, dass dieser Verbindungs-String in Ihrer Kubeconfig sein wird: „Server: https://127.0.0.1:6445“, um sich mit diesem Cluster zu verbinden.

Dieser Port wird vom Load Balancer auf Ihr Hostsystem abgebildet. Von dort aus werden die Anfragen an Ihre Serverknoten (Agents) weitergeleitet, wodurch eine Produktionsumgebung simuliert wird, in der auch Serverknoten (Agents) ausfallen können und Sie auf einen anderen Server ausweichen müssen.

Der _--volume /home/me/mycode:/code@agent[*]_ Bind mountet das lokale Verzeichnis _/home/me/mycode_ in den Pfad _/code_ innerhalb aller (_[*]_) Agents. Ersetze _*_ durch einen Index (hier: 0 oder 1), um es nur in einen von ihnen zu mounten.

Die Spezifikation, die k3d mitteilt, auf welcher Node es das Volume mounten soll, wird „Node-Filter“ genannt und wird auch für andere Flags verwendet, wie z.B. das _--port_-Flag für Port-Zuordnungen.

_--port '8080:80@loadbalancer'_ bedeutet, dass der Port _8080_ Ihres lokalen Hosts auf den Port _80_ des Loadbalancers (serverlb) abgebildet wird, der zur Weiterleitung des HTTP-Eingangsverkehrs zu Ihrem Cluster verwendet werden kann.
