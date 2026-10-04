# Modmail
[Github Repo](https://github.com/Minor-League-Esports/modmail)

## Maintainers
- **Keegabyte**
- **Icy_Fatal99**

Before messaging either one with questions, please reference the FAQ section of this page.

## Appropriation
MLE Modmail is a custom implementation of [Dragory's Modmail Bot](https://github.com/Dragory/modmailbot) with added customization and dockerfiles for MLE specifically.

## Setup
Please reference the [Modmail Readme](https://github.com/Minor-League-Esports/modmail/blob/main/readme.md) for setup instructions.

## FAQ

### How do I use this?
You can see a list of commands for this bot [here](https://github.com/Minor-League-Esports/modmail/blob/main/docs/commands.md).

### This is brand new but looks a lot like the old one, are the logs and snippets saved?
No, unfortunately. Due to the nature of how the old bot was hosted, we were unable to save the old logs and message snippets. You will have to make new snippets.

### Are we at risk of data loss in the future?
After update 1.0.1, we made that pretty much impossible. If there is data loss, we will have much bigger problems.

### Where is Modmail data stored?
The Docker deployment uses persistent host-mounted directories for the SQLite database, logs, attachments, and configuration.

The important mounts are:

```text
/root/modmail/logs        -> /usr/src/app/logs
/root/modmail/db          -> /usr/src/app/db
/root/modmail/attachments -> /usr/src/app/attachments
/root/modmail/config.ini  -> /usr/src/app/config.ini
```

The SQLite database is stored at:

```text
/usr/src/app/db/data.sqlite
```

Because these directories are stored on the host, replacing or recreating the Docker container should not remove the database, attachments, or configuration as long as the same mounts are preserved.

### How are transcript logs hosted?
Modmail currently uses local log storage. Unless otherwise configured, Modmail's internal web server listens on port `8890`.

Transcript links use the following format:

```text
http://<public-ip>:8890/logs/<thread-id>
```

Because of this, Docker must publish port `8890` from the container to the host:

```bash
-p 8890:8890
```

If this port mapping is missing, Modmail may still generate transcript URLs, but those URLs will not be accessible externally.

## Known Issues

### Knex Crash: Unable to contact internal db
- **Cause:** Currently unknown.
- **Quick Fix:** Restart container.

```bash
docker restart mle-modmail-bot
```

If the issue continues, check the recent container logs:

```bash
docker logs --tail 200 mle-modmail-bot
```

### Transcript Link Loads Forever / Does Not Open

#### Symptoms
Modmail successfully generates a transcript URL similar to:

```text
http://<public-ip>:8890/logs/<thread-uuid>
```

but opening the URL causes the browser to load indefinitely or fail to connect.

#### Known Cause
This occurred on September 29, 2026 because Modmail's internal web server was correctly listening on TCP port `8890` inside the Docker container, but Docker was not publishing that port to the DigitalOcean host.

The bot itself, Discord connection, SQLite database, and transcript generation were all working correctly.

#### Check the Port Mapping

Run:

```bash
docker port mle-modmail-bot
```

The working deployment should show port `8890` published.

You can also run:

```bash
docker ps --filter name=mle-modmail
```

The port mapping should resemble:

```text
0.0.0.0:8890->8890/tcp
[::]:8890->8890/tcp
```

#### Test the Web Server

Run:

```bash
curl -I --max-time 5 http://127.0.0.1:8890/
```

An HTTP `404` response from `/` is expected because Modmail does not define a page at the root URL. This still confirms that the web server is reachable.

To test an actual transcript:

```bash
curl -I --max-time 5 http://127.0.0.1:8890/logs/<thread-uuid>
```

A working transcript should return an HTTP `200` response.

#### Fix
If port `8890` was not published when the container was created, the container must be recreated with:

```bash
-p 8890:8890
```

Do not recreate the container without preserving the persistent mounts.

## Current Docker Deployment

The production container uses the following configuration:

```text
Container: mle-modmail-bot
Image: mle-modmail
Restart Policy: unless-stopped
Network Mode: bridge
Web Server Port: 8890
```

The container should be created with:

```bash
docker run -d \
  --name mle-modmail-bot \
  --restart unless-stopped \
  -p 8890:8890 \
  -v /root/modmail/logs:/usr/src/app/logs \
  -v /root/modmail/db:/usr/src/app/db \
  -v /root/modmail/attachments:/usr/src/app/attachments \
  -v /root/modmail/config.ini:/usr/src/app/config.ini:ro \
  mle-modmail
```

> **Important:** Do not remove `-p 8890:8890` unless the deployment is intentionally changed to use a reverse proxy or another networking setup. Without this mapping, public transcript links will not be accessible.

## Useful Diagnostic Commands

### Check container health and published ports

```bash
docker ps --filter name=mle-modmail
```

### Check recent application output

```bash
docker logs --tail 200 mle-modmail-bot
```

### Confirm port mapping

```bash
docker port mle-modmail-bot
```

### Test the web server locally

```bash
curl -I --max-time 5 http://127.0.0.1:8890/
```

A `404 Not Found` response from `/` is expected.

### Test a known transcript

```bash
curl -I --max-time 5 http://127.0.0.1:8890/logs/<thread-uuid>
```

### Inspect configured mounts

```bash
docker inspect mle-modmail-bot --format '{{json .Mounts}}'
```

### Inspect restart policy

```bash
docker inspect mle-modmail-bot --format '{{json .HostConfig.RestartPolicy}}'
```

## Safe Container Replacement Procedure

If the Modmail container needs to be recreated but the current installation and persistent data are still available, use the following procedure.

### 1. Save the existing Docker configuration

```bash
docker inspect mle-modmail-bot > /root/modmail-container-backup.json
```

### 2. Stop the current container

```bash
docker stop mle-modmail-bot
```

### 3. Rename the old container

Instead of immediately deleting the existing container, rename it so it can be restored if necessary:

```bash
docker rename mle-modmail-bot mle-modmail-bot-old
```

### 4. Create the replacement container

```bash
docker run -d \
  --name mle-modmail-bot \
  --restart unless-stopped \
  -p 8890:8890 \
  -v /root/modmail/logs:/usr/src/app/logs \
  -v /root/modmail/db:/usr/src/app/db \
  -v /root/modmail/attachments:/usr/src/app/attachments \
  -v /root/modmail/config.ini:/usr/src/app/config.ini:ro \
  mle-modmail
```

### 5. Verify the replacement

Check that the container is running:

```bash
docker ps --filter name=mle-modmail
```

Check the application logs:

```bash
docker logs --tail 200 mle-modmail-bot
```

Confirm the port mapping:

```bash
docker port mle-modmail-bot
```

Test the web server:

```bash
curl -I --max-time 5 http://127.0.0.1:8890/
```

Then test a known transcript:

```bash
curl -I --max-time 5 http://127.0.0.1:8890/logs/<thread-uuid>
```

Finally, confirm that the public transcript URL loads in a browser.

Do not delete `mle-modmail-bot-old` until the replacement container has been tested under normal use and the team is comfortable with the new deployment.

## Catastrophic Loss Rebuild Procedures
The following instructions are in place in case the working bot, docker container, docker image, and Github repo are all somehow deleted simultaneously.

1. Clone [Dragory's Modmail Bot](https://github.com/Dragory/modmailbot).

2. Follow the setup procedures listed in the bot's documentation. Edit the config file as stated, except add the following to the optional section:

```ini
allowMove = on
botMentionResponse = Thanks for pinging our mailbox bot. You can contact staff by DMing me. Our Community team will receive your message and route it to the proper department.
categoryAutomation.newThread = 632480538527137812
closeMessage = This thread has been closed.
rolesInThreadHeader = on
```

3. Create a standard `Dockerfile` and `.dockerignore` file in the bot's root directory.

4. Upload the new build to a Github repo. Copy the URL.

5. Clone the repository:

```bash
git clone [URL]
```

6. `cd` to the bot's new directory. This will likely be `modmail`:

```bash
cd modmail
```

7. Build the Docker image:

```bash
docker build -t mle-modmail .
```

8. Create the Docker container with persistent storage and the required transcript port:

```bash
docker run -d \
  --name mle-modmail-bot \
  --restart unless-stopped \
  -p 8890:8890 \
  -v $(pwd)/logs:/usr/src/app/logs \
  -v $(pwd)/db:/usr/src/app/db \
  -v $(pwd)/attachments:/usr/src/app/attachments \
  -v $(pwd)/config.ini:/usr/src/app/config.ini:ro \
  mle-modmail
```

> **Important:** Do not omit `-p 8890:8890`. Modmail's local transcript web server uses this port. Without the port mapping, transcript URLs can still be generated but will not be reachable from outside the Docker container.

9. Ensure the container is running:

```bash
docker ps --filter name=mle-modmail
```

10. Check the application logs:

```bash
docker logs mle-modmail-bot
```

11. Verify that port `8890` is published:

```bash
docker port mle-modmail-bot
```

12. Test the local web server:

```bash
curl -I --max-time 5 http://127.0.0.1:8890/
```

A `404` response from `/` is expected.

13. Open and close a test Modmail thread. Then test its transcript:

```bash
curl -I --max-time 5 http://127.0.0.1:8890/logs/<thread-uuid>
```

The transcript should return an HTTP `200` response and should also load using its public transcript URL.

## Critical Deployment Notes

When modifying or rebuilding Modmail, always preserve the following:

```text
Port Mapping:
8890:8890

Persistent Storage:
/root/modmail/logs
/root/modmail/db
/root/modmail/attachments
/root/modmail/config.ini

Restart Policy:
unless-stopped
```

The most important networking requirement for the current logging configuration is:

```bash
-p 8890:8890
```

If Modmail continues using local log storage, removing this mapping will cause public transcript links to become inaccessible even though the bot itself may otherwise appear healthy.
