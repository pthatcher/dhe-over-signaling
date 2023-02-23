You probably want to install [https://github.com/cabo/kramdown-rfc](kramdown-rfc) and [https://github.com/ietf-tools/xml2rfc](xml2rfc):

```
gem install kramdown-rfc
pip install xml2rfc
```

Then to generate the .xml, .txt, and .html files, do this:

```
kramdown-rfc draft-thatcher-webrtc-sans-dtls.md > draft-thatcher-webrtc-sans-dtls.xml && xml2rfc --html --text draft-thatcher-webrtc-sans-dtls.xml
```



