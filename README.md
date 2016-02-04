# secmodel-pledge

Idea for a [secmodel(9)](http://netbsd.gw.com/cgi-bin/man-cgi?secmodel) implementation that approximates OpenBSD's [pledge(2)](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man2/pledge.2).

### Goal

Apply the principle of least privilege to apps without code changes.

### Observations

* Apps signal a switch from *privileged* to *unprivileged* state by calling a few special, privileged syscalls. These includes variants of setuid(2) and setgid(2), chroot(2), etc.

* Values passed to the above syscalls are unique across apps. There are dedicated users and paths for each app. For example, `_smtpd` and `/var/chroot/ntpd`.

### Theory

* Associate a unique identifier, like a user or a path, with a set of *promises*, like `dpath` or `settime`.

* When the kernel detects a switch to that unique identifier, it limits the app by preventing it from doing things that should be prevented by the associated *promise*.

### Notes

* Not all *promises* map to privileged or even full operations. Some enforce limits, even on *unprivileged* operations, based on parameters.

* Pay special attention to the `id` promise.

### Configuration

`/etc/pledge.conf`:

```json
{
	"pledge": [
		{
			"user": "_foo",
			"promises": "foo,bar"
		},
		{
			"group": "_foo",
			"promises": "foo,bar"
		},
		{
			"chroot": "/path/to/foo",
			"promises": "foo,bar"
		}
	]
}
```

### Extend

This idea is very similar to role-based access controls, only that it focuses on apps rather than users. One could extend the configuration to allow for `privileges` in addition to `promises`, which could be more intuitive when limiting "real" users.
