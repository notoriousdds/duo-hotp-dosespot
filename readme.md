# Duo's HOTP for Dosespot.

This fork is simply for the purpose of modifing the instructions of <a href="https://github.com/WillForan/duo-hotp">the work by WillForan</a> to be used with Duo's HOTP via Dosespot. Duo can authenticate using HOTP - _Hash(message authentication code)-based One-Time Password_. But it has some proprietary covers over the OATH (Initiative for Open Authentication) standard.

[simonseo/nyuad-spammer](https://github.com/simonseo/nyuad-spammer/tree/master/spammer/duo) has code to work around this. 
`duo.py` is largely copied from there

## Usage
also see `duo.py -h` or the doc string of [duo.py](duo.py)

Unlike most services, Dosespot doesn't provide QR codes or links with which to use in the duo.py program. As such, some fiddling around is required to create the needed URL. <i>(Scroll down to see the edited instructions.)</i>

<blockquote><s>
1. generate a new duo QR code for an android tablet within your institution's device management portal<br>
2. copy the url of the QR code image   <img src="img/copy_qr_code.png?raw=True" width=100>. it should look like `https://api-e4c9863e.duosecurity.com/frame/qr?value=c53Xoof7cFSOHGxtm69f-YXBpLWU0Yzk4NjNlLmR1b3NlY3VyaXR5LmNvbQ`<br>
3. `./duo.py new 'https://URL-OF-IMAGE'` to register<br>
4. push continue in the browser<br>
5. `./duo.py next` for future authentication
</s></blockquote>

1. Instead of producing a QR code or URL, Dosespot sends an activation link (via SMS only) to your phone.  <strong>This link WILL NOT work with the duo.py program!</strong> However, it is necessary in order to create the URL that <i>will</i> work with the program.
2. <strong>DO NOT CLICK ON THE LINK FROM YOUR PHONE!</strong> Clicking the link will start the importing of the secret key into the DUO app (which will prevent you from being able to use the duo.py program with the URL you're about to create.)
3. Instead of clicking the link, copy and paste it into an email message that is addressed to yourself.
4. From a desktop or laptop, click on the link from the email you just sent and it should open to something that looks like this:
   <img src="img/duo.png?raw=True">
   <br>
5. Copy the entire code (circled in red above).  <strong>NOTE: The code is larger than the box in which it is displayed! Be sure to copy the entire code.</strong>
6. Using the code you just copied AND the example URL in WillForan's orginal instructions, create your own personal URL by replacing the back half of the eample URL with the code you just copied.  (I.E. it should look like this:
   `https://api-e4c9863e.duosecurity.com/frame/qr?value=ABCDEFGHIJKLMNOPQRST-YXBpLWYzNzNmOGIxLmR1b3NlY3VyaXR5LmNvbQ`)
7. You now have a URL that can you used in the duo.py progam with which to extract the HTOP secret! In other words, just run:
   `./duo.py new 'https://YOUR-PERSONAL-URL'` to register.
9. Duo.py stores the secret key in UTF-8 format in a filed named secrets.json.  If your chosen authenticator app requires that the key be in encoded in Base32 (like my authenticator app does), you need to encode it.  I used https://emn178.github.io/online-tools/base32_encode.html to encode the key. (You can also find the b32 encoded secret key simply by scrolling through the output of the duo.py output.)
   
### Convenience
consider adding binding in `sxkd`, `xbindkeys`, etc for
```
duo.py next -s ~/secure/myinstitution_duo.json  | xclip -i
```

## Warnings
 * The default `secret.json` file is not encrypted! Be careful where you store it (see `-s` switch).
 * if you generate too many `next` calls w/out passing on to duo, you'll leave the validation window and duo will not authenticate.

## Install

```
pip install -r requirements.txt # pyotp docopt requests
./duo.py -h
```

## Tests
testing is limited.
```
python -m doctest duo.py
```

## TODO
 * support GPG to secure secret file

## TOTP
`duo.py` is specific to duo's HOTP.
For time based one time passwords (Google Authenticator, Microsoft Authenticator), look at `oath-toolkit`

```
KEY=$(zbarimg /path/to/qr-image.png)
oathtool --totp --base32 $KEY
```
