```python
#!/usr/bin/env python3

from urllib.parse import urlencode
from urllib.request import urlopen, Request
import json
from imaplib import IMAP4_SSL

# create a Google app here
# https://console.developers.google.com
# then fill the following variables
GMAIL_CLIENT_ID = ""
GMAIL_CLIENT_SECRET = ""

# generate and print authorization link
url = "https://accounts.google.com/o/oauth2/auth?" + urlencode({
    "client_id": GMAIL_CLIENT_ID,
    "redirect_uri": 'urn:ietf:wg:oauth:2.0:oob',
    "scope": "https://mail.google.com/ email",
    "response_type": "code",
})
print("visit\n%s\n" % url)
code = input("paste reponse code: ")
print("")

# exchange code with access token
with urlopen(Request("https://accounts.google.com/o/oauth2/token", data=urlencode({
        "client_id": GMAIL_CLIENT_ID,
        "client_secret": GMAIL_CLIENT_SECRET,
        "code": code,
        "redirect_uri": 'urn:ietf:wg:oauth:2.0:oob',
        "grant_type": "authorization_code",
    }).encode())) as res:
    access_token = json.loads(res.read())["access_token"]

# request user email
with urlopen(Request("https://www.googleapis.com/oauth2/v2/userinfo", headers={
        "Authorization": "Bearer %s" % access_token,
    })) as res:
    user = json.loads(res.read())["email"]

# connect to imap
conn = IMAP4_SSL("imap.gmail.com", 993)

# authenticate the gmail way
conn.authenticate('XOAUTH2', lambda x:
    'user=%s\1auth=Bearer %s\1\1' % (user, access_token))

# the following is just an example that shows available folders
# you can use any function provided by imaplib
# https://docs.python.org/3/library/imaplib.html

print("available folders:")
res,data = conn.list('""', '*')
for mbox in data:
    print(mbox.decode())

print("")
```
