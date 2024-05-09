# login shell and non-login shell

A **login shell** and a **non-login shell** refer to different ways a shell session can be initiated in Unix-like operating systems, and they determine how certain configuration files are read and executed.

### Login Shell
A login shell is the shell session that is initiated when a user logs into the system. This can happen through:

- Direct login on a terminal (e.g., physical console or terminal connected via SSH).
- Remote login using protocols like SSH.
- Switching to a user with the `su -` command (the hyphen is crucial).

**Characteristics of a login shell:**

1. **Configuration Files:**
   - **/etc/profile**: System-wide configuration script.
   - **~/.bash_profile**: User-specific configuration script.
   - **~/.bash_login**: Another user-specific configuration script.
   - **~/.profile**: Another user-specific configuration script.

   The login shell reads these files in the above order, and stops at the first one that exists and is readable.

2. **Environment Setup:**
   - Typically used to set environment variables, execute startup scripts, and prepare the user environment.
   - It ensures that the session is properly initialized.

### Non-Login Shell
A non-login shell is any shell session that is not a login shell. These are typically started by invoking a terminal emulator from an already logged-in graphical session or by running a new shell instance from an existing shell.

**Characteristics of a non-login shell:**

1. **Configuration Files:**
   - **~/.bashrc**: User-specific configuration script.

   The non-login shell reads this file for configuration.

2. **Shell Invocation:**
   - Used for running scripts or commands in a shell that doesn’t need to reinitialize the user environment.

### Examples
- **Login Shell:**
  ```sh
  ssh user@hostname
  ```
  Logging in via SSH starts a login shell.

  ```sh
  su - username
  ```
  Using `su -` to switch users starts a login shell for the target user.

- **Non-Login Shell:**
  ```sh
  bash
  ```
  Starting a new bash shell from an existing shell session.

  Opening a terminal emulator (like GNOME Terminal or xterm) within a graphical desktop environment.

### Summary
- **Login Shell:**
  - Initializes the user environment by reading /etc/profile, ~/.bash_profile, ~/.bash_login, or ~/.profile.
  - Typically started at system login or remote login.

- **Non-Login Shell:**
  - Reads ~/.bashrc for configuration.
  - Typically started from an existing shell or terminal emulator.

Understanding the distinction is important for configuring your shell environment correctly and ensuring that scripts and commands have the appropriate environment variables and settings.

Reference Links:

- https://en.wikipedia.org/wiki/Unix_shell
- https://en.wikipedia.org/wiki/Su_(Unix)
- https://www.ruanyifeng.com/blog/2013/08/linux_boot_process.html