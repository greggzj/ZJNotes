
In a "WPA2" only network, all clients must support WPA2(AES) to be able to authenticate.

In a "WPA2/WPA mixed mode" network, one can connect with both WPA(TKIP) and WPA2(AES) clients.


Note that TKIP is not as secure as AES, and therefore WPA2/AES should be used exclusively, if possible. The only exception would be if there are some older WPA/TKIP wireless clients on the network that do not support WPA2/AES.

WPA2/WPA mixed mode allows for the coexistence of WPA and WPA2 clients on a common SSID. The passphrase for both WPA and WPA2 clients remains the same, the access point just advertises the different encryption cyphers available to be selected for use by the client. Clients choose which cypher to use for the wireless connection.


Notes:
This mixed mode is also known as: "WPA2 TKIP+AES mode", or "PSK2-mixed mode."
Using TKIP may force all wireless connections to 802.11g mode, i.e. 54Mbps or less.