[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/ToDesktop/self-hosted)

# ToDesktop Self-Hosted

## Setup

1. Use an existing Cloudflare account or create a new one
2. Click "Deploy to Cloudflare Workers" above
3. Follow the instructions. At the end, this repo will be forked in your organization and the following will be created on your Cloudflare account:
   - `release-relay` worker
   - `desktop-cdn` worker
   - `desktop-download-cdn` worker
   - `desktop-app-distributables-staging` R2 bucket
   - `desktop-app-distributables` R2 bucket
4. Create a [GitHub PAT (Personal Access Token)](https://github.com/settings/personal-access-tokens/new) with the following scopes:
   - Scope it to the `self-hosted` repo that has just been forked
   - Read access to repository _Metadata_
   - Read and Write access to repository _Pull Requests_
   - Read and Write access to repository _Contents_
5. Create a "Release webhook" on ToDesktop.
   - Go to https://www.todesktop.com/apps/{YOUR_APP_ID}/settings
   - Scroll to "App webhooks"
   - Click the `+` button and select "Release webhook"
   - Enter the URL of the `new-release-webhook` endpoint of the `release-relay` worker. It may look like this: https://release-relay.your-org.workers.dev/new-release-webhook
   - Click "Save" and note the webhook secret
6. Add the variables and secrets to the `release-relay` worker:
   - `WEBHOOK_HMAC_KEY` (SECRET) - the webhook secret from step 5
   - `GITHUB_TOKEN` (SECRET) - a github token with the required scopes from step 4
   - `GITHUB_OWNER` (VAR) - the owner of the forked `self-hosted` repo
   - `GITHUB_REPO` (VAR) - the name of the forked `self-hosted` repo
7. (Optional) Create custom domains for your self-hosted workers.

That's it!

## Migrating your app to self-hosted

1. Make sure that you have the latest version of `@todesktop/cli` installed.
2. Add `updateUrlBase` to your `todesktop.json` file and set it to the URL of the `desktop-cdn` worker.
3. Run `todesktop build` to build your app.
4. "Partially release" your app on ToDesktop to your local IP address (or multiple IPs if you wish to test the migration with multiple users).
5. Merge the PR created by the `release-relay` worker.
6. When you open the app then it should update to the new version. From now on, the app will check for updates on your self-hosted workers.
7. Repeat steps 3-6 again to test a full update on your self-hosted workers.
8. If all looks good, then do a full "Release" of your app on ToDesktop and merge the PR created by the `release-relay` worker.
9. You may also update your download URLs to point to your `desktop-download-cdn` worker.

## How it works from now on

Now every time you create a release on ToDesktop, the following will happen:

1. The `release-relay` worker will be triggered with the release data
2. It will upload the artifacts to the `desktop-app-distributables-staging` R2 bucket
3. It will create a pull request on the `self-hosted` repo with details of the release
4. If you choose to merge the PR, the artifacts will be uploaded to the `desktop-app-distributables` R2 bucket and your release will go live

See [desktop-download-cdn/README.md](packages/desktop-download-cdn/README.md) for more details on how the download CDN works and available routes.

![Sequence Diagram](./docs/sequence-diagram.svg)

![Screenshot of a PR to release new version](https://media.cleanshot.cloud/media/42811/iTPbQub2zeCBnuGiKaOCz9GkeXFosGINno5vYsWn.jpeg?Expires=1739830695&Signature=p4dt8Fa2sEdWrUE65Bo73hQjWMEb~g38Uh~9mW3UkRVpVqQsYbVHAksfYd-HKGYRzJhZRXQX-y7S0MPrjGf893lPysmKGf22Hy-qIUnSi7qmA~WYnEWxHXUETwI~tx~5HMvjQfRNT-LVSKXIfqAG4iTmX7JvvfcpAvdHX~1h8ohEZjhrndw64IPEWpsyoDOkw6N2l2jphbyoZcLKgUY-1Kjm4DylWr-UEJOwc6LDwwNpTjdaVxLI4-Usbi4FXAKLf5kXCL3t1ZAs~tx3evENHtJbl82E-~-tOiWj~hOhOwugbyTjjx73qZ2V~pKMF08rvnIEfrjRfaaT9KrcIXorhw__&Key-Pair-Id=K269JMAT9ZF4GZ)
