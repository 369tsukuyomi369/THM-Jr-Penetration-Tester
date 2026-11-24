# CSRF Introduction

Section 07 · Web Application Vulnerabilities I · Room 02

## Task 1 — Introduction

In StaffHub, the room’s employee portal, viewing the email address on a profile and saving a replacement have different consequences. Saving changes the address the portal keeps on file.

A signed-in session tells StaffHub which employee account a request belongs to. Cross-site request forgery (CSRF) occurs when another site can cause the browser to use that session for an action the employee never chose, such as changing their email address.

[TryHackMe room](https://tryhackme.com/room/csrfintroduction) · [Back to Section 07](./README.md)
