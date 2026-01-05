+++
title = "Vessel"
description = "Simple NGINX-based ACDC credential schema hosting server"
weight = 3
date = 2024-01-15

[extra]
github = "https://github.com/TetraVeda/vessel"
demo = "https://vessel.tetraveda.com"
tags = ["nginx", "docker", "acdc", "keri", "schema"]
+++

A lightweight Docker container hosting static ACDC schemas via NGINX. Vessel handles the full schema lifecycle: raw schemas are SAIDified and renamed based on their Self-Addressing Identifier (SAID) for proper OOBI resolution. Includes schema rules extraction and automated deployment workflows for credential schema hosting.

