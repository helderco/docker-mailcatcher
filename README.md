# docker-mailcatcher

[Mailcatcher](http://mailcatcher.me) docker image to catch emails during development.


## Basic usage

There are two exposed ports. Use port 25 to send emails to, and 80 to access the web interface where your emails will be shown.

Run mailcatcher process:

    docker run -d -p 1080:80 --name mail helder/mailcatcher

Link to another container to send emails:

    docker run -it --link mail ...

In this example emails should be sent to host `mail` and port 25.

### Docker-compose syntax
    php:
      build:
        context: .
        dockerfile: ./php/Dockerfile
        link:
          - mail
    mail:
      image: helder/mailcatcher
      ports:
        - 1080:80

## Example with a PHP container and ssmtp

ssmtp is a very lightweight send-only sendmail emulator. Let's see an example of how we can override sendmail:

    # Dockerfile
    FROM helder/php

    RUN apt-get install ssmtp -y && \
        echo "sendmail_path = /usr/sbin/ssmtp -t" > /usr/local/etc/php/conf.d/sendmail.ini && \
        echo "mailhub=mail:25\nUseTLS=NO\nFromLineOverride=YES" > /etc/ssmtp/ssmtp.conf

Now test PHP's `mail()` function:

    docker run -it --rm --link mail helder/php php -r 'mail("to@address.com", "Test", "Testing!", "From: my@example.com");'

Open your browser at http://localhost:1080 to see your emails.
