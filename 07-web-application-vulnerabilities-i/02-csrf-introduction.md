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

## Task 2 — What is CSRF?

Imagine an employee leaves StaffHub open in one tab and visits another website in a second tab. A page in the second tab can cause the browser to send a request to StaffHub. If the browser’s cookie settings allow it, the request can carry the employee’s session cookie. StaffHub can then recognize the account even though the employee never opened its settings form.

Cross-site request forgery occurs if StaffHub accepts that request and makes a change the employee never chose. An unwanted email change, for example, could send future account notices to a different inbox. The impact depends on what the signed-in account can do. In another application, an unwanted request could change a security preference or initiate a payment.

## Task 3 — Why CSRF Works

A request can start on one site and arrive at another. When cookie rules permit, the browser sends StaffHub’s session cookie directly to StaffHub. The page that started the request never needs to read the cookie or know the employee’s password.

Three conditions explain how an unwanted profile edit could take effect in the room’s example.

1. The employee is still signed in, and the request reaches StaffHub with a valid session cookie.
2. The employee’s account has permission to save the edit, such as changing a profile preference.
3. StaffHub treats the session as sufficient to save the change and skips checking whether the request came from a trusted page.

## Task 4 — Finding CSRF Vulnerabilities

A CSRF review starts with what the server does when someone uses a feature. Opening StaffHub’s settings displays the employee’s email address. Saving a replacement updates the account record. Other examples worth reviewing include changing a password or security preference, linking an account, and adding or removing a payee.

### GET and POST

`GET` is intended for reading data, and forms often use `POST` to save edits. If a handler writes to an account, either method leaves the server with the same question. What evidence ties this update to a trusted page? For an email change, the server could validate a CSRF token tied to the employee’s session before saving the new address.

## Task 5 — Exploitation Using an HTML Form

In the room’s demonstration, an employee is signed in to StaffHub when a form on another page initiates an email change. StaffHub associates the request with the employee’s session and saves the new address without checking where the update originated. The replacement address then appears in StaffHub’s own settings, even though the employee never selected **Update Email**.

StaffHub’s settings form requires an address and checks its format in the browser. An unwanted address can pass both checks. The security decision belongs on the server, where StaffHub can validate a token tied to the employee’s session before saving an edit.

## Task 6 — Exploitation over Weak Tokens

StaffHub checks a hidden token before saving a change to an employee’s role. The value comes from predictable account information, so a match gives the server little evidence that the request came through its own settings page. In the room’s demonstration, an employee interacts with a banner on another page and StaffHub accepts an unintended change that reduces the account’s permissions.

Two employees who share a role should still receive independent, unpredictable CSRF tokens for their sessions. Before saving a role change, the server should check the submitted token against the one it issued for the current session and reject the update if the token is missing or mismatched.

[TryHackMe room](https://tryhackme.com/room/csrfintroduction) · [Back to Section 07](./README.md)
