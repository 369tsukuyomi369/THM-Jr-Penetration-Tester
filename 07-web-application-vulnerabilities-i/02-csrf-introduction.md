# CSRF Introduction

Section 07 · Web Application Vulnerabilities I · Room 02

## Task 1 — Introduction

When an employee signs in to StaffHub, the browser stores a cookie that identifies their session. Depending on the cookie settings, the browser can include it with a request initiated by another site. If StaffHub accepts that request and changes the employee’s email address without the employee choosing to do so, that is cross-site request forgery (CSRF).

### Learning objectives

- Identify which features change account data. In StaffHub, viewing a profile reads existing data, while saving an email address or a profile preference changes it.
- Explain how an HTML form on one site can submit a request to another.
- Describe the role of a CSRF token in protecting an account update and why a weak token can leave that update exposed.

### Lab setup

1. Start the target VM. Use the TryHackMe AttackBox as the second machine, or connect your own machine through the TryHackMe VPN.
2. Give the target about 1–2 minutes to initialize. When using the AttackBox in split view, switch between machines with the tabs at the bottom.
3. If Chrome displays a keyring prompt in the VM, select **Cancel** to continue.

[TryHackMe room](https://tryhackme.com/room/csrfintroduction) · [Back to Section 07](./README.md)
