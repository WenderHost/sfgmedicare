# nginx deploy: sfgmedicare.com

This directory mirrors the live nginx config for sfgmedicare.com at `/etc/nginx/sites-available/sfgmedicare.com/` on venus.wenmarkdigital.com (206.189.224.134). It was set up with spinupwp-nginx-deploy (`/Users/mwender/Dropbox/Projects/spinupwp-nginx-deploy`), which holds the full documentation.

## Commands

```bash
deploy/nginx-deploy diff    # show how the server differs from the working tree (changes nothing)
deploy/nginx-deploy pull    # overwrite the working tree with the server's files, then review with git diff
deploy/nginx-deploy push    # deploy the committed custom files; nginx -t gated, auto-rollback
deploy/nginx-deploy test    # run nginx -t on the server
deploy/nginx-deploy check   # compare custom-files.txt with the server's allowlist
```

`push` deploys `HEAD`, not the working tree, so commit first. It sends only the files in `deploy/custom-files.txt`, and the server refuses anything not on its own copy of that list. If `nginx -t` or the reload fails, the previous files are restored. Replaced files are kept in `/var/backups/spinupwp-nginx-deploy/sfgmedicare.com/<timestamp>/` (last 30 deploys).

## Rules

- Run `pull` (or `diff`) before editing, never between committing and pushing: `pull` matches the working tree to the server, so it removes a committed custom file that isn't deployed yet (`git checkout -- .` restores it). SpinupWP rewrites its managed files during server updates and when settings change in its dashboard, so the mirror drifts.
- Never edit SpinupWP's own files (`fastcgi-cache.conf`, `https.conf`, and so on). Put customizations in separately named files in the same include dir. Files load alphabetically, so an override must sort after the file it overrides.
- Adding a custom file means adding it to `deploy/custom-files.txt` and re-registering with `bin/nginx-deploy-admin install` from the spinupwp-nginx-deploy project, which needs the server's sudo password.
- The tool never deletes. Removing a custom file from the server is a manual step for the sudo user.
