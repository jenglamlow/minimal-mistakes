---
title: "Dockerize Github Jekyll Blog for Windows"
categories:
  - Knowledge Sharing
tags:
  - Docker
  - Jekyll
  - Github
---

I've been writing blog using VirtualBox running Ubuntu. VM is quite a heavy application and my laptop is having diffulty in running it smoothly. Thus, I'm try to dockerize Jekyll blog, sot that I can modify my blog post and build using Docker.

My blog is using [minimal-mistake](https://mademistakes.com/work/minimal-mistakes-jekyll-theme/) theme. It is a decent Jekyll theme which contains many rich features. The best thing is the author, Michael Rose is constantly contributing the theme to add in new features and support latest technology stack.

## Download Minimal-Mistake Theme
Since, we are hosting in github, I would suggest to fork the [Minimal-Mistake Github repository](https://github.com/mmistakes/minimal-mistakes.git) and change the name to ```<username>.github.io```. Then clone the forked repository to the local machine.

```bash
git clone https://github.com/<username>/<username>.github.io.git
```

Or we can directly clone the repository for testing purpose.

```bash
git clone https://github.com/mmistakes/minimal-mistakes.git
```

According to 
[Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/) for Minimal-Mistake theme, replace the contents of ```GemFile``` in the repository with the following:

```ruby
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins

group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
end
```

## Run Docker

Thanks to the friendly community, I managed to find a Docker image, ```starefossen/github-pages``` on [Docker Hub](https://hub.docker.com/r/starefossen/github-pages/) that run well with Minimal-Mistake theme. 

Download ```starefossen/github-pages``` Docker image to Docker machine.

```bash
docker pull starefossen/github-pages
```

Change directory to cloned repository.

```bash
cd <username>.github.io.git
```

Start Jekyll Site by mounting current directory to the container's ```/usr/src/app``` directory.

```bash
docker run -v "$PWD":/usr/src/app -p "4000:4000" starefossen/github-pages
```

## Browse Jekyll Site

For Windows, check the Docker Machine IP using the following command:

```bash
docker-machine ip <machine_instance>
```

Then, open web browser and insert the address ```http://<docker-machine-ip>:4000``` to see your site goes live.


## Docker Compose

The command ``` 
docker run -v "$PWD":/usr/src/app -p "4000:4000" starefossen/github-pages
``` is quite a long command to remember

We can make use of Docker Compose to pass in the required options to create a container.

Create a ```docker-compose.yml``` with the following content:

```yml
jekyll:
    image: starefossen/github-pages
    command: jekyll serve -d /_site --force_polling --watch --drafts --incremental -H 0.0.0.0
    ports:
        - 4000:4000
    volumes:
        - .:/usr/src/app
```

and place it in the jekyll site root directory.

Windows require ```--force_polling``` to be enabled to poll for any changes in the files and regenerate the static site page.

Start the jekyll site by running

```bash
docker-compose up
```

## Start Editing Theme

Now, we have successfully hosted Jekyll page using Docker. Let's start modifying theme. Recommended to goes through the Minimal-Mistake [Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/) for sample page.

Have fun editting! =)

## Performance

My Machine Specification:

```
ASUS K53SV
Windows 7 64-bit 
Intel i5 2410M 2.1Ghz
8GB RAM
```

Generating Jekyll site for Minimal-Mistakes Theme took me around 50 seconds when running in Docker, which kind of slow. For my Ubuntu VM with 4GB memory allocated, it took around 20 seconds to regenerate.

I'm okay with the slow regeneration as I'm seldom regenerate the browser when writting blog. I guess I need to change my laptop to speedup the generation speed. Haha