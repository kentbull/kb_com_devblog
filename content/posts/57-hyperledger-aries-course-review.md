+++
draft = true
title = "Course Review: edX’s LinuxFoundation LFS173x – Becoming a Hyperledger Aries Developer"
slug = "hyperledger-aries-course-review"
date = "2026-01-04"

[taxonomies]
tags=["course-review", "hyperledger", "aries", "ssi"]

[extra]
comment = true
+++

# Course Review: edX’s LinuxFoundation LFS173x – Becoming a Hyperledger Aries Developer


Course link: [https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS173x+3T2021/home](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS173x+3T2021/home)

Audience: The intended audience for this course is developers who want to learn how to build applications that use self-sovereign identity (SSI) and Trust over IP (ToIP) capabilities.

## Chapter 1

Chapter one is a very light introduction to where Aries fits in the overall ecosystem. The first mention of AnonCreds occurred in this chapter.

## Chapter 2

The demo of verifiable credentials with the British Colombia system and the Conference Book system was really cool! It was awesome to use the Trinsic App. Now I want to go deeper and understand the software architecture of it all. I want to make one and to run everything in my own infrastructure to facilitate this.

Would providing a library that does schema translation between the different credential types and a workflow process wrapper for each type of credential workflow be sufficient to provide interoperability? And could credential selection, as well as cryptographic handshake selection, not be a decision by the user for their desired user experience?

### Core Concepts

Framework– has all core protocol logic– interacts with DIDComm– receives messages and sends webhook notification to controller– typically included as a dependency, not a lot of time is spent here for line of business applications– sometimes the word “agent” refers to the framework and other times to the combination of the framework and the controller. Some renaming is in order here.

Controller– executes business logic– responds to events– makes requests to framework based on business rules– where most of your development time will be spent

Comments

Setting up the demo locally seems like a lot of work. I want one command.Running in Docker didn’t work. It had timeout errors for agent startup.Running locally with Python and libindy didn’t work because the Docker build failed due to indy-sdk [#2412](https://github.com/hyperledger/indy-sdk/issues/2412).The [Running In A Browser](https://github.com/hyperledger/aries-cloudagent-python/tree/main/demo#running-in-a-browser) version with Play with Docker ended up working properly on the second try.

The Aries RFCs [index](https://github.com/hyperledger/aries-rfcs/blob/main/index.md) shows each RFC and the concepts behind it.

Building an Aries agent from scratch would be a really good blog post, especially targeting a relevant Aries Interop Profile (AIP).

Using the VON network was very helpful in getting me up and off the ground for Chapter 2

## Chapter 3

The genesis file makes sense. Can or should it be stored in IPFS?

