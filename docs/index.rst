hello world index

Installation on Raspberry Pi 4

Mini Tutorial:

So I assume we are starting off with a blank raspbian image (Raspbian Buster Lite) on a Pi4 with a coral stick attached. Then just follow one of the various existing tutorials for installing docker and docker-compose on a Pi4 (e.g. https://dev.to/rohansawant/installing-docker-and-docker-compose-on-the-raspberry-pi-in-5-simple-steps-3mgl)

Now for the dry dock for the frigate.

Note: we're directly using @kpine modified version for the Pi4 with the last updates from @blakeblackshear. It's still crashing after a while so we'll add a cronjob to restart it frequently.

    Create directories
    mkdir /code
    mkdir /code/02-Frigate
    mkdir /code/02-Frigate/config

    Create config files
    cd /code/02-Frigate
    /code/02-Frigate# nano docker-compose.yml

paste the following into this file for the configuration of the docker image:

frigate:
    container_name: frigate
    restart: unless-stopped
    privileged: true
   shm_size: '2g' # should work for 5-7 cameras
   image: kpine/frigate-raspberrypi:latest
   volumes:
     - /dev/bus/usb:/dev/bus/usb
     - /etc/localtime:/etc/localtime:ro
     - /code/02-Frigate/config:/config
   ports:
     - "5000:5000"

Config file for the frigate. You will need to adapt this to your own needs e.g. urls to the cam streams etc.

/code/02-Frigate/config# nano config.yml
config.example.yml

Now we pull the image and do a first start

/code/02-Frigate# docker-compose up

This will take a while but once everything is up you should see lines like:

    Starting frigate ... done
    Attaching to frigate
    frigate | On connect called

ctrl-c to stop as we want to run this in the background.

/code/02-Frigate# docker start frigate

This should fire up the frigate in the background. Now as mentioned we'll add a cronjob to restart it due to the current bug.

/code/02-Frigate# crontab -e

Note that the first time you run this it will ask you which editor you prefer. I like nano.

add the following line, which restarts frigate every hour.

    0 */1 * * * docker restart frigate

This should do it, configuration for home assistant as @blakeblackshear described it here
