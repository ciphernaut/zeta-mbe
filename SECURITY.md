# Security

Report a vulnerability privately through GitHub's private vulnerability reporting, at
<https://github.com/ciphernaut/zeta-mbe/security/advisories/new> (the same form as **Security**,
then **Report a vulnerability**, on the repository page; GitHub asks you to sign in). Only the
maintainer sees the report. Please do not open a public issue for it. Say what an attacker can
reach and do, with the commit and the steps to reproduce. This is a one-person project, so allow
a few days for an answer.

`zeta-mbe` reads untrusted input: frames demodulated off the air. A crash or memory error on any
input is in scope. A weakness in mbelib-neo itself belongs with
[its authors](https://github.com/arancormonk/mbelib-neo).
