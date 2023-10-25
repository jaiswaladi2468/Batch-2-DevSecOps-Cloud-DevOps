Assignment that covers basic Linux commands for user management, permissions, and package management:

---

## Linux Commands Assignment

### Part 1: User Management

#### Task 1: Create Users

1. Create a new user named `john` with the following command:
   ```bash
   sudo adduser john
   ```

2. Set a password for the user `john`:
   ```bash
   sudo passwd john
   ```

3. Verify if the user `john` was created successfully.

#### Task 2: Modify User Properties

1. Change the password of the user `john` to a new one.

2. Add `john` to the `sudo` group so that he has administrative privileges.

3. Check if `john` has been added to the `sudo` group.

#### Task 3: Remove Users

1. Delete the user `john`.

2. Verify if the user `john` has been removed.

---

### Part 2: Permissions

#### Task 4: File Permissions

1. Create a new file named `testfile.txt` using the `touch` command.

2. Change the permissions of `testfile.txt` to be readable, writable, and executable by the owner only.

3. Verify the permissions of `testfile.txt` using `ls -l`.

#### Task 5: Directory Permissions

1. Create a new directory named `testdir` using the `mkdir` command.

2. Change the permissions of `testdir` so that it is readable, writable, and executable by everyone.

3. Verify the permissions of `testdir` using `ls -l`.

---

### Part 3: Package Management

#### Task 6: Update and Install Packages

1. Update the package list and upgrade the installed packages using the package manager (e.g., `apt` for Debian-based systems, `yum` for Red Hat-based systems).

2. Install the `tree` package, which provides a directory listing in a tree-like format.

3. Verify if the `tree` package was installed successfully.

#### Task 7: Remove Packages

1. Uninstall the `tree` package.

2. Verify if the `tree` package has been successfully removed.

---

