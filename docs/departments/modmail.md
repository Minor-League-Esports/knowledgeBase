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

## Known Issues

### Knex Crash: Unable to contact internal db
- **Cause:** Currently unknown.
- **Quick Fix:** Restart container.

## Catastrophic Loss Rebuild Procedures
The following instructions are in place in case the working bot, docker container, docker image, and Github repo are all somehow deleted simultaneously.

1. Clone Dragory's Modmail Bot (see above).
2. Follow the setup procedures listed in the bot's documentation. Edit the config file as stated, except add the following to the optional section: 
```
allowMove = on
botMentionResponse = Thanks for pinging our mailbox bot. You can contact staff by DMing me. Our Community team will receive your message and route it to the proper department.
categoryAutomation.newThread = 632480538527137812
closeMessage = This thread has been closed.
rolesInThreadHeader = on
```
3. Creat a standard Dockerfile and .dockerignore file in the bot's root directory.
4. Upload the new build to a Github repo. Copy the URL.
5. ```bash
   git clone [URL]
   ```
6. cd to the bot's new directory (will likely be "modmail")
7. ```bash
   # Build the docker image
   docker build -t mle-modmail .
   ```
8. ```bash
   # Create the docker container with persistent storage
   docker run -d \
   --name mle-modmail-bot \
   -v $(pwd)/logs:/usr/src/app/logs \
   -v $(pwd)/db:/usr/src/app/db \
   -v $(pwd)/attachments:/usr/src/app/attachments \
   -v $(pwd)/config.ini:/usr/src/app/config.ini:ro \
   --restart unless-stopped \
   mle-modmail
   ```
9. ```bash
   # Ensure the container is running
   docker logs mle-modmail-bot
   ```
