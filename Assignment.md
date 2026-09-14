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
GPG verified commit test
### 6. Deploy

The application is actually placed into the required environment.

### 7. Operate

The application runs and serves users.

### 8. Monitor

Engineers continuously check:

* Errors
* CPU
* Memory
* Response time
* Availability

---

## DevOps Infinity Loop

Deployment is **not the end of software development**.

After deployment:

```text
Deploy
  ↓
Monitor
  ↓
Find Problem
  ↓
Fix Code
  ↓
Test
  ↓
Deploy Again
  ↓
Monitor
```

This creates a continuous cycle.

That is why DevOps is often represented as an **infinity loop**.

---

# 3. ENVIRONMENTS

A company normally does not directly take new code from a developer's laptop and put it into production.

Instead, the application moves through different environments.

## Main Environments

```text
LOCAL
  ↓
DEVELOPMENT
  ↓
STAGING
  ↓
PRODUCTION
```

---

## 3.1 Local Environment

This is the developer's own computer.

Example:

```text
Developer Laptop
     ↓
VS Code
     ↓
Application
```

Used by:

**Individual Developer**

Purpose:

* Write code
* Run application
* Debug problems
* Test basic functionality

---

# 3.2 Development Environment

Development environment is a shared environment where developers can test how different parts of the application work together.

Used mainly by:

**Development Team**

Example:

Developer A develops login.

Developer B develops payment.

Both changes can be tested together in the development environment.

---

# 3.3 Staging Environment

Staging is a **pre-production environment**.

It should be as similar to production as possible.

Used by:

* QA
* Developers
* Product team

Purpose:

**Final testing before production.**

Example:

```text
Development
     ↓
Staging
     ↓
Final Testing
     ↓
Production
```

---

# 3.4 Production Environment

Production is the real environment used by actual users.

Example:

When you open a live website, you are interacting with the **production environment**.

Used by:

**Real Users**

---

## Why not directly deploy to Production?

Because bugs can exist.

Without staging:

```text
Developer
    ↓
Production
    ↓
BUG 😨
    ↓
Users affected
```

With staging:

```text
Developer
    ↓
Development
    ↓
Staging
    ↓
Testing
    ↓
Production
```

The intermediate environments act as filters that catch problems before users see them.

---

# 4. VERSION CONTROL & BRANCHING

## What is Version Control?

Version control keeps track of changes made to source code.

The most commonly used tool is:

**Git**

Git allows developers to:

* Save versions
* Create branches
* Merge code
* See previous changes
* Work together

---

# What is a Branch?

A branch is basically a separate line of development.

Example:

```text
main
  |
  |------ feature-login
  |
  |------ feature-payment
```

Developers can work on features without directly disturbing `main`.

---

# Common Branching Strategies

## 4.1 Git Flow

Git Flow generally contains:

```text
main
develop
feature/*
release/*
hotfix/*
```

It is a relatively heavy branching process and is useful in some release-based products.

---

# 4.2 Trunk-Based Development

Developers frequently merge small changes into `main`.

```text
Developer A ─┐
Developer B ─┼──→ main
Developer C ─┘
```

Feature flags can be used to hide unfinished features.

Advantage:

**main**