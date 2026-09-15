# Active Directory Security Basics

## Objective

Understand the core security principles of Active Directory before learning attack and defense techniques.


# What is Active Directory Security?

Active Directory Security ensures that only authorized users can access authorized resources within a Windows domain environment.

Its purpose is to protect users, computers, services, and sensitive organizational data.

# Core Security Principles

## Authentication

Authentication verifies the identity of a user or computer.

Question answered:

Who are you?

Example:

A user enters their domain username and password.

The Domain Controller verifies the credentials.

If valid, the user is authenticated.

## Authorization

Authorization determines what an authenticated user is allowed to access.

Question answered:

What are you allowed to access?

Example:

A Finance employee cannot access the HR shared folder unless the required permissions are granted.

## Auditing

Auditing records security-related events such as:

- Successful logins
- Failed logins
- Password changes
- File access
- Group membership changes

These logs help defenders detect suspicious activity.

## Administration

Administration involves managing:

- Users
- Groups
- Computers
- Organizational Units
- Group Policies
- Permissions

# Why is Active Directory Security Important?

Organizations rely on Active Directory to centrally manage authentication, authorization, and access to critical business resources.

If Active Directory is compromised, attackers may gain access to sensitive systems and data across the enterprise.

# Summary

Active Directory Security is built on four major principles:

- Authentication
- Authorization
- Auditing
- Administration

Understanding these concepts is essential before learning Active Directory attack techniques.