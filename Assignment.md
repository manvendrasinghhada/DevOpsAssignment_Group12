# DEPLOYMENT & DEVOPS

## Detailed Notes — Easy to Understand + Exam Oriented

---

# 1. INTRODUCTION TO DEPLOYMENT

## What is Deployment?

*Deployment* means taking an application that has been developed and tested and making it available for real users.

### Simple Example

Suppose you create a website on your laptop.

Initially:

*Developer's Laptop → Application*

But users cannot access it.

You need to put the application on a server/cloud:

*Developer's Laptop → Server/Cloud → Real Users*

This process of moving the application to the server and making it available is called *Deployment*.

---

## Why is Deployment Important?

Deployment should be:

1. *Safe* — should not break the existing application.
2. *Repeatable* — same process should work every time.
3. *Automated* — minimum manual work.
4. *Fast* — new features should reach users quickly.
5. *Reversible* — if something goes wrong, we should be able to go back.

Modern organizations generally use automated *CI/CD pipelines* instead of manually copying files to servers.

---

# 2. WHERE DEPLOYMENT FITS IN SDLC

## What is SDLC?

SDLC = *Software Development Life Cycle*

A simplified flow is:

*Plan → Code → Build → Test → Release → Deploy → Operate → Monitor*

### Understand Each Step

### 1. Plan

Decide:

* What feature is required?
* What problem are we solving?
* What should the application do?

### 2. Code

Developers write the actual program.

### 3. Build

Source code is converted into a usable application.

For example:

text
Source Code
     ↓
Compile / Build
     ↓
Application


### 4. Test

Check whether the application works correctly.

Examples:

* Unit testing
* Integration testing
* Functional testing

### 5. Release

The tested version is prepared for users.