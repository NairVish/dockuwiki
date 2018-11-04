This is a slightly modified version of [Eric Barch's Dockuwiki](https://github.com/ericbarch/dockuwiki).

Sometimes, if you self-host a personal git server (like I do), then you do not expose SSH on the standard port 22. As a result, if you choose to use Dockuwiki with your git server, then the bootstrap script does not work properly, because `ssh-keyscan` is not called with the `-p` option. To fix this, I have slightly modified this script to look for a port in the `SSH_DOMAIN` environment variable.

For example, if your server is located at `git.example.com` and the SSH port is 8000. Then, `SSH_DOMAIN` can be defined (on the command line or in your docker-compose) as `git.example.com:8000`, and `ssh-keyscan` will be called with `-p 8000`. The script still works as intended if you do not define a custom port.

The original README for Dockuwiki (kept below) still stands in every other aspect. To use this modified image, pull `nairvish/dockuwiki` instead of `ericbarch/dockuwiki`.

---

# DockuWiki
A good old fashioned self backup-ing doku**wiki** in a box.


### But what is it?
A self installing instance of the venerable [dokuwiki](https://dokuwiki.org) that backs itself up to a git repo every hour. All wrapped up in a docker image (sorry, bow not included).


### But why is it?
I wanted a dead simple method of setting up wikis without hassle. Using a git repo for backups simplifies deployment and eliminates the need for a host with persistent storage.


### How do I use it?
1. [Install Docker](https://docs.docker.com/engine/installation/) if you haven't already.


2. Create an empty git repo (i.e. BitBucket, GitHub, your own server). In step 3, let’s pretend you created a repo named "wiki" on BitBucket.


3. ```$ docker run -d --restart=always --name=wiki -e SSH_DOMAIN=bitbucket.org -e REMOTE_URL=git@bitbucket.org:YOUR_BITBUCKET_USERNAME/wiki.git -p 3000:3000 ericbarch/dockuwiki```


4. On the first run, the container will generate a unique SSH key. ```docker logs wiki``` to get the public key. Add this public key to the SSH Keys section of BitBucket, GitHub, your own server, etc.


5. Wait a few moments, then access your freshly minted wiki at http://YOUR_DOCKER_HOST_IP:3000. The initial configuration page can be accessed at http://YOUR_DOCKER_HOST_IP:3000/install.php.


### Can I run it on a Pi?
Sure, I'm not going to tell you how to live your life:

1. Install Docker (it's officially supported for ARM!)

2. Give the pi user access to the Docker daemon: ```sudo usermod -a -G docker pi```. Reboot time!

3. Follow the same steps from the "How do I use it?" section above, but replace "ericbarch/dockuwiki" with "ericbarch/dockuwiki:rpi".

4. [Do this](https://i.imgur.com/893Smv1.gif)


### What if I want to use this with Let's Encrypt on my Pi?
Lucky for you, [I wrote a blog post about that](https://ericbarch.com/post/dockuwiki-letsencrypt-pi/)!


### What if I accidentally ignite my thermite packed PC and need to redeploy the wiki to a new machine?
Just run the same docker command again and add the new SSH key it generates. If dockuwiki finds an existing wiki, it clones and hosts it.


### Can I get that with SSL?
Nope. TLS only, baby. But that’s where Let’s Encrypt saves the day:

1. Start the nginx container:
    ```bash
    $ docker run -d --restart=always --name=nginxproxy \
        -p 80:80 -p 443:443 \
        --name nginx-proxy \
        -v /etc/nginx/certs \
        -v /etc/nginx/vhost.d \
        -v /usr/share/nginx/html \
        -v /var/run/docker.sock:/tmp/docker.sock:ro \
        jwilder/nginx-proxy
    ```

2. Start the Let’s Encrypt container:
    ```bash
    $ docker run -d --restart=always --name=letsencrypt \
        --volumes-from nginx-proxy \
        -v /var/run/docker.sock:/var/run/docker.sock:ro \
        jrcs/letsencrypt-nginx-proxy-companion
    ```

3. Start the Dockuwiki container with your domain name and email:
    ```bash
    $ docker run -d --restart=always --name=wiki \
    -e SSH_DOMAIN=bitbucket.org -e REMOTE_URL=git@bitbucket.org:YOUR_BITBUCKET_USERNAME/wiki.git \
    -e VIRTUAL_HOST=foo.bar.com -e LETSENCRYPT_HOST=foo.bar.com \
    -e LETSENCRYPT_EMAIL=youremail@yourdomain.com ericbarch/dockuwiki
    ```

4. Make sure to add the SSH key that was generated by the dockuwiki container to your git host. Note that the Let's Encrypt container will take its sweet time generating keys and certs before HTTPS comes up.


### How does it work?
[Read up, friend!](https://ericbarch.com/post/dockuwiki/)


### I don’t get the name
I tried to be cute by combining "Docker" and "Doku". Get it? Hah!


### Got any config recommendations?
I live and die by [indexmenu](https://www.dokuwiki.org/plugin:indexmenu), [upgrade](https://www.dokuwiki.org/plugin:upgrade), and [bootstrap3](https://www.dokuwiki.org/template:bootstrap3). You can install all of these from the built in extension manager in your dockuwiki instance.


### I’m going on vacation to the moon and won’t have internet access. How can I access my wiki?
No problemo, moon bound traveler. Either take the machine with you that hosts dockuwiki, or use something like a Raspberry Pi to deploy a new instance with the same repo URL. I don’t suggest running multiple instances simultaneously, but your stand in wiki host will collect all your changes and continue attempting to reach the internet until it is safely back on Earth.


### Who created this?
A dude named Eric Barch that is hoping the community will embrace the project so he can collect mad internet karma and retire early.
