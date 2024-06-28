# HPC File Permission Handler
The code in this repository consists of two Linux kernel patches and a PAM module that implement restrictions on the file permissions available to users on a HPC system. When used with a smask setting of 0007 on a system which implements the user private group scheme, these changes effectively prevent users sharing data via the filesystem unless they are both members of the same supplemental group.

This code has been developed by the MIT Lincoln Laboratory Supercomputing Center and is released under the GPLv2 and LGPLv2 licenses. The kernel patches are based on the v5.10.x branch of the Linux kernel.

## smask
smask (Security File Mode Mask) is a patch to the Linux kernel. smask is similar to the umask feature of Linux, except it is mandatory instead of advisory. Like with the umask feature, the setting is a mask of permission bits that may not be used by the process. Unlike umask, the smask is enforced on all system calls including explicit requests to change file permissions such as chmod. Processes with CAP_FOWNER in the root namespace are not subject to smask checks.

No privilege is required for a process to set it's own smask and ptrace read access (PTRACE_MODE_READ) is required to set a smask value on another process. However, without CAP_FOWNER in the root namespace the new value may not be less restrictive than the preexisting value. A process's smask value is inherited by child processes.

The current smask can be read or modified using the psudo-file /proc/$$/smask, represented textually as four octal digits. Any value to be set is AND'd against S_IRWXUGO – smask is not intended to work on special bits.

## pam_smask
pam_smask is a PAM session module that can be used to configure smask settings upon user login. Through thoughtful configuration of a PAM stack that includes the pam_smask module, different settings may be applied as needed to different accounts, such as those running systems services which should not be restricted. See config.example.

## smask_relax
smask_relax is a utility intended to be used by HPC support personnel who are not full administrators of the system to relax their smask restrictions. This may be desirable to allow certain personnel to make files available to all users on the system. It is meant to be setuid root, and can optionally be setgid as well. It will set the smask to 0002, drop root privileges, and then launch a new shell based on the SHELL environment variable. If the program was setgid, the group will be retained as the primary group of this new shell and thus be the default group-owner for any files created in this shell. To return to normal operations, the user can exit the subshell. Access to this utility is controlled by standard file permissions, and if needed by multiple sub-groups of users it is expected that multiple copies of this utility may be present with different group-ownership and permissions.

## ACL Must-Be-In-Group
ACL must-be-in-group is a patch to the Linux kernel that restricts the ability of users to set File Access Control Lists (FACLs). With this patch, processes may not set any user-based FACLs nor group-based FACLs unless the group specified in the FACL is the effective group (fsgid) or in the supplemental groups list of the process. Processes with CAP_CHOWN in the root namespace are exempt from this check.

Due to the read-modify-write nature of FACLs, this patch also prevents modifying a FACL that includes a group that the process is not a member of even if no changes were made to that specific entry. In this case, the change is aborted entirely and the FACL is unmodified by the attempt.
