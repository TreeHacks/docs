PiControl is TreeHacks' NFC check-in and attendee tracking system, used at TreeHacks 2020. With PiControl, hackers badge into various events by tapping their ID cards onto "cubes" equipped with NFC readers.

## ID badges

At TreeHacks 2020, we printed a custom badge for every hacker. We used [these PVC ID cards](https://store.gototags.com/nfc-pvc-badge-vertical) and an ID card printer (TODO: which one? although any should work).

The process we eventually used relied on the hardware IDs for each card being unique. This means that NTAG210-215 and MIFARE cards should work, but to be safe you should check the specification for the protocol you're planning on using to make sure that there won't be any ID collisions. From our research, it seems that most manufacturers like to assign IDs in sequence, but that's obviously not guaranteed.

## Hardware

Our hardware "cube" were assembled from the following:

- A plastic cube case. These were surprisingly hard to find, so we bought [these](https://www.amazon.com/Mr-Go-Rechargeable-Soothing-Decorative-Nightstand/dp/B01E7685IG/ref=sr_1_2?dchild=1&keywords=Mr.%2BGO%2Blight%2Bcube&qid=1592967810&sr=8-2&th=1) and replaced their internals with our hardware.

- Raspberry Pi 3 Model B+ (but any Raspberry Pi with Wi-Fi capability should work)

- [USB NFC readers](https://www.amazon.com/gp/product/B07Q5KXM6J/ref=ppx_yo_dt_b_asin_title_o06_s00?ie=UTF8&psc=1). These emulate a keyboard -- when an NFC badge is scanned onto them, they 'type' its ID into whatever they're connected to.
  - **PLEASE** make note of the model: if that link isn't dead, you'll see it links to a '14H' reader. '14H' (or '14D') is required to get the specificity necessary to distinguish between the NFC badge IDs: these readers read the most significant bit first, and since the card IDs are assigned sequentially this means that most of them have the same first 8-10 hex digits. You are in for much pain if you do not purchase the correct readers.
- Micro-USB battery pack. Best to get flat ones.
- *Optionally*: LEDs

### Hardware: Future Considerations

- Raspberry Pi's are certainly overkill for this application. Since the boxes only need to make a single web request, a simpler IoT board (e.g. ESP8266 or ESP32) would work just as well, consume far less power, and cost an order of magnitude less.
  - As a corollary, "PiControl" is probably a bad name for this system as pretty much all of the logic is hardware-agnostic. Even the small Python script run on the Pi could probably run on a much simpler microcontroller (with a couple modifications) through MicroPython.
- The keyboard emulation NFC readers were used because Katz (the author of this doc) had never touched hardware in his life and thus couldn't figure out how to use [an actual breakout board](https://www.adafruit.com/product/364). In the first week of CS107E, he realized that he was connecting the board's TX and RX pins to the Pi's TX and RX pins (respectively). He has now learned that TX == "transmit" and RX == "receive" and thus TX on one board should be connected to RX on the other and vice versa. If you ever see Katz, feel free to chastise him for his foolishness.

## Software

### On the Pi

The script run on the Pi can be found at [https://github.com/MYKatz/PiControl](https://github.com/MYKatz/PiControl). If you're using the aforementioned USB NFC readers, you should be running 'read_keyboard.py'.

The script is very simple: upon receiving input from the NFC reader (a card's ID), it sends that card ID to a specified API endpoint over HTTP.

#### OS and dependencies

We used a standard install of Raspbian. You will need to configure the Pi to connect to Wi-Fi -- there are many guides online to do this, such as [this one](https://raspberrypihq.com/how-to-connect-your-raspberry-pi-to-wifi/).

An installation of Python 3.7 should include all of the necessary dependencies except for [requests](https://requests.readthedocs.io/en/master/). A 'pip3 install requests' should work, provided you're connected to the internet.

### Web App

The PiControl web app comprises the vast majority of the logic. You can find it [here](https://github.com/treehacks/picontrol).

PiControl is developed on Postgres, Go (w/ Gin), and React. You can call this the PGRG stack if you hate pronounceable acronyms.

When the script on a Pi is ran for the first time, the Pi is registered with Picontrol and will show up on the 'Pis' page. You can then name the Pi and associate it with an event. When badges are scanned onto the Pi, they'll appear in the logs.

#### Eventive Integration

PiControl works by scanning NFC IDs into events on [Eventive](https://eventive.org/), which does the heavy lifting matching these IDs to hackers. If you're going to use Eventive (and you probably should, it's great!) then remember to update the EVENTIVE_API_KEY environment variable and change the eventive API link in the code (which is currently hard-coded.. oops!)

#### Environment Variables

```
DATABASE_URL= /* postgresql uri string */
EVENTIVE_API_KEY=
GIN_MODE=release /* either 'debug' or 'release' */
PASSWORD=/* password required to access web app */

```

#### Development Set-up

- Set "homepage" in package.json to the url you want.
- Run `go get -u github.com/gin-gonic/gin github.com/joho/godotenv github.com/lib/pq`
- Run `npm run dev-start`

Pushing and pulling to docker: [https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html](https://ropenscilabs.github.io/r-docker-tutorial/04-Dockerhub.html)

#### Installation

- Build using the dockerfile
- Get the build onto your server
- create .env file
- $ sudo docker run -d -t --env-file=".env" -p 80:80 [container id]
