---
published: false
---
## How to set up protractor in docker image running on jenkins

Update: With browsers now supporting headless mode, it gets much easier now.
You should be able to use Chrome in headless mode since version .. and firefox since version 57.

It it doesn't work, you can still use these steps:

1) Create a docker image with `xvfb` and `chrome`
2) Create startup script which starts `xvfb` and `sshd -D`
3) Configure jenkins to use that startup script
Jenkins normally executes `sshd -D` in docker image. We need to execute `xvfb` first.
`xvfb` cannot be executed in `RUN` command (it doesn't support background processes). And we don't have control over `CMD` command.
4) Configure jenkins to use prepend command `DISPLAY=:99`
(or any number used in `xvfb`)
We could potentially start `xvfb` here (and maybe use append command to stop it).
